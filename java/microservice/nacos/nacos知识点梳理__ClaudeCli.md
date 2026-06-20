# Nacos 源码核心知识点梳理

> 基于 `nacos笔记.md` (nacos 2.2.0) 整理。多文件/多组件调用的流程均附 ASCII 执行流程图。

---

## 目录

- [一、环境与启动](#一环境与启动)
- [二、服务发现模块](#二服务发现模块)
- [三、配置模块](#三配置模块)
- [四、核心机制](#四核心机制)
- [五、设计亮点与编码技巧](#五设计亮点与编码技巧)

---

## 一、环境与启动

### 1.1 集群环境

- 版本：nacos 2.2.0 / JDK 1.8+ / Maven 3.2+
- 构建命令：`mvn clean install -DskipTests -Drat.skip=true`（需注释 `apache-rat-plugin` 与 `maven-checkstyle-plugin`）
- 本地集群：3 节点（8848 / 8850 / 8852）

### 1.2 三大端口体系

| 端口偏移 | 端口示例 | 协议    | 作用                                       |
| -------- | -------- | ------- | ------------------------------------------ |
| +1000    | 9848     | gRPC    | **客户端 → 服务端**，Client 连接与请求     |
| +1001    | 9849     | gRPC    | **服务端 → 服务端**，集群间数据同步        |
| -1000    | 7848     | JRaft   | **服务端 → 服务端**，Raft 选举/心跳        |

### 1.3 启动流程 ASCII 图

```
                          ┌─────────────────────────┐
                          │   Nacos Server 启动      │
                          └────────────┬────────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              ▼                        ▼                        ▼
   ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
   │ JRaft 协议初始化 │    │  GrpcSdkServer   │    │ GrpcClusterServer│
   │  port -1000      │    │  port +1000      │    │  port +1001      │
   │  (7848)          │    │  (9848)          │    │  (9849)          │
   │                  │    │                  │    │                  │
   │  CP: 选举/心跳   │    │  Client→Server   │    │  Server→Server   │
   └──────────────────┘    └────────┬─────────┘    └────────┬─────────┘
                                    │                       │
                          注册两个 Acceptor       注册两个 Acceptor
                                    │                       │
                          ┌─────────┴─────────┐   ┌─────────┴─────────┐
                          ▼                   ▼   ▼                   ▼
                  CommonRequest        BiStreamRequest         (集群间同步)
                  Acceptor.request     Acceptor.requestBiStream
```

---

## 二、服务发现模块

### 2.1 领域模型

- **数据模型**：Namespace → Group → Service → Cluster → Instance
- **服务模型**：Service 聚合了多个 Cluster，每个 Cluster 含多个 Instance（临时/持久化）

### 2.2 服务注册流程（gRPC vs HTTP）

```
   ┌──────────────────┐                          ┌──────────────────┐
   │  Nacos Client    │                          │  Nacos Server    │
   └────────┬─────────┘                          └────────┬─────────┘
            │                                             │
            │  ┌─── gRPC 通道 (9848) ─────────────────────┤
            │  │   InstanceRequest                        │
            │  │   ──► grpcCommonRequestAcceptor.request  │
            │  │       ──► InstanceRequestHandler.handle  │
            │  │           ──► EphemeralClientOperationService
            │  │               .registerInstance()        │
            │  │                                          │
            │  └─── HTTP (8848)                           │
            │      POST /nacos/v1/ns/instance             │
            │      ──► InstanceController.register        │
            │          ──► InstanceOperator.register      │
            │                                             │
            ▼                                             ▼
    [本地缓存 RedoService]              [ClientManager + Service 索引]
        启动时定时重做                       发布 ClientChangedEvent
                                                  │
                                                  ▼
                                          NamingSubscriberServices
                                          推送给所有订阅者 (Push)
```

### 2.3 心跳检测

| 方向       | HTTP                                   | gRPC                                       |
| ---------- | -------------------------------------- | ------------------------------------------ |
| Client 端  | 定时 `/nacos/v1/ns/instance/beat`      | 构造 `HealthCheckRequest`                  |
| Server 端  | 更新 lastBeat 时间                     | 长连接，直接返回；通过断连感知            |

#### Server 端健康检查执行流

```
       ┌──────────────────────────────────────────┐
       │   定时任务 (ClientBeatCheckTask /        │
       │            HealthCheckProcessor)         │
       └────────────────────┬─────────────────────┘
                            │
              ┌─────────────┴──────────────┐
              ▼                            ▼
       ┌──────────────┐            ┌──────────────────┐
       │  临时实例     │            │  持久化实例       │
       │ (Ephemeral)  │            │ (Persistent)     │
       └──────┬───────┘            └────────┬─────────┘
              │                              │
       超时未心跳:                       主动探活:
       1.标记不健康                       TCP / HTTP / MySQL
       2.超时则下线                        Processor 探测
              │                              │
              ▼                              ▼
       发布 ClientDeregisterEvent     更新 healthy 状态 + CP 同步
```

### 2.4 服务下线（页面操作）

```
  Web UI 点击下线
      │
      ▼
  InstanceController.update
      │
      ▼
  通过 CP 协议(JRaft) 更新元数据 (metadata)
      │
      ▼
  集群内所有节点同步状态 → 推送订阅者
```

### 2.5 服务订阅（推拉结合）

```
   Client 启动
      │
      ├──► 立即拉取一次 (HTTP) / 订阅请求 (gRPC)
      │
      ├──► 定时拉取  (HTTP 每 10s)
      │
      └──► 接收 Server 推送 (gRPC 长连接 / UDP)
                │
                ▼
      ┌────────────────────────────┐
      │  Server 端事件链路:         │
      │  ServiceChangeEvent         │
      │   ──► NamingSubscriberServiceV2Impl
      │       ──► PushExecutor.doPush
      │           ──► RpcPushService / UdpPushService
      └────────────────────────────┘
```

### 2.6 服务查询

| 方式 | 入口                                                            |
| ---- | --------------------------------------------------------------- |
| HTTP | `CatalogController.instances()` → `serviceStorage.getData()`    |
| gRPC | `ServiceQueryRequestHandler.handle()` → `serviceStorage.getData()` |

---

## 三、配置模块

### 3.1 领域模型

`Namespace(tenant) → Group → DataId → Configuration`

### 3.2 配置发布（HTTP / SDK 殊途同归）

```
   ┌──────────────┐         ┌──────────────────┐
   │ HTTP POST    │         │ SDK ConfigPublish │
   │ /v1/cs/configs│        │ Request (gRPC)   │
   └──────┬───────┘         └────────┬─────────┘
          └────────────┬──────────────┘
                       ▼
            ConfigController.publishConfig
                       │
                       ▼
            ConfigOperationService.publishConfig
                       │
        ┌──────────────┼──────────────────────┐
        ▼              ▼                      ▼
   ┌─────────┐   ┌──────────┐          ┌──────────────┐
   │  写 DB   │   │ 写缓存    │          │ 写本地文件    │
   │ (MySQL/  │   │ (CacheItem)         │ DiskUtil.   │
   │ derby)   │   │          │          │ saveToDisk  │
   └─────────┘   └──────────┘          └──────────────┘
                       │
                       ▼
       发布 ConfigDataChangeEvent (异步)
                       │
                       ▼
     AsyncNotifyService → 集群其他节点
                       │
                       ▼
        各节点 LongPollingService 唤醒
                       │
                       ▼
            推送给订阅 Client (HTTP 长轮询 / gRPC 推送)
```

> 核心思想：**同一份数据同时写入 DB + 缓存 + 文件**，读取走文件以减压数据库。

### 3.3 配置订阅

#### HTTP 长轮询

```
   Client                                    Server
     │                                         │
     │  NacosContextRefresher.onApplicationEvent
     │  ──► configService.addListener           │
     │                                         │
     │  ClientWorker 启动长轮询线程             │
     │  POST /v1/cs/configs/listener (30s 挂起) │──►  LongPollingService
     │                                         │      .addLongPollingClient
     │                                         │            │
     │                                         │            ▼
     │                                         │  挂起 30s, 等待 ConfigDataChangeEvent
     │                                         │            │
     │  ◄─────── 配置变更, 立即返回 ────────────│  事件触发 → 提前响应
     │                                         │
     │  拉取最新内容 → 触发 Listener            │
     │  ──► Spring 属性刷新 (RefreshEvent)      │
     ▼                                         ▼
```

#### gRPC 订阅

```
   Client                                       Server
     │  ConfigChangeNotifyRequest 监听          │
     │                                          │
     │  ←──── 长连接推送 ConfigChange ──────────│  ConfigChangeNotifier
     │                                          │
     │  ConfigQueryRequest 拉最新               │
     │  ──────────────────────────────────────►│
     │  ConfigChangeListener.receiveConfigInfo  │
     ▼                                          ▼
```

### 3.4 配置查询（最终走文件）

```
HTTP : ConfigController.getConfig
gRPC : ConfigQueryRequestHandler.handle
                │
                ▼
       DiskUtil.targetFile(dataId, group, tenant)
                │
                ▼
        读取本地磁盘文件 → 返回
```

### 3.5 灰度发布

```
   新增灰度配置
       │
       ▼
   写入灰度缓存 + 标记灰度 IP
       │
       ▼
   推送阶段: 过滤 IP, 只推送给灰度 IP
       │
       ▼
   查询阶段: 灰度 IP 命中 → 返回灰度配置; 其他 IP → 返回正式配置

  停止灰度: 删除灰度标记 → 灰度 IP 下次拉取得到正式配置
  全量发布 = 用灰度内容修改正式配置 + 删除灰度
```

### 3.6 多版本管理 & 回滚

支持配置历史保存，回滚 = 取历史版本重新 publish。

---

## 四、核心机制

### 4.1 一致性协议

#### AP - 自研 Distro

```
   节点启动
      │
      ▼
   ┌──────────────────────────────┐
   │  ① 启动阶段                  │
   │     新节点从邻居拉取全量数据  │
   └──────────────────────────────┘
              │
              ▼
   ┌──────────────────────────────┐
   │  ② 数据校验(定时)             │
   │     节点广播 verify 自己负责的│
   │     数据 checksum             │
   └──────────────────────────────┘
              │
              ▼
   ┌──────────────────────────────┐
   │  ③ 写数据                    │
   │     根据 hash 路由到责任节点   │
   │     责任节点本地写 + 异步广播  │
   └──────────────────────────────┘
              │
              ▼
   ┌──────────────────────────────┐
   │  ④ 读数据                    │
   │     直接本地读 (最终一致)      │
   └──────────────────────────────┘
```

#### CP - JRaft（sofa-jraft 1.3.12）

- 用于持久化实例 / 元数据 / 服务上下线状态
- 关键点：**Leader 选举 → 日志复制(过半写入) → 日志提交 → 状态机应用**

### 4.2 寻址机制

```
                  ┌──────────────────────┐
                  │  LookupFactory       │  ← 工厂模式
                  │  .createLookUp()     │
                  └──────────┬───────────┘
                             │
            ┌────────────────┴────────────────┐
            ▼                                 ▼
   ┌───────────────────────┐      ┌──────────────────────────┐
   │ FileConfigMemberLookup │      │ AddressServerMemberLookup │
   │ (cluster.conf)         │      │ (HTTP 询问地址服务器)      │
   │                        │      │                          │
   │ JDK WatchService       │      │ 定时 HTTP GET 地址列表    │
   │ 监听文件变化            │      │                          │
   │ ──► FileWatcher.onChange │     │ ──► memberManager        │
   │      ──► memberManager   │     │      .memberChange()     │
   │       .memberChange()    │     │                          │
   └───────────────────────┘      └──────────────────────────┘
                  │                                 │
                  └────────────────┬────────────────┘
                                   ▼
                       AbstractMemberLookup.afterLookup
                                   │
                                   ▼
                       ServerMemberManager 更新内存节点表
```

> 文件方式缺点：手工运维成本高，扩缩容需逐机修改 cluster.conf。

### 4.3 事件机制（观察者模式）

```
   Publisher                NotifyCenter                 Subscriber
       │                        │                            │
       │ publishEvent ────────► │                            │
       │                        │ 路由到对应 EventPublisher   │
       │                        │ (按事件类型)                │
       │                        │     │                       │
       │                        │     ▼                       │
       │                        │  内部 BlockingQueue         │
       │                        │     │                       │
       │                        │     ▼                       │
       │                        │  独立线程消费 ─────────────► onEvent()
       │                        │                            │
```

特性：**异步、解耦、高性能**，是 Nacos 主要扩展点。

### 4.4 网络连接（gRPC 长连接管理）

#### 通道建立 + 断线重连

```
   Client                                       Server
     │  RpcClient.start()                        │
     │  ──► connectToServer ──────────────────► │  GrpcServerInterceptor
     │       (gRPC BiStream + ServerCheckRequest)│   ──► ConnectionManager
     │                                           │       .register(connId)
     │                                           │
     │   [ServerListFactory 选下一台]            │
     │                                           │
     │   定时健康检查 (HealthCheckRequest)        │
     │  ──────────────────────────────────────► │
     │                                           │
     │  ✗ 心跳失败:                              │  ✗ 检测断链:
     │   ──► switchServerAsync()                 │   ──► ConnectionManager
     │       重连下一台                           │       .unregister(connId)
     │       (sleep retryTurns*100ms,            │       清空内存数据 + close
     │        最大 5s)                            │
     ▼                                           ▼
```

### 4.5 高可用机制

- **同城容灾**：多机房部署 + 寻址服务器
- **多级数据容灾**：
  - Server 全量缓存（DB 挂只影响写）
  - SDK 本地文件缓存（Server 全挂仍能调用）

### 4.6 插件机制（SPI）

```
   插件仓库: nacos-plugin
       │
       ▼
   Client / Server 端均通过 NacosServiceLoader 加载
       │
       ▼
   JDK SPI: /META-INF/services/<接口全限定名>
       │
       ▼
   排序 (按 order) + 链式调用
       │
       ▼
   典型应用: 鉴权、加密、追踪
```

### 4.7 回调机制（监听器模式）

- 服务模块、配置模块均通过 Listener 接口让用户感知变更
- 配置回调 → Spring `RefreshEvent` → 属性刷新（参见 `NacosContextRefresher`）

### 4.8 数据存储抽象

`com.alibaba.nacos.core.storage.kv.KvStorage` 统一接口，实现：

- **File** （磁盘文件）
- **Memory** （内存 Map）
- **RocksDB** （持久化 KV）

API：`get / batchGet / put / batchPut / delete / batchDelete / doSnapshot / snapshotLoad / allKeys / shutdown`

---

## 五、设计亮点与编码技巧

| 编号 | 亮点                  | 关键点                                                      |
| ---- | --------------------- | ----------------------------------------------------------- |
| 1    | 跳跃表                | `ConcurrentSkipListMap`，有序并发                           |
| 2    | Nacos SPI             | 封装 JDK SPI，加载 `/META-INF/services/`                     |
| 3    | 拦截器链              | `AbstractNamingInterceptorChain`，SPI 加载 + order 排序     |
| 4    | UDP 可靠推送          | `UdpConnector`，ACK 机制                                    |
| 5    | 简单读写锁            | `SimpleReadWriteLock`：status 0=无锁 / 负=写 / 正=读次数    |
| 6    | 配置加载顺序          | `PROPERTIES → JVM → ENV → DEFAULT_SETTING`                  |
| 7    | 过滤器链              | `VirtualFilterChain`，递归 doFilter                         |
| 8    | 客户端限流            | `Limiter.isLimit()`，请求级 hash 限流                       |
| 9    | 失败重试间隔          | 0s → 2s → 4s （`tryTimes * 2`）                            |
| 10   | CountDownLatch        | `peers.majorityCount()` 等待过半响应（CP 写日志）            |
| 11   | 生命周期钩子          | 监听 + 策略模式（`NacosApplicationListener`）                |
| 12   | Nacos SPI 机制        | `NacosServiceLoader.load(...)`                              |
| 13   | 内存缓存              | 读热路径全部走内存                                          |
| 14   | 文件监听              | `WatchFileCenter.registerWatcher(...)`                      |
| 15   | Builder 模式          | `Member.builder().ip(...).port(...).build()`                |
| 16   | 工厂模式              | `LookupFactory.createLookUp()` + `LookupType` 枚举          |
| 17   | 模板方法 + 策略       | `AbstractMemberLookup.doStart/doDestroy` 抽象               |
| 18   | 线程池集中化          | `GlobalExecutor` 统一管理 sdkRpc / clusterRpc / distro      |
| 19   | JVM Shutdown Hook     | `Runtime.getRuntime().addShutdownHook(...)`                 |
| 20   | 配置优先级            | 参考亮点 6                                                  |
| 21   | 延时 sleep            | `Math.min(retryTurns+1, 50) * 100L`，渐进退避               |
| 22   | 策略模式：请求处理器  | 一种请求一个 Handler，按 RequestType 路由                   |
| 23   | 模板方法 + 过滤器     | `handleRequest` 先跑 filter chain, 再 `handle`              |
| 24   | 延时任务缓冲          | 批量合并短时间内重复事件，降低下游压力                      |

---

## 附：核心调用链速查

```
服务注册 (gRPC):
  Client.NamingGrpcClientProxy
    └─ InstanceRequest
       └─ Server.GrpcCommonRequestAcceptor
          └─ InstanceRequestHandler
             └─ EphemeralClientOperationServiceImpl.registerInstance
                └─ ClientManager.getClient + Service 索引更新
                   └─ NotifyCenter.publish(ClientChangedEvent)
                      └─ NamingSubscriberServiceV2Impl 推送订阅者

配置发布 (HTTP):
  ConfigController.publishConfig
    └─ ConfigOperationService.publishConfig
       ├─ PersistService.insertOrUpdate (DB)
       ├─ CacheItem 更新 (内存)
       ├─ DiskUtil.saveToDisk (文件)
       └─ NotifyCenter.publish(ConfigDataChangeEvent)
          └─ AsyncNotifyService.executeAsyncInvokeTask (集群同步)
             └─ LongPollingService / RpcConfigChangeNotifier (推送客户端)

集群寻址:
  LookupFactory.createLookUp
    └─ FileConfigMemberLookup / AddressServerMemberLookup
       └─ AbstractMemberLookup.afterLookup
          └─ ServerMemberManager.memberChange
             └─ NotifyCenter.publish(MembersChangeEvent)
```
