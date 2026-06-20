# Spring 整理稿

> 原笔记版本基准:**Spring Framework 5.2.8**
> 涵盖 BeanDefinition 解析、refresh 生命周期、IoC/DI、AOP、事务

---

## 一、知识点总览

```
Spring 5.2.8
├─ refresh() 生命周期(13 步)
├─ BeanDefinition 解析
│   ├─ XML:默认标签(bean/import/alias)+ 自定义标签(NamespaceHandler)
│   ├─ Annotation:@ComponentScan + ConfigurationClassPostProcessor
│   └─ Programmatic:DefaultListableBeanFactory#registerBeanDefinition
├─ Bean 实例化
│   ├─ 构造函数推断 / @Autowired 构造器 / FactoryBean / factory-method
│   ├─ 三级缓存与循环依赖
│   └─ @Autowired / @Value / @Resource / @PostConstruct 解析
├─ AOP
│   ├─ 入口:AbstractAutoProxyCreator#postProcessAfterInitialization
│   ├─ Advisor 收集(@Aspect → ReflectiveAspectJAdvisorFactory)
│   ├─ ProxyFactory → JDK/CGLIB
│   └─ ReflectiveMethodInvocation 责任链
└─ 事务
    ├─ @EnableTransactionManagement → AutoProxyRegistrar
    ├─ BeanFactoryTransactionAttributeSourceAdvisor
    ├─ TransactionInterceptor#invoke
    └─ 传播属性 / 挂起 / NESTED savepoint
```

---

## 二、refresh() 完整生命周期(多文件调用流程)

```
AbstractApplicationContext#refresh()
  │
  ├─ ①  prepareRefresh()                       初始化属性源、校验必要属性
  │
  ├─ ②  obtainFreshBeanFactory()
  │       └─> refreshBeanFactory()              子类实现:创建 DefaultListableBeanFactory
  │             └─> loadBeanDefinitions(bf)     XML / 注解扫描入口
  │                   ↓
  │              [XML 路径]
  │              XmlBeanDefinitionReader.loadBeanDefinitions(Resource)
  │                 └─ doLoadBeanDefinitions(InputSource, Resource)
  │                     └─ registerBeanDefinitions(Document, Resource)
  │                         └─ DefaultBeanDefinitionDocumentReader
  │                              .doRegisterBeanDefinitions(Element root)
  │                              └─ parseBeanDefinitions(root, delegate)
  │                                  ├─ 默认命名空间 → parseDefaultElement
  │                                  │     ├─ import / alias / bean / nested
  │                                  │     └─ processBeanDefinition →
  │                                  │           parseBeanDefinitionElement →
  │                                  │           注册到 BeanDefinitionRegistry
  │                                  └─ 自定义命名空间 → delegate.parseCustomElement
  │                                        └─ NamespaceHandlerResolver.resolve(uri)
  │                                              ↳ 加载所有 jar 的
  │                                                META-INF/spring.handlers
  │                                              ↳ 命中 NamespaceHandler.init()
  │                                                如 ContextNamespaceHandler 注册了
  │                                                ComponentScanBeanDefinitionParser
  │                                        └─ handler.parse(ele, parserCtx)
  │                                              └─ doScan(basePackages)
  │                                                    └─ findCandidateComponents
  │                                                          └─ ASM 读取 .class → BD
  │
  ├─ ③  prepareBeanFactory(bf)                 设置 ClassLoader / SPEL / Aware 后处理器
  ├─ ④  postProcessBeanFactory(bf)              子类钩子(SpringBoot 在这里注册 web bf 后处理器)
  │
  ├─ ⑤  invokeBeanFactoryPostProcessors(bf)    🌟核心
  │       │
  │       ├─ 1. 先调 BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry
  │       │     ↳ ConfigurationClassPostProcessor 在此触发:
  │       │       processConfigBeanDefinitions(registry)
  │       │         └─ ConfigurationClassParser.parse()
  │       │             └─ doProcessConfigurationClass()
  │       │                 ├─ 内部类(@Component)递归
  │       │                 ├─ @PropertySource
  │       │                 ├─ @ComponentScan → ClassPathBeanDefinitionScanner.doScan
  │       │                 ├─ @Import         → processImports
  │       │                 │     ├─ ImportSelector  → selectImports
  │       │                 │     ├─ ImportBeanDefinitionRegistrar → registerBeanDefinitions
  │       │                 │     └─ 普通类       → 当作 @Configuration 递归
  │       │                 ├─ @ImportResource
  │       │                 └─ @Bean 方法收集
  │       │             └─ reader.loadBeanDefinitions(configClasses)
  │       │                 ├─ @Import 进来的类注册成 BD
  │       │                 └─ @Bean 方法注册成 BD
  │       │
  │       └─ 2. 再调 BeanFactoryPostProcessor#postProcessBeanFactory
  │             ↳ ConfigurationClassPostProcessor 在此:
  │               enhanceConfigurationClasses(bf)
  │                 └─ CGLIB 增强所有 Full Configuration 类
  │                    (让 @Bean 方法间相互调用走 BeanFactory,保证单例)
  │
  ├─ ⑥  registerBeanPostProcessors(bf)
  │       └─ 找出所有 BeanPostProcessor 类型的 BD
  │          按 PriorityOrdered → Ordered → 普通 排序
  │          bf.addBeanPostProcessor(bpp)
  │
  ├─ ⑦  initMessageSource()                    国际化
  ├─ ⑧  initApplicationEventMulticaster()      事件广播器
  ├─ ⑨  onRefresh()                            子类钩子(SpringBoot 在这里启动内嵌 Tomcat)
  ├─ ⑩  registerListeners()                    把 ApplicationListener 加到广播器
  │
  ├─ ⑪  finishBeanFactoryInitialization(bf)   🌟最关键
  │       └─ bf.preInstantiateSingletons()
  │            for each beanName:
  │              getBean(beanName)
  │                └─ doGetBean →
  │                   ├─ getSingleton(name)             三级缓存
  │                   └─ createBean → doCreateBean
  │                       ├─ createBeanInstance         反射 / @Autowired 构造器
  │                       ├─ applyMergedBeanDefinitionPostProcessors
  │                       │     ↳ Autowired/Common 注解收集
  │                       ├─ addSingletonFactory(三级缓存写入)
  │                       ├─ populateBean              DI 入口
  │                       │     ↳ AutowiredAnnotationBeanPostProcessor
  │                       │        .postProcessProperties → resolveDependency → getBean
  │                       └─ initializeBean
  │                            ├─ invokeAwareMethods
  │                            ├─ applyBeanPostProcessorsBeforeInitialization
  │                            │     ↳ @PostConstruct (InitDestroyAnnotationBeanPostProcessor)
  │                            ├─ invokeInitMethods(afterPropertiesSet / init-method)
  │                            └─ applyBeanPostProcessorsAfterInitialization
  │                                 ↳ 🌟 AOP 入口:AbstractAutoProxyCreator
  │
  └─ ⑫  finishRefresh()                        发布 ContextRefreshedEvent / lifecycleProcessor.onRefresh
```

