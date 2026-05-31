# Flink 基础 整理稿

> 原笔记版本基准:**Flink 1.15.2**(Standalone / Flink on YARN)
> 涵盖:运行时架构、三种部署模式、StreamExecutionEnvironment、FlinkSQL、时间语义、窗口、Watermark、Checkpoint。

---

## 一、知识点总览

```
Flink 1.15.2
├─ 部署模式
│   ├─ Standalone
│   └─ On YARN
│       ├─ Session 模式      集群常驻,多 Job 共享 JM
│       ├─ Per-job 模式      每个 Job 一个 JM(1.15 已 deprecated)
│       └─ Application 模式  main() 在 JM 上执行(推荐)
├─ 运行时架构
│   ├─ JobManager
│   │    ├─ Dispatcher       接收 Job 提交,拉起 JobMaster
│   │    ├─ ResourceManager  申请/释放 Slot
│   │    └─ JobMaster        StreamGraph → JobGraph → ExecutionGraph,调度 Task
│   ├─ TaskManager           执行 Task,管理 Slot/网络/内存
│   └─ Client                生成 StreamGraph/JobGraph(Application 模式除外)
├─ API 分层
│   ├─ SQL / Table API      声明式
│   ├─ DataStream API       流式核心
│   └─ ProcessFunction      底层(定时器、状态)
├─ 时间语义:Event / Ingestion / Processing
├─ 窗口:Tumble / Hop / Session / Cumulate / Over
├─ Watermark:乱序事件的进度标记
└─ 容错
    ├─ Checkpoint(Chandy-Lamport 异步屏障)
    ├─ Restart Strategy (none/fixed-delay/failure-rate/exponential-delay)
    └─ StateBackend (HashMap / RocksDB) + CheckpointStorage (Jobmanager / FileSystem)
```

---

## 二、运行时架构与三种 YARN 部署模式

### 2.1 JobManager / TaskManager 协作

```
┌──────────── Client ────────────┐
│ env.execute()                  │
│   ├─ StreamGraph (用户算子图)   │
│   └─ JobGraph    (chain 后)     │
└─────────────┬──────────────────┘
              │ submitJob
              ▼
┌──────────── JobManager ────────────────────────────────────┐
│ Dispatcher.submitJob(JobGraph)                             │
│   └─> 启动 JobMaster                                       │
│        ├─ JobGraph → ExecutionGraph (并行化)               │
│        ├─ 向 ResourceManager 申请 Slot                     │
│        │     └─> ResourceManager 向 TaskExecutor 要 Slot   │
│        └─ deploy(ExecutionVertex) 到 TaskManager           │
└─────────────┬──────────────────────────────────────────────┘
              │ submitTask (RPC)
              ▼
┌──────────── TaskManager ───────────────────────────────────┐
│ TaskExecutor.submitTask()                                  │
│   └─> new Task(...).run()                                  │
│        └─> StreamTask.invoke()                             │
│             └─> OperatorChain 链式调用 processElement(...) │
└────────────────────────────────────────────────────────────┘
```

### 2.2 三种 YARN 部署模式对比

| 模式 | main() 执行位置 | JM 生命周期 | 集群隔离 | 适用场景 |
| --- | --- | --- | --- | --- |
| Session | Client | 集群常驻 | 多 Job 共享 | 大量小作业、交互式查询 |
| Per-job | Client | 一 Job 一 JM | 隔离 | 已 deprecated(1.15+ 建议 Application) |
| Application | **JobManager** | 一 Job 一 JM | 隔离 | 生产首选,Client 负担小 |

**⚠ 原笔记纠错(关键点 1):** 原文把 Per-job 当成"生产标配",实际从 Flink 1.15 起 **Per-job 已 `@Deprecated`**(`bin/flink run -m yarn-cluster ...`),官方推荐 **Application 模式**(`flink run-application -t yarn-application`)。两者最大差别:Application 模式 `main()` 方法在 JobManager 上执行,Client 只负责提交一个 jar,不需要在本地构建 StreamGraph,Client 内存压力小、网络流量小。

### 2.3 Application 模式提交链路

```
Client                       YARN ResourceManager        NodeManager
  │ flink run-application       │                           │
  ├──────────────────────────►  │                           │
  │  (上传用户 jar + flink dist) │                           │
  │                             │ allocateContainer         │
  │                             ├─────────────────────────► │
  │                             │                           │ 启动 AM (JobManager 容器)
  │                             │                           │   ├─ 在 JM 上执行 user main()
  │                             │                           │   ├─ env.execute() → JobGraph
  │                             │                           │   └─ Dispatcher.submitJob
  │                             │                           │
  │                             │  申请 TM 容器(动态)       │
  │                             │ ◄───────────────────────► │
  │                             │                           │ 启动 TaskManager 容器
  │                             │                           │   注册回 JM,接收 Task
```

---

## 三、StreamExecutionEnvironment 与表

