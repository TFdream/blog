
## 什么是分布式锁

## 实现方式
分布式锁一般有三种实现方式：
1. 基于数据库乐观锁；
2. 基于Redis的分布式锁；
3. 基于ZooKeeper的分布式锁。

本篇博客将介绍第二种方式，基于Redis实现分布式锁。
本篇博客将详细介绍如何正确地实现Redis分布式锁。

## 满足条件
实现分布式锁需要满足哪些条件呢？我们至少要确保锁的实现同时满足以下条件：
* 高性能：
* 互斥性：在任意时刻，只有一个客户端能持有锁，同一个方法在同一时间只能被一台机器上的一个线程执行。
* 避免发生死锁：即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁；另外这把锁是一把可重入锁。
* 容错性：只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。

## Redis实现分布式锁
Redis通常可以使用setnx来实现分布式锁。

首先我们要通过Maven引入Jedis开源组件，在pom.xml文件加入下面的代码：
```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

### 基本版
1. 加锁

```
public void lock(){
    while(true){
        ret = tryLock();
        if(ret){  //获取到了锁
            return;
        }
        sleep(100);
    }
}
```

```
public boolean tryLock(){
    Long result = jedis.setnx(lockKey, requestId);
    if (result == 1) {
        return true;
    }
    return false;
}
```
2. 解锁
```
public void unlock(){
    jedis.del(lockKey);
}
```

setnx来创建一个key，如果key不存在则创建成功返回1，如果key已经存在则返回0。依照上述来判定是否获取到了锁，获取到锁的执行业务逻辑，完毕后删除lock_key，来实现释放锁。

### 改进版
1. 加锁
```
public boolean tryLock(long leaseTime, TimeUnit unit) {
	String resp = jedis.set(resourceName, getLockName(Thread.currentThread().getId()), "NX", "PX", unit.toMillis(leaseTime));
	if ("OK".equals(resp)) {
		return true;
	}
	return false;
}

protected String getLockName(long threadId) {
	return String.format("%s:%d", id, threadId);
}
```

2. 解锁
```
    public void unlock() {
        String holder = jedis.get(resourceName);
        if (getLockName(Thread.currentThread().getId()).equals(holder)) {  //release
            jedis.del(resourceName);
        }
    }
```

### 终极版
1. 加锁
```
import java.util.concurrent.TimeUnit;

/**
 * Distributed Lock
 * @author Ricky Fung
 */
public interface DLock {

    void lock();

    boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException;

    boolean isLocked();

    boolean isHeldByCurrentThread();

    void forceUnlock();

    void unlock();
}
```

RedisLock
```
import pandora.distlock.DLock;
import pandora.redis.RedisTemplate;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

/**
 * 参考：[Distributed locks with Redis](https://redis.io/topics/distlock)
 * @author Ricky Fung
 */
public class RedisLock implements DLock {

    private String id; //唯一标识id
    private String resourceName;    //资源名称
    private RedisTemplate redisTemplate;
    private LuaScript luaScript = new LuaScript();

    private final List<String> keys = new ArrayList<>(1);

    public RedisLock(String id, String resourceName, RedisTemplate redisTemplate) {
        this.id = id;
        this.resourceName = resourceName;
        this.redisTemplate = redisTemplate;
        this.keys.add(resourceName);
    }

    @Override
    public void lock() {

    }

    @Override
    public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        boolean res = tryAcquire(leaseTime, unit);
        // lock acquired
        if (res) {
            return true;
        }
        time -= (System.currentTimeMillis() - current);
        if (time <= 0) {
            return false;
        }
        while (true) {
            long currentTime = System.currentTimeMillis();
            res = tryAcquire(leaseTime, unit);
            // lock acquired
            if (res) {
                return true;
            }
            time -= (System.currentTimeMillis() - currentTime);
            if (time <= 0) {
                return false;
            }
        }
    }

    @Override
    public boolean isLocked() {
        return redisTemplate.exists(resourceName);
    }

    @Override
    public boolean isHeldByCurrentThread() {
        List<String> args = new ArrayList<>(1);
        args.add(getLockValue());
        Long update = (Long) redisTemplate.eval(luaScript.buildHeldByCurrentThreadScript(), keys,
                args);
        if (update==1) {
            return true;
        }
        return false;
    }

    @Override
    public void forceUnlock() {
        redisTemplate.del(resourceName);
    }

    @Override
    public void unlock() {
        List<String> args = new ArrayList<>(1);
        args.add(getLockValue());
        Long update = (Long) redisTemplate.eval(luaScript.buildUnLockScript(), keys, args);

    }

    private boolean tryAcquire(long leaseTime, TimeUnit unit) {
        List<String> args = new ArrayList<>(4);
        args.add(getLockValue());
        args.add("NX");
        args.add("PX");
        args.add(String.valueOf(unit.toMillis(leaseTime)));
        Long update = (Long) redisTemplate.eval(luaScript.buildLockScript(), keys, args);
        if (update==1) {
            return true;
        }
        return false;
    }

    protected String getLockValue() {
        return String.format("%s:%d", id, Thread.currentThread().getId());
    }

}

```

LuaScript
```
/**
 * @author Ricky Fung
 */
public class LuaScript {

    /**
     * SET resource_name my_random_value NX PX 30000
     * @return
     */
    public String buildLockScript() {
        StringBuilder sb = new StringBuilder(120);
        sb.append("if redis.call('set', KEYS[1], ARGV[1], ARGV[2], ARGV[3], tonumber(ARGV[4])) == 1 then ")
                .append("\t return 1 ")
                .append("end ")
                .append("if redis.call('get', KEYS[1]) == ARGV[1] then ")
                .append("\t return 1 ")
                .append("else ")
                .append("\t return 0 ")
                .append("end");
        return sb.toString();
    }

    public String buildUnLockScript() {
        StringBuilder sb = new StringBuilder(120);
        sb.append("if redis.call('get', KEYS[1]) == ARGV[1] then ")
                .append("\t return redis.call('del', KEYS[1]) ")
                .append("else ")
                .append("\t return 0 ")
                .append("end");
        return sb.toString();
    }

    public String buildHeldByCurrentThreadScript() {
        StringBuilder sb = new StringBuilder(120);
        sb.append("if redis.call('get', KEYS[1]) == ARGV[1] then ")
                .append("\t return 1 ")
                .append("else ")
                .append("\t return 0 ")
                .append("end");
        return sb.toString();
    }

}
```

```
/**
 * @author Ricky Fung
 */
public class Panda {
    private final RedisTemplate redisTemplate;

    public Panda(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public DLock getLock(String name) {
        return new RedisLock(UUIDUtils.getUUID(), name, redisTemplate);
    }
}
```

```
    Panda panda = new Panda(redisTemplate);
    
    @Test
    public void testReentrantLock() throws InterruptedException {
        String res = "res1";
        DLock lock = panda.getLock(res);
        try {
            boolean success = lock.tryLock(5, 30, TimeUnit.SECONDS);
            System.out.println("key:"+res+" value:"+redisTemplate.get(res));
            if (success) {
                System.out.println("re-entry:"+lock.tryLock(5, 30, TimeUnit.SECONDS));
                System.out.println("YES");
            } else {
                System.out.println("NO");
            }
        } finally {
            lock.unlock();
        }
        System.out.println("key:"+res+" value:"+redisTemplate.get(res));
    }
```


