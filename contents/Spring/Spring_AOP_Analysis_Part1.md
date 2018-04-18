
## Spring AOP源码解析（上篇）

** 本文基于Spring 4.3.12.RELEASE源码分析 **

### Spring AOP 入门
为了探究AOP实现原理，首先定义几个类，一个Service接口：
```
public interface UserService {

	String login(String username, String password);
	
	String register(String username, String password);
}
```
Service接口实现类：
```
import org.springframework.stereotype.Service;

@Service("userService")
public class UserServiceImpl implements UserService {

	@Override
	public String login(String username, String password) {
		System.out.println("login username:"+username+",password:"+password);
		if(username==null){
			throw new NullPointerException("username is null");
		}
		return "OK";
	}

	@Override
	public String register(String username, String password) {
		System.out.println("register username:"+username+",password:"+password);
		return "OK";
	}

}
```
Spring AOP配置文件spring-aop.xml：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xmlns:context="http://www.springframework.org/schema/context"
	   xmlns:aop="http://www.springframework.org/schema/aop"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

	<context:component-scan base-package="demo.spring4.aop.xml" />

	<bean id="userService" class="demo.spring4.aop.service.impl.UserServiceImpl" />

	<!-- 配置切面 -->
	<bean id="logInterceptor" class="demo.spring4.aop.aspect.LogInterceptor" />
	<bean id="validationInterceptor" class="demo.spring4.aop.aspect.ValidationInterceptor" />

	<!-- aop配置 -->
	<aop:config>
		<!-- 配置切点表达式 -->
		<aop:pointcut expression="execution(* demo.spring4.aop.service.UserService.*(..))"
			id="pointcut"/>
		<!-- 配置切面及通知 -->
		<aop:aspect id="log" ref="logInterceptor" order="1">
			<aop:before method="beforeExecute" pointcut-ref="pointcut"/>
			<aop:after method="afterExecute" pointcut-ref="pointcut"/>
			<aop:after-throwing method="afterThrowing" pointcut-ref="pointcut" throwing="exception"/>
			<aop:after-returning method="afterReturning" pointcut-ref="pointcut" returning="result"/>
			<aop:around method="around" pointcut-ref="pointcut"/>
		</aop:aspect>
		<!--切面2-->
		<aop:aspect ref="validationInterceptor" order="2">
			<aop:before method="validateArgs" pointcut-ref="pointcut"/>
		</aop:aspect>
	</aop:config>

</beans>

```

测试代码：
```

public class SpringAopXmlApp {

	public static void main(String[] args) {
		
		ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:spring-aop-xml.xml");

		UserService userService = (UserService) ctx.getBean("userService");
		userService.login("ricky", "123");

		ctx.close();
	}

}
```

有了以上的内容，我们就可以根据这些跟一下代码，看看Spring到底是如何实现AOP的。

### Spring AOP NamespaceHandler
有很多朋友不愿意去看AOP源码的一个很大原因是因为找不到AOP源码实现的入口在哪里。其实magic主要在<aop:config>标签上，熟悉Spring Schema的同学应该知道它对应的Namespace 是http://www.springframework.org/schema/aop，对应的NamespaceHandler是```org.springframework.aop.config.AopNamespaceHandler```类，这可以在spring-aop-4.3.12.RELEASE.jar包里的 ```META-INF/spring.schemas```和```META-INF/spring.handler``` 看到。

```
public class AopNamespaceHandler extends NamespaceHandlerSupport {

	/**
	 * Register the {@link BeanDefinitionParser BeanDefinitionParsers} for the
	 * '{@code config}', '{@code spring-configured}', '{@code aspectj-autoproxy}'
	 * and '{@code scoped-proxy}' tags.
	 */
	@Override
	public void init() {
		// In 2.0 XSD as well as in 2.1 XSD.
		registerBeanDefinitionParser("config", new ConfigBeanDefinitionParser());
		registerBeanDefinitionParser("aspectj-autoproxy", new AspectJAutoProxyBeanDefinitionParser());
		registerBeanDefinitionDecorator("scoped-proxy", new ScopedProxyBeanDefinitionDecorator());

		// Only in 2.0 XSD: moved to context namespace as of 2.1
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
	}

}
```
不难看出，解析<aop:config>标签的是ConfigBeanDefinitionParser。


### AOP Bean定义加载
ConfigBeanDefinitionParser类主要代码如下：
```
class ConfigBeanDefinitionParser implements BeanDefinitionParser {