```
StreamExecutionEnvironment.getExecutionEnvironment()
        │
        ├─ 本地启动     → LocalStreamEnvironment
        ├─ 提交到远端    → RemoteStreamEnvironment
        └─ 集群内执行    → ContextEnvironment(命令行/Web 提交时由 ExecutionEnvironmentFactory 注入)

StreamTableEnvironment.create(env)
        ├─ 临时表    生命周期 ≤ Session,catalog 内存
        ├─ 永久表    元数据存 Catalog(默认 default_catalog.default_database)
        └─ 外部表    CREATE TABLE ... WITH ('connector'='kafka' ...)
```

**⚠ 原笔记纠错(关键点 2):** 原文说 "`getExecutionEnvironment()` 根据运行环境自动判断,本地 IDE 启动会返回 `LocalStreamEnvironment`"。这描述对一半。实际实现在 `StreamExecutionEnvironment.getExecutionEnvironment()` 里:
1. 先看 `ThreadLocal<StreamExecutionEnvironmentFactory>` —— Flink 在提交 jar 时会注入 `StreamContextEnvironment`,这样用户代码里写 `getExecutionEnvironment()` 才能"自动"拿到上下文。
2. 否则回退到 `createLocalEnvironment()`。

所以"自动识别"靠的是 **CliFrontend 注入 Factory**,不是 JVM 自我感知。

---

## 四、时间语义 & 窗口

### 4.1 三种时间

| 名称 | 取自 | 特点 |
| --- | --- | --- |
| Processing Time | 算子所在机器系统时钟 | 最快,不可重放 |
| Ingestion Time | Source 算子接收时刻 | 不依赖事件本身 |
| Event Time | 事件本身携带的字段 | 可重放、可处理乱序,**必须配 Watermark** |

### 4.2 五类窗口

```
Tumble  : [0, 10) [10, 20) [20, 30)     固定大小、无重叠
Hop     : [0, 10) [5, 15) [10, 20)      slide=5、size=10
Session : 由间隙(gap)切分,大小可变
Cumulate: [0, 2) [0, 4) [0, 6) ... [0, T]  渐进式,每个 step 触发
Over    : 不切分,以"行"为窗口(RANGE/ROWS BETWEEN ...)
```

**⚠ 原笔记纠错(关键点 3):** 原文把 `HOP` 写成"slice",这不准确。FlinkSQL 中函数名是 `HOP(time, slide, size)`,**slice 是 Flink 内部对 Hop 窗口的一种实现优化技术**(把重叠区间切成最小公共片段以减少状态拷贝),不是用户层概念。

### 4.3 窗口划分起点(很多人会错)

Flink 内部 Tumble 窗口起点采用 `TimeWindow.getWindowStartWithOffset(timestamp, offset, size)`,默认 offset=0,公式:

```
windowStart = timestamp - (timestamp - offset + size) % size
```

举例:size=10s,事件时间戳=1714661385000(2024-05-02 22:09:45 UTC+8)
→ windowStart 不是 22:09:40,需按 epoch ms 余数计算,**与本地时区无关**。
原文笔记把"窗口起点"等同于"自然时间整点",在跨时区时会算错。

---

## 五、Watermark

### 5.1 概念

Watermark 是一种**带时间戳的特殊事件**,在数据流中随事件向下游传播。语义为:**"我向下游声明,不会再有 ≤ T 的事件到达"**。

```
事件流(乱序到达):  3   5   2   4   7   6   9
                                      │
              Watermark(=max_event_ts - delay):
              T1=3  T2=5  T2=5  T3=4  T4=7  T4=7  T5=9
              (delay=0 时即 max_event_ts;delay=2s 时为 max-2s)
```

### 5.2 触发窗口

```
窗口 [10, 20) 何时触发?
  收到 watermark W ≥ 20 时触发 (默认 EventTimeTrigger)
  
带 allowedLateness(2s):
  W ≥ 20 时第一次触发(输出)
  W < 22 期间,迟到事件仍可入窗,再次触发更新
  W ≥ 22 时窗口状态被清理,之后迟到 → sideOutputLateData
```

### 5.3 多并行度水印传播

```
Source-1 ─┐ wm=5 ─┐
Source-2 ─┼ wm=8 ─┼─►  Operator(并行)
Source-3 ─┘ wm=3 ─┘            │
                          下游 watermark = MIN(5, 8, 3) = 3
```

**⚠ 原笔记纠错(关键点 4):** 原笔记写"多并行度下取最大的水印"。**错。下游算子收到上游多个 Channel 的 Watermark 时,取的是 MIN**(取最小才能保证所有上游都不会再发更早的事件)。Flink 1.11 起还引入了 **Idleness 机制**:某分区长时间没数据时可标记为 idle,从 MIN 计算中排除,避免一个空闲分区拖住下游窗口触发。

---

## 六、Checkpoint 与状态后端

### 6.1 Checkpoint 执行流程(Chandy-Lamport 异步屏障快照)

