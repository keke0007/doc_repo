# Spring 源码知识点梳理（基于 Spring 5.2.8）

> 本文档基于 `Spring.md` 笔记内容整理，按主题归类核心知识点，并对涉及**多文件/多类协作调用**的流程使用 ASCII 流程图说明。

---

## 目录

1. [BeanDefinition 与 XML 标签解析](#一beandefinition-与-xml-标签解析)
2. [默认标签与自定义标签解析](#二默认标签与自定义标签解析)
3. [component-scan 标签解析](#三component-scan-标签解析)
4. [BeanPostProcessor 与 Bean 实例化总流程](#四beanpostprocessor-与-bean-实例化总流程)
5. [Bean 实例化与注解收集](#五bean-实例化与注解收集)
6. [@Autowired / @PostConstruct / @PreDestroy](#六autowired--postconstruct--predestroy)
7. [循环依赖与三级缓存](#七循环依赖与三级缓存)
8. [配置文件解析与 Environment](#八配置文件解析与-environment)
9. [自定义 scope 与 FactoryBean](#九自定义-scope-与-factorybean)
10. [ConfigurationClassPostProcessor](#十configurationclasspostprocessor)
11. [AOP 基础与切面创建](#十一aop-基础与切面创建)
12. [代理生成与链式调用](#十二代理生成与链式调用)
13. [事务切面](#十三事务切面)
14. [事务传播属性](#十四事务传播属性)
15. [BeanPostProcessor 注册时机](#十五beanpostprocessor-注册时机)

---

## 一、BeanDefinition 与 XML 标签解析

### 核心要点

- `BeanDefinition` 是 Bean 元数据的抽象描述（class、scope、属性、构造参数等）
- XML 解析的入口是 `AbstractApplicationContext#refresh()` → `obtainFreshBeanFactory()`
- 解析过程大量使用**委托模式（Delegate）** 与**模板方法模式**

### 多文件调用执行流程图

```
┌──────────────────────────────────────────────────────────────────────────┐
│                  XML 解析为 BeanDefinition 的完整调用链                  │
└──────────────────────────────────────────────────────────────────────────┘

  AbstractApplicationContext#refresh()
              │
              ▼
  obtainFreshBeanFactory()
              │
              ▼
  AbstractApplicationContext#refreshBeanFactory()         [模板方法]
              │
              ▼
  AbstractRefreshableApplicationContext#refreshBeanFactory()
              │  创建 DefaultListableBeanFactory
              ▼
  AbstractRefreshableApplicationContext#loadBeanDefinitions()
              │                                          [模板方法,子类实现]
              ▼
  AbstractXmlApplicationContext#loadBeanDefinitions()
              │  new XmlBeanDefinitionReader(beanFactory)  ◄── 委托模式
              ▼
  AbstractBeanDefinitionReader#loadBeanDefinitions(Resource...)
              │  for(resource : resources)
              ▼
  XmlBeanDefinitionReader#loadBeanDefinitions(Resource)
              │
              ▼
  XmlBeanDefinitionReader#doLoadBeanDefinitions()
              │  解析 XML → Document 对象
              ▼
  XmlBeanDefinitionReader#registerBeanDefinitions(Document, Resource)
              │  createBeanDefinitionDocumentReader()       ◄── 委托模式
              ▼
  DefaultBeanDefinitionDocumentReader#registerBeanDefinitions()
              │
              ▼
  DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions()
              │  preProcessXml(root)
              ▼
  parseBeanDefinitions(root, delegate)
              │
              ├─── delegate.isDefaultNamespace(ele) ? ──┐
              │                                         │
              ▼                                         ▼
   parseDefaultElement()                    delegate.parseCustomElement()
   [bean / import / alias / beans]          [context: / aop: / tx: 等]
```

---

## 二、默认标签与自定义标签解析

### 默认标签（`<bean>` 为例）

```
parseDefaultElement(ele, delegate)
        │
        ├── IMPORT_ELEMENT  →  importBeanDefinitionResource()
        ├── ALIAS_ELEMENT   →  processAliasRegistration()
        ├── BEAN_ELEMENT    →  processBeanDefinition()        ◄── 重点
        └── NESTED_BEANS    →  doRegisterBeanDefinitions(ele) [递归]

processBeanDefinition()
        │
        ▼
BeanDefinitionParserDelegate#parseBeanDefinitionElement()
        │  解析 id / name / class / scope / property
        ▼
AbstractBeanDefinition (持有 MutablePropertyValues)
        │
        ▼
BeanDefinitionReaderUtils.registerBeanDefinition()
        │
        ▼
DefaultListableBeanFactory (BeanDefinitionRegistry)
```

### 自定义标签解析流程

自定义标签解析五步法：
1. 根据当前解析标签的头信息找到对应的 `namespaceUri`
2. 加载所有 jar 中的 `META-INF/spring.handlers` 文件，建立映射
3. 根据 `namespaceUri` 从映射中找到 `NamespaceHandler` 实现类
4. 调用 `handler.init()` 方法（注册各种自定义标签的解析类）
5. 根据标签名找到解析类，调用 `parse()` 完成解析

```
┌──────────────────────────────────────────────────────────────────┐
│  自定义标签解析调用链（以 <context:component-scan> 为例）        │
└──────────────────────────────────────────────────────────────────┘

  BeanDefinitionParserDelegate#parseCustomElement()
              │  namespaceUri = "http://www.springframework.org/schema/context"
              ▼
  XmlReaderContext#getNamespaceHandlerResolver().resolve(uri)
              │
              ▼
  DefaultNamespaceHandlerResolver#resolve(namespaceUri)
              │  getHandlerMappings() ──► 加载所有 META-INF/spring.handlers
              │                          PropertiesLoaderUtils.loadAllProperties()
              ▼
  ContextNamespaceHandler (反射实例化)
              │  init() 注册各 BeanDefinitionParser:
              │    "component-scan"        → ComponentScanBeanDefinitionParser
              │    "property-placeholder"  → PropertyPlaceholderBeanDefinitionParser
              │    "annotation-config"     → AnnotationConfigBeanDefinitionParser
              ▼
  handler.parse(element, parserContext)
              │
              ▼
  ComponentScanBeanDefinitionParser#parse()
```

> 💡 Dubbo 等框架就是基于自定义标签机制实现的扩展。

---

## 三、component-scan 标签解析

### 关键执行流程

```
┌────────────────────────────────────────────────────────────────────┐
│      <context:component-scan> 扫描 → BeanDefinition 注册           │
└────────────────────────────────────────────────────────────────────┘

  ComponentScanBeanDefinitionParser#parse()
              │
              │  basePackages = StringUtils.tokenizeToStringArray(
              │                    basePackage, ",;\t\n")
              ▼
  configureScanner(parserContext, element)
              │  use-default-filters 默认 true
              ▼
  ClassPathBeanDefinitionScanner#doScan(basePackages)
              │
              ▼
  ClassPathScanningCandidateComponentProvider
              │  findCandidateComponents(basePackage)
              │       │
              │       ▼
              │   scanCandidateComponents(basePackage)
              │       │
              │       │  1. 扫描包路径下 .class 文件 (递归)
              │       │  2. File → Resource
              │       │  3. MetadataReader 读取类元数据
              │       │  4. isCandidateComponent() 判断注解
              │       │     默认匹配: @Component @Service @Controller
              │       │              @Repository @Configuration
              │       │  5. 封装成 ScannedGenericBeanDefinition
              ▼
  BeanDefinitionReaderUtils.registerBeanDefinition()
              │
              ▼
  BeanDefinitionRegistry (DefaultListableBeanFactory)
              │
              ▼
  ComponentScanBeanDefinitionParser#registerComponents()
              │
              ▼
  AnnotationConfigUtils.registerAnnotationConfigProcessors()
              │  注册顶层注解处理器:
              │    ┌─ ConfigurationClassPostProcessor      (BeanFactoryPostProcessor)
              │    ├─ AutowiredAnnotationBeanPostProcessor (BeanPostProcessor)
              │    ├─ CommonAnnotationBeanPostProcessor    (BeanPostProcessor)
              │    ├─ EventListenerMethodProcessor
              │    └─ DefaultEventListenerFactory
```

### 各处理器的职责

| 处理器 | 类型 | 解析的注解 |
|---|---|---|
| `ConfigurationClassPostProcessor` | BeanFactoryPostProcessor | `@Configuration` `@Import` `@Bean` `@ComponentScan` |
| `AutowiredAnnotationBeanPostProcessor` | BeanPostProcessor | `@Autowired` `@Value` |
| `CommonAnnotationBeanPostProcessor` | BeanPostProcessor | `@Resource` `@PostConstruct` `@PreDestroy` |

> 自定义扫描指定注解示例：
> ```java
> ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(registry);
> scanner.addIncludeFilter(new AnnotationTypeFilter(MyService.class));
> scanner.scan("com.enjoy.jack2021.customBean");
> ```

---

## 四、BeanPostProcessor 与 Bean 实例化总流程

### `refresh()` 关键步骤

```
AbstractApplicationContext#refresh()
   │
   ├── obtainFreshBeanFactory()          创建 BeanFactory + 解析 XML 标签
   ├── prepareBeanFactory(beanFactory)   设置 BeanFactory 属性
   ├── invokeBeanFactoryPostProcessors() 执行 BFPP / BDRPP
   ├── registerBeanPostProcessors()      把 BPP 实例化并加入 BeanFactory
   ├── initApplicationEventMulticaster() 初始化事件管理器
   ├── onRefresh()                       模板方法 (SpringBoot 内嵌 Tomcat)
   ├── registerListeners()               注册监听器
   └── finishBeanFactoryInitialization() ⭐ 最重要,实例化所有非懒加载单例
```

### Bean 实例化核心调用链

```
┌──────────────────────────────────────────────────────────────────────┐
│             finishBeanFactoryInitialization 调用链                   │
└──────────────────────────────────────────────────────────────────────┘

  finishBeanFactoryInitialization(beanFactory)
              │
              ▼
  DefaultListableBeanFactory#preInstantiateSingletons()
              │  for(beanName : beanNames)
              │    getMergedLocalBeanDefinition()  ← 合并父子 BD
              │    if(!abstract && singleton && !lazyInit)
              ▼
  AbstractBeanFactory#getBean()
              │
              ▼
  AbstractBeanFactory#doGetBean()
              │
              ├──► getSingleton(beanName)               ◄── 一级缓存优先
              │    [命中则直接返回]
              │
              ├──► getMergedLocalBeanDefinition()
              ├──► 检查 dependsOn (先实例化依赖)
              │
              ▼
  getSingleton(beanName, ObjectFactory)              ◄── 单例创建入口
              │  beforeSingletonCreation()
              │      [加入 singletonsCurrentlyInCreation]
              ▼
  AbstractAutowireCapableBeanFactory#createBean()
              │
              │  resolveBeforeInstantiation()         ← AOP 提前生成代理入口
              │  [若返回非 null,直接返回,跳过 doCreateBean]
              │
              ▼
  AbstractAutowireCapableBeanFactory#doCreateBean()
              │
              ├──► createBeanInstance()                ★ 实例化(堆中开辟内存)
              │
              ├──► applyMergedBeanDefinitionPostProcessors()
              │      [AutowiredAnnotationBPP / CommonAnnotationBPP 收集注解]
              │
              ├──► addSingletonFactory()                ★ 加入三级缓存
              │      (解决循环依赖关键)
              │
              ├──► populateBean()                       ★ DI 依赖注入
              │
              ├──► initializeBean()                     ★ Aware/init/AOP
              │      ├── invokeAwareMethods()
              │      ├── applyBeanPostProcessorsBeforeInitialization()
              │      │     └─ @PostConstruct 调用
              │      ├── invokeInitMethods()
              │      │     └─ afterPropertiesSet / init-method
              │      └── applyBeanPostProcessorsAfterInitialization()
              │            └─ AOP 代理生成入口
              │
              └──► registerDisposableBeanIfNecessary()  注册销毁回调
```

---

## 五、Bean 实例化与注解收集

### `createBeanInstance` 实例化逻辑（按优先级）

1. **`factoryMethod` 实例化** (`<bean factory-method>` 或 `@Bean`)
   - 另一个类的非静态方法
   - 当前类的静态方法
2. **带 `@Autowired` 的有参构造函数**
3. **不带注解的有参构造函数**
4. **无参构造函数**（兜底）

```java
Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
if (ctors != null || ...) {
    return autowireConstructor(beanName, mbd, ctors, args);
}
```

> ⚠️ 一个类有多个构造函数时，必须 `@Autowired(required = false)`，否则报错。

### `applyMergedBeanDefinitionPostProcessors` 注解收集

```
AutowiredAnnotationBeanPostProcessor#findAutowiringMetadata()
        │
        ▼
buildAutowiringMetadata(clazz)
        │
        ├── ReflectionUtils.doWithLocalFields()   收集字段上的 @Autowired
        └── ReflectionUtils.doWithLocalMethods()  收集方法上的 @Autowired
        │
        ▼
封装为 List<InjectionMetadata.InjectedElement>
        │
        ▼
缓存到 injectionMetadataCache,等 populateBean 阶段调用
```

---

## 六、@Autowired / @PostConstruct / @PreDestroy

### 注解归属

| 注解 | 处理器 |
|---|---|
| `@Autowired` `@Value` | `AutowiredAnnotationBeanPostProcessor` |
| `@PostConstruct` `@PreDestroy` `@Resource` | `CommonAnnotationBeanPostProcessor` |

### @Autowired 依赖注入调用链

```
┌──────────────────────────────────────────────────────────────────────┐
│         populateBean → @Autowired 触发引用类型 getBean              │
└──────────────────────────────────────────────────────────────────────┘

  populateBean(beanName, mbd, instanceWrapper)
              │
              ▼
  InstantiationAwareBeanPostProcessor#postProcessProperties()
              │
              ▼
  AutowiredAnnotationBeanPostProcessor#postProcessProperties()
              │
              ▼
  findAutowiringMetadata(beanName, beanClass, pvs)
              │  metadata = buildAutowiringMetadata(clazz)
              ▼
  InjectionMetadata#inject(bean, beanName, pvs)
              │  element.inject(target, beanName, pvs)  [模板方法]
              ▼
  AutowiredFieldElement#inject()
              │
              ▼
  DefaultListableBeanFactory#resolveDependency()
              │
              ▼
  DefaultListableBeanFactory#doResolveDependency()
              │  getAutowireCandidateResolver()
              │      .getSuggestedValue(descriptor)   ← @Value ${...}
              │
              ▼
  findAutowireCandidates(beanName, type, descriptor)
              │
              ▼
  DefaultListableBeanFactory#addCandidateEntry()
              │
              ▼
  descriptor.resolveCandidate()  ◄── 触发被依赖 Bean 的 getBean
```

### initializeBean 关键调用

```
initializeBean(beanName, exposedObject, mbd)
        │
        ├── invokeAwareMethods()                              [BeanNameAware 等]
        │
        ├── applyBeanPostProcessorsBeforeInitialization()
        │     ├── ApplicationContextAwareProcessor            [Aware 接口]
        │     │     ├ EnvironmentAware
        │     │     ├ EmbeddedValueResolverAware
        │     │     ├ ResourceLoaderAware
        │     │     ├ ApplicationEventPublisherAware
        │     │     ├ MessageSourceAware
        │     │     └ ApplicationContextAware
        │     ├── InitDestroyAnnotationBPP                    [@PostConstruct]
        │     └── ImportAwareBeanPostProcessor                [ImportAware]
        │
        ├── invokeInitMethods()                               [afterPropertiesSet / init-method]
        │
        └── applyBeanPostProcessorsAfterInitialization()      ★ AOP 入口
```

---

## 七、循环依赖与三级缓存

### 三种循环依赖情况

| 情况 | 能否解决 |
|---|---|
| 单例 + 属性注入循环依赖 | ✅ 可以 |
| 构造函数循环依赖 | ❌ 不行 |
| 多例（prototype）循环依赖 | ❌ 不行 |

### 三级缓存

| 缓存 | 字段 | 作用 |
|---|---|---|
| 一级 | `singletonObjects` | 完全创建好的 Bean |
| 二级 | `earlySingletonObjects` | 提前暴露但尚未注入完成 |
| 三级 | `singletonFactories` | `ObjectFactory` 工厂（可生成代理） |

### 单例循环依赖解决流程图

```
┌──────────────────────────────────────────────────────────────────────────┐
│            A 依赖 B,B 依赖 A 的循环依赖解决（单例 + 字段注入）          │
└──────────────────────────────────────────────────────────────────────────┘

  ① getBean(A)
      │
      ▼ DefaultSingletonBeanRegistry#getSingleton(A)
        三级缓存均未命中
      │
      ▼ createBean(A) → doCreateBean(A)
        ├─ createBeanInstance(A)             ──► new A() 仅堆中实例
        │
        ├─ addSingletonFactory(A, ()->getEarlyBeanReference)
        │                       ★ A 放入三级缓存
        │
        └─ populateBean(A)
                │
                │  解析 @Autowired B b;
                │
                ▼ ② getBean(B)
                  │
                  ▼ createBean(B) → doCreateBean(B)
                    ├─ createBeanInstance(B)
                    ├─ addSingletonFactory(B, ...)  ★ B 放入三级缓存
                    │
                    └─ populateBean(B)
                          │
                          │  解析 @Autowired A a;
                          │
                          ▼ ③ getBean(A)
                            │
                            ▼ getSingleton(A, true)
                              ├─ 一级:无
                              ├─ 二级:无
                              └─ 三级:命中 ObjectFactory
                                  │
                                  ▼ singletonFactory.getObject()
                                    [可经 SmartIABPP 生成代理]
                                    │
                                    ├─► 升级到二级缓存
                                    └─► 移除三级缓存
                            │
                            返回 A 的早期引用 ◄────────────┐
                          │                                │
                          B 注入 A 完成                    │
                    │                                      │
                    initializeBean(B) → 完成 B 创建        │
                    addSingleton(B,...) ★ B 放入一级缓存   │
                  │                                        │
                  返回完整的 B ◄──────────────────────┐    │
                │                                     │    │
                A 注入 B 完成                         │    │
        │                                             │    │
        initializeBean(A) → 完成                      │    │
        addSingleton(A,...) ★ A 放入一级缓存          │    │
      │                                               │    │
      返回完整的 A                                    │    │
                                                      │    │
※ 关键: A 在 populateBean 之前已放入三级缓存,所以      │    │
        B 注入 A 时能拿到 A 的早期引用 ────────────────┴────┘
```

### 三级缓存检索代码

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);   // 一级
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName); // 二级
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName); // 三级
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject); // 升级二级
                    this.singletonFactories.remove(beanName);                  // 删除三级
                }
            }
        }
    }
    return singletonObject;
}
```

### 为什么构造函数循环依赖无法解决？

> A 通过构造方法实例化时即触发 B 的实例化，此时 A **还未来得及把自己引用放入三级缓存**，故 B 在构造期间拿不到 A 的早期引用。

---

## 八、配置文件解析与 Environment

### 占位符 `${enjoy.name}` 解析机制

1. XML 解析时把 `<property value="${enjoy.name}"/>` 封装到 `MutablePropertyValues`，**值保持原样**
2. `PropertySourcesPlaceholderConfigurer` 实现 `BeanFactoryPostProcessor`，在 `postProcessBeanFactory` 阶段统一替换占位符
3. 数据源：`properties 文件` + `Environment`
4. XML 中必须配置：`<context:property-placeholder location="classpath:application.properties"/>`，否则启动报错
5. 也可通过 `@Bean` 方式注册 `PropertySourcesPlaceholderConfigurer`
6. 或者实现 `BeanDefinitionRegistryPostProcessor` 自定义修改 BD

### @Value 与 @Autowired 的处理逻辑相同：先收集，后注入。

---

## 九、自定义 scope 与 FactoryBean

### 在 Environment 中添加自定义属性

```java
// 1. 实现 EnvironmentAware
public class PropertiesPro implements EnvironmentAware {
    @Override
    public void setEnvironment(Environment environment) {
        StandardEnvironment bean = (StandardEnvironment) environment;
        Properties properties = new Properties();
        properties.put("enjoy.name", "James");
        PropertiesPropertySource propertiesCustom =
            new PropertiesPropertySource("propertiesCustom", properties);
        bean.getPropertySources().addLast(propertiesCustom);
    }
}
```

### FactoryBean

```java
public class FactoryBeanDemo implements FactoryBean<Student> {
    @Override
    public Student getObject() { return new Student(); }   // 调用时才执行
    @Override
    public Class<?> getObjectType() { return Student.class; }
}

// applicationContext.getBean("factoryBeanDemo")          ← 拿到 Student
// applicationContext.getBean("&factoryBeanDemo")         ← 拿到 FactoryBean 本身
```

### 让对象被 Spring 管理的 4 种方式

1. 自定义 `BeanDefinition` 并注册到 Registry
2. `applicationContext.getBeanFactory().registerSingleton("jack", new Jack())`
3. 自定义 `FactoryBean` 接口
4. `@Bean` 方法

---

## 十、ConfigurationClassPostProcessor

### 注册路径

```
AnnotationConfigApplicationContext()
        │
        ▼
new AnnotatedBeanDefinitionReader(this)
        │
        ▼
AnnotationConfigUtils.registerAnnotationConfigProcessors(registry)
        │
        ├─ new RootBeanDefinition(ConfigurationClassPostProcessor.class)
        ├─ new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class)
        └─ new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class)
```

### 负责解析的注解

`@ComponentScan` `@Import` `ImportSelector` `ImportBeanDefinitionRegistrar` `@ImportResource` `@Bean` `@Configuration`

### 解析调用链

```
┌──────────────────────────────────────────────────────────────────────┐
│            ConfigurationClassPostProcessor 解析调用链               │
└──────────────────────────────────────────────────────────────────────┘

  invokeBeanFactoryPostProcessors(beanFactory)
              │
              ▼
  ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry()
              │
              ▼
  processConfigBeanDefinitions(registry)
              │  candidateNames = registry.getBeanDefinitionNames()
              ▼
  ConfigurationClassParser parser = new ConfigurationClassParser(...)
              │
              ▼
  parser.parse(candidates)
              │
              ▼
  parser.processConfigurationClass(new ConfigurationClass(metadata, beanName))
              │
              ├─► ConditionEvaluator#shouldSkip()           [@Conditional 过滤]
              │
              ▼
  doProcessConfigurationClass(configClass, sourceClass, filter)
              │
              ├── processMemberClasses()            内部类(带 @Component)
              ├── processPropertySource()           @PropertySource
              ├── ComponentScan 处理                 @ComponentScan(s)
              ├── processImports()                  @Import / ImportSelector / ImportBeanDefinitionRegistrar
              ├── processImportResource()           @ImportResource (xml)
              ├── retrieveBeanMethodMetadata()      收集 @Bean 方法
              └── processInterfaces()               接口上的 @Bean
              │
              ▼
  this.configurationClasses.put(configClass, configClass)
              │
              ▼
  reader.loadBeanDefinitions(configClasses)        ★ 最终注册为 BD
              │
              ├─ registerBeanDefinitionForImportedConfigurationClass()
              └─ loadBeanDefinitionsForBeanMethod()    @Bean → BD
              │
              ▼
  ConfigurationClassPostProcessor#postProcessBeanFactory()
              │
              ▼
  enhanceConfigurationClasses(beanFactory)         ★ CGLIB 增强
              │  ConfigurationClassEnhancer.enhance()
              │  → BeanMethodInterceptor 拦截 @Bean 方法,从容器拿
```

### @Component vs @Configuration

| 维度 | @Component + @Bean | @Configuration + @Bean |
|---|---|---|
| CGLIB 增强 | ❌ 否 | ✅ 是 |
| `@Bean` 方法多次调用 | 违背单例（每次 new） | 走容器缓存（同一实例） |

### ImportSelector / DeferredImportSelector

- `ImportSelector#selectImports` 在 `BeanDefinitionRegistry` 调用前执行
- `DeferredImportSelector` 在 SpringBoot 自动装配中大量使用，内部 `Group` 接口流程：
  1. 先执行 `Group#process` 收集配置类
  2. 再执行 `Group#selectImports`
  3. 最后执行外部 `selectImports`

---

## 十一、AOP 基础与切面创建

### 核心概念

- **Advisor** = `Pointcut` + `Advice`
- **Pointcut**: 匹配/拦截规则
  - `ClassFilter` 类匹配
  - `MethodMatcher` 方法匹配
- **Advice**: 增强逻辑

### AOP 入口处理器

- `AbstractAutoProxyCreator` （基类）
- `AspectJAwareAdvisorAutoProxyCreator`
- `AnnotationAwareAspectJAutoProxyCreator`
- `InfrastructureAdvisorAutoProxyCreator` （事务 AOP 用）

### AOP 调用链

```
┌──────────────────────────────────────────────────────────────────────┐
│                  AOP 切面收集与代理创建调用链                        │
└──────────────────────────────────────────────────────────────────────┘

  AbstractAutoProxyCreator#postProcessAfterInitialization()
              │
              ▼
  wrapIfNecessary(bean, beanName, cacheKey)
              │
              ▼
  getAdvicesAndAdvisorsForBean(beanClass, beanName, null)
              │
              ▼
  AbstractAdvisorAutoProxyCreator#findEligibleAdvisors(beanClass, beanName)
              │
              ├──► findCandidateAdvisors()                  ★ 找出所有候选切面
              │       │
              │       ├── BeanFactoryAdvisorRetrievalHelper#findAdvisorBeans()
              │       │     [自定义 implements Advisor]
              │       │
              │       └── AnnotationAwareAspectJAutoProxyCreator#findCandidateAdvisors()
              │             └── aspectJAdvisorsBuilder.buildAspectJAdvisors()
              │                   │
              │                   ▼
              │             ReflectiveAspectJAdvisorFactory#getAdvisors()
              │                   │
              │                   ▼
              │             getAdvisor(method, factory, order, aspectName)
              │                   │
              │                   ├── getPointcut()       ◄── AspectJExpressionPointcut
              │                   │
              │                   └── new InstantiationModelAwarePointcutAdvisorImpl()
              │                         │
              │                         └── instantiateAdvice(declaredPointcut)
              │                                │
              │                                ▼
              │                         ReflectiveAspectJAdvisorFactory#getAdvice()
              │                                │  根据注解类型创建对应 Advice:
              │                                │  @Around  → AspectJAroundAdvice
              │                                │  @Before  → AspectJMethodBeforeAdvice
              │                                │  @After   → AspectJAfterAdvice
              │                                │  ...
              │                                │  (均实现 MethodInterceptor)
              │
              ├──► findAdvisorsThatCanApply()    ★ 匹配当前 Bean
              │
              └──► extendAdvisors(eligibleAdvisors)
                       │
                       └── advisors.add(0, ExposeInvocationInterceptor.ADVISOR)
                             [负责切面间参数链式传递]
              │
              ▼
  createProxy(beanClass, beanName, specificInterceptors,
              new SingletonTargetSource(bean))
```

### 切面排序规则

1. `ExposeInvocationInterceptor.ADVISOR` 排第一位
2. 实现 `Order` 接口的按值升序
3. `@Aspect` 注解切面按代码顺序，排最后

---

## 十二、代理生成与链式调用

### 代理工厂创建流程

```
createProxy(beanClass, beanName, specificInterceptors, targetSource)
        │
        ▼
ProxyFactory proxyFactory = new ProxyFactory()
        │  proxyFactory.copyFrom(this)   ← 从 AnnotationAwareAspectJAutoProxyCreator
        │  buildAdvisors(beanName, specificInterceptors)
        │       │
        │       └── advisorAdapterRegistry.wrap(...) 把 interceptor 包装成 Advisor
        │  proxyFactory.addAdvisors(advisors)
        │  proxyFactory.setTargetSource(targetSource)
        │
        ▼
proxyFactory.getProxy(classLoader)
        │
        ├── proxyTargetClass = true   →  CglibAopProxy   (CGLIB 子类代理)
        └── proxyTargetClass = false  →  JdkDynamicAopProxy (JDK 接口代理)
              [最终选择还取决于 targetClass 是否有接口]
```

> `@EnableAspectJAutoProxy(proxyTargetClass=true/false)` 控制策略
> 每个代理对象都有自己独立的 `ProxyFactory` 与 `JdkDynamicAopProxy`，**非全局共享**

### 链式调用流程图

```
┌─────────────────────────────────────────────────────────────────┐
│              AOP 代理方法调用 → 责任链执行                      │
└─────────────────────────────────────────────────────────────────┘

  proxy.businessMethod(args)
        │
        ▼
  JdkDynamicAopProxy#invoke(proxy, method, args)
        │
        ├── if (advised.exposeProxy)
        │     AopContext.setCurrentProxy(proxy)   ← 放入 ThreadLocal
        │
        ├── TargetSource targetSource = advised.targetSource
        │   target = targetSource.getTarget()
        │
        ├── chain = advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass)
        │
        ▼
  new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain)
        │
        ▼
  ReflectiveMethodInvocation#proceed()
        │
        │  if (currentInterceptorIndex == chain.size() - 1)
        │      return invokeJoinpoint()   ← 反射调用目标方法
        │
        │  Object interceptor = chain.get(++currentInterceptorIndex)
        │
        │  if (interceptor instanceof InterceptorAndDynamicMethodMatcher)
        │      // pointcut.isRuntime() == true 时处理参数
        │      matcher.matches(method, class, args)?
        │             interceptor.invoke(this) : proceed()
        │  else
        │      return ((MethodInterceptor)interceptor).invoke(this)
        │
        ▼
  各 MethodInterceptor.invoke():
        ├── ExposeInvocationInterceptor    [传递 invocation]
        ├── AspectJAroundAdvice
        ├── MethodBeforeAdviceInterceptor
        ├── AspectJAfterAdvice
        ├── AfterReturningAdviceInterceptor
        └── AspectJAfterThrowingAdvice
        每个 interceptor 调用 mi.proceed() 递归推进链表
```

### 代理提前生成

```java
// AbstractAutowireCapableBeanFactory#createBean
Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
// 若 customTargetSourceCreators 不为空,可在此提前生成多例代理
// 生成后直接返回,不再执行 doCreateBean
```

### scope 代理（多例代理）

```java
@Scope(value = SCOPE_PROTOTYPE, proxyMode = ScopedProxyMode.TARGET_CLASS)
```

流程：

```
ClassPathBeanDefinitionScanner#doScan
        │
        ▼
AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, registry)
        │
        ▼
RootBeanDefinition proxyDefinition = new RootBeanDefinition(ScopedProxyFactoryBean.class)
        │
        ▼
ScopedProxyFactoryBean#setBeanFactory()
        │  添加 DelegatingIntroductionInterceptor 切面
        ▼
生成代理对象 → 每次方法调用都返回新实例
```

---

## 十三、事务切面

### 入口

```
@EnableTransactionManagement
        │
        ▼
@Import(TransactionManagementConfigurationSelector.class)
        │
        ▼
selectImports() 返回:
        ├── AutoProxyRegistrar                    ← 注册 InfrastructureAdvisorAutoProxyCreator
        └── ProxyTransactionManagementConfiguration
                ├── BeanFactoryTransactionAttributeSourceAdvisor (事务切面)
                ├── TransactionAttributeSource                   (@Transactional 解析后封装)
                └── TransactionInterceptor                       (Advice = MethodInterceptor)
```

### 事务切面调用链

```
┌──────────────────────────────────────────────────────────────────────┐
│                @Transactional 方法调用 → 事务执行                    │
└──────────────────────────────────────────────────────────────────────┘

  proxy.transactionalMethod(args)
        │
        ▼
  TransactionInterceptor#invoke(invocation)
        │
        ▼
  TransactionAspectSupport#invokeWithinTransaction(method, targetClass, callback)
        │
        ├── TransactionAttribute txAttr = getTransactionAttribute()
        │         │
        │         ▼
        │   AbstractFallbackTransactionAttributeSource#getTransactionAttribute()
        │         │  Method spec = AopUtils.getMostSpecificMethod(method, targetClass)
        │         ▼
        │   SpringTransactionAnnotationParser#parseTransactionAnnotation()
        │         (解析 @Transactional 属性)
        │
        ├── createTransactionIfNecessary(tm, txAttr, joinpointId)
        │         │
        │         ▼
        │   AbstractPlatformTransactionManager#getTransaction(txAttr)
        │         │
        │         ├── doGetTransaction()  ← DataSourceTransactionManager
        │         │     DataSourceTransactionObject:
        │         │       └─ ConnectionHolder (从 ThreadLocal 拿:
        │         │             TransactionSynchronizationManager.getResource(dataSource))
        │         │
        │         ├── 是否已有事务?
        │         │     ├─ 有 → handleExistingTransaction()  [传播属性处理]
        │         │     └─ 无 → startTransaction()
        │         │             └─ doBegin() 关闭 autoCommit
        │
        ├── try {
        │       retVal = invocation.proceedWithInvocation()   ← 火炬传递,执行业务
        │   } catch (Throwable ex) {
        │       completeTransactionAfterThrowing(txInfo, ex)  ← 异常回滚
        │   }
        │
        ├── cleanupTransactionInfo(txInfo)
        │
        └── commitTransactionAfterReturning(txInfo)            ← 提交
                │
                ▼
        AbstractPlatformTransactionManager#commit()
                │  processCommit(defStatus)
                │  └─ doCommit() conn.commit()
                ▼
        cleanupAfterCompletion(status)   ← 恢复挂起的连接(传播属性用)
```

### 关键对象

- `DataSourceTransactionObject`：事务对象
- `ConnectionHolder`：连接对象包装器，通过 `TransactionSynchronizationManager` 绑定到 `ThreadLocal`
- `SuspendedResourcesHolder`：挂起事务时保存的连接资源

> 💡 **核心结论**：事务必然与连接对象挂钩；同一事务下 DML 操作必然共享同一个 `Connection`，且 `autoCommit=false`。

---

## 十四、事务传播属性

### 嵌套调用伪代码

```java
@Transactional
public void outer() {
    try {
        createTransactionIfNecessary();      // ① 开启外层事务
        areaService.addArea();               // 内部嵌入新方法切面
        goodsService.addGoods();
        commitTransactionAfterReturning();   // 提交(根据状态决定)
    } catch (Exception ex) {
        completeTransactionAfterThrowing();  // 回滚
    }
}
```

### PROPAGATION_REQUIRES_NEW 流程

```
进入新事务方法
     │
     ▼
suspend(transaction)                       挂起当前事务
     │  doSuspend() → 解绑 ConnectionHolder
     │  return SuspendedResourcesHolder
     ▼
startTransaction(definition, transaction, debug, suspendedResources)
     │  newTransaction = true
     │  doBegin() → 新 Connection,autoCommit=false
     ▼
执行业务方法
     │
     ▼
commit() / rollback()
     │
     ▼
cleanupAfterCompletion(status)
     │  doCleanupAfterCompletion() 释放新连接
     │  resume(transaction, suspendedResources) ◄── 恢复绑定旧连接
```

### NESTED 传播属性

- 相对 `PROPAGATION_REQUIRED` **多创建一个回滚点（savepoint）**
- 异常未匹配回滚规则时仍会 commit

### 编程式事务

- **没有传播属性概念**，事务粒度更细
- `TransactionSynchronizationManager.registerSynchronization(new DoOnAfterCommit())`：注册提交后回调钩子
- `TransactionSynchronizationManager.getResource(dataSource)`：拿到当前线程绑定的连接

### 缓存切面 / 异步切面

- 实现机制与事务一致：**Advice + Pointcut 的 AOP 代理**
- 缓存切面 Advice 中执行查缓存/写缓存逻辑
- 异步切面 Advice 中提交线程池执行

---

## 十五、BeanPostProcessor 注册时机

```
AbstractApplicationContext.refresh()
    ├── prepareRefresh()
    ├── obtainFreshBeanFactory()
    ├── prepareBeanFactory()
    ├── postProcessBeanFactory()
    ├── invokeBeanFactoryPostProcessors()         ← BFPP / BDRPP 执行完毕
    ├── registerBeanPostProcessors(beanFactory)   ★ 在此注册所有 BPP
    │     │
    │     │  PostProcessorRegistrationDelegate.registerBeanPostProcessors():
    │     │     1. getBeanNamesForType(BeanPostProcessor.class)
    │     │     2. 按 PriorityOrdered → Ordered → 普通 排序
    │     │     3. 提前实例化所有 BPP (getBean)
    │     │     4. beanFactory.addBeanPostProcessor(pp)
    │     ▼
    ├── initMessageSource()
    ├── initApplicationEventMulticaster()
    ├── onRefresh()
    ├── registerListeners()
    ├── finishBeanFactoryInitialization()         ★ 普通 Bean 在此创建,会被 BPP 拦截
    └── finishRefresh()
```

### 关键结论

1. **BPP 早于普通 Bean 创建** —— 因为 BPP 要参与后续所有 Bean 的生命周期
2. **BPP 自身不会被自己处理**，也不会被其他默认 BPP 处理
3. 注册由 Spring 自动完成，通过 `beanFactory.addBeanPostProcessor()`，无需手动调用

---

## 附：refresh() 全流程一图速记

```
┌────────────────────────────────────────────────────────────────────┐
│              AbstractApplicationContext#refresh()                  │
└────────────────────────────────────────────────────────────────────┘
  1. prepareRefresh()                       准备工作
  2. obtainFreshBeanFactory()               ★ 创建 BF + 解析标签 → BD
  3. prepareBeanFactory()                   设置 BF 属性 / Aware
  4. postProcessBeanFactory()               子类扩展
  5. invokeBeanFactoryPostProcessors()      ★ 执行 BFPP / BDRPP
                                              (ConfigurationClassPostProcessor 在此)
  6. registerBeanPostProcessors()           ★ 注册所有 BPP
  7. initMessageSource()                    国际化
  8. initApplicationEventMulticaster()      事件广播器
  9. onRefresh()                            模板 (SpringBoot 启动 Tomcat)
 10. registerListeners()                    注册监听器
 11. finishBeanFactoryInitialization()      ★ 实例化非懒加载单例
       └─ doGetBean → createBean → doCreateBean
          ├─ createBeanInstance
          ├─ applyMergedBeanDefinitionPostProcessors  (注解收集)
          ├─ addSingletonFactory                       (三级缓存)
          ├─ populateBean                              (DI)
          └─ initializeBean                            (Aware/init/AOP)
 12. finishRefresh()                        发布 ContextRefreshedEvent
```
