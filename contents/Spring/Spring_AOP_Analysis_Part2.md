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
上文分析了Bean生成代理的时机是在每个Bean初始化之后，下面把代码定位到Bean初始化之后，AbstractAutoProxyCreator的postProcessAfterInitialization方法，代码如下：
```
/**
 * Create a proxy with the configured interceptors if the bean is
 * identified as one to proxy by the subclass.
 * @see #getAdvicesAndAdvisorsForBean
 */
@Override
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
	if (bean != null) {
		Object cacheKey = getCacheKey(bean.getClass(), beanName);
		if (!this.earlyProxyReferences.contains(cacheKey)) {
			return wrapIfNecessary(bean, beanName, cacheKey);
		}
	}
	return bean;
}
```

AbstractAutoProxyCreator的wrapIfNecessary方法如下：
```
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
	if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
		return bean;
	}
	if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
		return bean;
	}
	if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
		this.advisedBeans.put(cacheKey, Boolean.FALSE);
		return bean;
	}

	// Create proxy if we have advice.
	Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
	if (specificInterceptors != DO_NOT_PROXY) {
		this.advisedBeans.put(cacheKey, Boolean.TRUE);
		Object proxy = createProxy(
				bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
		this.proxyTypes.put(cacheKey, proxy.getClass());
		return proxy;
	}

	this.advisedBeans.put(cacheKey, Boolean.FALSE);
	return bean;
}
```

哪些目标对象需要生成代理？只要getAdvicesAndAdvisorsForBean方法返回的Advisor数组不为空，那么就会通过createProxy方法为<bean>创建代理，代码如下：
```
protected Object createProxy(
		Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {

	if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
		AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
	}

	ProxyFactory proxyFactory = new ProxyFactory();
	proxyFactory.copyFrom(this);

	if (!proxyFactory.isProxyTargetClass()) {
		if (shouldProxyTargetClass(beanClass, beanName)) {
			proxyFactory.setProxyTargetClass(true);
		}
		else {
			evaluateProxyInterfaces(beanClass, proxyFactory);
		}
	}

	Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
	proxyFactory.addAdvisors(advisors);
	proxyFactory.setTargetSource(targetSource);
	customizeProxyFactory(proxyFactory);

	proxyFactory.setFrozen(this.freezeProxy);
	if (advisorsPreFiltered()) {
		proxyFactory.setPreFiltered(true);
	}

	return proxyFactory.getProxy(getProxyClassLoader());
}
```

AbstractAutoProxyCreator的wrapIfNecessary方法是否需要生成代理，getAdvicesAndAdvisorsForBean代码如下：
```
@Override
protected Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, TargetSource targetSource) {
	List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
	if (advisors.isEmpty()) {
		return DO_NOT_PROXY;
	}
	return advisors.toArray();
}

```
findEligibleAdvisors方法为指定class寻找合适的Advisor，如下：
```
protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
	List<Advisor> candidateAdvisors = findCandidateAdvisors();
	List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
	extendAdvisors(eligibleAdvisors);
	if (!eligibleAdvisors.isEmpty()) {
		eligibleAdvisors = sortAdvisors(eligibleAdvisors);
	}
	return eligibleAdvisors;
}
```

findCandidateAdvisors方法的作用是寻找候选Advisors，根据上文的配置文件，有两个候选Advisor，分别是<aop:aspect>节点下的<aop:before>和<aop:after>这些节点，它们在XML解析的时候已经被转换生成了RootBeanDefinition。
```
protected List<Advisor> findCandidateAdvisors() {
	return this.advisorRetrievalHelper.findAdvisorBeans();
}
```

跳过第3行的代码，先看下第4行的代码extendAdvisors方法，它作用是向候选Advisor链的开头（也就是List.get(0)的位置）添加一个org.springframework.aop.support.DefaultPointcutAdvisor，代码实现在AspectJAwareAdvisorAutoProxyCreator类中，如下：
```
@Override
protected void extendAdvisors(List<Advisor> candidateAdvisors) {
	AspectJProxyUtils.makeAdvisorChainAspectJCapableIfNecessary(candidateAdvisors);
}
```

