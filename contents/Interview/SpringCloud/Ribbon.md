# Ribbon
Spring Cloud Ribbon是一个基于HTTP和TCP的客户端负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。Spring Cloud Ribbon虽然只是一个工具类框架，它不像服务注册中心、配置中心、API网关那样需要独立部署，但是它几乎存在于每一个Spring Cloud构建的微服务和基础设施中。因为微服务间的调用，API网关的请求转发等内容，实际上都是通过Ribbon来实现的，包括后续我们将要介绍的Feign，它也是基于Ribbon实现的工具。所以，对Spring Cloud Ribbon的理解和使用，对于我们使用Spring Cloud来构建微服务非常重要。

## Ribbon负载均衡策略

## Ribbon底层原理
通过Spring Cloud Ribbon的封装，我们在微服务架构中使用客户端负载均衡调用非常简单，只需要如下两步：
* 服务提供者只需要启动多个服务实例并注册到一个注册中心或是多个相关联的服务注册中心。
* 服务消费者直接通过调用被@LoadBalanced注解修饰过的RestTemplate来实现面向服务的接口调用。

这样，我们就可以将服务提供者的高可用以及服务消费者的负载均衡调用一起实现了。
```
@Target({ ElementType.FIELD, ElementType.PARAMETER, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Qualifier
public @interface LoadBalanced {

}
```
从@LoadBalanced注解码的注释中，可以知道该注解用来给RestTemplate标记，以使用负载均衡的客户端（LoadBalancerClient）来配置它。

从LoadBalancerAutoConfiguration类上的注解可知，Ribbon实现负载均衡自动化配置需要满足下面两个条件：
* @ConditionalOnClass(RestTemplate.class)：RestTemplate必须存在于当前工程的环境中。
* @ConditionalOnBean(LoadBalancerClient.class)：在Spring的Bean工程中必须有LoadBalancerClient的实现bean。

该自动配置类，主要做了下面三件事：
1. 创建了一个LoadBalancerInterceptor的Bean，用于实现对客户端发起请求时进行拦截，以实现客户端负载均衡。
2. 创建了一个RestTemplateCustomizer的Bean，用于给RestTemplate增加LoadbalancerInterceptor
3. 维护了一个被@LoadBalanced注解修饰的RestTemplate对象列表，并在这里进行初始化，通过调用RestTemplateCustomizer的实例来给需要客户端负载均衡的RestTemplate增加LoadBalancerInterceptor拦截器。


