## Spring Bean生命周期详解
在Spring中 Bean 可谓是一个核心的元素，当我们结合Spring进行编程的时候也离不开Bean，面对这样重要的一个角色，了解其生命周期和该生命周期所涉及的环节对我们更加熟练灵活地使用Bean是很有Bean必要的，下面我们就来详细分析下Bean的生命周期吧。
此处，可以借鉴Java EE中Servlet的生命周期，实例化、初始init、接收请求service、销毁destroy。

### Spring Bean的生命周期概述
1. 实例化BeanFactoryPostProcessor接口实现类；
2. 执行BeanFactoryPostProcessor的postProcessBeanFactory方法；
3. 实例化BeanPostProcessor接口实现类；
4. 实例化InstantiationAwareBeanPostProcessor接口实现类；
5. 执行InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation方法；
6. 执行Bean的构造方法；
7. 执行InstantiationAwareBeanPostProcessor的postProcessPropertyValues方法；
8. 为Bean注入属性；
9. 调用BeanNameAware接口的setBeanName方法；
10. 调用BeanFactoryAware接口的setBeanFactory()方法；
11. 调用ApplicationContextAware接口的setApplicationContext(ApplicationContext)方法；
12. 执行BeanPostProcessor接口的postProcessBeforeInitialization方法；
13. 执行InitializingBean接口的afterPropertiesSet方法；
14. 调用<bean>的init-method属性指定的初始化方法；
15. 执行BeanPostProcessor接口的postProcessAfterInitialization方法；
16. 执行InstantiationAwareBeanPostProcessor接口的postProcessAfterInstantiation方法；
17. 容器初始化成功，执行业务代码后，下面开始销毁容器；

18. 调用DisposableBean接口的destroy方法；
19. 调用<bean>中的destroy-method指定的销毁方法。

### Spring Bean生命周期流程图

