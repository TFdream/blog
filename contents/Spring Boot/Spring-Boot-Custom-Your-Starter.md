SpringBoot最大的特点就是提供了很多默认的配置，Spring4.x提供了基于条件来配置Bean的能力，SpringBoot就是通过这一原理来实现的。 

## 创建自己的Spring Boot Starter
如果你想要自己创建一个starter，那么基本上包含以下几步：
1. 创建一个starter项目，关于项目的命名你可以参考[这里](https://docs.spring.io/spring-boot/docs/1.5.12.RELEASE/reference/htmlsingle/#boot-features-custom-starter-naming)
2. 创建一个ConfigurationProperties用于保存你的配置信息（如果你的项目不使用配置信息则可以跳过这一步，不过这种情况非常少见）
3. 创建一个AutoConfiguration，引用定义好的配置信息；在AutoConfiguration中实现所有starter应该完成的操作，并且把这个类加入spring.factories配置文件中进行声明
4. 打包项目，之后在一个SpringBoot项目中引入该项目依赖，然后就可以使用该starter了

Spring 官方 Starter通常命名为spring-boot-starter-{name}如 spring-boot-starter-web， Spring官方建议非官方Starter命名应遵循{name}-spring-boot-starter的格式。

```
package com.alibaba.druid.spring.boot.autoconfigure;

import javax.sql.DataSource;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.spring.boot.autoconfigure.properties.DruidStatProperties;
import com.alibaba.druid.spring.boot.autoconfigure.stat.DruidFilterConfiguration;
import com.alibaba.druid.spring.boot.autoconfigure.stat.DruidSpringAopConfiguration;
import com.alibaba.druid.spring.boot.autoconfigure.stat.DruidStatViewServletConfiguration;
import com.alibaba.druid.spring.boot.autoconfigure.stat.DruidWebStatFilterConfiguration;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.autoconfigure.AutoConfigureBefore;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
/**
 * @author lihengming [89921218@qq.com]
 */
@Configuration
@ConditionalOnClass(DruidDataSource.class)
@AutoConfigureBefore(DataSourceAutoConfiguration.class)
@EnableConfigurationProperties({DruidStatProperties.class, DataSourceProperties.class})
@Import({DruidSpringAopConfiguration.class,
    DruidStatViewServletConfiguration.class,
    DruidWebStatFilterConfiguration.class,
    DruidFilterConfiguration.class})
public class DruidDataSourceAutoConfigure {

    private static final Logger LOGGER = LoggerFactory.getLogger(DruidDataSourceAutoConfigure.class);

    @Bean(initMethod = "init")
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        LOGGER.info("Init DruidDataSource");
        return new DruidDataSourceWrapper();
    }
}
```
我们首先讲一下源码中注解的作用：
* @Configuration,被该注解注释的类会提供一个或则多个@Bean修饰的方法并且会被spring容器处理来生成bean definitions。
* @Bean注解是必须修饰函数的，该函数可以提供一个bean。而且该函数的函数名必须和bean的名称一致，除了首字母不需要大写。
* @ConditionalOnClass注解是条件判断的注解，表示对应的类在classpath目录下存在时，才会去解析对应的配置文件。
* @EnableConfigurationProperties注解给出了该配置类所需要的配置信息类，也就是StorageServiceProperties类，这样spring容器才会去读取配置信息到StorageServiceProperties对象中。
* @ConditionalOnMissingBean注解也是条件判断的注解，表示如果不存在对应的bean条件才成立，这里就表示如果已经有StorageService的bean了，那么就不再进行该bean的生成。这个注解十分重要，涉及到默认配置和用户自定义配置的原理。也就是说用户可以自定义一个StorageService的bean,这样的话，spring容器就不需要再初始化这个默认的bean了。
* ConditionalOnProperty注解是条件判断的注解，表示如果配置文件中的响应配置项数值为true,才会对该bean进行初始化。


## 解决Spring Boot Parent依赖


## 参考资料
* [Spring Boot custom-starter](https://docs.spring.io/spring-boot/docs/1.5.12.RELEASE/reference/htmlsingle/#boot-features-custom-starter-naming)
* [dubbo-spring-boot-starter](https://github.com/alibaba/dubbo-spring-boot-starter)
* [druid-spring-boot-starter](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
* [Condition annotations](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/#boot-features-bean-conditions)
[Spring Boot - parent pom when you already have a parent pom
](https://stackoverflow.com/questions/21317006/spring-boot-parent-pom-when-you-already-have-a-parent-pom/21318359#21318359)
