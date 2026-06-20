# Sentinel 整理稿

> 原笔记版本基准：**Sentinel 1.8.7**(`mvn clean install -DskipTests` 构建)
> 适用范围：dashboard 控制台 + sentinel-core 客户端 + 与 Nacos 整合的规则持久化

---

## 一、知识点总览

```
Sentinel
├─ 源码环境
│   ├─ 编译问题:注释 maven-pmd-plugin、sentinel-quarkus-adapter
│   └─ JDK 1.8 + Maven 3.x
├─ Dashboard
│   ├─ 首页 / 实时监控 / 簇点链路 / 机器列表(心跳)
│   ├─ 流控规则(QPS、并发线程、流控模式、流控效果)
│   ├─ 熔断规则(慢调用比例、异常比例、异常数)
│   ├─ 热点规则(参数级限流)
│   ├─ 系统规则(load、CPU、RT、入口 QPS、线程数)
│   ├─ 授权规则(黑白名单)
│   ├─ 集群流控(Token Server / Token Client)
│   └─ 规则持久化(Nacos 数据源)
└─ Core 内核
    ├─ SPI 加载机制(SpiLoader)
    ├─ 配置加载顺序
    ├─ 命令处理器(CommandHandler) + 端口监听
    ├─ ProcessorSlotChain(责任链)
    └─ 滑动时间窗口(LeapArray)
```

---

## 二、Dashboard 核心机制

### 2.1 流控效果三种实现

| 效果 | 实现类 | 算法本质 |
| --- | --- | --- |
| 快速失败 | `DefaultController` | 计数器,超过 count 直接 return false |
| Warm Up(冷启动) | `WarmUpController` | 改良的令牌桶,初始桶满,逐步降低生成速率 |
| 排队等待 | `ThrottlingController` | **匀速器(虚拟队列)**,按预计通过时间排队 |

**⚠ 原笔记纠错(关键点 1):**
原文写"排队等待 ... 对应的是漏桶算法"。
更精确的说法是:`ThrottlingController` 实现的是**匀速放行(virtual queue / RateLimiter 风格)**,它通过 `latestPassedTime + costTime` 计算下一个请求的预计通过时刻,效果上类似漏桶,但**不是经典漏桶**(没有真实的"桶"作为缓冲区),叫"匀速排队"更准。

### 2.2 流控模式

- **直接**:统计自身资源的 QPS
- **关联**:`/write` 达到阈值 → 限制 `/read`(保护关联资源)
- **链路**:同一资源被多个入口调用,只统计指定入口的调用链

### 2.3 熔断规则三种类型

```
                      ┌─────────────────┐
                      │      CLOSED     │  正常放行,统计指标
                      └────────┬────────┘
                               │ 触发阈值(慢调用比例 / 异常比例 / 异常数)
                               ▼
                      ┌─────────────────┐
                      │       OPEN      │  熔断期 timeWindow,全部拒绝
                      └────────┬────────┘
                               │ 时间窗口过去,放一个探测请求
                               ▼
                      ┌─────────────────┐
                      │    HALF_OPEN    │  探测中
                      └───┬─────────┬───┘
                          │成功      │失败
                          ▼          ▼
                       CLOSED       OPEN(继续熔断 timeWindow)
```

**⚠ 原笔记纠错(关键点 2):** 原文写"**短路器**的三种状态",应为"**熔断器 / 断路器(Circuit Breaker)**"。Sentinel 中类名为 `CircuitBreaker`。

### 2.4 热点规则

依赖 `sentinel-parameter-flow-control`。

**⚠ 原笔记纠错(关键点 3):** 原文将 `WPS=6` 写为示例,应为 **QPS=6**(笔误)。
另外,自 Sentinel 1.4 起 `sentinel-parameter-flow-control` 已默认包含在 `sentinel-core` 内,1.8.x 客户端通常**无需再单独引入依赖**,只在做参数热点规则的 dashboard 推送时需要(可通过 `ParamFlowRuleManager` 直接管理)。

---

## 三、规则持久化:Dashboard ↔ Nacos ↔ Client 调用流程

涉及三方组件(dashboard / nacos / client),画清楚:

