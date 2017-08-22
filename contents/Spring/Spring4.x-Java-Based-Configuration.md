# Spring4.x新特性-基于Java配置
基于 Java 的配置选项，可以使你在不用配置 XML 的情况下编写大多数的 Spring。

## @Configuration 和 @Bean 注解
带有 @Configuration 的注解类表示这个类可以使用 Spring IoC 容器作为 bean 定义的来源。@Bean 注解告诉 Spring，一个带有 @Bean 的注解方法将返回一个对象，该对象应该被注册为在 Spring 应用程序上下文中的 bean。

最简单可行的 @Configuration 类如下所示：
```
package com.mindflow.spring4;

import org.springframework.context.annotation.*;

@Configuration
public class AppConfig {
   @Bean
   public Foo foo() {
      return new Foo(bar());
   }
   @Bean
   public Bar bar() {
      return new Bar();
   }
}
```

上面的代码将等同于下面的 XML 配置：
```
<beans>
   <bean id="foo" class="com.mindflow.spring4.Foo" />
   <bean id="bar" class="com.mindflow.spring4.Bar" />
</beans>
```

在这里，带有 @Bean 注解的方法名称作为 bean 的 ID，它创建并返回实际的 bean。你的配置类可以声明多个 @Bean。一旦定义了配置类，你就可以使用 AnnotationConfigApplicationContext 来加载并把他们提供给 Spring 容器，如下所示：
```
package com.mindflow.spring4;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.*;

public class MainApp {
   public static void main(String[] args) {
      ApplicationContext ctx = 
      new AnnotationConfigApplicationContext(AppConfig.class);

      Foo foo = ctx.getBean(Foo.class);

      foo.say("Hello World!");
   }
}
```

## @Import 注解
@Import 注解允许从另一个配置类中加载 @Bean 定义。就像 Spring 的 XML 文件中使用的<import/>元素帮助模块化配置一样，@Import注解允许从其它配置类中加载@Bean的配置。

考虑 ConfigA 类，如下所示：
```
@Configuration
public class ConfigA {
   @Bean
   public A a() {
      return new A(); 
   }
}
```
你可以在另一个 Bean 声明中导入上述 Bean 声明，如下所示：
```
@Configuration
@Import(ConfigA.class)
public class ConfigB {
   @Bean
   public B a() {
      return new A(); 
   }
}
```

现在，当实例化上下文时，不需要同时指定 ConfigA.class 和 ConfigB.class，只有 ConfigB 类需要提供，如下所示：
```
public static void main(String[] args) {
   ApplicationContext ctx = 
   new AnnotationConfigApplicationContext(ConfigB.class);
   // now both beans A and B will be available...
   A a = ctx.getBean(A.class);
   B b = ctx.getBean(B.class);
}
```

## AnnotationConfigApplicationContext
AnnotationConfigApplicationContext可以使用无参构造方法来实例化，之后使用register()方法来配置。这个方法当编程式地构建AnnotationConfigApplicationContext时尤其有用:
```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

要开启组件扫描，仅仅需要在你的@Configuration类上加上如下注解：
```
@Configuration
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```

有经验的 Spring 用户肯定会熟悉下面这个 Spring 的 context:命名空间中的常用 XML声明：
```
<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
```
在上面的示例中，com.acme包会被扫描到，去查找任意@Component注解的类，那些类就会被注册为 Spring 容器中的 bean。AnnotationConfigApplicationContext 暴露出 scan(String ...)方法，允许相同的组件扫描功能：
```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```


## @Bean注解
### 生命周期回调
@Bean 注解支持指定任意的初始化和销毁的回调方法，就像在 bean 元素中 Spring 的 XML 的初始化方法和销毁方法的属性：
```
public class Foo {
   public void init() {
      // initialization logic
   }
   public void cleanup() {
      // destruction logic
   }
}

@Configuration
public class AppConfig {
   @Bean(initMethod = "init", destroyMethod = "cleanup" )
   public Foo foo() {
      return new Foo();
   }
}
```

### 指定 Bean 的范围

默认范围是单实例，但是你可以重写带有 @Scope 注解的该方法，如下所示：
```
@Configuration
public class AppConfig {
   @Bean
   @Scope("prototype")
   public Foo foo() {
      return new Foo();
   }
}
```

### 指定bean的名称
默认地，配置类使用@Bean方法的名称来作为注册bean的名称。这个方法可以被重写，当然，使用的是name属性。
```
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }
}
```


