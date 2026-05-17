## 1.spring源码编译和xml解析

>
>
>编译spring的版本 5.2.8
>

Jack的博客

https://blog.csdn.net/luoyang_java?type=blog

学员博客

https://yangzhiwen911.github.io

## 2.BeanDefinition和默认标签、自定义标签解析

### 2.1 BeanDefinition精讲

```
1.
AbstractApplicationContext#refresh();
2. 创建BeanFactory对象
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
3. 创建BeanFactory对象 的 核心方法，必须读，重要程度：5
AbstractApplicationContext#refreshBeanFactory();
4. 创建BeanFactory对象 的 执行代码
AbstractRefreshableApplicationContext#refreshBeanFactory()
5. 解析xml，并把xml中的标签封装成BeanDefinition对象
AbstractRefreshableApplicationContext#loadBeanDefinitions
6. 解析xml 执行代码 入口  通过流的形式加载xml配置文件
AbstractXmlApplicationContext#loadBeanDefinitions
//创建xml的解析器，这里是一个委托模式
XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
7. 重载方法,执行 加载的xml配置文件 解析xml文件为BeanDefinition
AbstractXmlApplicationContext#loadBeanDefinitions
8. Reader 加载 xml的入口
AbstractBeanDefinitionReader#loadBeanDefinitions
9.重载方法 执行加载BeanDefinition的逻辑
public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
		Assert.notNull(resources, "Resource array must not be null");
		int count = 0;
		for (Resource resource : resources) {
			//模板设计模式，调用到子类中的方法
			count += loadBeanDefinitions(resource);
		}
		return count;
	}
	
10. 重载方法调用 最终执行加载的逻辑
XmlBeanDefinitionReader#loadBeanDefinitions
doLoadBeanDefinitions(inputSource, encodedResource.getResource());
主要看这个方法，根据解析出来的document对象，拿到里面的标签元素封装成BeanDefinition
XmlBeanDefinitionReader#registerBeanDefinitions

public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		//又来一记委托模式，BeanDefinitionDocumentReader委托这个类进行document的解析
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		int countBefore = getRegistry().getBeanDefinitionCount();

		//主要看这个方法，createReaderContext(resource) XmlReaderContext上下文，封装了XmlBeanDefinitionReader对象
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
	
11 执行注册BeanDefinition逻辑
DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions

protected void doRegisterBeanDefinitions(Element root) {

		preProcessXml(root);
		//主要看这个方法，标签具体解析过程
		parseBeanDefinitions(root, this.delegate);
		postProcessXml(root);

		this.delegate = parent;
	}
	
parseBeanDefinitions  这个方法来执行默认标签与自定义标签的解析

12 
if (delegate.isDefaultNamespace(ele)) {
						//默认标签解析
						parseDefaultElement(ele, delegate);
					}
					else {
						//自定义标签解析
						delegate.parseCustomElement(ele);
					}
					
13

```

2.1.13

```java
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
		//import标签解析  重要程度 1 ，可看可不看
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
		//alias标签解析 别名标签  重要程度 1 ，可看可不看
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
		//bean标签，重要程度  5，必须看
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}

processBeanDefinition 执行逻辑
1.
DefaultBeanDefinitionDocumentReader#processBeanDefinition
//重点看这个方法，重要程度 5 ，解析document，封装成BeanDefinition
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);

BeanDefinitionParserDelegate#parseBeanDefinitionElement
//<bean> 标签解析的核心方法  重要程度5
		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
// 这个方法执行解析Bean标签的逻辑
BeanDefinitionParserDelegate#parseBeanDefinitionElement

保存对象中的属性和值
MutablePropertyValues
```



### 2.2 默认标签解析

```
传统标签解析：bean、import等
```

### 2.3 自定义标签解析

```java
* 	自定义标签解析 如：<context:component-scan base-package="com.xiangxue.jack"/>
* 	自定义标签解析流程：
* 		a、根据当前解析标签的头信息找到对应的namespaceUri
* 		b、加载spring所有jar中的spring.handlers文件。并建立映射关系
* 		c、根据namespaceUri从映射关系中找到对应的实现了NamespaceHandler接口的类
* 		d、调用类的init方法，init方法是注册了各种自定义标签的解析类
* 		e、根据namespaceUri找到对应的解析类，然后调用paser方法完成标签解析
|
|
自定义标签解析的入口
NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);

public NamespaceHandler resolve(String namespaceUri) {
		//获取spring中所有jar包里面的 "META-INF/spring.handlers"文件，并且建立映射关系
		Map<String, Object> handlerMappings = getHandlerMappings();
		//根据namespaceUri：http://www.springframework.org/schema/context，获取到这个命名空间的处理类
		Object handlerOrClassName = handlerMappings.get(namespaceUri);
		
//加载"META-INF/spring.handlers"文件过程
						Properties mappings =								PropertiesLoaderUtils.loadAllProperties(this.handlerMappingsLocation, this.classLoader);

这种方式可以用来自定义框架  例如dubbo 就是采用这种自定义标签的方式实现的
property-placeholder  是 ContextNamespaceHandler 来初始化,通过
new PropertyPlaceholderBeanDefinitionParser() 来实现解析
```

### 2.4 装饰器设计模式

```
装饰器设计模式  与 动态代理 的区别
```



## 3.component-scan标签解析和bean实例化初探

### 1.Context 标签配置的标签明细

```java
public class ContextNamespaceHandler extends NamespaceHandlerSupport {

	@Override
	public void init() {
		registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
		registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
		registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
		registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
		registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
	}

}
```

### 2.component-scan标签的解析

