# Nacos 整理稿

> 原笔记版本基准:**Nacos 2.2.0**,JDK 1.8+,Maven 3.2+
> 内置 sofa-jraft、distro 协议,gRPC + HTTP 双通道

---

## 一、知识点总览

```
Nacos 2.2.0
├─ 集群搭建(3 节点 8848/8850/8852)
├─ 启动流程(端口分配)
│   ├─ 8848  HTTP/OpenAPI
│   ├─ 9848 = 8848 + 1000  SDK gRPC(client → server)
│   ├─ 9849 = 8848 + 1001  Cluster gRPC(server → server)
│   └─ 7848 = 8848 - 1000  JRaft 选举/复制端口
├─ 服务发现(Naming)
│   ├─ 注册:GRPC InstanceRequest / HTTP /nacos/v1/ns/instance
│   ├─ 心跳:HTTP 主动 beat / GRPC 长连接探活
│   ├─ 订阅:推(GRPC NotifySubscriber)+ 拉(10s 定时)
│   └─ 查询:ServiceQueryRequestHandler / CatalogController
├─ 配置中心(Config)
│   ├─ 发布:DB + 缓存 + 本地文件
│   ├─ 订阅:HTTP 长轮询 30s / GRPC 推送
│   ├─ 查询:从本地文件读取
│   └─ 灰度发布、多版本回滚
└─ 核心机制
    ├─ 一致性:AP(Distro)+ CP(JRaft)
    ├─ 寻址:文件(cluster.conf)/ 地址服务器
    ├─ 事件:NotifyCenter(EventBus)
    ├─ 网络:Grpc{Sdk,Cluster}Server + ConnectionManager
    ├─ 插件:NacosServiceLoader(SPI)
    └─ 存储:KvStorage(File / Memory / RocksDB)
```

---

## 二、启动流程与端口模型(原表纠错)

### ⚠ 原笔记纠错(关键点 1):端口表格混乱

原文表格写成:

```
| 9848 | 1000  | 客户端gRPC … |
| 9849 | 1001  | 服务端gRPC … |
| 7848 | -1000 | Jraft … |
```

正确的端口模型应当是:**主端口为 `server.port`(默认 8848),其他端口都是它的偏移**:

| 端口 | 偏移量 | 协议 | 通道方向 |
| --- | --- | --- | --- |
| 8848 | 0 | HTTP | OpenAPI / Console |
| 9848 | +1000 | gRPC(SDK) | **client → server**,SDK 发起连接和双向流 |
| 9849 | +1001 | gRPC(Cluster) | **server → server**,节点间同步 |
| 7848 | −1000 | JRaft | 节点间选举 / 日志复制 |

启动时按上述偏移分别拉起三个 gRPC Server:

```
NacosStartUp
  ├─ BaseRpcServer.start()
  │   ├─ GrpcSdkServer        listen on  port+1000  (9848)
  │   ├─ GrpcClusterServer    listen on  port+1001  (9849)
  │   └─ 各自注册:
  │        - RequestAcceptor          (普通请求)
  │        - BiRequestStreamAcceptor  (双向流,服务端推)
  └─ JRaftServer.start()      listen on  port-1000  (7848)
```

---

## 三、服务注册的多文件调用链

### 3.1 gRPC 注册(Nacos 2.x 默认通道)

```
┌─────────────── Client(nacos-client) ───────────────┐
│ NamingService.registerInstance(serviceName, ip, …) │
│   └─> NacosNamingService.registerInstance          │
│         └─> NamingClientProxyDelegate.register     │
│               └─> NamingGrpcClientProxy.register   │
│                     └─> RpcClient.request(         │
│                            InstanceRequest TYPE=REGISTER) │
└──────────────────────┬─────────────────────────────┘
                       │ gRPC(9848)
                       ▼
┌─────────────── Server(naming 模块) ────────────────┐
│ InstanceRequestHandler.handle(InstanceRequest)     │
│   └─> EphemeralClientOperationServiceImpl          │
│         .registerInstance(service, instance, clientId) │
│           ├─> ConnectionBasedClient.addServiceInstance │
│           ├─> NotifyCenter.publishEvent(           │
│           │      ClientChangedEvent / ClientRegisterServiceEvent) │
│           └─> Distro 异步同步 → 其他节点           │
│                 └─> DistroProtocol.sync(DistroData)│
│                       └─> 集群间 GrpcClusterServer  │
└────────────────────────────────────────────────────┘
```