---

## 三、自定义标签解析(spring.handlers + spring.schemas)

```
配置:<context:component-scan base-package="x"/>

加载:
  ① BeanDefinitionParserDelegate.parseCustomElement(ele)
  ② getNamespaceURI(ele) → "http://www.springframework.org/schema/context"
  ③ DefaultNamespaceHandlerResolver.resolve(uri):
       └─ getHandlerMappings():
           Properties = PropertiesLoaderUtils.loadAllProperties(
             "META-INF/spring.handlers",  classLoader)
           // 合并所有 jar 包的同名文件
       └─ handlerMappings.get(uri) → "org.springframework.context.config.ContextNamespaceHandler"
  ④ 反射实例化,调用 init():
       register("property-placeholder", new PropertyPlaceholderBDP())
       register("component-scan",       new ComponentScanBDP())
       …
  ⑤ handler.parse(ele, parserCtx) → 找到对应的 BDP → 解析标签

应用:Dubbo 早期就是用同样手法实现 <dubbo:service /> 等自定义标签
```

---

## 四、三级缓存与循环依赖

```
DefaultSingletonBeanRegistry 的三级结构:
   singletonObjects        (一级) Map<name, Object>     完成品
   earlySingletonObjects   (二级) Map<name, Object>     半成品(已实例化,DI 未完成)
   singletonFactories      (三级) Map<name, ObjectFactory>  Bean 工厂(可生成早期引用,可能是代理)

getSingleton(name, allowEarlyReference=true) 的查找顺序:
   一级 → 找到则返回
        ↓ 未找到 且 isSingletonCurrentlyInCreation(name)
   二级 → 找到则返回
        ↓ 未找到 且 allowEarlyReference
   三级 → 调用 singletonFactory.getObject()
          升级到二级,移除三级
          返回早期引用
```

