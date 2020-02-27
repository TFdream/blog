# Spring Cloud面试专题
[Spring Cloud](https://spring.io/projects/spring-cloud/) focuses on providing good out of box experience for typical use cases and extensibility mechanism to cover others.
* Distributed/versioned configuration
* Service registration and discovery
* Routing
* Service-to-service calls
* Load balancing
* Circuit Breakers
* Global locks
* Leadership election and cluster state
* Distributed messaging

## Spring Cloud体系

### Eureka
[Eureka专题](Eureka.md)

### Hystrix
[Hystrix专题](Hystrix.md)

### Ribbon
[Ribbon专题](Ribbon.md)

### Feign
[Feign专题](Feign.md)

### Zuul路由网关

### Spring Boot和 Spring Cloud 联系与区别？
Spring Boot是一个快速整合第三方框架，关注的是**微观**，具体关注快速方便的开发单个个体的服务；
Spring Cloud 关注的是**宏观**，具体关注全局的微服务协调整理治理框架，将spring boot开发的一个个单体服务整合并管理起来；它为各个微服务之间提供 配置管理 服务发现 断路器路由 微代理 全局锁 分布式会话 等 集成服务。

### Spring Cloud和 Dubbo有哪些区別?
本质区别： 
* dubbo 是 基于 RPC 远程 过程调用 
* Spring Cloud 是基于 http rest api调用；

### 请说说eureka、consul和 zookeeper 的区別?
这里不得不提一下，我们知道分布式里一个重要的理论，那就是CAP原则。指的是在一个分布式系统中，Consistency(一致性)、Availability(可用性)、Partition Tolerance(分区容错性)，不能同时成立。
* 一致性：它要求在同一时刻点，分布式系统中的所有数据备份都处于同一状态。
* 可用性：在系统集群的一部分节点宕机后，系统依然能够响应用户的请求。
* 分区容错性：在网络区间通信出现失败，系统能够容忍。

一般来讲，基于网络的不稳定性，分布容错是不可避免的，所以我们默认CAP中的P总是成立的。一致性的强制数据统一要求，必然会导致在更新数据时部分节点处于被锁定状态，此时不可对外提供服务，影响了服务的可用性，反之亦然。因此一致性和可用性不能同时满足。在注册中心的发展上面，一直有两个分支：一个就是 CP 系统，追求数据的强一致性。还有一个是 AP 系统，追求高可用与最终一致。

我们接下来介绍的服务注册和发现组件中，Eureka满足了其中的AP，Consul和Zookeeper满足了其中的CP。
