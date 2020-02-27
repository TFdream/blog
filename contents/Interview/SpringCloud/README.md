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