```
JobMaster (CheckpointCoordinator)
   │
   │  ① 周期触发 triggerCheckpoint(chkId=N)
   │     向所有 Source Task 注入 CheckpointBarrier(N)
   ▼
Source Task                           Operator Task                Sink Task
   │                                        │                          │
   ├─ 收到触发 → snapshotState() → 上报 ack  │                          │
   ├─ 把 Barrier(N) emit 到下游          ─► │ 等待所有输入 channel       │
   │                                        │ 都到 Barrier(N)           │
   │                                        │   ├─ 已到的 channel 暂存   │
   │                                        │   └─ 未到的 channel 继续读 │
   │                                        │ (对齐式 checkpoint)        │
   │                                        │ → snapshotState() → ack   │
   │                                        │ → 把 Barrier 转发下游    ─►│
   │                                        │                          │ 同上,最终 ack
   │                                        │                          │
   │  ② JobMaster 收齐所有 Task 的 ack       │                          │
   │     → notifyCheckpointComplete(N)      │                          │
   │     → 各 Operator 提交本次外部事务      │                          │
   ▼
```

**⚠ 原笔记纠错(关键点 5):** 原笔记把 Checkpoint 描述为"同步保存所有状态后再继续处理"。
实际上 Flink 默认 Checkpoint 是**异步、对齐**的:
- 异步:`snapshotState()` 触发后,状态拷贝由后台线程写入 DFS,主线程继续处理新数据;
- 对齐:对齐式(Aligned)是默认;Flink 1.11+ 提供 **Unaligned Checkpoint**,Barrier 一到就直接转发,把 in-flight 数据也写入快照,反压场景下显著缩短 chk 时间。

### 6.2 重启策略

| 策略 | 配置 | 行为 |
| --- | --- | --- |
| none / disable | `RestartStrategies.noRestart()` | 失败即终止 |
| fixed-delay | `fixedDelayRestart(3, 5s)` | 固定次数 + 固定间隔 |
| failure-rate | `failureRateRestart(3, 5min, 10s)` | 5 分钟内最多 3 次失败 |
| exponential-delay | `exponentialDelayRestart(...)`(1.13+) | 指数退避,封顶 |

### 6.3 StateBackend (1.13+ 新分层)

Flink 1.13 起把"状态访问"和"快照存储"拆成两层,**原笔记里 `MemoryStateBackend / FsStateBackend / RocksDBStateBackend` 这三个名字已被废弃**:

| 旧名 | 新结构 |
| --- | --- |
| MemoryStateBackend | `HashMapStateBackend` + `JobManagerCheckpointStorage` |
| FsStateBackend | `HashMapStateBackend` + `FileSystemCheckpointStorage` |
| RocksDBStateBackend | `EmbeddedRocksDBStateBackend` + `FileSystemCheckpointStorage` |

**⚠ 原笔记纠错(关键点 6):** 原笔记仍按旧三件套讲。1.15 中:
- **StateBackend** 决定状态在算子运行时**怎么存**(JVM 堆 / RocksDB);
- **CheckpointStorage** 决定快照**写到哪**(JM 内存 / 文件系统)。
两者正交。如线上一般用 `EmbeddedRocksDBStateBackend` + HDFS/S3 的 CheckpointStorage。

---

## 七、动态表 & 连续查询

```
   外部流(Kafka)        Dynamic Table        Changelog Stream
   ┌─────────┐         ┌─────────────┐       ┌──────────────┐
   │ event1  │ ──►     │ row1        │  ──►  │ +I row1      │
   │ event2  │ ──►     │ row1,row2   │  ──►  │ +I row2      │
   │ upd1    │ ──►     │ row1',row2  │  ──►  │ -U row1      │
   │         │         │             │       │ +U row1'     │
   └─────────┘         └─────────────┘       └──────────────┘

显示模式:
   table      只展示最终状态(类似数据库结果集),会替换
   changelog  打印 +I -U +U -D 全过程
   tableau    类似 mysql 命令行表格,流式追加
```

---

## 八、关键纠错清单(汇总)

| # | 原笔记表述 | 正确表述 |
| --- | --- | --- |
| 1 | "Per-job 是生产标配" | Flink 1.15 起 Per-job 已 deprecated,推荐 **Application 模式**(main 在 JM 执行) |
| 2 | "getExecutionEnvironment 根据 JVM 自动识别" | 通过 `StreamExecutionEnvironmentFactory` ThreadLocal 注入,提交时由 CliFrontend 设置 |
| 3 | "HOP 又叫 slice" | HOP 是用户层函数,slice 是 Flink 内部的优化技术 |
| 4 | "多并行度水印取最大值" | **取最小值**(MIN),并支持 idleness 排除空闲分区 |
| 5 | "Checkpoint 同步保存,期间停处理" | 默认是**异步对齐**;1.11+ 支持 Unaligned Checkpoint |
| 6 | "MemoryStateBackend / FsStateBackend / RocksDBStateBackend" | 1.13+ 拆为 **StateBackend(HashMap/RocksDB)+ CheckpointStorage(JM/FileSystem)** |
| 7 | "窗口起点 = 自然时间整点" | 起点按 epoch ms 余数算,与本地时区无关 |

---