```java
component-scan标签解析  
由org.springframework.context.config.ContextNamespaceHandler 名称空间来初始化解析类
new ComponentScanBeanDefinitionParser()

调试入口
BeanDefinitionParserDelegate

public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
		String namespaceUri = getNamespaceURI(ele);
		if (namespaceUri == null) {
			return null;
		}
		NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
		if (handler == null) {
			error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
			return null;
		}
		return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
	}
	
handler.parse  方法可以debug  componentScan标签的解析过程

最后找到解析器 ComponentScanBeanDefinitionParser
用来解析执行componentScan的逻辑

String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
				ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS)
    
字符串 转 数组的 方法 支持 ,;\t\n这4种方式的字符串

//创建注解扫描器
		// Actually scan for bean definitions and register them.
		ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
		//扫描并把扫描的类封装成beanDefinition对象  核心方法，重要程度 5
		Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);

use-default-filters  过滤器属性  默认为true

ComponentScan 默认扫描的注解
@Component
@Service
@Controller
@Repository
@Configuration

doScan 的执行逻辑
1.去扫描基本包的路径,找.class文件
2.递归找.class文件
找到 .class文件 File,会包装成 Resource 对象
3.判断.class文件里面是否有注解,includeFilter里面的注解@Component
4.变成beanDefinition对象

//扫描到有注解的类并封装成BeanDefinition对象
			Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
scanCandidateComponents(basePackage)  // 实现将File文件转换成Resource
一个Resource 对应 一个 File

//包装了类的基本信息的对象
						MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
						//如果类上面有includeFilters注解
						if (isCandidateComponent(metadataReader)) {
							ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
							sbd.setSource(resource);
							if (isCandidateComponent(sbd)) {
								if (debugEnabled) {
									logger.debug("Identified candidate component class: " + resource);
								}
								candidates.add(sbd);
							}
// 这几行代码 实现了将Resource转换成BeanDefinition对象
//   isCandidateComponent(metadataReader)  这个过滤器会判断哪些类可以实现     Resource转换成BeanDefinition对象 
                            
1.将Resource解析成BeanDefinition
2.补全BeanDefinition缺失的值
BeanDefinitionReaderUtils#registerBeanDefinition
将BeanDefinition注册到BeanDefinitionRegistry(本质是一个BeanFactory)中
    
 ComponentScan完成注解扫描之后
 会注册几个定义的处理器组件
1.ComponentScanBeanDefinitionParser#registerComponents
2.AnnotationConfigUtils#registerAnnotationConfigProcessors
3.会注册
ConfigurationClassPostProcessor.class
AutowiredAnnotationBeanPostProcessor.class
CommonAnnotationBeanPostProcessor.class
EventListenerMethodProcessor.class
DefaultEventListenerFactory.class
这个几个顶级的解析其他注解的处理器
ConfigurationClassPostProcessor.class是BeanFactoryPostProcessor接口
ConfigurationClassPostProcessor 
用来解析Configuration, Import,Bean注解  
AutowiredAnnotationBeanPostProcessor.class
CommonAnnotationBeanPostProcessor.class 是 BeanPostProcessor接口
AutowiredAnnotationBeanPostProcessor & CommonAnnotationBeanPostProcessor
用来解析 @Autowire 等注解
 
通过自定义标签的解析完成
ConfigurationClassPostProcessor.class
AutowiredAnnotationBeanPostProcessor.class
CommonAnnotationBeanPostProcessor.class  BeanDefinition对象的创建
    
以上的逻辑都是在
ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
这行代码中执行完成  创建BeanFactory并完成标签的解析        
    
prepareBeanFactory(beanFactory); 给给beanFactory设置一些属性值
    
    
/*
				 * BeanDefinitionRegistryPostProcessor
				 * BeanFactoryPostProcessor
				 * 完成对这两个接口的调用
				 * */
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);
                            
                            
                            
  BeanDefinitionRegistryPostProcessor  可以对BeanDefinition执行增删改查
  调用的时序 在 Bean的实例化之前
  invokeBeanFactoryPostProcessors 中会通过invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);完成对
 postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)方法的调用
      
  最后会通过
      invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);完成对  postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 方法的调用
      
      
通过这个方法可以完成指定注解扫描加载
ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(registry);
//需要过滤的注解
scanner.addIncludeFilter(new AnnotationTypeFilter(MyService.class));
scanner.scan("com.enjoy.jack2021.customBean");
```



## 4.BeanPostProcessor和Bean实例化初探

`AbstractApplicationContext#refresh()方法执行中 每个方法执行的逻辑解释`

```java
/*
* 把实现了BeanPostProcessor接口的类实例化，并且加入到BeanFactory中
				 * */
// Register bean processors that intercept bean creation.
registerBeanPostProcessors(beanFactory);
|
|
//初始化事件管理类
// Initialize event multicaster for this context.
initApplicationEventMulticaster();
|
|
事件监听案例
 //为什么要add？因为启动后把事件监听加入
applicationContext.addApplicationListener(new EnjoyApplicationListener());
//发布事件
applicationContext.publishEvent(new EnjoyEvent("Jack", "enjoyEvent"));
applicationContext.start();
|
|
//这个方法着重理解模板设计模式，因为在springboot中，这个方法是用来做内嵌tomcat启动的
// Initialize other special beans in specific context subclasses.
onRefresh();
```

Bean实例化的方法

```java
/*
 * 这个方法是spring中最重要的方法，没有之一
 * 所以这个方法一定要理解要具体看
 * 1、bean实例化过程
 * 2、ioc
 * 3、注解支持
 * 4、BeanPostProcessor的执行
 * 5、Aop的入口
 * */
finishBeanFactoryInitialization(beanFactory);
|
|
//重点看这个方法，重要程度：5
beanFactory.preInstantiateSingletons();
|
|
//具体实例化过程
DefaultListableBeanFactory#preInstantiateSingletons()
|
// Trigger initialization of all non-lazy singleton beans...
for (String beanName : beanNames) {
//把父BeanDefinition里面的属性拿到子BeanDefinition中
RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
//如果不是抽象的，单例的，非懒加载的就实例化
if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
//判断bean是否实现了FactoryBean接口，这里可以不看  &factoryBeanDemo
if (isFactoryBean(beanName)) {
Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
if (bean instanceof FactoryBean) {
FactoryBean<?> factory = (FactoryBean<?>) bean;
boolean isEagerInit;
if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
isEagerInit = AccessController.doPrivileged(
(PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
else {
isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
if (isEagerInit) {
getBean(beanName);
}
}
}
else {
//主要从这里进入，看看实例化过程
getBean(beanName);
}
}
}
```

```java
// 获得Bean
AbstractBeanFactory#doGetBean
|
|
//从缓存中拿实例
// 场景一 //如果缓存里面能拿到实例
// Eagerly check singleton cache for manually registered singletons.
Object sharedInstance = getSingleton(beanName);
//该方法是FactoryBean接口的调用入口
bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);

// 场景二 如果缓存里面不能拿到实例
//如果singletonObjects缓存里面没有，则走下来
// Fail if we're already creating this bean instance:
// We're assumably within a circular reference.
//如果是scope 是Prototype的，校验是否有出现循环依赖，如果有则直接报错
if (isPrototypeCurrentlyInCreation(beanName)) {
throw new BeanCurrentlyInCreationException(beanName);
}
|
|
//父子BeanDefinition合并
RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
//获取依赖对象属性，依赖对象要先实例化
// Guarantee initialization of beans that the current bean depends on.
String[] dependsOn = mbd.getDependsOn();
|
|
//着重看，大部分是单例的情况
sharedInstance = getSingleton(beanName, () -> {
	try {
		return createBean(beanName, mbd, args);
        }
|
|
// getSingleton 三级缓存获取逻辑
|
|
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
//根据beanName从缓存中拿实例
//先从一级缓存拿
Object singletonObject = this.singletonObjects.get(beanName);
//如果bean还正在创建，还没创建完成，其实就是堆内存有了，属性还没有DI依赖注入
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		synchronized (this.singletonObjects) {
//从二级缓存中拿
		singletonObject = this.earlySingletonObjects.get(beanName);
//如果还拿不到，并且允许bean提前暴露
		if (singletonObject == null && allowEarlyReference) {
//从三级缓存中拿到对象工厂
		ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
		if (singletonFactory != null) {
//从工厂中拿到对象
		singletonObject = singletonFactory.getObject();
//升级到二级缓存
		this.earlySingletonObjects.put(beanName, singletonObject);
//删除三级缓存
		this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
|
|    
// 创建Bean
AbstractAutowireCapableBeanFactory#createBean
/*
* TargetSource接口的运用，可以在用改一个类实现该接口，然后在里面定义实例化对象的方式，然后返回
* 也就是说不需要spring帮助我们实例化对象
*
*
* 这里可以直接返回实例本身
*
* 这个代码不用看，实际开发过程中用不到，我会做为一个甜点分享
* */
// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
|
// AbstractAutowireCapableBeanFactory#doCreateBean会触发执行的逻辑
|    
//主要看这个方法，重要程度 5
Object beanInstance = doCreateBean(beanName, mbdToUse, args);
|
|
//创建实例,,重点看，重要程度：5 
instanceWrapper = createBeanInstance(beanName, mbd, args);
// createBeanInstance 执行完成,只是在堆中划分了一个内存地址,对象的属性还没有赋值
|
|
//CommonAnnotationBeanPostProcessor  支持了@PostConstruct，@PreDestroy,@Resource注解
//AutowiredAnnotationBeanPostProcessor 支持 @Autowired,@Value注解
//BeanPostProcessor接口的典型运用，这里要理解这个接口
//对类中注解的装配过程
//重要程度5，必须看
applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
|
|    
//这里着重理解，对理解循环依赖帮助非常大，重要程度 5   添加三级缓存
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
|
|
//ioc di，依赖注入的核心方法，该方法必须看，重要程度：5
populateBean(beanName, mbd, instanceWrapper);
//bean 实例化+ioc依赖注入完以后的调用，非常重要，重要程度：5
exposedObject = initializeBean(beanName, exposedObject, mbd);
|
|    
//注册bean销毁时的类DisposableBeanAdapter
registerDisposableBeanIfNecessary(beanName, bean, mbd);
```

