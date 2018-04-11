## 概述
在开发高并发系统时有三把利器用来保护系统：缓存、降级和限流。缓存的目的是提升系统访问速度和增大系统能处理的容量，可谓是抗高并发流量的银弹；而降级是当服务出问题或者影响到核心流程的性能则需要暂时屏蔽掉，待高峰或者问题解决后再打开；而有些场景并不能用缓存和降级来解决，比如稀缺资源（秒杀、抢购）、写服务（如评论、下单）、频繁的复杂查询（评论的最后几页），因此需有一种手段来限制这些场景的并发/请求量，即限流。

## 目的
限流的目的是通过对并发访问/请求进行限速或者一个时间窗口内的的请求进行限速来保护系统，一旦达到限制速率则可以拒绝服务（定向到错误页或告知资源没有了）、排队或等待（比如秒杀、评论、下单）、降级（返回兜底数据或默认数据，如商品详情页库存默认有货）。

## 应用场景
一般开发高并发系统常见的限流有：限制总并发数（比如数据库连接池、线程池）、限制瞬时并发数（如nginx的limit_conn模块，用来限制瞬时并发连接数）、限制时间窗口内的平均速率（如Guava的RateLimiter、nginx的limit_req模块，限制每秒的平均速率）；其他还有如限制远程接口调用速率、限制MQ的消费速率。另外还可以根据网络连接数、网络流量、CPU或内存负载等来限流。


## 实战
### 1. RedisRateLimiter
```
package redis.ratelimit;

import demo.pandora.redis.RedisRateLimiterDemo;
import pandora.redis.RedisTemplate;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.time.Instant;
import java.util.Arrays;
import java.util.List;

/**
 * @author Ricky Fung<br/>
 * Created on 2018-04-11<br/>
 */
public class RedisRateLimiter {
    private static final String PREFIX = "META-INF/scripts/";

    private RedisTemplate redisTemplate;
    private final String redisKeyPrefix;

    /** How many requests per second do you want a user to be allowed to do? **/
    private final int replenishRate;
    /** How much bursting do you want to allow **/
    private final int burstCapacity;
    /** Lua script**/
    private String script;

    public RedisRateLimiter(RedisTemplate redisTemplate, String redisKeyPrefix,
                            int replenishRate) {
        this(redisTemplate, redisKeyPrefix, replenishRate, replenishRate);
    }

    public RedisRateLimiter(RedisTemplate redisTemplate, String redisKeyPrefix,
                            int replenishRate, int burstCapacity) {
        this.redisTemplate = redisTemplate;
        this.redisKeyPrefix = redisKeyPrefix;
        this.replenishRate = replenishRate;
        this.burstCapacity = burstCapacity;
        this.script = loadScript();
    }

    public Response acquire(String id, int permits) {
        // Make a unique key per user.
        String prefix = redisKeyPrefix+".{" + id+"}";

        // You need two Redis keys for Token Bucket.
        List<String> keys = Arrays.asList(prefix + ".tokens", prefix + ".timestamp");
        // The arguments to the LUA script. time() returns unixtime in seconds.
        List<String> scriptArgs = Arrays.asList(replenishRate + "", burstCapacity + "", Instant.now().getEpochSecond() + "", permits+"");

        List<Long> results;
        try {
            //执行 Redis Lua 脚本，获取令牌。返回结果为 [是否获取令牌成功, 剩余令牌数] ，其中，1 代表获取令牌成功，0 代表令牌获取失败。
            results = (List<Long>) redisTemplate.eval(script, keys, scriptArgs);
        } catch (Exception e) {
            //当 Redis Lua 脚本过程中发生异常，忽略异常，返回 Arrays.asList(1L, -1L) ，即认为获取令牌成功。
            // 为什么？在 Redis 发生故障时，我们不希望限流器对 Reids 是强依赖，并且 Redis 发生故障的概率本身就很低。
            results = Arrays.asList(1L, -1L);
        }
        boolean allowed = results.get(0) == 1L;
        Long tokensLeft = results.get(1);
        Response response = new Response(allowed, tokensLeft);
        return response;
    }

    private String loadScript() {
        try {
            String fullName = PREFIX + "request_rate_limiter.lua";
            InputStream in = RedisRateLimiterDemo.class.getClassLoader().getResourceAsStream(fullName);
            BufferedReader br = new BufferedReader(new InputStreamReader(in, "UTF-8"));

            String separator = System.getProperty("line.separator");

            StringBuilder sb = new StringBuilder(2048);
            String line = null;
            while ((line=br.readLine()) !=null) {
                sb.append(line).append(separator);
            }
            return sb.toString();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
}

```

### 2. Redis Lua 脚本
```META-INF/scripts/request_rate_limiter.lua```，Redis Lua 脚本，实现基于令牌桶算法实现限流。代码如下 ：
```
local tokens_key = KEYS[1]
local timestamp_key = KEYS[2]

local rate = tonumber(ARGV[1])
local capacity = tonumber(ARGV[2])
local now = tonumber(ARGV[3])
local requested = tonumber(ARGV[4])

local fill_time = capacity/rate
local ttl = math.floor(fill_time*2)

local last_tokens = tonumber(redis.call("get", tokens_key))
if last_tokens == nil then
  last_tokens = capacity
end

local last_refreshed = tonumber(redis.call("get", timestamp_key))
if last_refreshed == nil then
  last_refreshed = 0
end

local delta = math.max(0, now-last_refreshed)
local filled_tokens = math.min(capacity, last_tokens+(delta*rate))
local allowed = filled_tokens >= requested
local new_tokens = filled_tokens
local allowed_num = 0
if allowed then
  new_tokens = filled_tokens - requested
  allowed_num = 1
end

redis.call("setex", tokens_key, ttl, new_tokens)
redis.call("setex", timestamp_key, ttl, now)

return { allowed_num, new_tokens }
```

