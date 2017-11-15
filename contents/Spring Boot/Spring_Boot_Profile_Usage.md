## 前言
在Spring Boot的开发中,应用程序在不同的环境可能会有不同的配置，例如数据库连接、日志级别等，开发，测试，生产每个环境可能配置都不一致。

使用Spring Boot的Profile可以实现多场景下的配置切换，方便开发中进行测试和部署生产环境。
下面就大致介绍一下yml配置文件跟properties配置文件怎么使用profile配置不同环境的配置文件。

## 开发环境
开发环境如下：
* JDK 1.8
* Maven 3.x
* Spring Boot 1.5.8 
* Intellij Idea 2017

## 实战

### 1. 使用yml文件
 1.首先,我们先创建一个名为 application.yml的属性文件,如下:
```
server:
  port: 8080

my:
  name: demo

spring:
  profiles:
    active: dev

---
#development environment
spring:
  profiles: dev

server:
  port: 8160

my:
  name: ricky

---
#test environment
spring:
  profiles: test

server:
  port: 8180

my:
  name: test

---
#production environment
spring:
  profiles: prod

server:
  port: 8190

my:
  name: prod

```

application.yml文件分为四部分,使用```---``` 来作为分隔符，第一部分通用配置部分，表示三个环境都通用的属性，
后面三段分别为：开发，测试，生产，用spring.profiles指定了一个值(开发为dev，测试为test，生产为prod)，这个值表示该段配置应该用在哪个profile里面。

如果我们是本地启动，在通用配置里面可以设置调用哪个环境的profil，也就是第一段的```spring.profiles.active=XXX```，
其中XXX是后面3段中spring.profiles对应的value,通过这个就可以控制本地启动调用哪个环境的配置文件，例如:
```
spring:
    profiles:
        active: dev
```

加载的就是开发环境的属性，如果dev换成test，则会加载测试环境的属性，以此类推。

**注意**<br>
如果spring.profiles.active没有指定值，那么只会使用没有指定spring.profiles文件的值，也就是只会加载通用的配置。

#### 打包发布
如果是部署到服务器的话,我们正常打成jar包，启动时通过
```--spring.profiles.active=xxx```来控制加载哪个环境的配置，完整命令如下:
```
java -jar --spring.profiles.active=test xxx.jar  表示加载测试环境的配置

java -jar --spring.profiles.active=prod xxx.jar  表示加载生产环境的配置
```
上面介绍完了,我们本地启动一下,先将spring.profiles.active的值设置为dev,看看日志打印结果:


#### 使用多个yml配置文件进行配置属性文件
如果是使用多个yml来配置属性，通过与配置文件相同的明明规范，创建application-{profile}.yml文件,将于环境无关的属性,放置到application.yml文件里面,可以通过这种形式来配置多个环境的属性文件,在application.yml文件里面指定```spring.profiles.active=xxx```,来加载不同环境的配置,如果不指定,则默认只使用application.yml属性文件,不会加载其他的profiles的配置


### 2. 使用properties文件
如果使用application.properties进行多个环境的配置,原理跟使用多个yml配置文件一致,也是通过application-{profile}.properties来控制加载哪个环境的配置,将于环境无关的属性,放置到application.properties文件里面,通过```spring.profiles.active=xxx```,加载不同环境的配置,如果不指定,则默认加载application.properties的配置,不会加载带有profile的配置


### 3. Maven中的场景配置
```


```

## 源码
[spring-boot-profiles](https://github.com/TFrise/spring-boot-tutorials/tree/master/spring-boot-profiles)

## 参考文档
[Spring Boot Reference Guide - Profiles](https://docs.spring.io/spring-boot/docs/1.5.8.RELEASE/reference/htmlsingle/#boot-features-profiles)