## 5.bean的实例化和注解的收集

`AbstractAutowireCapableBeanFactory#createBeanInstance执行逻辑`

```java
1.实例化factoryMethod方法对应的实例
factoryMethod方法 触发条件
(mbd.getFactoryMethodName() != null)
<bean>标签里面  配置了factory-method属性
方法上面加上@Bean注解
@Bean 是 另一个类的非静态方法 实现的
factory-method 有两种调用形式
1.另一个类的非静态方法
2.当前类中的静态方法调用
|
|
2.实例化带有@Autowire注解的有参构造函数
3.实例化没有@Autowire注解的有参构造函数
4.实例化无参构造函数

BeanWrapper 的实例对象 BeanWrapperImpl
|
|
//@Autowire注解的有参构造函数执行逻辑
Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
			//如果ctors不为空，就说明构造函数上有@Autowired注解
			return autowireConstructor(beanName, mbd, ctors, args);
		}
// autowireConstructor 执行 @Autowire注解的有参构造

// new ConstructorResolver(this).autowireConstructor(beanName, mbd, ctors, explicitArgs);

// 通过 ConstructorResolver#autowireConstructor 实现 @Autowire注解的有参构造 解析

// @Autowire注解的方法或者属性都会触发getBean操作

//@Autowired 修饰有多个构造函数,需要@Autowired(required = false),不然执行会报错
```

### applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);

```java
这个方法是收集注解解析的处理器
//CommonAnnotationBeanPostProcessor  支持了@PostConstruct，@PreDestroy,@Resource注解
//AutowiredAnnotationBeanPostProcessor 支持 @Autowired,@Value注解
//1.AutowiredAnnotationBeanPostProcessor核心方法
AutowiredAnnotationBeanPostProcessor#findAutowiringMetadata

//主要看这个方法
metadata = buildAutowiringMetadata(clazz);

收集对象中的 属性 与 方法  是否有@Autowired注解
把收集到注解的属性,包装成一个对象

//寻找field上面的@Autowired注解并封装成对象
ReflectionUtils.doWithLocalFields(targetClass, field -> {

//寻找Method上面的@Autowired注解并封装成对象
ReflectionUtils.doWithLocalMethods(targetClass, method -> {

收集到的包装对象
final List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();
```



## 6.@Autowired@PostConstruct@PreDestroy注解

### 1.applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);解析



>@Autowired 注解由 AutowiredAnnotationBeanPostProcessor解析
>
>@PostConstruct & @PreDestroy注解 由 CommonAnnotationBeanPostProcessor 解析

```java
public CommonAnnotationBeanPostProcessor() {
		setOrder(Ordered.LOWEST_PRECEDENCE - 3);
		setInitAnnotationType(PostConstruct.class);
		setDestroyAnnotationType(PreDestroy.class);
		ignoreResourceType("javax.xml.ws.WebServiceContext");
	}
// 此处初始化 PostConstruct.class & PreDestroy.class 注解处理类型

CommonAnnotationBeanPostProcessor 的
super.postProcessMergedBeanDefinition(beanDefinition, beanType, beanName); // 收集处理 PostConstruct.class & PreDestroy.class
InjectionMetadata metadata = findResourceMetadata(beanName, beanType, null);
// 收集处理 webServiceRefClass  ejbClass  Resource.class
```



### 2.populateBean(beanName, mbd, instanceWrapper);

```java
// Aware 接口处理器的处理逻辑
InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
return;
}
//依赖注入过程，@Autowired的支持
PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);

// 依赖注入引用数据类型 会触发getBean
代码执行链路
1.
PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
2.
AutowiredAnnotationBeanPostProcessor#postProcessProperties
3.
InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
4.
metadata = buildAutowiringMetadata(clazz);
metadata.inject(bean, beanName, pvs);
InjectionMetadata#inject
element.inject(target, beanName, pvs); // 模板方法
AutowiredFieldElement#inject
//这里会触发依赖注入属性的getBean操作
value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
DefaultListableBeanFactory#resolveDependency
DefaultListableBeanFactory#doResolveDependency
findAutowireCandidates(beanName, elementType,
					new MultiElementDescriptor(descriptor));
DefaultListableBeanFactory#addCandidateEntry
//这里会调用getBean
descriptor.resolveCandidate(candidateName, requiredType, this);


//获取@Value中的值，值类似于${enjoy.name}
Object value = getAutowireCandidateResolver().getSuggestedValue(descriptor);


```

### 3.initializeBean(beanName, exposedObject, mbd)

```java
//调用Aware方法
invokeAwareMethods(beanName, bean);

//对类中某些特殊方法的调用，比如@PostConstruct，Aware接口，非常重要 重要程度 ：5
//ApplicationContextAwareProcessor 对Aware接口的调用如：
//EnvironmentAware EmbeddedValueResolverAware  ResourceLoaderAware ApplicationEventPublisherAware MessageSourceAware  ApplicationContextAware
			
//ImportAwareBeanPostProcessor 对ImportAware的支持
wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);

//InitializingBean接口，afterPropertiesSet，init-method属性调用,非常重要，重要程度：5
invokeInitMethods(beanName, wrappedBean, mbd);

//这个地方可能生出代理实例，是aop的入口
wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
```

```java
applyBeanPostProcessorsBeforeInitialization

Object current = processor.postProcessBeforeInitialization(result, beanName);
InitDestroyAnnotationBeanPostProcessor#postProcessBeforeInitialization // 初始化注解处理器
    
ConfigurationClassPostProcessor 解析到@Import注解会调用
registerImport()方法

ImportAwareBeanPostProcessor会处理importRegistry的元数据

如果定义的类想拿到注解信息  不能通过@Component注解
需要使用@import注解
ImportAware  可以拿到 注解信息
```

## 7.spring中循环依赖详解

