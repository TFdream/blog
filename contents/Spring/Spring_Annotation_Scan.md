
在使用Spring时，有时候有会有一些自定义Annotation的需求，比如一些Listener的回调函数。

例如，在配置中心的设计上，我们期望在配置发生变更时，client能收到通知，如下：
```
@Service
public class DemoService {

  @Value("${batch:100}")
  private int batch;
  
  @ConfigChangeListener
  private void onChange(ConfigChangeEvent changeEvent) {
    //update injected value of batch if it is changed in Apollo
    if (changeEvent.isChanged("batch")) {
      batch = config.getIntProperty("batch", 100);
    }
  }
}
```

一开始的时候，我是在Spring的ContextRefreshedEvent事件里，通过context.getBeansWithAnnotation(Component.class) 来获取到所有的bean，然后再检查method是否有@MyListener的annotation。

后来发现这个方法有缺陷，当有一些spring bean的@Scope设置为session/request时，创建bean会失败。

参考：http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-factory-scopes

## BeanPostProcessor接口
后来看了下spring jms里的@JmsListener的实现，发现实现BeanPostProcessor接口才是最合理的办法。
```
public interface BeanPostProcessor {

    /**
     * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
     * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
     * or a custom init-method). The bean will already be populated with property values.
     * The returned bean instance may be a wrapper around the original.
     * @param bean the new bean instance
     * @param beanName the name of the bean
     * @return the bean instance to use, either the original or a wrapped one;
     * if {@code null}, no subsequent BeanPostProcessors will be invoked
     * @throws org.springframework.beans.BeansException in case of errors
     * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
     */
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

    /**
     * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
     * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
     * or a custom init-method). The bean will already be populated with property values.
     * The returned bean instance may be a wrapper around the original.
     * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
     * instance and the objects created by the FactoryBean (as of Spring 2.0). The
     * post-processor can decide whether to apply to either the FactoryBean or created
     * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
     * <p>This callback will also be invoked after a short-circuiting triggered by a
     * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
     * in contrast to all other BeanPostProcessor callbacks.
     * @param bean the new bean instance
     * @param beanName the name of the bean
     * @return the bean instance to use, either the original or a wrapped one;
     * if {@code null}, no subsequent BeanPostProcessors will be invoked
     * @throws org.springframework.beans.BeansException in case of errors
     * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
     * @see org.springframework.beans.factory.FactoryBean
     */
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```
所有的bean在创建完之后，都会回调postProcessAfterInitialization函数，这时就可以确定bean是已经创建好的了。

所以扫描自定义的annotation的代码大概是这个样子的：
```
public class ConfigChangeListenerProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        Method[] methods = ReflectionUtils.getAllDeclaredMethods(bean.getClass());
        if (methods != null) {
            for (Method method : methods) {
                ConfigChangeListener listener = AnnotationUtils.findAnnotation(method, ConfigChangeListener.class);
                // process
            }
        }
        return bean;
    }
}
```