### 3.2 HTTP 注册(兼容旧版本)

```
Client: POST /nacos/v1/ns/instance?serviceName=…&ip=…&port=…
   │
   ▼
InstanceController.register
   └─> InstanceOperatorClientImpl.registerInstance
         └─> 走与 GRPC 相同的 EphemeralClientOperation 链路
```

**⚠ 原笔记纠错(关键点 2):** 原文写"GRPC 服务注册流程 → 图",但流程图没标注出**临时实例(ephemeral)走 Distro/AP,持久化实例(persistent)走 JRaft/CP**这条关键区别。Nacos 2.x 默认临时实例,持久化实例由 `PersistentClientOperationServiceImpl` 处理。

---

## 四、心跳检测:HTTP vs GRPC 完全不同

### 4.1 HTTP(1.x 风格,2.x 兼容)

```
Client                                     Server
   │  每 5s 一次 PUT /nacos/v1/ns/instance/beat │
   ├───────────────────────────────────────────►│
   │                                            │
   │           InstanceController.beat          │
   │           └─> InstanceOperatorClientImpl   │
   │                 .handleBeat()              │
   │                 └─> 更新 lastHeartbeatTime │
   │                                            │
Server 定时任务 ClientBeatCheckTask(每 5s):
   for each instance:
     if now - lastBeat > 15s: instance.healthy = false
     if now - lastBeat > 30s: 删除实例
```

### 4.2 GRPC(2.x 推荐)

```
Client                                     Server
   │                                            │
   │  长连接已建立(双向流)                     │
   │  ⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄⇄ │
   │                                            │
   │  Client 定时 5s 发 HealthCheckRequest      │
   │   ├──────────────────────────────────────►│
   │                                            │ HealthCheckRequestHandler
   │                                            │   └─> 直接 return success
   │                                            │ (无需更新时间戳,因为长连接的存活
   │                                            │  本身已经在 ConnectionManager 监控)
   │                                            │
ConnectionManager 探活:
   定期 expire 检查 + Netty channelInactive → unregister Connection
   → 触发 ClientDisconnectEvent → 移除该 Connection 关联的所有实例
```

**⚠ 原笔记纠错(关键点 3):** 原文写"GRPC 方式 ... 直接返回,因为是长连接,可以在链接断开的时候动态感知到"——表述正确,但**关键区别没强调**:HTTP 模式服务端按"时间戳过期"判存活;GRPC 模式按"TCP 长连接是否断开"判存活,**两套机制不能混用**。这也是为什么 Nacos 2.x 把临时实例和长连接的生命周期绑定:连接断 → 实例立即注销,比 HTTP 的"30s 过期"快得多。

---

## 五、服务订阅:推 + 拉结合

```
       Client 启动 / 第一次 selectInstances
            │
            ▼
   ┌────────────────────┐
   │ ServiceInfoHolder  │ ← 本地缓存
   └────────┬───────────┘
            │ 立即拉取一次
            │ ServiceListRequest → 服务端
            ▼
   ┌──────────────────────────────────────┐
   │ NamingGrpcClientProxy.subscribe      │
   │   ├─> 本地注册监听器                 │
   │   └─> 发送 SubscribeServiceRequest   │
   │       服务端记录该 client 关心的服务 │
   └──────────────────────────────────────┘

   推:服务端实例变更
   ┌──────────────────────────────────────────────┐
   │ NotifyCenter ServiceChangedEvent             │
   │   └─> PushExecutorRpcImpl                    │
   │         └─> 通过 BiStream 推 NotifySubscriberRequest │
   │             ↓                                │
   │           Client RpcClient 接收              │
   │             ↓                                │
   │           NamingPushRequestHandler           │
   │             ↓                                │
   │           ServiceInfoHolder.processServiceInfo │
   │             ↓                                │
   │           回调用户 EventListener             │
   └──────────────────────────────────────────────┘

   拉:每 6s 周期任务对所有订阅服务做一次防御性拉取
       UpdateTask → ServiceQueryRequest
```