	@Override
	public BeanDefinition parse(Element element, ParserContext parserContext) {
		CompositeComponentDefinition compositeDef =
				new CompositeComponentDefinition(element.getTagName(), parserContext.extractSource(element));
		parserContext.pushContainingComponent(compositeDef);

		configureAutoProxyCreator(parserContext, element);

		List<Element> childElts = DomUtils.getChildElements(element);
		for (Element elt: childElts) {
			String localName = parserContext.getDelegate().getLocalName(elt);
			if (POINTCUT.equals(localName)) {
				parsePointcut(elt, parserContext);
			}
			else if (ADVISOR.equals(localName)) {
				parseAdvisor(elt, parserContext);
			}
			else if (ASPECT.equals(localName)) {
				parseAspect(elt, parserContext);
			}
		}

		parserContext.popAndRegisterContainingComponent();
		return null;
	}
  ```

configureAutoProxyCreator方法的作用有以下几点：
* 向Spring容器注册了一个BeanName为org.springframework.aop.config.internalAutoProxyCreator的Bean定义，可以自定义也可以使用Spring提供的（根据优先级来）
* Spring默认提供的是org.springframework.aop.aspectj.autoproxy.AspectJAwareAdvisorAutoProxyCreator，这个类是AOP的核心类，留在下篇讲解
* 根据配置proxy-target-class和expose-proxy，设置是否使用CGLIB进行代理以及是否暴露最终的代理。

### 处理<aop:aspect>流程
<aop:config>下的节点为<aop:aspect>，对应的是执行parseAspect方法，代码如下：
```