```java
// AbstractAdvisingBeanPostProcessor  AOP的切面处理器
1.单例循环依赖 (可以)
2.构造函数循环依赖 (不行)
3.多例循环依赖(不行)
    
// 循环依赖的入口
getBean
|
|
getSingleton(beanName)
// 第一次进来 getSingleton拿不到实例,需要走
 getSingleton(String beanName, ObjectFactory<?> singletonFactory)   
sharedInstance = getSingleton(beanName, () -> {
						try {
							return createBean(beanName, mbd, args);
						}
					});

//把beanName添加到singletonsCurrentlyInCreation Set容器中，在这个集合里面的bean都是正在实例化的bean
		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
			throw new BeanCurrentlyInCreationException(beanName);
		}

//DefaultSingletonBeanRegistry#getSingleton  这个方法是单例Bean创建的入口
1.
//把beanName添加到singletonsCurrentlyInCreation Set容器中，在这个集合里面的bean都是正在实例化的bean
beforeSingletonCreation(beanName);
2.
//如果这里有返回值，就代表这个bean已经结束创建了，已经完全创建成功
singletonObject = singletonFactory.getObject();
--> 这个方法就是调用 return createBean(beanName, mbd, args);
3.
//bean创建完成后singletonsCurrentlyInCreation要删除该bean
afterSingletonCreation(beanName);
4.
//创建对象成功时，把对象缓存到singletonObjects缓存中,bean创建完成时放入一级缓存
addSingleton(beanName, singletonObject);



```

### 循环依赖的解决代码

```java
在createBean的代码块中
doCreateBean
// 在 instanceWrapper = createBeanInstance(beanName, mbd, args);
// 与 populateBean(beanName, mbd, instanceWrapper);之间 有一段允许Bean提前暴露的逻辑

//是否	单例bean提前暴露
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			//这里着重理解，对理解循环依赖帮助非常大，重要程度 5   添加三级缓存
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
// 这里会把Bean的引用添加到三级缓存中,在循环引用时,就可以被另外一个Bean实例化的时候获取到

() -> getEarlyBeanReference(beanName, mbd, bean)
可以通过BeanPostProcessor创建代理对象

循环依赖中
A,B
A 的getBean触发 在populateBean中触发B的getBean
B 依赖注入完成,B 创建成功,
此时A的依赖注入成功,A创建完成
//SmartInstantiationAwareBeanPostProcessor  这个处理器可以在Bean的创建过程中修改Bean的属性
//SmartInstantiationAwareBeanPostProcessor#getEarlyBeanReference
```

### 构造函数的循环依赖

```
构造函数的循环依赖 A,B 
A在创建Bean的时候,通过构造方法实例化,会触发B的实例化,此时A还没来得及将自己的引用放在三级缓存中,就触发了B的getBean方法

```



## 8.spring中配置文件解析和Environment对象 

### Spring的配置文件解析

````java
1.xml解析将Bean中的属性,封装成Property属性,
Property属性 的 key = xxx ,Value = "${enjoy.name}"

2.可以通过BeanPostProcessor处理器,在Bean的属性赋值阶段,将占位符替换成对应的值

3.执行占位符解析的处理器,优先级时最低的,确保在最后一个执行,可以替换Bean最后的占位符

4.xml 执行占位符解析必须加上
<context:property-placeholder location="classpath:application.properties"/>
如果不加这个配置Spring就解析不了占位符,启动就会报错

5.也可以通过PropertySourcesPlaceholderConfigurer来实现占位符的解析
 @Bean
public PropertySourcesPlaceholderConfigurer getPropertySourcesPlaceholderConfigurer() {
PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer = new PropertySourcesPlaceholderConfigurer();
        propertySourcesPlaceholderConfigurer.setLocation(resourceLoader.getResource("application-1.properties"));
        return propertySourcesPlaceholderConfigurer;
    }

// PropertySourcesPlaceholderConfigurer 就是一个实现了BeanFactoryPostProcessor的类 可以在 postProcessBeanFactory 中修改 BeanDefinition 来实现占位符的解析
占位符 的解析  有两个数据源properties文件与environment中查找解析 占位符中对应的值
PropertySourcesPlaceholderConfigurer#postProcessBeanFactory 占位符执行解析的入口

6.也可以通过实现BeanDefinitionRegistryPostProcessor自定义处理器修改BeanDefinition实现


占位符 也可以用来实现化Bean,但是没必要

@Value 的 处理逻辑 与 @Autowire的逻辑一样,也是先收集,后面处理
依赖注入的解析  与  xml中配置的解析流程一样,只是执行的时机不一样
````

## 9.自定义scope和factoryBean接口

在environment中添加属性

````java
1.实现 EnvironmentAware 
2.在setEnvironment(Environment environment)添加值
System.out.println("PropertiesPro.setEnvironment");
StandardEnvironment bean = (StandardEnvironment)environment;

Properties properties = new Properties();
properties.put("enjoy.name","James");

PropertiesPropertySource propertiesCustom = new PropertiesPropertySource("propertiesCustom", properties);

MutablePropertySources propertySources = bean.getPropertySources();
propertySources.addLast(propertiesCustom);
````

### 自定义scope

```
自定义scope,可以完成bean的自定义管理
```

### 基于注解AnnotationConfigApplicationContext的启动

```java
常用注解的解析
@PropertySource
@ComponentScan
@Import
@Bean
@ImportSource
@Configuration
importSelector

// @PropertySource 这个注解相当于<context:property-placeholder location="classpath:application.properties"/>
```

### AnnotationConfigApplicationContext的启动过程

```java
ConfigurationClassPostProcessor的注册入口

public AnnotationConfigApplicationContext() {
this.reader = new AnnotatedBeanDefinitionReader(this);
this.scanner = new ClassPathBeanDefinitionScanner(this);
}

public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {

public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
		this.registry = registry;
		this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
		AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
	}
	
AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);

// 这个方法完成定义BeanFactoryPostProcessor处理器的注册
registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
		registerAnnotationConfigProcessors(registry, null);
	}
	
RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
//ConfigurationClassPostProcessor implements BeanDefinitionRegistryPostProcessor
//操作BeanDefinition并且加入到BeanDefinitionRegistry中
//ConfigurationClassPostProcessor的优先级最低,需要在所有的BeanDefinitionRegistry处理完了之后才执行BeanDefinitionRegistry的操作逻辑

RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);

RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
```

### ConfigurationClassPostProcessor

```java
ConfigurationClassPostProcessor处理逻辑的入口
ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry
//核心逻辑，重点看，重要程度5
processConfigBeanDefinitions(registry);

//候选BeanDefinition的解析器
// Parse each @Configuration class
ConfigurationClassParser parser = new ConfigurationClassParser
		
//解析核心流程，重点看，重要程度5
//其实就是把类上面的特殊注解解析出来最终封装成beanDefinition
parser.parse(candidates);

// ConfigurationClassParser#parse方法
//把metadata对象和beanName封装成ConfigurationClass对象
processConfigurationClass(new ConfigurationClass(metadata, beanName), DEFAULT_EXCLUSION_FILTER);

//这个对象理解为跟类或者接口对应，然后把metadata对象包装进去了
SourceClass sourceClass = asSourceClass(configClass, filter);

//核心代码，认真读
// ConfigurationClassParser#doProcessConfigurationClass
sourceClass = doProcessConfigurationClass(configClass, sourceClass, filter);


//判断类上面是否有Component注解
if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
// Recursively process any member (nested) classes first
//递归处理有@Component注解的内部类
processMemberClasses(configClass, sourceClass, filter);
		}

//处理@Import注解 getImports(sourceClass) 获取类上面的@Import注解并封装成SourceClass
// Process any @Import annotations
processImports(configClass, sourceClass, getImports(sourceClass), filter, true);

//@Bean @Import 内部类 @ImportedResource ImportBeanDefinitionRegistrar具体处理逻辑
this.reader.loadBeanDefinitions(configClasses);
```