### 循环依赖案例:A 依赖 B,B 依赖 A

```
① getBean(A)
   └─ 一级无 → createBean(A)
       └─ createBeanInstance(A)               // A 已分配内存
       └─ addSingletonFactory(A, factory→A)   // 三级缓存写入 A
       └─ populateBean(A)                     // 注入 b 字段 → getBean(B)
            ↓
            ② getBean(B)
               └─ 一级无 → createBean(B)
                   └─ createBeanInstance(B)
                   └─ addSingletonFactory(B, factory→B)
                   └─ populateBean(B)         // 注入 a 字段 → getBean(A)
                        ↓
                        ③ getBean(A)
                           └─ 一级无 → 但 A in singletonsCurrentlyInCreation
                           └─ 二级无 → 三级:factory.getObject() → 返回 A 的早期引用
                           └─ A 升二级,return earlyA
                        ← B 拿到 A 早期引用,完成 DI
                   └─ initializeBean(B)
                   └─ B 放入一级缓存
            ← A 拿到完整 B,完成 DI
       └─ initializeBean(A)
       └─ A 放入一级缓存
```

### 为什么需要三级而不是二级?

如果只有二级,把"早期 bean"直接放进去就行了——但**当 A 需要被 AOP 代理时**,我们希望注入到 B 中的 a 字段是**A 的代理对象**,而不是原始 A。三级缓存里存的是一个 `ObjectFactory`,只有当真正发生循环依赖时才调用它,通过 `SmartInstantiationAwareBeanPostProcessor#getEarlyBeanReference` **提前生成代理**。如果没有循环依赖,这个 factory 永不调用,代理仍按正常流程在 `initializeBean → applyBeanPostProcessorsAfterInitialization` 生成。

### 哪些情况不能解决?

- **构造器循环依赖**:`createBeanInstance` 阶段 A 还没机会把工厂放进三级缓存,B 反过来 getBean(A) 时一级二级三级都没有 → `BeanCurrentlyInCreationException`。
- **多例(prototype)循环依赖**:Spring 不缓存 prototype,直接报错。
- **@Async 修饰的方法所在 bean 循环依赖**:AsyncAnnotationBeanPostProcessor 不属于 SmartInstantiationAwareBeanPostProcessor,无法提前暴露代理 → 5.x 在 `populateBean` 注入时会发现引用与最终代理不一致并报错(需要用 `@Lazy` 规避)。

---

## 五、Bean 实例化阶段的构造器选择

```
createBeanInstance 内部判断顺序:
  ① mbd.getFactoryMethodName() != null      → instantiateUsingFactoryMethod
       场景:<bean factory-method="…"/>  或  @Bean 方法
  ② determineConstructorsFromBeanPostProcessors(beanClass)
       ↳ AutowiredAnnotationBeanPostProcessor 扫描 @Autowired 构造器
       若返回非空 → autowireConstructor(选最合适的构造器)
  ③ AUTOWIRE_CONSTRUCTOR 模式 / 构造参数 / args 不空 → autowireConstructor
  ④ 否则                                          → instantiateBean(默认无参)
```

**注意:** `@Autowired` 修饰**多个构造器**时,必须把它们都标 `required=false`,否则启动报错(Spring 不知道该选哪个)。

---

## 六、AOP 多文件调用流程(@Aspect 解析 → 代理 → 链式调用)

### 6.1 收集 Advisor

```
触发点:Bean 创建完毕(initializeBean 最后)
  AbstractAutoProxyCreator#postProcessAfterInitialization
    └─ wrapIfNecessary(bean, name, cacheKey)
         └─ getAdvicesAndAdvisorsForBean(beanClass, name, null)
              └─ AbstractAdvisorAutoProxyCreator#findEligibleAdvisors
                  ├─ findCandidateAdvisors()
                  │    ├─ BeanFactoryAdvisorRetrievalHelper.findAdvisorBeans()
                  │    │    // 容器内显式声明的 Advisor 类型 bean
                  │    └─ AnnotationAwareAspectJAutoProxyCreator extends 中:
                  │         aspectJAdvisorsBuilder.buildAspectJAdvisors()
                  │         // 扫描所有 @Aspect 注解的 bean
                  │         └─ ReflectiveAspectJAdvisorFactory.getAdvisors(aspect)
                  │             for each method with @Before/@Around/@After…:
                  │               getAdvisor(method, …)
                  │                 ├─ getPointcut(method) → AspectJExpressionPointcut
                  │                 └─ new InstantiationModelAwarePointcutAdvisorImpl(…)
                  │                      .instantiateAdvice():
                  │                          根据注解类型 new 不同 Advice
                  │                          (AspectJAroundAdvice / AspectJAfterAdvice / …)
                  ├─ findAdvisorsThatCanApply(候选, beanClass, name)
                  │    // 用 PointCut 匹配
                  └─ extendAdvisors(eligible)
                       // 头部插入 ExposeInvocationInterceptor.ADVISOR
                       // 为了在链式调用中通过 ThreadLocal 共享 MethodInvocation
```

