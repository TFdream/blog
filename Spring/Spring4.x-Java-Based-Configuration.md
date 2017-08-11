## Spring4.x新特性-基于Java配置
基于 Java 的配置选项，可以使你在不用配置 XML 的情况下编写大多数的 Spring，但是一些有帮助的基于 Java 的注解，解释如下：

@Configuration 和 @Bean 注解
带有 @Configuration 的注解类表示这个类可以使用 Spring IoC 容器作为 bean 定义的来源。@Bean 注解告诉 Spring，一个带有 @Bean 的注解方法将返回一个对象，该对象应该被注册为在 Spring 应用程序上下文中的 bean。

最简单可行的 @Configuration 类如下所示：
```
package com.mindflow.spring4;

import org.springframework.context.annotation.*;

@Configuration
public class AppConfig {

   @Bean 
   public HelloWorld helloWorld(){
      return new HelloWorld();
   }
}
```

上面的代码将等同于下面的 XML 配置：
```
<beans>
   <bean id="helloWorld" class="com.mindflow.spring4.HelloWorld" />
</beans>
```

