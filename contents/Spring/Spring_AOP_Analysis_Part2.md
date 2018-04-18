## Spring源码分析 - AOP源码解析（下篇）
本篇重点关注AspectJAwareAdvisorAutoProxyCreator及为Bean生成代理时机分析。

上篇中负责解析<aop:config> 标签ConfigBeanDefinitionParser中的configureAutoProxyCreator方法如下：
```
	private void configureAutoProxyCreator(ParserContext parserContext, Element element) {
		AopNamespaceUtils.registerAspectJAutoProxyCreatorIfNecessary(parserContext, element);
	}
```

AopNamespaceUtils的registerAspectJAutoProxyCreatorIfNecessary方法如下：
```

	public static void registerAspectJAutoProxyCreatorIfNecessary(
			ParserContext parserContext, Element sourceElement) {

		BeanDefinition beanDefinition = AopConfigUtils.registerAspectJAutoProxyCreatorIfNecessary(
				parserContext.getRegistry(), parserContext.extractSource(sourceElement));
		useClassProxyingIfNecessary(parserContext.getRegistry(), sourceElement);
		registerComponentIfNecessary(beanDefinition, parserContext);
	}

	private static void useClassProxyingIfNecessary(BeanDefinitionRegistry registry, Element sourceElement) {
		if (sourceElement != null) {
			boolean proxyTargetClass = Boolean.valueOf(sourceElement.getAttribute(PROXY_TARGET_CLASS_ATTRIBUTE));
			if (proxyTargetClass) {
				AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
			}
			boolean exposeProxy = Boolean.valueOf(sourceElement.getAttribute(EXPOSE_PROXY_ATTRIBUTE));
			if (exposeProxy) {
				AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
			}
		}
	}

	private static void registerComponentIfNecessary(BeanDefinition beanDefinition, ParserContext parserContext) {
		if (beanDefinition != null) {
			BeanComponentDefinition componentDefinition =
					new BeanComponentDefinition(beanDefinition, AopConfigUtils.AUTO_PROXY_CREATOR_BEAN_NAME);
			parserContext.registerComponent(componentDefinition);
		}
	}
```

其中AopConfigUtils的registerAspectJAutoProxyCreatorIfNecessary方法如下：
```
	public static BeanDefinition registerAspectJAutoProxyCreatorIfNecessary(BeanDefinitionRegistry registry, Object source) {
		return registerOrEscalateApcAsRequired(AspectJAwareAdvisorAutoProxyCreator.class, registry, source);
	}
	
	private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
				int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
				int requiredPriority = findPriorityForClass(cls);
				if (currentPriority < requiredPriority) {
					apcDefinition.setBeanClassName(cls.getName());
				}
			}
			return null;
		}
		RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
		beanDefinition.setSource(source);
		beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
		beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
		return beanDefinition;
	}
```

```org.springframework.aop.aspectj.autoproxy.AspectJAwareAdvisorAutoProxyCreator```这个类是Spring提供给开发者的AOP的核心类，就是AspectJAwareAdvisorAutoProxyCreator完成了【类/接口-->代理】的转换过程，首先我们看一下AspectJAwareAdvisorAutoProxyCreator的层次结构：
![AspectJAwareAdvisorAutoProxyCreator_Hierarchical_Structure](https://github.com/TFdream/blog/blob/master/docs/image/Spring/AspectJAwareAdvisorAutoProxyCreator.png)

这里最值得注意的一点是最左下角的那个方框，我用几句话总结一下：
* AspectJAwareAdvisorAutoProxyCreator是BeanPostProcessor接口的实现类
* postProcessBeforeInitialization方法与postProcessAfterInitialization方法实现在父类AbstractAutoProxyCreator中
* postProcessBeforeInitialization方法是一个空实现
* 逻辑代码在postProcessAfterInitialization方法中

基于以上的分析，将Bean生成代理的时机已经一目了然了：在每个Bean初始化之后，如果需要，调用AspectJAwareAdvisorAutoProxyCreator中的postProcessBeforeInitialization为Bean生成代理。


### 判断是否为<bean>生成代理对象
上文分析了Bean生成代理的时机是在每个Bean初始化之后，下面把代码定位到Bean初始化之后，先是AbstractAutowireCapableBeanFactory的initializeBean方法进行初始化：
```

```