第3行findAdvisorsThatCanApply方法根据候选Advisors，寻找可以使用的Advisor，代码如下：
```
protected List<Advisor> findAdvisorsThatCanApply(
		List<Advisor> candidateAdvisors, Class<?> beanClass, String beanName) {

	ProxyCreationContext.setCurrentProxiedBeanName(beanName);
	try {
		return AopUtils.findAdvisorsThatCanApply(candidateAdvisors, beanClass);
	}
	finally {
		ProxyCreationContext.setCurrentProxiedBeanName(null);
	}
}
```
AopUtils的findAdvisorsThatCanApply方法如下：
```
public static List<Advisor> findAdvisorsThatCanApply(List<Advisor> candidateAdvisors, Class<?> clazz) {
	if (candidateAdvisors.isEmpty()) {
		return candidateAdvisors;
	}
	List<Advisor> eligibleAdvisors = new LinkedList<Advisor>();
	for (Advisor candidate : candidateAdvisors) {
		if (candidate instanceof IntroductionAdvisor && canApply(candidate, clazz)) {
			eligibleAdvisors.add(candidate);
		}
	}
	boolean hasIntroductions = !eligibleAdvisors.isEmpty();
	for (Advisor candidate : candidateAdvisors) {
		if (candidate instanceof IntroductionAdvisor) {
			// already processed
			continue;
		}
		if (canApply(candidate, clazz, hasIntroductions)) {
			eligibleAdvisors.add(candidate);
		}
	}
	return eligibleAdvisors;
}
```

整个方法的主要判断都围绕canApply展开方法：
```
public static boolean canApply(Advisor advisor, Class<?> targetClass, boolean hasIntroductions) {
	if (advisor instanceof IntroductionAdvisor) {
		return ((IntroductionAdvisor) advisor).getClassFilter().matches(targetClass);
	}
	else if (advisor instanceof PointcutAdvisor) {
		PointcutAdvisor pca = (PointcutAdvisor) advisor;
		return canApply(pca.getPointcut(), targetClass, hasIntroductions);
	}
	else {
		// It doesn't have a pointcut so we assume it applies.
		return true;
	}
}
```
第一个参数advisor的实际类型是AspectJPointcutAdvisor，它是PointcutAdvisor的子类，因此执行第7行的方法：
```
public static boolean canApply(Pointcut pc, Class<?> targetClass, boolean hasIntroductions) {
	Assert.notNull(pc, "Pointcut must not be null");
	if (!pc.getClassFilter().matches(targetClass)) {
		return false;
	}

	MethodMatcher methodMatcher = pc.getMethodMatcher();
	if (methodMatcher == MethodMatcher.TRUE) {
		// No need to iterate the methods if we're matching any method anyway...
		return true;
	}

	IntroductionAwareMethodMatcher introductionAwareMethodMatcher = null;
	if (methodMatcher instanceof IntroductionAwareMethodMatcher) {
		introductionAwareMethodMatcher = (IntroductionAwareMethodMatcher) methodMatcher;
	}

	Set<Class<?>> classes = new LinkedHashSet<Class<?>>(ClassUtils.getAllInterfacesForClassAsSet(targetClass));
	classes.add(targetClass);
	for (Class<?> clazz : classes) {
		Method[] methods = ReflectionUtils.getAllDeclaredMethods(clazz);
		for (Method method : methods) {
			if ((introductionAwareMethodMatcher != null &&
					introductionAwareMethodMatcher.matches(method, targetClass, hasIntroductions)) ||
					methodMatcher.matches(method, targetClass)) {
				return true;
			}
		}
	}

	return false;
}
```

个方法其实就是拿当前Advisor对应的expression做了两层判断：
* 目标类必须满足expression的匹配规则
* 目标类中的方法必须满足expression的匹配规则，当然这里方法不是全部需要满足expression的匹配规则，有一个方法满足即可

如果以上两条都满足，那么容器则会判断该<bean>满足条件，需要被生成代理对象，具体方式为返回一个数组对象，该数组对象中存储的是<bean>对应的Advisor。
