# Sentinel 知识点梳理

> 基于 `sentinel.md` 文档整理，源码版本：1.8.7

---

## 一、知识点总览

```
Sentinel
├── 1. 源码环境搭建
├── 2. Dashboard 控制台
│   ├── 首页（应用列表）
│   ├── 实时监控
│   ├── 簇点链路
│   ├── 流控规则（快速失败 / Warm Up / 排队等待 + 流控模式）
│   ├── 熔断规则（慢调用比例 / 异常数 / 异常比例）
│   ├── 热点规则
│   ├── 系统规则
│   ├── 授权规则（黑白名单）
│   ├── 集群流控（Token Server / Token Client）
│   ├── 机器列表（心跳上报）
│   └── 规则持久化（Nacos）
└── 3. Sentinel 核心机制
    ├── SPI 加载机制
    ├── 配置加载
    ├── 命令处理器注册与端口监听
    ├── 处理器链条（Slot Chain）
    └── 滑动时间窗口统计 QPS
```

---

## 二、源码环境搭建

- 官网：https://sentinelguard.io/zh-cn/index.html
- 仓库：https://github.com/alibaba/Sentinel
- 环境：maven 3.x、JDK 1.8、源码分支 1.8.7

构建步骤：

```shell
git clone https://github.com/alibaba/Sentinel.git
git branch 1.8.7 1.8.7
git checkout 1.8.7
mvn clean install -DskipTests
```

构建踩坑：
1. 注释掉 `maven-pmd-plugin` 插件
2. 注释掉 `sentinel-quarkus-adapter` 模块

---

## 三、Dashboard 控制台

### 3.1 首页（应用列表）

- 请求 URL：`GET http://localhost:8080/app/briefinfos.json`
- 返回当前注册的应用、所属机器、心跳时间、健康状态等

### 3.2 实时监控

- 每 **10 秒** 轮询一次
- URL：`/metric/queryTopResourceMetric.json?app=xxx&desc=true&pageIndex=1&pageSize=6`

### 3.3 簇点链路

流控规则 + 熔断规则 + 热点规则 + 授权规则 的综合视图。

### 3.4 流控规则

#### 3.4.1 三种限流效果

| 效果 | 实现类 | 算法 | 特点 |
| --- | --- | --- | --- |
| 快速失败 | `DefaultController` | 计数器 | 超出阈值立即拒绝 |
| Warm Up | `WarmUpController` | 令牌桶（带预热） | 冷启动保护，慢慢放量 |
| 排队等待 | `ThrottlingController` | 漏桶 | 严格匀速通过 |

**快速失败核心源码（DefaultController#canPass）**：

```java
public boolean canPass(Node node, int acquireCount, boolean prioritized) {
    int curCount = avgUsedTokens(node);
    if (curCount + acquireCount > count) {
        // 超阈值，支持优先级请求"借未来令牌"
        if (prioritized && grade == RuleConstant.FLOW_GRADE_QPS) {
            long waitInMs = node.tryOccupyNext(...);
            if (waitInMs < OccupyTimeoutProperty.getOccupyTimeout()) {
                sleep(waitInMs);
                throw new PriorityWaitException(waitInMs);
            }
        }
        return false; // 拒绝
    }
    return true;     // 通过
}
```

#### 3.4.2 流控模式

- **直接**：直接对当前资源限流
- **关联**：A 资源被压垮时，限流 B 资源（例如 `/write` QPS 过高，限流 `/read`）
- **链路**：按调用链入口区分限流（仅 `/test1` 触发某资源时限流，`/test2` 放行）

### 3.5 熔断规则

支持三种策略：**慢调用比例 / 异常数 / 异常比例**。

**断路器三种状态转换**：

```
                     失败率/慢调用比例达标
              ┌─────────────────────────────┐
              ▼                             │
       ┌────────────┐   时间窗口到期    ┌────────────┐
       │   OPEN     │ ───────────────▶  │  HALF_OPEN │
       │  （熔断）  │                   │ （探测请求）│
       └────────────┘                   └────────────┘
              ▲                                │
              │ 探测失败                       │ 探测成功
              └────────────────┐               ▼
                               │        ┌────────────┐
                               └────────│  CLOSED    │
                                        │  （正常）  │
                                        └────────────┘
```

实测现象：进入 OPEN 后每隔 2 秒发一次探测请求，失败则继续熔断。

### 3.6 热点规则

