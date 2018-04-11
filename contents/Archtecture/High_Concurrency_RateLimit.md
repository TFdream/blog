## 概述
在开发高并发系统时有三把利器用来保护系统：缓存、降级和限流。缓存的目的是提升系统访问速度和增大系统能处理的容量，可谓是抗高并发流量的银弹；而降级是当服务出问题或者影响到核心流程的性能则需要暂时屏蔽掉，待高峰或者问题解决后再打开；而有些场景并不能用缓存和降级来解决，比如稀缺资源（秒杀、抢购）、写服务（如评论、下单）、频繁的复杂查询（评论的最后几页），因此需有一种手段来限制这些场景的并发/请求量，即限流。

## 目的
限流的目的是通过对并发访问/请求进行限速或者一个时间窗口内的的请求进行限速来保护系统，一旦达到限制速率则可以拒绝服务（定向到错误页或告知资源没有了）、排队或等待（比如秒杀、评论、下单）、降级（返回兜底数据或默认数据，如商品详情页库存默认有货）。

## 
一般开发高并发系统常见的限流有：限制总并发数（比如数据库连接池、线程池）、限制瞬时并发数（如nginx的limit_conn模块，用来限制瞬时并发连接数）、限制时间窗口内的平均速率（如Guava的RateLimiter、nginx的limit_req模块，限制每秒的平均速率）；其他还有如限制远程接口调用速率、限制MQ的消费速率。另外还可以根据网络连接数、网络流量、CPU或内存负载等来限流。

## 实战
```
package demo.pandora.redis;

import org.springframework.context.support.ClassPathXmlApplicationContext;
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
public class RedisRateLimiterDemo {
    private static final String PREFIX = "META-INF/scripts/";

    public static void main(String[] args) {

        new RedisRateLimiterDemo().testRateLimit();
    }

    public void testRateLimit() {

        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-redis.xml");

        RedisTemplate redisTemplate = (RedisTemplate) context.getBean("redisTemplate");

        Long userId = 15L;
        // Make a unique key per user.
        String prefix = "request_rate_limiter.{" + userId+"}";

        //How many requests per second do you want a user to be allowed to do?
        Long replenishRate = 10L;
        //How much bursting do you want to allow
        Long burstCapacity = 2 * replenishRate;

        // You need two Redis keys for Token Bucket.
        List<String> keys = Arrays.asList(prefix + ".tokens", prefix + ".timestamp");
        // The arguments to the LUA script. time() returns unixtime in seconds.
        List<String> scriptArgs = Arrays.asList(replenishRate + "", burstCapacity + "", Instant.now().getEpochSecond() + "", "1");

        String script = loadScript();

        for (int i=0; i<50; i++) {
            //返回数组结果，[是否获取令牌成功, 剩余令牌数]
            List<Long> result = (List<Long>) redisTemplate.eval(script, keys, scriptArgs);
            System.out.println(result);
        }
    }

    private String loadScript() {
        try {
            String fullName = PREFIX + "request_rate_limiter.lua";
            InputStream in = RedisRateLimiterDemo.class.getClassLoader().getResourceAsStream(fullName);
            BufferedReader br = new BufferedReader(new InputStreamReader(in, "UTF-8"));

            String separator = System.getProperty("line.separator");
            System.out.println("分隔符："+separator);
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



### 3. Redis Lua 脚本
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

- 第 1 至 2 行 ：KEYS 方法参数 ：
 - 第一个参数 ：request_rate_limiter.${id}.tokens ，令牌桶剩余令牌数。
 - 第二个参数 ：request_rate_limiter.${id}.timestamp ，令牌桶最后填充令牌时间，单位：秒。
- 第 4 至 7 行 ：ARGV 方法参数 ：
 - 第一个参数 ：replenishRate 。
 - 第二个参数 ：burstCapacity 。
 - 第三个参数 ：得到从 1970-01-01 00:00:00 开始的秒数。
 - 第四个参数 ：消耗令牌数量，默认 1 。

- 第 9 行 ：计算令牌桶填充满令牌需要多久时间，单位：秒。
- 第 10 行 ：计算 request_rate_limiter.${id}.tokens / request_rate_limiter.${id}.timestamp 的 ttl 。* 2 保证时间充足。
- 第 12 至 20 行 ：调用 get 命令，获得令牌桶剩余令牌数( last_tokens ) ，令牌桶最后填充令牌时间(last_refreshed) 。
- 第 22 至 23 行 ：填充令牌，计算新的令牌桶剩余令牌数( filled_tokens )。填充不超过令牌桶令牌上限。
- 第 24 至 30 行 ：获取令牌是否成功。
 - 若成功，令牌桶剩余令牌数(new_tokens) 减消耗令牌数( requested )，并设置获取成功( allowed_num = 1 ) 。
 - 若失败，设置获取失败( allowed_num = 0 ) 。

- 第 32 至 33 行 ：设置令牌桶剩余令牌数( new_tokens ) ，令牌桶最后填充令牌时间(now) 。
- 第 35 行 ：返回数组结果，[是否获取令牌成功, 剩余令牌数] 。



附上Stripe的[Request rate limiter](https://gist.github.com/ptarjan/e38f45f2dfe601419ca3af937fff574d) 原始脚本 ：
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
if allowed then
  new_tokens = filled_tokens - requested
end

redis.call("setex", tokens_key, ttl, new_tokens)
redis.call("setex", timestamp_key, ttl, now)

return { allowed, new_tokens }
```



## 参考资料
[Scaling your API with rate limiters](https://stripe.com/blog/rate-limiters)
[Spring-Cloud-Gateway 源码解析 —— 过滤器 (4.10) 之 RequestRateLimiterGatewayFilterFactory 请求限流](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-request-rate-limiter/)