### 6.2 创建代理

```
createProxy(beanClass, name, specificInterceptors,
            new SingletonTargetSource(bean))
  └─ ProxyFactory proxyFactory = new ProxyFactory()
     proxyFactory.copyFrom(this)               拷贝 AnnotationAware… 的属性
     proxyFactory.addAdvisors(buildAdvisors(...))
     proxyFactory.setTargetSource(targetSource)
     proxyFactory.getProxy(classLoader)
        └─ 选择 AopProxy:
             - 接口 + !proxyTargetClass        → JdkDynamicAopProxy
             - 类   或  proxyTargetClass=true → ObjenesisCglibAopProxy
```

**⚠ 原笔记纠错(关键点 1):** 原文写"代理的最终选择不是由 proxyTargetClass 属性决定的"。
更准确的规则:

```
proxyTargetClass=false (默认):
   目标实现了接口?  → JDK 代理
   目标只是普通类?  → CGLIB 代理(自动 fallback)
proxyTargetClass=true:
   无论目标是接口还是类,一律 CGLIB(对接口实现类也强制走子类代理)
```

也就是说 `proxyTargetClass=true` **会**决定一定走 CGLIB;但 `=false` 时**最终是否走 JDK 取决于目标是否实现了接口**。

### 6.3 链式调用

```
代理方法被调用:
  JdkDynamicAopProxy#invoke(proxy, method, args)
   ├─ this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass)
   │     // 把 Advisor 适配成 List<MethodInterceptor>(经 AdvisorAdapter)
   │     // 头部是 ExposeInvocationInterceptor
   ├─ new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain)
   └─ invocation.proceed()
        currentInterceptorIndex++
        ├─ 还有 interceptor → chain.get(i).invoke(this)
        │     例:AspectJAroundAdvice.invoke
        │       └─ 用户切面方法(ProceedingJoinPoint.proceed) → 再回到 invocation.proceed()
        │     例:MethodBeforeAdviceInterceptor.invoke
        │       └─ before(...) → 再调 mi.proceed()
        └─ 已是最后一个 → invokeJoinpoint() (反射调用目标方法)

排序规则(Order 内的 Advisor 内部排序):
  1. ExposeInvocationInterceptor.ADVISOR 始终第一
  2. 实现 Ordered 的按 order 升序
  3. @Aspect 中同类多个 advice 按代码声明顺序
```

**⚠ 原笔记纠错(关键点 2):** 原文写"`InfrastructureAdvisorAutoProxyCreator` 这个是 AOP 增强的处理器"。
更准确的说:

- `InfrastructureAdvisorAutoProxyCreator`:**只处理 Infrastructure 类型的 Advisor**(主要用于事务),不会扫描 `@Aspect`。
- `AspectJAwareAdvisorAutoProxyCreator`:支持 XML `<aop:config>` 风格。
- `AnnotationAwareAspectJAutoProxyCreator`:**才是 `@EnableAspectJAutoProxy` 注册的那个**,既扫描 `@Aspect` 也合并 Infrastructure Advisor。

三个类是继承关系:`Annotation… extends Aspect… extends Infrastructure…`。

---

## 七、事务多文件调用流程