### 

```
注解加载配置文件
```

### 

```
注解包扫描
```

### factoryBean接口

```java
1.实现factoryBean接口
//FactoryBeanDemo implements FactoryBean
2.可以通过这个接口实现Spring与第三方接口的整合
  /**
     * 只有要用才会调用到
     */
    @Override
    public Object getObject() throws Exception {
        return new Student();
    }

    @Override
    public Class<?> getObjectType() {
        return Student.class;
    }
    
// 只有当调用Student()对象时,才会触发getObject()执行
Student student = (Student)applicationContext.getBean("factoryBeanDemo");
```

在Spring里面如果你想创建一个 对象并且被Spring管理

1.自定义BeanDefinition

2.applicationContext.getBeanFactory().registerSingleton("jack",new jack())

3.自定义factoryBean接口

4.@Bean

## 10.ConfigurationClassPostProcessor类源码1

```java
registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
		registerAnnotationConfigProcessors(registry, null);
	}ConfigurationClassPostProcessor的注册位置
1.AnnotationConfigApplicationContext的构造方法
2.this.reader = new AnnotatedBeanDefinitionReader(this);
3.public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
4.registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {
		registerAnnotationConfigProcessors(registry, null);
}
// 这个地方完成 ConfigurationClassPostProcessor 的注册
5.RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);

ConfigurationClassPostProcessor类用来完成以下注解的解析
1.@ComponentScan
2.@Import
3.ImportSelect 和 ImportBeanDefinitionRegistrar接口
4.@ImportResource
5.@Bean
6.@Configuration

// 实现了ImportSelect 接口的类,需要用@Import导入才能解析 触发selectImports方法的执行逻辑
selectImports 执行的时序在 BeanDefinitionRegistry调用之前

DeferredImportSelector接口
SpringBoot中有用到
DeferredImportSelector接口有两个内部接口
Group 
1.先执行Group的process,后执行Group的selectImports
2.如果没有实现Group接口,会调用Spring内部的Group类
3.调完了内部类Group的方法逻辑,再执行外部类的 selectImport 方法
4.Group 返回的对象是一个Entry,会遍历Entry拿到元数据信息
Group#process 收集处理 配置类
Entry
```

## 11.ConfigurationClassPostProcessor类源码2

````java
1.condition原理
2.@Bean的实现原理
3.ImportBeanDefinitionRegistrar接口的调用
4.@Configuration原理
5.@Component 和 @Configuration区别
6.ImportBeanDefinitionRegistry实战

ConfigurationClassPostProcessor 实现 condition 注解的解析入口

1.ConfigurationClassPostProcessor#processConfigBeanDefinitions
2.ConfigurationClassParser#parser
3.ConfigurationClassParser#parse(AnnotationMetadata metadata, String beanName)
4.ConfigurationClassParser#processConfigurationClass
5.ConditionEvaluator#shouldSkip  // 这个方法会判断 condition的逻辑
// condition 会过滤掉一些类的导入
// 触发这个过滤逻辑  需要类先实现Conditioni接口


ConfigurationClassParser#doProcessConfigurationClass
// 这个方法会解析 @Component,@Import,

// @Import添加的类 实现了ImportSelect接口 本身不会被Spring管理,
// @Import 的类 添加了 @Component 注解,该类也不会被Spring管理

ConfigurationClassParser#processConfigurationClass
this.configurationClasses.put(configClass, configClass);
// 这行代码是ConfigurationClassPostProcessor解析的类交给Spring管理

ConfigurationClass 收集 ConfigurationClassParser 解析 注解的 类

// 处理 Component注解 逻辑
//判断类上面是否有Component注解
		if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
			// Recursively process any member (nested) classes first
			//递归处理有@Component注解的内部类
			processMemberClasses(configClass, sourceClass, filter);
}

//处理PropertySources和 PropertySource注解 核心逻辑
processPropertySource(propertySource);

//处理ComponentScans和ComponentScan注解
		Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
				sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);

//处理@Import注解 getImports(sourceClass) 获取类上面的@Import注解并封装成SourceClass
processImports(configClass, sourceClass, getImports(sourceClass), filter, true);

//处理@ImportResource注解 ，没啥用，加载xml配置文件
AnnotationAttributes importResource =				AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);

//处理@Bean注解，重点
//收集有@bean 注解的方法
Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);

//处理接口里面方法有@Bean注解的，逻辑差不多
processInterfaces(configClass, sourceClass);


//@Bean @Import 内部类 @ImportedResource ImportBeanDefinitionRegistrar具体处理逻辑
this.reader.loadBeanDefinitions(configClasses);


//@Import进来的类，和内部类走这里变成BeanDefinition
		if (configClass.isImported()) {
			registerBeanDefinitionForImportedConfigurationClass(configClass);
		}
//@Bean注解的方法变成BeanDefinition
for (BeanMethod beanMethod : configClass.getBeanMethods()) {
// 处理 方法上面的@Bean注解
loadBeanDefinitionsForBeanMethod(beanMethod);
}






@Component标记的类  里面有@Bean 注解 会违背单例.每次拿到的值不一样
@Configuration标记的类  里面有@Bean 注解 会从缓存中拿.每次拿到的值一样

postProcessBeanDefinitionRegistry 会比 postProcessBeanFactory先执行

postProcessBeanFactory中有一行代码
enhanceConfigurationClasses(beanFactory); // 会生成代理对象

前置知识  cglib的使用案例
cglib的作用 是自己生成一个字节码并加载到JVM虚拟机里面
如何@Configuration标记的类有@Bean注解,动态代理调用的类会用BeanMethodInterceptor,通过BeanFactory取获取Bean的实例对象,这样每次获取到的Bean都是同一个
BeanMethodInterceptor 就是一个切莫

// 这个方法可以调式 Spring中ConfigurationClassPostProcessor解析BeanDefinition的添加过程
	public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
		List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
		//获取所有的beanNames
		String[] candidateNames = registry.getBeanDefinitionNames();
		
		
虽然 @Configuration 本身没有太多参数，但它决定了：
是否是 Full Configuration
是否触发 CGLIB 增强
@Configuration 与 @Component的区别
@Configuration 类被 CGLIB 增强的代码入口

@Component 类不会被CGLIB 增强

CGLIB 增强的代码入口
ConfigurationClassPostProcessor#postProcessBeanFactory
enhanceConfigurationClasses(beanFactory);
ConfigurationClassEnhancer enhancer = new ConfigurationClassEnhancer();
Class<?> enhancedClass = enhancer.enhance(configClass, classLoader);
````



## 12.spring的实战代码案例1及AOP基础 

