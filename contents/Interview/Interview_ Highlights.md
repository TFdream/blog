
## java基础篇
### 集合
* Arraylist 与 LinkedList 区别
* ArrayList 与 Vector 区别
* HashMap 的工作原理及代码实现
* HashMap 和 Hashtable 的区别
* HashSet 和 HashMap 区别
* ConcurrentHashMap 的工作原理及代码实现
* COW集合

### 线程
* 创建线程的方式及实现
* sleep() 、join（）、yield（）有什么区别
* 说说 CountDownLatch 原理
* 说说 CyclicBarrier 原理
* 说说 Semaphore 原理
* 说说 Exchanger 原理
* 说说 CountDownLatch 与 CyclicBarrier 区别
* ThreadLocal 原理分析
* 讲讲线程池的实现原理
* 线程池的几种方式
* 线程的生命周期

### 锁机制
* 说说线程安全问题
* volatile 实现原理
* synchronize 实现原理
* synchronized 与 lock 的区别
* CAS 乐观锁
* ABA 问题
* 乐观锁的业务场景及实现方式

## HTTP/TCP
* HTTP 请求的 GET 与 POST 方式的区别
* session 与 cookie 区别

## 框架篇
### Spring
* BeanFactory 和 ApplicationContext 有什么区别
* Spring Bean 的生命周期
* Spring IOC 如何实现
* 说说 Spring AOP
* Spring AOP 实现原理
* 动态代理（cglib 与 JDK）
* Spring 事务实现方式
* Spring 事务底层原理
* 如何自定义注解实现功能
* Spring MVC 运行流程
* Spring MVC 启动流程
* Spring 的单例实现原理
* Spring 框架中用到了哪些设计模式

### Mybatis
* #{}和${}的区别是什么？
* 通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？
* Dao接口里的方法，参数不同时，方法能重载吗？
* Mybatis 是如何进行分页的？分页插件的原理是什么？
* Mybatis的一级、二级缓存


### Dubbo
* dubbo都支持什么协议，推荐用哪种？
* Dubbo推荐使用什么序列化框架，你知道的还有哪些？推荐使用Hessian序列化，还有Duddo、FastJson、Java自带序列化。
* Dubbo有哪几种集群容错方案，默认是哪种？
* Dubbo有哪几种负载均衡策略，默认是哪种？
* 当一个服务接口有多种实现时怎么做？
* 服务上线怎么兼容旧版本？


### Netty

### Spring Boot

## 核心篇
### 缓存
* Redis 有哪些类型
* Redis 内部结构
* 聊聊 Redis 使用场景
* Redis 持久化机制
* Redis 如何实现持久化
* Redis 集群方案与实现
* Redis 为什么是单线程的
* 缓存雪崩
* 缓存降级
* 使用缓存的合理性问题

### MQ
* 消息队列的使用场景
* 消息的重发补偿解决思路
* 消息的幂等性解决思路
* 消息的堆积解决思路
* 自己如何实现消息队列
* 如何保证消息的有序性

### 数据库
* MySQL 索引使用的注意事项
* 说说反范式设计
* 说说分库与分表设计
* 分库与分表带来的分布式困境与应对之策
* 说说 SQL 优化之道
* MySQL 遇到的死锁问题
* 存储引擎的 InnoDB 与 MyISAM
* 数据库索引的原理
* 为什么要用 B-tree
* 聚集索引与非聚集索引的区别
* 什么叫做覆盖索引？
* 聚集索引与非聚集索引的区别
* 大数据量分页优化
* 选择合适的数据存储方案

### 搜索引擎
* 倒排索引
* 聊聊 ElasticSearch 使用场景

## 微服务篇
### 微服务
* 前后端分离是如何做的
* 微服务哪些框架
* 你怎么理解 RPC 框架
* 说说 RPC 的实现原理
* 说说 Dubbo 的实现原理
* 你怎么理解 RESTful
* 说说如何设计一个良好的 API
* 如何理解 RESTful API 的幂等性
* 如何保证接口的幂等性
* 说说 CAP 定理、 BASE 理论
* 怎么考虑数据一致性问题
* 说说最终一致性的实现方案
* 你怎么看待微服务
* 微服务与 SOA 的区别
* 如何拆分服务
* 如何应对微服务的链式调用异常
* 对于快速追踪与定位问题
* 微服务的安全

### 分布式
* 谈谈业务中使用分布式的场景
* Session 分布式方案
* 分布式锁的场景
* 分布是锁的实现方案
* 分布式事务
* 集群与负载均衡的算法与实现
* 说说分库与分表设计
* 分库与分表带来的分布式困境与应对之策

### 性能优化
* 性能指标有哪些
* 如何发现性能瓶颈
* 性能调优的常见手段
* 说说你在项目中如何进行性能调优

## 工程篇