```
配置:@EnableTransactionManagement
       │
       │ @Import(TransactionManagementConfigurationSelector.class)
       ▼
TransactionManagementConfigurationSelector#selectImports
       └─ 返回 [AutoProxyRegistrar, ProxyTransactionManagementConfiguration]

AutoProxyRegistrar.registerBeanDefinitions
   └─ AopConfigUtils.registerAutoProxyCreatorIfNecessary
         └─ 注册 InfrastructureAdvisorAutoProxyCreator(若已存在更高级的就升级)

ProxyTransactionManagementConfiguration
   └─ @Bean BeanFactoryTransactionAttributeSourceAdvisor advisor
        ├─ setAdvice(transactionInterceptor)
        └─ setTransactionAttributeSource(AnnotationTransactionAttributeSource)
              └─ SpringTransactionAnnotationParser.parseTransactionAnnotation(@Transactional)

运行期一次带事务的方法调用:
  代理对象.method()
   ├─ TransactionInterceptor#invoke(MethodInvocation)
   │   └─ invokeWithinTransaction(method, targetClass, () -> mi.proceed())
   │       └─ TransactionAspectSupport#invokeWithinTransaction
   │            ├─ TransactionAttribute txAttr = tas.getTransactionAttribute(method, targetClass)
   │            ├─ TransactionInfo info = createTransactionIfNecessary(tm, txAttr, joinpointId)
   │            │    └─ tm.getTransaction(txAttr)              开启事务 / 复用 / 挂起
   │            │       AbstractPlatformTransactionManager#getTransaction
   │            │        ├─ doGetTransaction()                获取 DataSourceTransactionObject
   │            │        │  └─ TransactionSynchronizationManager.getResource(dataSource)
   │            │        │     // ThreadLocal 拿 ConnectionHolder
   │            │        ├─ if isExistingTransaction → handleExistingTransaction(传播属性逻辑)
   │            │        │     ├─ REQUIRES_NEW → suspend(当前) + startTransaction(新)
   │            │        │     ├─ NESTED       → 同事务但建 Savepoint
   │            │        │     ├─ NOT_SUPPORTED → suspend(当前),无事务执行
   │            │        │     ├─ NEVER        → 抛异常
   │            │        │     └─ REQUIRED/SUPPORTS/MANDATORY → 复用当前事务
   │            │        └─ else if PROPAGATION_REQUIRED/...  → startTransaction
   │            ├─ try { retVal = invocation.proceedWithInvocation(); }
   │            │   catch (Throwable ex) {
   │            │     completeTransactionAfterThrowing(info, ex)    // 回滚 or 提交
   │            │   }
   │            ├─ finally { cleanupTransactionInfo(info) }
   │            └─ commitTransactionAfterReturning(info)       // 提交 / 恢复挂起的事务
```

### 关键事务对象

```
DataSourceTransactionObject
  ├─ ConnectionHolder connectionHolder   // 包装 java.sql.Connection
  ├─ Integer previousIsolationLevel
  ├─ boolean readOnly
  └─ boolean savepointAllowed             // NESTED 用到

TransactionSynchronizationManager(全是 ThreadLocal)
  ├─ resources           Map<DataSource, ConnectionHolder>
  ├─ synchronizations    List<TransactionSynchronization>
  ├─ currentTransactionName
  ├─ currentTransactionReadOnly
  └─ actualTransactionActive
```

### 挂起与恢复(REQUIRES_NEW)

```
父事务运行中  → 内层方法 @Transactional(REQUIRES_NEW)
   suspend(transaction)
     ├─ 触发所有 TransactionSynchronization.suspend()
     ├─ 把当前 ConnectionHolder 从 ThreadLocal 解绑
     └─ 打包成 SuspendedResourcesHolder 暂存到新事务对象上
   startTransaction(新事务)
     ├─ dataSource.getConnection() 新连接
     └─ 绑定到 ThreadLocal
   ... 内层执行 ...
   commit/rollback
   cleanupAfterCompletion
     └─ resume(挂起的资源)
         └─ 重新绑定父事务的 ConnectionHolder
```

---

## 八、@Component vs @Configuration 区别(原文说法纠错)

**⚠ 原笔记纠错(关键点 3):** 原文写:
> "@Component 标记的类里面有 @Bean 注解会违背单例,每次拿到的值不一样"

更精确的描述:

| 维度 | @Configuration(Full) | @Component(Lite) |
| --- | --- | --- |
| ConfigurationClassPostProcessor 是否给类做 CGLIB 增强 | **是** | 否 |
| 容器内 `applicationContext.getBean("xxBean")` 取值 | 单例 | 单例(也走 BeanFactory) |
| **配置类内方法间互调** `methodA()` 直接调 `@Bean methodB()` | **走 CGLIB 代理 → BeanFactory.getBean,单例** | **直接 new,每次新实例** |
| 用法定位 | 装配中心 | 普通组件 |