需引入依赖：

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-parameter-flow-control</artifactId>
</dependency>
```

特点：可对**指定参数值**单独设置阈值（如 VIP 用户 QPS=6，普通用户 QPS=2）。

### 3.7 系统规则

文档：https://sentinelguard.io/zh-cn/docs/system-adaptive-protection.html
按系统级指标（Load、CPU、RT、入口 QPS、线程数）做自适应保护。

### 3.8 授权规则

针对**调用来源**（origin）的黑白名单控制。

### 3.9 集群流控

官方文档：https://sentinelguard.io/zh-cn/docs/cluster-flow-control.html

包含 **Token Server 初始化** 与 **Token Client 初始化** 两条独立流程，详见后文执行流程图。

### 3.10 机器列表

- 客户端定期发送心跳到 Dashboard
- Dashboard 接收心跳，标记机器存活/下线
- 详见后文「心跳上报执行流程图」

### 3.11 规则持久化（Nacos）

文档：https://github.com/alibaba/Sentinel/wiki/动态规则扩展

#### Dashboard 改造步骤

1. 复制 `nacos` 目录到 `rule` 下，注释掉 `@Scope`
2. 修改 Nacos 注册地址
3. 更改 Controller 注入的实现类（改为 Nacos 版）
4. 取消页面相关注释
5. 在 Dashboard 添加流控规则 → 自动同步到 Nacos 配置中心

#### Client 改造步骤

1. 添加依赖：

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

2. 通过 SPI 注册 `InitFunc`：

```java
public class DataSourceInitFunc implements InitFunc {
    @Override
    public void init() throws Exception {
        ReadableDataSource<String, List<FlowRule>> ds = new NacosDataSource<>(
            "localhost:8848", "SENTINEL_GROUP",
            "xxx-flow-rules",
            source -> JSON.parseObject(source, new TypeReference<List<FlowRule>>() {}));
        FlowRuleManager.register2Property(ds.getProperty());
    }
}
```

3. 在 `META-INF/services/com.alibaba.csp.sentinel.init.InitFunc` 中登记类全限定名

---

## 四、Sentinel 核心机制

### 4.1 SPI 机制

- 借鉴 SpringBoot 自动装配思路，但实现的是 **Java SPI**
- 启动时扫描 `META-INF/services/` 下的接口实现，缓存到 `SpiLoader`
- 应用场景：`Slot` 链装配、`InitFunc` 初始化扩展、`ProcessorSlot` 等

### 4.2 配置加载

加载优先级（**从低到高**，后加载的覆盖前面）：

```
classpath:sentinel.properties  →  System env  →  System property
```

核心代码逻辑：

```java
// 1. 先从 -D 系统属性取文件名
// 2. 再从环境变量取
// 3. 兜底用默认文件 sentinel.properties
// 4. 最后再用 System.getProperties() 覆盖一遍 → JVM 参数优先级最高
```

### 4.3 命令处理器注册与端口监听

| 命令名称 | 处理器 | 作用 |
| --- | --- | --- |
| `setRules` | `ModifyRulesCommandHandler` | 修改规则 |
| `getRules` | `FetchActiveRuleCommandHandler` | 拉取规则 |
| `metric` | `SendMetricCommandHandler` | 上报指标 |
| `version` | `VersionCommandHandler` | 版本号 |

客户端启动一个 HTTP server（默认 8719 起，端口冲突自动 +1），Dashboard 通过 HTTP 调用这些命令。

### 4.4 Slot Chain（处理器链条）

通过 SPI 加载并按 `@SpiOrder` 顺序组装，每个 Slot 各司其职：

```
NodeSelectorSlot → ClusterBuilderSlot → LogSlot → StatisticSlot
   → AuthoritySlot → SystemSlot → ParamFlowSlot
   → FlowSlot → DegradeSlot
```

### 4.5 滑动时间窗口统计 QPS

- 通过 `LeapArray` + `WindowWrap` 实现
- 把 1 秒切分为 N 个小窗口，每次统计取窗口内已落桶的指标聚合
- 时间推进时旧窗口自动 reset、复用，避免分配压力

---

## 五、多文件调用执行流程图

### 5.1 客户端心跳上报流程

涉及：Client 端 `HeartbeatSender` → Dashboard 端 `MachineRegistryController`

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              Client（被监控应用）                          │
│                                                                          │
│  InitExecutor.doInit()                                                   │
│        │                                                                 │
│        ▼  (SPI 加载 InitFunc)                                            │
│  HeartbeatSenderInitFunc#init                                            │
│        │                                                                 │
│        ▼  schedule 每 10s 一次                                           │
│  SimpleHttpHeartbeatSender#sendHeartbeat                                 │
│        │   构造参数: app / appType / hostname / ip / port / version      │
│        ▼                                                                 │
│  HttpClient.GET  →  http://dashboard:8080/registry/machine               │
└──────────────────────────────┬───────────────────────────────────────────┘
                               │
                               ▼   HTTP
┌──────────────────────────────────────────────────────────────────────────┐
│                                Dashboard                                  │
│                                                                          │
│  MachineRegistryController#receiveHeartBeat                              │
│        │                                                                 │
│        ▼                                                                 │
│  AppManagement.addMachine(MachineInfo)                                   │
│        │                                                                 │
│        ▼                                                                 │
│  ConcurrentHashMap<appName, AppInfo>                                     │
│        │                                                                 │
│        ▼  前端轮询                                                       │
│  /app/briefinfos.json  →  返回机器列表 + healthy/dead 状态                │
└──────────────────────────────────────────────────────────────────────────┘

判活规则：lastHeartbeat 距今 >  阈值（默认 ~60s）→ dead = true
```