	private void parseAspect(Element aspectElement, ParserContext parserContext) {
		String aspectId = aspectElement.getAttribute(ID);
		String aspectName = aspectElement.getAttribute(REF);

		try {
			this.parseState.push(new AspectEntry(aspectId, aspectName));
			List<BeanDefinition> beanDefinitions = new ArrayList<BeanDefinition>();
			List<BeanReference> beanReferences = new ArrayList<BeanReference>();

			List<Element> declareParents = DomUtils.getChildElementsByTagName(aspectElement, DECLARE_PARENTS);
			for (int i = METHOD_INDEX; i < declareParents.size(); i++) {
				Element declareParentsElement = declareParents.get(i);
				beanDefinitions.add(parseDeclareParents(declareParentsElement, parserContext));
			}

			// We have to parse "advice" and all the advice kinds in one loop, to get the
			// ordering semantics right.
			NodeList nodeList = aspectElement.getChildNodes();
			boolean adviceFoundAlready = false;
			for (int i = 0; i < nodeList.getLength(); i++) {
				Node node = nodeList.item(i);
				if (isAdviceNode(node, parserContext)) {
					if (!adviceFoundAlready) {
						adviceFoundAlready = true;
						if (!StringUtils.hasText(aspectName)) {
							parserContext.getReaderContext().error(
									"<aspect> tag needs aspect bean reference via 'ref' attribute when declaring advices.",
									aspectElement, this.parseState.snapshot());
							return;
						}
						beanReferences.add(new RuntimeBeanReference(aspectName));
					}
					AbstractBeanDefinition advisorDefinition = parseAdvice(
							aspectName, i, aspectElement, (Element) node, parserContext, beanDefinitions, beanReferences);
					beanDefinitions.add(advisorDefinition);
				}
			}

			AspectComponentDefinition aspectComponentDefinition = createAspectComponentDefinition(
					aspectElement, aspectId, beanDefinitions, beanReferences, parserContext);
			parserContext.pushContainingComponent(aspectComponentDefinition);

			List<Element> pointcuts = DomUtils.getChildElementsByTagName(aspectElement, POINTCUT);
			for (Element pointcutElement : pointcuts) {
				parsePointcut(pointcutElement, parserContext);
			}

			parserContext.popAndRegisterContainingComponent();
		}
		finally {
			this.parseState.pop();
		}
	}

```

for循环有一个关键的判断就是ifAdviceNode判断，ifAdviceNode方法：
```
	private boolean isAdviceNode(Node aNode, ParserContext parserContext) {
		if (!(aNode instanceof Element)) {
			return false;
		}
		else {
			String name = parserContext.getDelegate().getLocalName(aNode);
			return (BEFORE.equals(name) || AFTER.equals(name) || AFTER_RETURNING_ELEMENT.equals(name) ||
					AFTER_THROWING_ELEMENT.equals(name) || AROUND.equals(name));
		}
	}
```

即这个for循环只用来处理<aop:aspect>标签下的<aop:before>、<aop:after>、<aop:after-returning>、<aop:after-throwing method="">和<aop:around method="">这五个标签的。

接着，如果是上述五种标签之一，那么进入parseAdvice方法：
```
	private AbstractBeanDefinition parseAdvice(
			String aspectName, int order, Element aspectElement, Element adviceElement, ParserContext parserContext,
			List<BeanDefinition> beanDefinitions, List<BeanReference> beanReferences) {

		try {
			this.parseState.push(new AdviceEntry(parserContext.getDelegate().getLocalName(adviceElement)));

			// create the method factory bean
			RootBeanDefinition methodDefinition = new RootBeanDefinition(MethodLocatingFactoryBean.class);
			methodDefinition.getPropertyValues().add("targetBeanName", aspectName);
			methodDefinition.getPropertyValues().add("methodName", adviceElement.getAttribute("method"));
			methodDefinition.setSynthetic(true);

			// create instance factory definition
			RootBeanDefinition aspectFactoryDef =
					new RootBeanDefinition(SimpleBeanFactoryAwareAspectInstanceFactory.class);
			aspectFactoryDef.getPropertyValues().add("aspectBeanName", aspectName);
			aspectFactoryDef.setSynthetic(true);

			// register the pointcut
			AbstractBeanDefinition adviceDef = createAdviceDefinition(
					adviceElement, parserContext, aspectName, order, methodDefinition, aspectFactoryDef,
					beanDefinitions, beanReferences);

			// configure the advisor
			RootBeanDefinition advisorDefinition = new RootBeanDefinition(AspectJPointcutAdvisor.class);
			advisorDefinition.setSource(parserContext.extractSource(adviceElement));
			advisorDefinition.getConstructorArgumentValues().addGenericArgumentValue(adviceDef);
			if (aspectElement.hasAttribute(ORDER_PROPERTY)) {
				advisorDefinition.getPropertyValues().add(
						ORDER_PROPERTY, aspectElement.getAttribute(ORDER_PROPERTY));
			}

			// register the final advisor
			parserContext.getReaderContext().registerWithGeneratedName(advisorDefinition);

			return advisorDefinition;
		}
		finally {
			this.parseState.pop();
		}
	}
```

方法主要做了三件事：
* 根据织入方式（before、after这些）创建RootBeanDefinition，名为adviceDef即advice定义
* 将上一步创建的RootBeanDefinition写入一个新的RootBeanDefinition，构造一个新的对象，名为advisorDefinition，即advisor定义
* 将advisorDefinition注册到DefaultListableBeanFactory中

下面来看做的第一件事createAdviceDefinition方法定义：
```
	private AbstractBeanDefinition createAdviceDefinition(
			Element adviceElement, ParserContext parserContext, String aspectName, int order,
			RootBeanDefinition methodDef, RootBeanDefinition aspectFactoryDef,
			List<BeanDefinition> beanDefinitions, List<BeanReference> beanReferences) {

		RootBeanDefinition adviceDefinition = new RootBeanDefinition(getAdviceClass(adviceElement, parserContext));
		adviceDefinition.setSource(parserContext.extractSource(adviceElement));

		adviceDefinition.getPropertyValues().add(ASPECT_NAME_PROPERTY, aspectName);
		adviceDefinition.getPropertyValues().add(DECLARATION_ORDER_PROPERTY, order);

		if (adviceElement.hasAttribute(RETURNING)) {
			adviceDefinition.getPropertyValues().add(
					RETURNING_PROPERTY, adviceElement.getAttribute(RETURNING));
		}
		if (adviceElement.hasAttribute(THROWING)) {
			adviceDefinition.getPropertyValues().add(
					THROWING_PROPERTY, adviceElement.getAttribute(THROWING));
		}
		if (adviceElement.hasAttribute(ARG_NAMES)) {
			adviceDefinition.getPropertyValues().add(
					ARG_NAMES_PROPERTY, adviceElement.getAttribute(ARG_NAMES));
		}

		ConstructorArgumentValues cav = adviceDefinition.getConstructorArgumentValues();
		cav.addIndexedArgumentValue(METHOD_INDEX, methodDef);

		Object pointcut = parsePointcutProperty(adviceElement, parserContext);
		if (pointcut instanceof BeanDefinition) {
			cav.addIndexedArgumentValue(POINTCUT_INDEX, pointcut);
			beanDefinitions.add((BeanDefinition) pointcut);
		}
		else if (pointcut instanceof String) {
			RuntimeBeanReference pointcutRef = new RuntimeBeanReference((String) pointcut);
			cav.addIndexedArgumentValue(POINTCUT_INDEX, pointcutRef);
			beanReferences.add(pointcutRef);
		}

		cav.addIndexedArgumentValue(ASPECT_INSTANCE_FACTORY_INDEX, aspectFactoryDef);

		return adviceDefinition;
	}
```

首先可以看到，创建的AbstractBeanDefinition实例是RootBeanDefinition，这和普通Bean创建的实例为GenericBeanDefinition不同。然后进入第6行的getAdviceClass方法看一下：
```
	private Class<?> getAdviceClass(Element adviceElement, ParserContext parserContext) {
		String elementName = parserContext.getDelegate().getLocalName(adviceElement);
		if (BEFORE.equals(elementName)) {
			return AspectJMethodBeforeAdvice.class;
		}
		else if (AFTER.equals(elementName)) {
			return AspectJAfterAdvice.class;
		}
		else if (AFTER_RETURNING_ELEMENT.equals(elementName)) {
			return AspectJAfterReturningAdvice.class;
		}
		else if (AFTER_THROWING_ELEMENT.equals(elementName)) {
			return AspectJAfterThrowingAdvice.class;
		}
		else if (AROUND.equals(elementName)) {
			return AspectJAroundAdvice.class;
		}
		else {
			throw new IllegalArgumentException("Unknown advice kind [" + elementName + "].");
		}
	}

```

### 处理<aop:pointcut>流程
回到ConfigBeanDefinitionParser的parseAspect方法：
···

			AspectComponentDefinition aspectComponentDefinition = createAspectComponentDefinition(
					aspectElement, aspectId, beanDefinitions, beanReferences, parserContext);
			parserContext.pushContainingComponent(aspectComponentDefinition);

			List<Element> pointcuts = DomUtils.getChildElementsByTagName(aspectElement, POINTCUT);
			for (Element pointcutElement : pointcuts) {
				parsePointcut(pointcutElement, parserContext);
			}

			parserContext.popAndRegisterContainingComponent();

···

代码拿到所有<aop:aspect>下的pointcut标签，进行遍历，由parsePointcut方法进行处理：
```
	private AbstractBeanDefinition parsePointcut(Element pointcutElement, ParserContext parserContext) {
		String id = pointcutElement.getAttribute(ID);
		String expression = pointcutElement.getAttribute(EXPRESSION);

		AbstractBeanDefinition pointcutDefinition = null;

		try {
			this.parseState.push(new PointcutEntry(id));
			pointcutDefinition = createPointcutDefinition(expression);
			pointcutDefinition.setSource(parserContext.extractSource(pointcutElement));

			String pointcutBeanName = id;
			if (StringUtils.hasText(pointcutBeanName)) {
				parserContext.getRegistry().registerBeanDefinition(pointcutBeanName, pointcutDefinition);
			}
			else {
				pointcutBeanName = parserContext.getReaderContext().registerWithGeneratedName(pointcutDefinition);
			}

			parserContext.registerComponent(
					new PointcutComponentDefinition(pointcutBeanName, pointcutDefinition, expression));
		}
		finally {
			this.parseState.pop();
		}

		return pointcutDefinition;
	}
```

createPointcutDefinition方法如下：
```
	protected AbstractBeanDefinition createPointcutDefinition(String expression) {
		RootBeanDefinition beanDefinition = new RootBeanDefinition(AspectJExpressionPointcut.class);
		beanDefinition.setScope(BeanDefinition.SCOPE_PROTOTYPE);
		beanDefinition.setSynthetic(true);
		beanDefinition.getPropertyValues().add(EXPRESSION, expression);
		return beanDefinition;
	}
```

### 参考资料
[Spring源码剖析（六）AOP实现原理剖析](https://www.cnblogs.com/xrq730/p/6753160.html)