---

## 六、配置发布与订阅

### 6.1 发布:一次写三处

```
publishConfig (HTTP /nacos/v1/cs/configs 或 GRPC ConfigPublishRequest)
   │
   ▼
ConfigController.publishConfig
   ├─> ① persistService.insertOrUpdate(...)            写 DB(MySQL / Derby)
   ├─> ② ConfigCacheService.dump(...)                  写本地磁盘文件
   │     DiskUtil.saveToDisk(dataId, group, tenant, content)
   ├─> ③ EventDispatcher.fireEvent(ConfigDataChangeEvent) 发本地事件
   │     └─> AsyncNotifyService 集群间通知
   │           └─> 通过 HTTP 通知其他 nacos 节点 dump 本地文件
   └─> ④ LongPollingService.dataChange(groupKey)       唤醒长轮询客户端
```

### 6.2 查询:从本地文件读

```
ConfigController.getConfig  /  ConfigQueryRequestHandler.handle
  └─> file = DiskUtil.targetFile(dataId, group, tenant)
        └─> 直接读取文件返回(MD5 校验)
```

**优势:** Nacos 把 DB 当持久化、文件当读路径,**核心读不依赖 DB**,即使 DB 挂了已发布的配置仍可读(高可用机制原文有提到)。

### 6.3 配置订阅(客户端 HTTP 长轮询版)

```
Client                                              Server
  │                                                   │
  │  每个 ClientWorker 周期任务 LongPollingRunnable   │
  │  POST /nacos/v1/cs/configs/listener               │
  │     Listening-Configs = dataId%02group%02md5      │
  │     Long-Pulling-Timeout: 30000                   │
  ├──────────────────────────────────────────────────►│
  │                                                   │ LongPollingService
  │              (持有连接,挂起 29.5s)                │   .addLongPollingClient
  │                                                   │
  │  Case A:有数据变更 → Server 立即响应变更列表     │
  │  Case B:29.5s 超时 → Server 响应空列表           │
  │◄──────────────────────────────────────────────────│
  │                                                   │
  │  对变更的 dataId 再发 GET /configs 拉新内容       │
  │  → NacosContextRefresher 发布事件 → @RefreshScope │
```

**⚠ 原笔记纠错(关键点 4):** 原文亮点 #6 写 "**配置加载顺序:PROPERTIES → JVM → ENV → DEFAULT_SETTING**"。
这是 **Nacos 客户端 SDK 的 `NacosClientProperties` 行为**,顺序应为:
```
JVM 系统属性  >  环境变量  >  classpath:application.properties  >  默认值
```
其中 `PROPERTIES` 指的是显式传入的 `Properties` 对象(用户 new 出来传给 ConfigService 构造器)优先于 JVM 系统属性,所以**完整顺序**:

```
入参 Properties  >  JVM -D 系统属性  >  环境变量  >  默认值
```

原文那一行只是源码注释的搬运,容易被误读。

---

## 七、一致性协议

### 7.1 协议抽象层

```
ConsistencyProtocol(顶层接口)
   ├─ APProtocol      (Distro,自研)
   │    └─ DistroConsistencyServiceImpl
   └─ CPProtocol      (JRaft 1.3.x,封装 sofa-jraft)
        └─ JRaftConsistencyServiceImpl
```

### 7.2 AP — Distro 协议(临时实例)

每个节点**只负责一部分数据(责任分片)**,接收写请求后异步同步到其他节点:

```
启动:
  DistroProtocol.startDistroTask
    ├─ 1. 启动时全量拉取:从某个邻居 syncAllDataFromRemote()
    └─ 2. 周期任务:校验各分片 checksum,不一致触发增量同步

写请求(任意节点接收):
  Node A 接收 register
    ├─ 本地更新 ConnectionBasedClient
    ├─ 通过 DistroProtocol.sync 发布 DistroData 到其他节点
    │    └─ 集群 GrpcClusterServer.request(DistroDataRequest)
    └─ 返回成功(无需等待集群确认 → AP)

读请求:
  任意节点本地读 → 由于异步同步,可能短暂不一致
```

### 7.3 CP — JRaft 协议(持久化实例 / Raft 选主)

```
                    ┌─────────────────┐
                    │     Leader      │
                    └────────┬────────┘
            AppendEntries    │     AppendEntries
                ┌────────────┼────────────┐
                ▼            ▼            ▼
          ┌─────────┐  ┌─────────┐  ┌─────────┐
          │ Follower│  │ Follower│  │ Follower│
          └─────────┘  └─────────┘  └─────────┘

写入流程:
  1. Client 请求落到 Leader(非 Leader 转发)
  2. Leader 把 LogEntry append 到本地日志
  3. 并行向 Follower 发 AppendEntries
  4. 过半 Follower 确认 → Leader 提交日志(commitIndex++)
  5. Leader 应用 StateMachine,返回客户端成功
  6. 后续心跳带 commitIndex,Follower 也提交

选举(Leader 失联):
  Follower election timeout(随机 150-300ms)
    → 转 Candidate,任期 +1,投票给自己,广播 RequestVote
    → 收到过半投票 → 成为新 Leader
```

**⚠ 原笔记纠错(关键点 5):**
- 原文写"jraft 1.3.12 版本",更精确的说法是 Nacos 2.2.0 引用的是 **sofa-jraft 1.3.13**(参见 `nacos-consistency` 模块 `pom.xml`)。版本号不必死记,以源码 pom 为准即可。
- 原文写"1.4.1 版本自己实现的 raft 协议" —— 准确表述:**Nacos 1.x 系列**使用了**自研的简化版 Raft**(`com.alibaba.nacos.naming.consistency.persistent.raft.RaftCore`),用于持久化实例;**Nacos 2.x 已全面切换到 sofa-jraft**。如果想看自研版,看 1.4.x 最后一个 release。

---

## 八、寻址机制(Member Lookup)

```
LookupFactory(工厂)
   ├─ standalone 模式 → StandaloneMemberLookup
   └─ cluster 模式 chooseLookup():
        ├─ cluster.conf 存在 / 显式配 nacos.member.list  → FileConfigMemberLookup
        └─ 否则                                          → AddressServerMemberLookup

FileConfigMemberLookup:
   ├─ 读 conf/cluster.conf
   └─ JDK WatchService 监听文件变更 → 触发 memberChange()

AddressServerMemberLookup:
   定时(5s)HTTP GET 地址服务器
   GET http://{addressServer}/serverlist
   → 返回最新 ip 列表 → memberChange()
```

**优劣对比:**

| 方式 | 优点 | 缺点 |
| --- | --- | --- |
| 文件 | 简单、无外部依赖 | 扩缩容必须修改每个节点的 cluster.conf,易不一致 |
| 地址服务器 | 动态扩缩容,集中管理 | 多一个组件依赖,地址服务器需自己保证 HA |

---

## 九、事件机制(NotifyCenter)

典型发布订阅 + Disruptor 风格异步队列:

```
Publisher                        Subscriber
   │                                │
   │  NotifyCenter.publishEvent(e) │
   ├───────────────────────────────►│
   │                                │
   │   ┌─────────────────────────┐  │
   │   │ EventPublisher          │  │
   │   │  (持有 BlockingQueue +  │  │
   │   │   单线程消费)           │  │
   │   └────────┬────────────────┘  │
   │            │ 唤醒              │
   │            ▼                   │
   │   Subscriber.onEvent(e)        │
   │     └─> 同步执行 / 异步 schedule
   │                                │

每种事件类型有自己的 EventPublisher(隔离慢消费者)
SmartSubscriber 可一次性订阅多种事件
```