```java
Advisor 必须要有 PointCut Advice

PiontCut 作用  匹配,拦截
目的 1.为了生成代理
2.用代理对象调用的时候
Advice
承载一些增强的逻辑
ClassFilter == 类是他拦截
MethodMatcher == 方法是它拦截
AbstractAutoProxyCreator#postProcessAfterInitialization
AspectJAwareAdvisorAutoProxyCreator
AnnotationAwareAspectJAutoProxyCreator
InfrastructureAdvisorAutoProxyCreator  这个是AOP增强的处理器

new AspectJAutoProxyBeanDefinitionParser() 这个负责AOP标签的解析


AOP的代码调入口
1.AbstractAutoProxyCreator#postProcessAfterInitialization
2.
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
		if (bean != null) {
			Object cacheKey = getCacheKey(bean.getClass(), beanName);
			if (this.earlyProxyReferences.remove(cacheKey) != bean) {
				return wrapIfNecessary(bean, beanName, cacheKey);
			}
		}
		return bean;
	}
3.	
//创建当前bean的代理，如果这个bean有advice的话，重点看，重要程度5
		// Create proxy if we have advice.
		Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
4.		
AbstractAdvisorAutoProxyCreator#getAdvicesAndAdvisorsForBean
5.
//找到合格的切面，重点看
List<Advisor> advisors = findEligibleAdvisors(beanClass, beanName);
6.
//找到候选的切面,其实就是一个寻找有@Aspectj注解的过程，把工程中所有有这个注解的类封装成Advisor返回
List<Advisor> candidateAdvisors = findCandidateAdvisors();
//判断候选的切面是否作用在当前beanClass上面，就是一个匹配过程。。现在就是一个匹配
List<Advisor> eligibleAdvisors = findAdvisorsThatCanApply(candidateAdvisors, beanClass, beanName);
//针对@Aspect注解切面添加了一个默认的切面 DefaultPointcutAdvisor
extendAdvisors(eligibleAdvisors);
7.
BeanFactoryAdvisorRetrievalHelper#findAdvisorBeans() // 这里找到所有的Advisor(自定义的Advisor)
AnnotationAwareAspectJAutoProxyCreator#findCandidateAdvisors
// 找到注解定义的Advisor(自定义的@Aspect注解的类进行处理)
this.aspectJAdvisorsBuilder.buildAspectJAdvisors()
// 通过这个方法收集 @Aspect注解 的 Advisor
8.
ReflectiveAspectJAdvisorFactory#getAdvisors
// 这个方法负责实际的解析@Aspect注解
//非常重要重点看看，重要程度 5
Advisor advisor = getAdvisor(method, lazySingletonAspectInstanceFactory, 0, aspectName);
9.
//获取pointCut对象，最重要的是从注解中获取表达式
AspectJExpressionPointcut expressionPointcut = getPointcut(
				candidateAdviceMethod, aspectInstanceFactory.getAspectMetadata().getAspectClass());
10.
//创建Advisor切面类，这才是真正的切面类，一个切面类里面肯定要有1、pointCut 2、advice
		//这里pointCut是expressionPointcut， advice 增强方法是 candidateAdviceMethod
		return new InstantiationModelAwarePointcutAdvisorImpl(expressionPointcut, candidateAdviceMethod,
				this, aspectInstanceFactory, declarationOrderInAspect, aspectName);
11.
//这个方法重点看看，创建advice对象
this.instantiatedAdvice = instantiateAdvice(this.declaredPointcut);
12.
//创建Advice对象
Advice advice = this.aspectJAdvisorFactory.getAdvice(this.aspectJAdviceMethod, pointcut,
				this.aspectInstanceFactory, this.declarationOrder, this.aspectName);
13.
ReflectiveAspectJAdvisorFactory#getAdvice
//根据不同的注解类型创建不同的advice类实例
14.
//实现了MethodInterceptor接口
springAdvice = new AspectJAroundAdvice(
						candidateAdviceMethod, expressionPointcut, aspectInstanceFactory);
15.AspectJAroundAdvice 的类结构(Advice创建成功)
public class AspectJAroundAdvice extends AbstractAspectJAdvice implements MethodInterceptor, Serializable 
16.创建代理对象
//把被代理对象bean实例封装到SingletonTargetSource对象中
Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
```



## 13.动态代理和AOP的初见 

## 14.代理的生成和链式调用流程 

```
Advisor实现的过程
1.自己写类实现Advisor接口,并且在类里面定义自己的PointCut和 Advice
List<Advisor> advisors = super.findCandidateAdvisors();
2.解析有@Aspect注解的类,解析有@Aroud@Before创建advice对象,通过注解中配置的value值创建PointCut对象,最终根据创建的advice对象和PointCut对象创建Advice对象
//主要看这里，创建候选的切面  对@Aspect注解的类进行处理
if (this.aspectJAdvisorsBuilder != null) {		advisors.addAll(this.aspectJAdvisorsBuilder.buildAspectJAdvisors());
}
--------------------------------------------		
1.切面的排序

排序的规则
1.ExposeInvocationInterceptor.ADVISOR 排第一位
2.实现了Order接口的排序从小到大 排第二位
3.实现的@Aspect注解的切面,根据代码从上至下排序   排到最后
---------------------------------------------------
2.代理生成
代码入口
1.AbstractAutoProxyCreator#wrapIfNecessary
2.//把被代理对象bean实例封装到SingletonTargetSource对象中
Object proxy = createProxy(
					bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
3.代理工厂
ProxyFactory proxyFactory = new ProxyFactory();
//把AnnotationAwareAspectJAutoProxyCreator中的某些属性copy到proxyFactory中
proxyFactory.copyFrom(this);
this指的是AnnotationAwareAspectJAutoProxyCreator
4.//组装advisor
Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);

默认进行下AnnotationAwareAspectJAutoProxyCreator.interceptorNames是空
可以在public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
设置AnnotationAwareAspectJAutoProxyCreator.interceptorNames的值

}
5.//对Advisor进行创建全局增强的代理对象
this.advisorAdapterRegistry.wrap(allInterceptors.get(i)
6.
JdkDynamicAopProxy.AdvisedSupport  就是代理实例
proxyFactory.addAdvisors(advisors); // 所有的advisor切面
targetSource 就是被代理的实例
//从代理工厂中拿到TargetSource对象，该对象包装了被代理实例bean
TargetSource targetSource = this.advised.targetSource;
7.exposeProxy = true
//如果该属性设置为true，则把代理对象设置到ThreadLocal中
if (this.advised.exposeProxy) {
				// Make invocation available if necessary.
				oldProxy = AopContext.setCurrentProxy(proxy);
				setProxyContext = true;
			}
----------------------
3.链式调用流程
AOP会先初始化一个Advisor  是为了解决参数的传递问题
ExposeInvocationInterceptor 在所有的切面中 进行参数的 链式传递
// 代码入口
//针对@Aspect注解切面添加了一个默认的切面 DefaultPointcutAdvisor
1.AbstractAdvisorAutoProxyCreator#findEligibleAdvisors
2.extendAdvisors(eligibleAdvisors);
3.AspectJProxyUtils.makeAdvisorChainAspectJCapableIfNecessary(candidateAdvisors);
4.advisors.add(0, ExposeInvocationInterceptor.ADVISOR);


@EnableAspectJAutoProxy(proxyTargetClass = false) // 使用JDK代理
@EnableAspectJAutoProxy(proxyTargetClass = true) // 使用CGLIB代理
JDK  代理接口
CGLIB  代理类
代理的最终选择不是由 proxyTargetClass属性决定的,是由proxyTargetClass属性与你的代理类是接口还是类来决定的

每个代理对象都有一个 ProxyFactory 与 JdkDynamicAopProxy ,并不是全局共享的
```