### 5.2 规则推送与持久化（Push 模式 + Nacos）

涉及：Dashboard 修改规则 → Nacos → Client 监听 → 本地 RuleManager

```
┌────────────────────┐  1. 用户点击「新增流控规则」
│   Dashboard 页面   │
└─────────┬──────────┘
          │ POST /v1/flow/rule
          ▼
┌────────────────────────────────────┐
│  FlowControllerV2 (改造后)         │
│   - 调 NacosDynamicRulePublisher   │ 2. 发布到 Nacos
│   - 调 SentinelApiClient           │ 3. (可选) 同步推送到 client 8719
└─────────┬──────────────────────────┘
          │ publishConfig(dataId, group, rulesJson)
          ▼
┌────────────────────────────────────┐
│           Nacos Server             │  dataId: {app}-flow-rules
│   持久化 JSON 格式规则              │  group : SENTINEL_GROUP
└─────────┬──────────────────────────┘
          │ 长轮询通知
          ▼
┌────────────────────────────────────────────────────────────┐
│                       Client                                │
│                                                            │
│  DataSourceInitFunc#init  (SPI: InitFunc)                  │
│        │                                                   │
│        ▼                                                   │
│  new NacosDataSource(addr, group, dataId, parser)          │
│        │                                                   │
│        ▼                                                   │
│  FlowRuleManager.register2Property(ds.getProperty())       │
│        │                                                   │
│        ▼  规则变更时 PropertyListener 触发                  │
│  FlowRuleManager#FlowPropertyListener.configUpdate         │
│        │                                                   │
│        ▼                                                   │
│  更新内存中的 flowRules（ConcurrentHashMap）                │
│        │                                                   │
│        ▼ 下次请求进入                                       │
│  FlowSlot#entry → checkFlow → DefaultController#canPass    │
└────────────────────────────────────────────────────────────┘
```

### 5.3 资源访问的 Slot Chain 调用链

涉及：`SphU.entry` → `CtSph` → `ProcessorSlotChain` → 各 Slot

```
应用代码: SphU.entry("resourceName")
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│ CtSph#entryWithPriority                                     │
│   - 获取/构建 ResourceWrapper                                │
│   - lookProcessChain(resource) → ProcessorSlotChain         │
│       └─ SlotChainProvider (SPI 加载 SlotChainBuilder)       │
└──────────────────────┬──────────────────────────────────────┘
                       │ chain.entry(context, resource, ...)
                       ▼
┌─────────────────────────────────────────────────────────────┐
│  ProcessorSlotChain (链表)                                  │
│                                                             │
│  ┌──────────────────┐                                       │
│  │ NodeSelectorSlot │  按 context 创建 DefaultNode (调用树) │
│  └────────┬─────────┘                                       │
│           ▼                                                 │
│  ┌────────────────────┐                                     │
│  │ ClusterBuilderSlot │  构建 ClusterNode（资源级聚合统计） │
│  └────────┬───────────┘                                     │
│           ▼                                                 │
│  ┌─────────┐    ┌────────────────┐                          │
│  │ LogSlot │ →  │ StatisticSlot  │  滑动窗口写入 QPS/RT/异常 │
│  └─────────┘    └────────┬───────┘                          │
│                          ▼                                  │
│  ┌────────────────┐   黑白名单校验                          │
│  │ AuthoritySlot  │                                         │
│  └────────┬───────┘                                         │
│           ▼                                                 │
│  ┌────────────┐   系统级保护（Load / CPU / RT）             │
│  │ SystemSlot │                                             │
│  └────────┬───┘                                             │
│           ▼                                                 │
│  ┌──────────────┐  热点参数限流                             │
│  │ ParamFlowSlot│                                           │
│  └────────┬─────┘                                           │
│           ▼                                                 │
│  ┌──────────┐  流控（quick fail / warm up / throttling）    │
│  │ FlowSlot │ → DefaultController / WarmUpController /      │
│  └────────┬─┘   ThrottlingController                        │
│           ▼                                                 │
│  ┌────────────┐  熔断（慢调用 / 异常数 / 异常比例）         │
│  │ DegradeSlot│                                             │
│  └────────────┘                                             │
└─────────────────────────────────────────────────────────────┘
       │ 任一 Slot 抛出 BlockException
       ▼
应用代码 catch(BlockException) → 走兜底/降级逻辑
```