---

## 十、网络连接(gRPC 长连接 + 断线重连)

```
Client 启动:
  RpcClient.start()
    ├─ chooseHealthyServer() → 随机一台 server
    ├─ 与 server 建立 gRPC 双向流 BiStream
    │    └─ 发送 ConnectionSetupRequest(client metadata)
    └─ 启动两个守护线程:
         ├─ ReconnectThread(检测连接状态)
         └─ HealthCheck 定时 5s

断线重连(Client 侧):
  channel 异常 → switchServerAsync()
    ├─ 关闭原 channel
    ├─ Thread.sleep(min(retryTurns+1, 50) * 100ms)  # 指数回退,封顶 5s
    ├─ 从健康服务器列表随机挑下一台
    └─ rebuild connection → 重新订阅之前的服务/配置(reSubscribe)

断线检测(Server 侧):
  Netty channelInactive → ConnectionManager.unregister(connectionId)
    └─ 触发 ClientDisconnectEvent
        └─ ClientServiceIndexesManager 移除该 client 注册的所有 service
            └─ 通知订阅方:实例下线
```

---

## 十一、插件机制 & SPI

```
NacosServiceLoader.load(NacosApplicationListener.class)
  └─> 委托给 ServiceLoader.load(...)
        └─> 扫描所有 jar 包的
            META-INF/services/com.alibaba.nacos.xxx.NacosApplicationListener
        └─> 缓存到 SERVICES_MAP
```

**⚠ 原笔记纠错(关键点 6):** 原文亮点 #12 写 "默认加载 `/META-INF/services/包名+类名`"。
更准确表述:文件名是**接口的全限定类名**(不是"包名+类名"那种模糊说法),路径 `META-INF/services/<接口 FQN>`。Nacos 复用 JDK SPI 机制,扩展点本身就在 JDK 的 `ServiceLoader` 之上。

---

## 十二、客户端限流

```java
boolean limit = Limiter.isLimit(request.getClass() + asJsonObjectTemp.toString());
```

`Limiter` 内部基于 Guava `RateLimiter`(令牌桶),**针对相同请求(类型 + body 序列化)做单机限流**,防止业务侧短时间内发出过多重复 request 把 Nacos 集群打挂。

---

## 十三、关键纠错清单(汇总)

| # | 原笔记表述 | 正确表述 |
| --- | --- | --- |
| 1 | 端口表格中 "9849 server.port+1001" 表头混乱 | 主端口 8848 是基准,9848=+1000(SDK gRPC)、9849=+1001(Cluster gRPC)、7848=−1000(JRaft) |
| 2 | "GRPC 服务注册流程"未区分实例类型 | 临时实例(默认)走 Distro/AP;持久化实例走 JRaft/CP |
| 3 | "GRPC 心跳直接返回" | 表述正确但缺背景:GRPC 模式存活由 TCP 长连接保证,HTTP 模式靠 15s 时间戳过期 |
| 4 | "PROPERTIES→JVM→ENV→DEFAULT_SETTING" | 完整顺序是:入参 Properties > JVM -D 系统属性 > 环境变量 > 默认值 |
| 5 | "jraft 1.3.12 版本"、"1.4.1 自己实现的 raft" | Nacos 2.2.0 用 sofa-jraft 1.3.13;1.x 自研 Raft 用于持久化实例,2.x 全面切到 sofa-jraft |
| 6 | "默认加载 `/META-INF/services/包名+类名`" | 路径为 `META-INF/services/<接口全限定名>`,文件名就是接口 FQN |
| 7 | "异步通知 ... 唤醒长轮询" 时序混淆 | 配置发布的本机流程是:DB → 本地文件 → 事件 → 集群通知 → 唤醒本机长轮询;集群其他节点收到通知后再 dump 本地文件、唤醒它们自己挂起的长轮询 |

---