## 15.链式调用过程和AOP周边 

```
1.链式调用过程
AOP切面的调用执行入口
1.
JdkDynamicAopProxy#invoke
2. 这个方法拿到拿到所有的切面Advice
this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
3.封装MethodInvocation类执行ReflectiveMethodInvocation中process的调用,进行链式调用
new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
4.process这个方法执行advice切面的调用.类似于责任链
ReflectiveMethodInvocation#process

InterceptorAndDynamicMethodMatcher 这个MethodInterceptor会处理切面拦截方法的参数,需要特别记忆,只有当pointcut中的isruntime=true

切面都是实现了MethodInterceptor接口的类
AfterReturningAdviceInterceptor
AspectJAfterThrowingAdvice

切面拦截  入参需要单独定义切面的args参数
invokeAdviceMethod(pjp, jpm, null, null); // 反射调用
------------------------------------------
2.代理的提前生成
代码入口
AbstractAutowireCapableBeanFactory#createBean
/*
* TargetSource接口的运用，可以在用改一个类实现该接口，然后在里面定义实例化对象的方式，然后返回
* 也就是说不需要spring帮助我们实例化对象
* 这里可以直接返回实例本身
* 这个代码不用看，实际开发过程中用不到，我会做为一个甜点分享
* */
// 这个方法会触发动态代理,提前生成代理对象
Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
代理对象生成之后直接返回,不会触发doCreateBean方法执行,bean的创建逻辑
当代理对象不为空,需要自己设置自定义的TargetSource
private TargetSourceCreator[] customTargetSourceCreators;

经过这个方法生成的代理对象是一个多例对象,这个方法的实现过程有一点复杂,这个功能可以实现 有切面也不生成代理对象
------------------------------------------
3.scopeProxy
@Scope(value = DefaultListableBeanFactory.SCOPE_PROTOTYPE, proxyMode = ScopedProxyMode.TARGET_CLASS)
这样每次的实例都不一样
autowire-candidate 属性来配置解析@Scope注解
@Scope对象使用@Autowire注入的流程
1.入口
ClassPathBeanDefinitionScanner#doScan
2.判断逻辑
if (checkCandidate(beanName, candidate)) {
					BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
					definitionHolder =
							AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
					beanDefinitions.add(definitionHolder);
					//BeanDefinition注册
					registerBeanDefinition(definitionHolder, this.registry);
				}
3.这个方法会触发代理的生成
AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
4.关键代码
RootBeanDefinition proxyDefinition = new RootBeanDefinition(ScopedProxyFactoryBean.class);
5.ScopedProxyFactoryBean这个类的setBeanFactory方法会添加new DelegatingIntroductionInterceptor(scopedObject)这个切面,用来生成代理对象
```



## 16.spring的实战代码案例2之巩固AOP 

```
1.自定义的Advisor
boolean annotationPresent = method.isAnnotationPresent(EasyCache.class);
Advisor 的match会调用两次
第一次是true
第二次是false
原因是
第一次是生成代理之前,是类中方法对象本身
第二次是用代理对象.方法调用,这时候方法的对象跟之前的不一样

 public boolean isRuntime() {
        return true;
    }
  这个返回值为true时,需要matches(Method method, Class<?> targetClass, Object... args) 这个方法返回true才执行切面的invoke方法
------------------
2.带分布式锁的缓存框架
------------------
3.基于参数做动态路由
```



## 17.spring的事务切面精讲1 

```java
如果多个DML操作是同一个事务,那么他们的连接对象肯定是同一个,
在同一个事务的前提是,必须要把自动提交关闭

事务的入口
@EnableTransactionManagement
TransactionManagementConfigurationSelector#selectImports
return new String[]{AutoProxyRegistrar.class.getName(), ProxyTransactionManagementConfiguration.class.getName()};
//注册aop的入口类InfrastructureAdvisorAutoProxyCreator					AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);
//把属性设置到入口类中，最终会copy到proxyFactory中						AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);

ProxyTransactionManagementConfiguration类中的
BeanFactoryTransactionAttributeSourceAdvisor就是事务切面,切面有两个元素,一个是advice,一个是PointCut

TransactionAttributeSource  这个是事务注解解析之后封装的对象
//设置切面的advice
advisor.setAdvice(transactionInterceptor);
BeanFactoryTransactionAttributeSourceAdvisor 对象里面有定义PointCut方法

// 方法match
tas.getTransactionAttribute(method, targetClass)
AbstractFallbackTransactionAttributeSource#getTransactionAttribute
AbstractFallbackTransactionAttributeSource#computeTransactionAttribute(method, targetClass);

//获取原始方法
Method specificMethod = AopUtils.getMostSpecificMethod(method, targetClass);
//method方法会被调用两次,第一次是原始对象,第二次是代理对象,通过AopUtils.getMostSpecificMethod拿到代理对象的方法

// 事务的解析收集
SpringTransactionAnnotationParser#parseTransactionAnnotation // 这个方法解析@Transactional注解
//解析Transactional注解中的属性，并封装成对象
return parseTransactionAnnotation(attributes);

// 事务的执行
// 事务对象的MethodInteceptor
TransactionInterceptor#invoke
invokeWithinTransaction(invocation.getMethod(), targetClass, invocation::proceed)
TransactionAspectSupport#invokeWithinTransaction // 这个方法是事务链执行的核心流程
 
// 一般这样定义事务管理器
@Bean
public PlatformTransactionManager annotationDrivenTransactionManager(DataSource dataSource) {
        DataSourceTransactionManager dtm = new DataSourceTransactionManager();
        dtm.setDataSource(dataSource);
        return dtm;
}


// 创建事务的逻辑
TransactionAspectSupport#createTransactionIfNecessary
//开启事务，这里重点看
status = tm.getTransaction(txAttr);
AbstractPlatformTransactionManager#getTransaction // 这个地方的hi开启事务的入口
//这里重点看，.DataSourceTransactionObject拿到对象
Object transaction = doGetTransaction();

DataSourceTransactionManager#doGetTransaction

 DataSourceTransactionObject  //这个是事务对象
 常用的属性
private ConnectionHolder connectionHolder;// 连接对象的包装对象
private Integer previousIsolationLevel;
private boolean readOnly = false;
private boolean savepointAllowed = false;
//这个代码是从ThreadLocal中获取到连接对象
ConnectionHolder conHolder =
				(ConnectionHolder) TransactionSynchronizationManager.getResource(obtainDataSource());

事务肯定是跟连接对象挂钩的
事务的传播属性 是为了控制  事务的流传

TransactionAspectSupport#invokeWithinTransaction 这个是事务执行流程的起点
final TransactionAttribute txAttr = (tas != null ? tas.getTransactionAttribute(method, targetClass) : null);//这行代码对事务的属性进行处理
//火炬传递
retVal = invocation.proceedWithInvocation();
```