* 第 1 至 2 行 ：KEYS 方法参数 ：
    - 第一个参数 ：request_rate_limiter.${id}.tokens ，令牌桶剩余令牌数。
    - 第二个参数 ：request_rate_limiter.${id}.timestamp ，令牌桶最后填充令牌时间，单位：秒。
* 第 4 至 7 行 ：ARGV 方法参数 ：
    - 第一个参数 ：replenishRate 。
    - 第二个参数 ：burstCapacity 。
    - 第三个参数 ：得到从 1970-01-01 00:00:00 开始的秒数。
    - 第四个参数 ：消耗令牌数量，默认 1 。

* 第 9 行 ：计算令牌桶填充满令牌需要多久时间，单位：秒。
* 第 10 行 ：计算 request_rate_limiter.${id}.tokens / request_rate_limiter.${id}.timestamp 的 ttl 。* 2 保证时间充足。
* 第 12 至 20 行 ：调用 get 命令，获得令牌桶剩余令牌数( last_tokens ) ，令牌桶最后填充令牌时间(last_refreshed) 。
* 第 22 至 23 行 ：填充令牌，计算新的令牌桶剩余令牌数( filled_tokens )。填充不超过令牌桶令牌上限。
* 第 24 至 30 行 ：获取令牌是否成功。
    - 若成功，令牌桶剩余令牌数(new_tokens) 减消耗令牌数( requested )，并设置获取成功( allowed_num = 1 ) 。
    - 若失败，设置获取失败( allowed_num = 0 ) 。

* 第 32 至 33 行 ：设置令牌桶剩余令牌数( new_tokens ) ，令牌桶最后填充令牌时间(now) 。
* 第 35 行 ：返回数组结果，[是否获取令牌成功, 剩余令牌数] 。


### RedisRateLimiter测试
```
package demo.redis;

import redis.ratelimit.RedisRateLimiter;
import redis.ratelimit.Response;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import pandora.core.util.JsonUtils;
import pandora.redis.RedisTemplate;

/**
 * @author Ricky Fung<br/>
 * Created on 2018-04-11<br/>
 */
public class RedisRateLimiterDemo {

    public static void main(String[] args) {

        new RedisRateLimiterDemo().testRateLimit();
    }

    public void testRateLimit() {

        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-redis.xml");

        RedisTemplate redisTemplate = (RedisTemplate) context.getBean("redisTemplate");

        String userId = "15";
        String keyPrefix = "request_rate_limiter";
        RedisRateLimiter redisRateLimiter = new RedisRateLimiter(redisTemplate, keyPrefix, 20, 30);

        for (int i=0; i<50; i++) {
            Response response = redisRateLimiter.acquire(userId, 1);
            System.out.println(JsonUtils.toJson(response));
        }
    }

}

```
打印结果:
```
{"allowed":true,"tokensRemaining":29}
{"allowed":true,"tokensRemaining":28}
{"allowed":true,"tokensRemaining":27}
{"allowed":true,"tokensRemaining":26}
{"allowed":true,"tokensRemaining":25}
{"allowed":true,"tokensRemaining":24}
{"allowed":true,"tokensRemaining":23}
{"allowed":true,"tokensRemaining":22}
{"allowed":true,"tokensRemaining":21}
{"allowed":true,"tokensRemaining":20}
{"allowed":true,"tokensRemaining":19}
{"allowed":true,"tokensRemaining":18}
{"allowed":true,"tokensRemaining":17}
{"allowed":true,"tokensRemaining":16}
{"allowed":true,"tokensRemaining":15}
{"allowed":true,"tokensRemaining":14}
{"allowed":true,"tokensRemaining":13}
{"allowed":true,"tokensRemaining":12}
{"allowed":true,"tokensRemaining":11}
{"allowed":true,"tokensRemaining":10}
{"allowed":true,"tokensRemaining":9}
{"allowed":true,"tokensRemaining":8}
{"allowed":true,"tokensRemaining":7}
{"allowed":true,"tokensRemaining":6}
{"allowed":true,"tokensRemaining":5}
{"allowed":true,"tokensRemaining":4}
{"allowed":true,"tokensRemaining":3}
{"allowed":true,"tokensRemaining":2}
{"allowed":true,"tokensRemaining":1}
{"allowed":true,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
{"allowed":false,"tokensRemaining":0}
```


## 参考资料
* [Scaling your API with rate limiters](https://stripe.com/blog/rate-limiters)
* [Scaling your API with rate limiters Lua Scripts](https://gist.github.com/ptarjan/e38f45f2dfe601419ca3af937fff574d)
* [Spring-Cloud-Gateway 源码解析 —— 过滤器 (4.10) 之 RequestRateLimiterGatewayFilterFactory 请求限流](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-request-rate-limiter/)
