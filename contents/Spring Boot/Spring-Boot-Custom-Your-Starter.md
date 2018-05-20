## Spring Boot
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
* [dubbo-spring-boot-starter](https://github.com/alibaba/dubbo-spring-boot-starter)
* [druid-spring-boot-starter](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
* [Condition annotations](https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/#boot-features-bean-conditions)
[Spring Boot - parent pom when you already have a parent pom
](https://stackoverflow.com/questions/21317006/spring-boot-parent-pom-when-you-already-have-a-parent-pom/21318359#21318359)