### 5.4 集群流控 - Token Server 初始化

```
ClusterServer 启动
      │
      ▼
┌──────────────────────────────────┐
│ DefaultEmbeddedTokenServer /     │
│ DefaultTokenServer               │
└──────────────┬───────────────────┘
               │ start()
               ▼
┌──────────────────────────────────┐
│ NettyTransportServer             │
│  - 绑定端口（默认 18730）         │
│  - 注册 ChannelHandler           │
└──────────────┬───────────────────┘
               ▼
┌──────────────────────────────────┐
│ TokenService 接口                │
│  - requestToken(ruleId, count)   │
│  - 内部委托给 FlowRuleChecker     │
└──────────────┬───────────────────┘
               ▼
┌──────────────────────────────────┐
│ ClusterFlowRuleManager           │
│  - 维护 namespace → ruleId → Rule │
│  - 通过 DataSource 监听 Nacos 等  │
└──────────────────────────────────┘
```

### 5.5 集群流控 - Token Client 初始化

```
SPI: InitFunc
      │
      ▼
┌──────────────────────────────────┐
│ ClusterStateManager              │
│  setToClient() / setToServer()   │
└──────────────┬───────────────────┘
               ▼
┌──────────────────────────────────┐
│ DefaultClusterTokenClient        │
│  - 读取 ClusterClientConfig       │
│  - 读取 ClusterClientAssignConfig │
│    （server host / port）         │
└──────────────┬───────────────────┘
               ▼
┌──────────────────────────────────┐
│ NettyTransportClient.start()     │
│  - Netty 建立长连接               │
│  - 失败自动重连                   │
└──────────────┬───────────────────┘
               ▼
应用请求 → FlowSlot → FlowRuleChecker
   ├── 集群模式: TokenClient.requestToken → 远程 Server
   │       ├── 成功 OK / SHOULD_WAIT → 通过
   │       └── BLOCKED / FAIL → 降级本地或直接拒绝
   └── 单机模式: 本地 Controller.canPass
```

### 5.6 配置加载流程

```
SentinelConfigLoader#load
      │
      ▼  优先级从低到高，后加载覆盖前面
┌──────────────────────────────────────────┐
│ 1. 读取 -Dcsp.sentinel.config.file=xxx    │
│      ↓ 为空                              │
│ 2. 读取环境变量 CSP_SENTINEL_CONFIG_FILE   │
│      ↓ 为空                              │
│ 3. 默认 classpath:sentinel.properties     │
└──────────────┬───────────────────────────┘
               ▼
         ConfigUtil.loadProperties(fileName)
               │
               ▼
         properties.putAll(p)
               │
               ▼   再遍历 System.getProperties()
         JVM -D 参数最终覆盖 → 优先级最高
```

### 5.7 滑动时间窗口统计

```
StatisticSlot#entry
      │
      ▼
ClusterNode / DefaultNode#addPassRequest
      │
      ▼
ArrayMetric#addPass
      │
      ▼
LeapArray<MetricBucket>.currentWindow(now)
      │
      ├── 计算 idx = (now / windowLengthInMs) % array.length
      ├── 计算 windowStart = now - now % windowLengthInMs
      │
      ▼
   ┌──────────────────────────┐
   │ array[idx] == null?      │── yes → CAS 新建 WindowWrap
   │ window.start == start?   │── yes → 复用
   │ window.start <  start?   │── yes → reset 旧窗口并复用
   │ window.start >  start?   │── 不可能（时间倒退）
   └─────────────┬────────────┘
                 ▼
          MetricBucket.add(PASS, 1)

查询时:
  passQps() = Σ 有效窗口内 PASS 计数 / intervalInSec
```

---

## 六、关键类速查

| 类 | 模块 | 作用 |
| --- | --- | --- |
| `SphU` / `SphO` | core | 资源埋点入口 |
| `CtSph` | core | `SphU` 的实际实现，构建 Slot Chain |
| `ProcessorSlotChain` | core | Slot 责任链 |
| `DefaultController` | flow | 快速失败限流 |
| `WarmUpController` | flow | 冷启动限流 |
| `ThrottlingController` | flow | 排队等待（漏桶） |
| `FlowRuleManager` | flow | 流控规则管理 + 监听 |
| `DegradeRuleManager` | degrade | 熔断规则管理 |
| `LeapArray` / `MetricBucket` | statistic | 滑动窗口数据结构 |
| `HeartbeatSender` | transport | 客户端心跳上报 |
| `CommandCenter` | transport | 端口监听 & 命令分发 |
| `NacosDataSource` | datasource-nacos | 规则动态数据源 |
| `InitFunc` | core | SPI 初始化扩展点 |
| `ClusterTokenClient` | cluster-client | 集群流控客户端 |
| `EmbeddedClusterTokenServer` | cluster-server | 集群流控服务端 |