所以更准的说法是:**只有"在 `@Component` 配置类内部、`@Bean` 方法之间互相调用"时**才会出现"违背单例"。从容器外 `getBean` 取仍然是单例。

CGLIB 增强代码入口:`ConfigurationClassPostProcessor#postProcessBeanFactory` → `enhanceConfigurationClasses(bf)` → `ConfigurationClassEnhancer.enhance(...)`,拦截器是 `BeanMethodInterceptor`(原笔记把"切面"写成了"切莫",属于错别字)。

---

## 九、其他纠错点

### 9.1 ComponentScan 默认扫描的注解

原文列了 `@Component / @Service / @Controller / @Repository / @Configuration`。
**实际上**:`ClassPathScanningCandidateComponentProvider` 默认只注册了一个 `AnnotationTypeFilter(@Component.class)`,而 `@Service / @Controller / @Repository / @Configuration` 这些注解上**自己标了 `@Component`**,所以是间接命中。理解为"凡是元注解里含 @Component 的都被扫到"更准。

### 9.2 "AbstractAdvisingBeanPostProcessor 是 AOP 切面处理器"

原文这句不准确。`AbstractAdvisingBeanPostProcessor` 是一个**简单版本**的 advisor-based BPP,被 `AsyncAnnotationBeanPostProcessor`、`MethodValidationPostProcessor` 等使用——**只能处理一个固定 Advisor**。
通用 AOP 入口是 `AbstractAutoProxyCreator`,它能动态匹配 Advisor 列表。

### 9.3 "JDK 代理接口,CGLIB 代理类" 不完全准确

JDK 代理需要目标类**实现接口**才能用,但 JDK 代理生成的不是接口的代理"接口",而是**接口的实现类**(`$Proxy0 implements 接口`)。所以更准的说法是:**JDK 代理基于接口,CGLIB 代理基于子类继承**。

### 9.4 "BeanPostProcessor 自己不会被它自己处理"(章节 28)

原文这一段总体正确,但少一句关键:**BeanPostProcessor 是 bean,所以它自己也是被 `getBean` 创建的**,创建过程中会经过 `populateBean` / DI / `initializeBean` 等,只是**注册时机**早于业务 bean,所以业务 bean 创建时它已就绪。"BeanPostProcessor 自己不会被它自己处理"是指 `applyBeanPostProcessorsBefore/AfterInitialization` 的循环中会**跳过那些尚未注册到 list 中的 BPP**(因为还没注册嘛),不是从设计上"刻意排除"。

---

## 十、关键纠错清单(汇总)

| # | 原笔记表述 | 正确表述 |
| --- | --- | --- |
| 1 | `proxyTargetClass` 不决定走 JDK 还是 CGLIB | `=true` 时强制 CGLIB;`=false` 时:目标实现接口则 JDK,否则 fallback 到 CGLIB |
| 2 | `InfrastructureAdvisorAutoProxyCreator` 是 AOP 增强的处理器 | 它只处理事务等 Infrastructure 类型;`@EnableAspectJAutoProxy` 用的是 `AnnotationAwareAspectJAutoProxyCreator` |
| 3 | "@Component 类里 @Bean 会违背单例" | 只是**配置类内部方法互调**时不走代理;从容器外 getBean 仍然单例 |
| 4 | "BeanMethodInterceptor 是一个切莫" | 错别字,应为"切面"(MethodInterceptor) |
| 5 | "AbstractAdvisingBeanPostProcessor AOP 的切面处理器" | 它只处理单 Advisor,通用 AOP 入口是 `AbstractAutoProxyCreator` |
| 6 | "JDK 代理接口,CGLIB 代理类" | JDK 基于接口生成实现类代理;CGLIB 基于子类继承目标类生成代理 |
| 7 | ComponentScan 默认扫描 `@Component/@Service/@Controller/@Repository/@Configuration` | 实际只扫 `@Component`,其他注解本身含 `@Component` 元注解,所以间接命中 |
| 8 | "ConfigurationClassPostProcessor 的优先级最低" | 实际上它实现了 `PriorityOrdered`,优先级**最高**;原文表述与实际相反 |
| 9 | 三级缓存只是为"循环依赖"存在 | 更准:为"循环依赖 + AOP 提前生成代理"两件事共同存在;无 AOP 时二级理论上够用 |

---