```
┌───────────────────────┐
│  Sentinel Dashboard   │
│  (修改后含 NacosRule  │
│   Provider/Publisher) │
└─────────┬─────────────┘
          │ ① 新增/修改规则
          │   FlowRuleApiController.apiAddFlowRule
          │     └─> nacosConfigService.publishConfig(
          │             dataId, group, JSON.toJSONString(rules))
          ▼
┌───────────────────────┐
│        Nacos          │   ② 配置变更事件 (长轮询 / GRPC 推送)
│  dataId =             │ ────────────────────────────────────┐
│   {appName}-flow-rules│                                     │
│  group = SENTINEL_GROUP                                     │
└───────────────────────┘                                     │
                                                              ▼
                                          ┌──────────────────────────┐
                                          │   Sentinel Client (App)  │
                                          │                          │
                                          │   InitFunc(SPI)          │
                                          │     └─> NacosDataSource  │
                                          │           ↓ listener     │
                                          │     FlowRuleManager      │
                                          │       .register2Property │
                                          │     ↓                    │
                                          │   规则热更新,无需重启    │
                                          └──────────────────────────┘
```

**关键文件调用链:**

```
Dashboard 启动(改造后):
  FlowControllerV2
    └─> DynamicRuleProvider<List<FlowRuleEntity>>     ← 改成 NacosRuleProvider
    └─> DynamicRulePublisher<List<FlowRuleEntity>>    ← 改成 NacosRulePublisher
                                │
                                └─> ConfigService.publishConfig(dataId, group, rules)

Client 启动(改造后):
  InitExecutor.doInit()
    └─> 通过 SPI 加载 META-INF/services/com.alibaba.csp.sentinel.init.InitFunc
        └─> DataSourceInitFunc.init()
            └─> new NacosDataSource(remoteAddress, group, dataId, parser)
                └─> FlowRuleManager.register2Property(ds.getProperty())
                    └─> 内部注册到 SentinelProperty,后续 Nacos 推送 → 自动 update
```

**⚠ 原笔记纠错(关键点 4):** 原文写"**复制 nacos 目录,copy 到 rule 规则下**"。准确表述是:Sentinel 仓库的 `sentinel-dashboard` 模块在 `test/java/.../rule/nacos/` 下已经**给出了示例代码**(Test 源码集,默认不会打入主包),需要把 `NacosConfig`、`NacosConfigUtil`、`FlowRuleNacosProvider`、`FlowRuleNacosPublisher` 等几个类**从 test 目录搬到 main 目录**,并把 `pom.xml` 中 `sentinel-datasource-nacos` 依赖的 `<scope>test</scope>` 注释掉。原文里的"注释掉 scope"指的是这件事,但叙述不够清楚。

---

## 四、集群流控

```
                ┌──────────────────────┐
                │   Token Server       │
                │  (独立部署 / 嵌入式) │
                └─────────┬────────────┘
                          │ Netty: 客户端发 FlowRequest
                          │ 服务端回 TokenResultStatus
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ Client A│       │ Client B│       │ Client C│
   │ (Embed) │       │ (Embed) │       │ (Embed) │
   └─────────┘       └─────────┘       └─────────┘

        全局阈值在 Server 端统一计算,各 Client 节点不再各自限流
```

- TokenServer 初始化:监听端口、注册 RequestHandler、加载 namespace 集群规则
- TokenClient 初始化:NettyClient 与 Server 建立长连接,失败回退到本地规则(fallback)

---

## 五、Sentinel SPI 机制(纠错重点)

**⚠ 原笔记纠错(关键点 5):** 原文写"加载 `/META_INF/Service/` 文件"。
错误两处:
1. 是 **`META-INF`**(`-`,不是 `_`)
2. 是 **`services`**(复数 `s`),不是 `Service`

Sentinel 自己实现了 `SpiLoader`,语法仿 JDK SPI:文件位于 `META-INF/services/<完整接口名>`,内容为实现类的全限定名,可选地用 `@Spi(order = …, isDefault = true)` 注解控制加载顺序与默认实现。与 Dubbo SPI 不同,**Sentinel SPI 不支持别名 key**,只按类名 + `@Spi` 注解控制。

---

## 六、配置加载优先级

```
源码:SentinelConfigLoader.load()
顺序(高 → 低):
  1. JVM 系统属性 (-Dcsp.sentinel.config.file=…)
  2. 环境变量    (CSP_SENTINEL_CONFIG_FILE)
  3. classpath:sentinel.properties
  4. legacy path

注意:上述是「文件路径」的确定顺序;
确定文件后,加载完文件再用 System.getProperties() 覆写,
所以 「JVM -D 参数 始终 > 配置文件 同名 key」。
```

原笔记代码注释("保证后面加载文件的优先级别计较高"是误读)——实际作用是用 JVM 参数**覆盖**已加载的文件值,**JVM 参数优先级最高**。

---

## 七、ProcessorSlotChain(责任链)

调用一次资源(`SphU.entry`)在 Slot Chain 上走完一圈:

```
Entry
  │
  ▼
┌──────────────────┐
│ NodeSelectorSlot │  构建调用链路上的 DefaultNode
├──────────────────┤
│ ClusterBuilder   │  按资源 + origin 构建 ClusterNode
├──────────────────┤
│ LogSlot          │  Block 异常日志
├──────────────────┤
│ StatisticSlot    │  统计 QPS / RT / 异常 / 线程数(滑动窗口写入)
├──────────────────┤
│ AuthoritySlot    │  黑白名单
├──────────────────┤
│ SystemSlot       │  系统自适应规则
├──────────────────┤
│ ParamFlowSlot    │  参数热点规则
├──────────────────┤
│ FlowSlot         │  普通流控规则(DefaultController / WarmUp / Throttling)
├──────────────────┤
│ DegradeSlot      │  熔断规则
└──────────────────┘
  │ 通过 → 业务方法执行 → entry.exit()
  ▼
```

各 Slot 通过 SPI 注册,顺序由 `@Spi(order = …)` 决定。

---

## 八、滑动时间窗口(LeapArray)

Sentinel 的 QPS 统计基于 `LeapArray<MetricBucket>`:

```
sampleCount = 2(默认),intervalInMs = 1000
                   ┌───────┬───────┐
windowArr  →      │ bkt 0 │ bkt 1 │  …  环形数组
                   └───┬───┴───┬───┘
                       ▼       ▼
                  start:0    start:500
                  pass:N     pass:M

写入逻辑:
  当前时间 now → calculateTimeIdx(now) → 取 bucket
    ├─ bucket.windowStart == 当前窗口起点 → 直接累加
    ├─ bucket.windowStart < 当前窗口起点 → CAS resetTo(now),清零
    └─ bucket.windowStart > now           → 时钟回拨,理论不可达
读取逻辑:遍历所有有效 bucket(过去 intervalInMs 内),求和。
```

**⚠ 原笔记缺失补充:** 原文只放了一张图,没提"滑动窗口由两层组成"——上层 `ArrayMetric` 用于秒级 QPS(`sampleCount=2,interval=1s`),下层 `OccupiableBucketLeapArray` 用于分钟级(`sampleCount=60,interval=60s`),两者目的不同,前者用于实时限流,后者用于 dashboard 展示。

---

## 九、机器心跳上下线(多文件流程)

```
┌─────────────────── Client (sentinel-transport) ──────────────────┐
│                                                                  │
│ SimpleHttpHeartbeatSender                                        │
│   schedule fixedRate 10s                                         │
│   └─> sendHeartbeat()                                            │
│        POST http://{dashboard}/registry/machine                  │
│             ?app=…&port=8720&version=…&v=…                       │
└──────────────────────────────┬───────────────────────────────────┘
                               │
                               ▼
┌─────────────────── Dashboard (sentinel-dashboard) ───────────────┐
│ MachineRegistryController#receiveHeartBeat                       │
│   └─> AppManagement.addMachine(MachineInfo)                      │
│        └─> AppInfo.addMachine() / 更新 lastHeartbeat             │
│                                                                  │
│ Dashboard 定时任务 healthCheck:                                  │
│   now - lastHeartbeat > 60s ⇒ healthy=false                      │
│   now - lastHeartbeat > 5min ⇒ dead=true(机器下线)              │
└──────────────────────────────────────────────────────────────────┘
```

---

## 十、关键纠错清单(汇总)

| # | 原笔记表述 | 正确表述 |
| --- | --- | --- |
| 1 | "排队等待对应漏桶算法" | 严格来说是**匀速排队(virtual queue)**,实现上接近漏桶但不等于漏桶 |
| 2 | "短路器三种状态" | **熔断器 / 断路器(Circuit Breaker)** |
| 3 | "WPS=6" | **QPS=6**(笔误) |
| 4 | "复制 nacos 目录到 rule 下" | 把 `test/java/.../rule/nacos/` 下的示例类**搬到 main 目录**,并改 `sentinel-datasource-nacos` 依赖的 `<scope>test</scope>` |
| 5 | "扫描 `/META_INF/Service/`" | `META-INF/services/`(中划线 + 复数) |
| 6 | "保证后面加载文件的优先级别计较高" | 代码意图是**让 JVM -D 参数覆盖文件配置**,JVM 参数优先级最高 |
| 7 | "添加 `sentinel-parameter-flow-control` 依赖" | 1.4 之后该模块已并入 `sentinel-core`,**客户端不必额外引入**;dashboard 仍单独依赖 |

---