## 18.spring的事务传播属性精讲1

```java
1.事务的执行流程
@Transactional
public void transation(ConsultConfigArea area, ZgGoods zgGoods) throws JamesException {
        areaService.addArea(area);
        goodsService.addGoods(zgGoods);
    }
    
transation方法执行的伪代码

1.transation方法执行开始
try{
createTransactionIfNecessary // 开启事务
areaService.addArea(area);
addArea方法执行开始
try{
createTransactionIfNecessary // 开启事务
int i = commonMapper.addArea(area);
return i;
}catch(E e){
//事务回滚
completeTransactionAfterThrowing(txInfo, ex);
}
//事务提交
commitTransactionAfterReturning(txInfo);//会根据状态值判断是否提交事务

goodsService.addGoods(zgGoods);
addGoods方法执行开始
try{
createTransactionIfNecessary // 开启事务
int i = commonMapper.addGood(zgGoods);
}catch(E e){
//事务回滚
completeTransactionAfterThrowing(txInfo, ex);
}
//事务提交
commitTransactionAfterReturning(txInfo);//会根据状态值判断是否提交事务
}catch(E e){
//事务回滚
completeTransactionAfterThrowing(txInfo, ex);
}
//事务提交
commitTransactionAfterReturning(txInfo); //会根据状态值判断是否提交事务


txInfo.getTransactionStatus() 这个属性会决定事务是否提交

SuspendedResourcesHolder suspendedResources = suspend(null);
suspendedResources = doSuspend(transaction);
SuspendedResourcesHolder // 这个对象封装保存挂起的连接资源对象
    
 // 这个是传播属性PROPAGATION_REQUIRES_NEW的逻辑
 if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW) {
			if (debugEnabled) {
				logger.debug("Suspending current transaction, creating new transaction with name [" +
						definition.getName() + "]");
			}
			//挂起
			SuspendedResourcesHolder suspendedResources = suspend(transaction);
			try {
				return startTransaction(definition, transaction, debugEnabled, suspendedResources);
			}
			catch (RuntimeException | Error beginEx) {
				resumeAfterBeginException(transaction, suspendedResources, beginEx);
				throw beginEx;
			}
		}
		
//创建一个新的事务状态，注意这里的 newTransaction 属性为 true了
// 将以前的连接对象挂起保存在SuspendedResourcesHolder中
AbstractPlatformTransactionManager#commit 执行成功之后会恢复以前挂起的连接对象
//执行事务提交
processCommit(defStatus);
finally {
//处理挂起事务，在这里恢复绑定关系
cleanupAfterCompletion(status);
}
doCleanupAfterCompletion
```



## 19.spring的事务传播属性精讲2

```
1.NESTED 传播属性
1.相对于PROPAGATION_REQUIRES,只是多创建了一个回滚点

回滚的异常不匹配,会做commit
-----------------
2.编程式事务

编程式事务,没有传播属性的概念,事务控制的粒度更细

TransactionSynchronizationManager.registerSynchronization(new DoOnAfterCommit()); // 这个方法可以注册一个钩子,等事务提交之后执行回调

ConnectionHolder resource = (ConnectionHolder) TransactionSynchronizationManager.getResource(dataSource);
// 这个方法可以拿到当前方法的事务连接对象
-----------------
3.缓存切面

缓存 与 异步 的实现 都是基于切面的AOP代理实现
缓存切面 需要 定义好Advice 与 PointCut,Advice中执行缓存的逻辑

同理 异步注解 也 同样是 Advice 与 PointCut, Advice中执行异步的逻辑
```



## 20.缓存切面和异步切面 

## 21.缓存框架和springmvc零配置原理 

## 22.springmvc中请求的调用流程 

## 23.springmvc中请求的调用流程 

## 24.JSON参数解析和视图响应 

## 25.异常处理、拦截器和跨域问题 

## 26.springmvc实战、spring总流程归档 

## 27.spring总流程归档 

## 28.BeanPostProcessor案例讲解

### ✔ 结论：`BeanPostProcessor` 注册发生在 **Spring 容器刷新（refresh）流程中非常早的阶段**

具体来说，调用点在：

### **① refresh() → invokeBeanFactoryPostProcessors() 之后**

### **② refresh() → registerBeanPostProcessors(beanFactory) 中**

也就是说：

- Spring **扫描到所有 BeanPostProcessor 类型的 bean 定义**
- 创建它们（提前创建！）
- 然后在该步骤中执行：
   👉 **beanFactory.addBeanPostProcessor()**

------

### 📌 所以 `BeanPostProcessor` 的注册发生在 Refresh 流程中的这个位置：

```
AbstractApplicationContext.refresh()
    ├── prepareRefresh()
    ├── obtainFreshBeanFactory()
    ├── prepareBeanFactory()
    ├── postProcessBeanFactory()
    ├── **invokeBeanFactoryPostProcessors()**
    ├── **registerBeanPostProcessors(beanFactory)**  ← 就在这里调用 addBeanPostProcessor
    ├── initMessageSource()
    ├── initApplicationEventMulticaster()
    ├── onRefresh()
    ├── registerListeners()
    ├── finishBeanFactoryInitialization()
    └── finishRefresh()
```

------

### 🧩 更细节：`registerBeanPostProcessors()` 做了什么？

源码（简化）：

```
public static void registerBeanPostProcessors(
        ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {

    // 1. 找出所有 BeanPostProcessor 的 beanName
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

    // 2. 按优先级排序（PriorityOrdered, Ordered, 普通）
    sortPostProcessors(postProcessors);

    // 3. 逐个创建并注册：
    beanFactory.addBeanPostProcessor(pp);
}
```

### 即：

- Spring 先扫描所有实现了 BeanPostProcessor 的 bean
- 立刻创建它们（提前初始化，不走普通 bean 的延迟加载）
- 通过 `beanFactory.addBeanPostProcessor()` 注册到 BeanFactory

注册完成后，后续所有 bean 在创建前后都会被它们处理。

------

### ✔ 补充：`BeanPostProcessor` 创建时间早于普通 bean

因为这些处理器要参与到后续 bean 的生命周期回调，所以 Spring 必须提前创建它们。

因此：

- **BeanPostProcessor 自己不会被它自己处理**
- **也不会被其他默认 BPP（比如 AutowiredAnnotationBeanPostProcessor）再处理**

------

### 📘 举例

你自己写了一个：

```
@Component
public class MyBPP implements BeanPostProcessor { ... }
```

Spring 会在 `registerBeanPostProcessors()` 执行时调用：

```
beanFactory.addBeanPostProcessor(new MyBPP());
```

而你的业务 bean 在后面 `finishBeanFactoryInitialization()` 阶段创建时，才会被你的 BPP 拦截：

- `postProcessBeforeInitialization`
- `postProcessAfterInitialization`

------

### ✔ 总结（最好记住这三点）

#### 1) BeanPostProcessor 是在 `refresh()` 的早期注册的

位置是：

```
registerBeanPostProcessors(beanFactory)
```

#### 2) 注册时调用的就是 `beanFactory.addBeanPostProcessor()`

即 Spring 自动管理，无需手动调用。

#### 3) 所有 BeanPostProcessor 会在普通 bean 创建之前准备好

确保它们能参与整个 bean 的生命周期回调。