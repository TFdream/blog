## Spring Boot核心原理－自动配置
Spring Boot出现之后，得益于COC(“习惯优于配置”)这个理念，再也没有繁琐的配置（大多数流行第三方技术都被集成在内）。
那么背后实现的核心原理到底是什么呢？ 其实是spring 4.x提供的基于条件配置bean的能力。
Spring boot关于自动配置的源码在spring-boot-autoconfigure-x.x.x.x.jar中，主要包含了如下图所示的配置（并未截全）：
