# SparkCore 整理稿

> 原笔记版本基准:**Spark 3.x**(以 RDD/Core 为主)
> 涵盖:RDD 算子、Stage/Task 调度、Shuffle 演进、广播变量、累加器、内存管理、Spark On YARN。

---

## 一、知识点总览

```
SparkCore
├─ 与 MR 对比
│   ├─ 内存计算、DAG 调度、惰性求值
│   └─ 中间结果可缓存,迭代场景比 MR 快 10~100x
├─ 集群模式
│   ├─ Standalone (client / cluster)
│   └─ Spark On YARN (client / cluster)
├─ 核心抽象 RDD
│   ├─ 五大特性:分区列表、计算函数、依赖、Partitioner、preferredLocations
│   ├─ Transformation(lazy)
│   └─ Action(触发 Job)
├─ 调度
│   ├─ Application → Job → Stage → TaskSet → Task
│   ├─ DAGScheduler:按 Shuffle 切 Stage
│   └─ TaskScheduler:把 TaskSet 投递到 Executor
├─ Shuffle
│   ├─ HashShuffle (老,已废弃)
│   └─ SortShuffle (默认) + Tungsten-Sort + Bypass
├─ 高级特性
│   ├─ 广播变量(BroadcastManager / TorrentBroadcast)
│   ├─ 累加器(LongAccumulator / CollectionAccumulator)
│   └─ Cache / Checkpoint
└─ 内存管理(Unified)
    ├─ Reserved(300MB 预留)
    ├─ User Memory (~25%)
    └─ Spark Memory = Execution + Storage(可动态借用)
```

---

## 二、Spark 与 MapReduce 对比(纠错)

| 维度 | MapReduce | Spark |
| --- | --- | --- |
| 中间结果 | 每个 Stage 落盘 | 优先内存,可 cache/checkpoint |
| 调度模型 | 两阶段(Map/Reduce) | DAG,多 Stage |
| 编程接口 | Mapper/Reducer | 丰富算子 + DataFrame/Dataset |
| 容错 | 任务重跑 | RDD 血缘(Lineage)回溯 |

**⚠ 原笔记纠错(关键点 1):** 原文说"Spark 全部在内存中,所以快 100 倍"。**错。**
- Spark Shuffle **一定会落盘**(Sort/Hash Shuffle 都把分桶后的数据写到本地磁盘,reducer 拉取);
- Spark 快的核心是:① DAG 调度减少不必要的落盘(MR 每个 stage 都落盘),② RDD 可缓存复用,③ JVM 进程级 Executor 复用而不是 MR 的进程级 Task。"100x" 只在迭代算法+全内存场景(论文实验)成立。

---

## 三、Application → Job → Stage → Task

```
Application (一个 SparkContext)
   │
   ├─ Job 1   ← 一次 Action 触发
   │   ├─ Stage 1 (ShuffleMapStage)
   │   │    └─ TaskSet [Task0, Task1, ...]  ← 每个分区一个 Task
   │   ├─ Stage 2 (ShuffleMapStage)
   │   └─ Stage 3 (ResultStage)
   │
   └─ Job 2

切分规则:
   从 Action 出发反向遍历 RDD 血缘,
   遇到 ShuffleDependency 就切一个 Stage 边界。
   一个 Stage 内部全是 NarrowDependency(窄依赖),
   可以 pipeline 在同一个 Task 内执行。
```

### 3.1 调度调用链

```
rdd.count()
  └─> sc.runJob(rdd, ...)
        └─> DAGScheduler.runJob
              └─> submitJob → JobWaiter
                    └─> DAGScheduler.handleJobSubmitted
                          ├─ finalStage = createResultStage(...)
                          ├─ submitStage(finalStage)
                          │    ├─ 递归向上 submitMissingTasks(parent)
                          │    └─ TaskScheduler.submitTasks(TaskSet)
                          │          └─ SchedulerBackend.reviveOffers()
                          │                └─ Executor.launchTask(serializedTask)
                          │                      └─ TaskRunner.run() → Task.runTask()
```

---

## 四、Shuffle 演进

### 4.1 HashShuffleManager(已废弃,Spark 2.0 移除)

每个 Map Task 为每个 Reducer 单独写一个文件:
- 一个 Stage 产生 `M × R` 个小文件
- 文件数爆炸,IO 极差

**优化版** (Consolidate): 同一个 Core 上的 Map Task 共享同一组输出文件,文件数降到 `Cores × R`。

### 4.2 SortShuffleManager(默认,Spark 1.2+)

```
每个 Map Task 输出:
   ┌────────────────────────┐
   │  data.shuffle  (一个文件) │  按 partitionId 排序写入
   ├────────────────────────┤
   │  index.shuffle (一个文件) │  partitionId → offset
   └────────────────────────┘

Reducer 端 fetch:
   找到对应 data 文件 + 通过 index 定位本 partition 的字节区间 → 拉取
   → 文件数从 M*R 降到 M(每个 Map Task 只产生 2 个文件)
```

三种写出路径:
- **UnsafeShuffleWriter (Tungsten-Sort)**:序列化后直接在堆外排序,数据 size ≤ 16M 且分区数 ≤ 16777215 时启用
- **BypassMergeSortShuffleWriter**:分区数 ≤ `spark.shuffle.sort.bypassMergeThreshold`(默认 200)且无 map 端聚合时启用,类似 HashShuffle 但最后合并成一个文件
- **SortShuffleWriter**:默认路径,带 map 端聚合或大数据量

**⚠ 原笔记纠错(关键点 2):** 原笔记说"SortShuffleManager 每个 Map Task 都对分区内数据排序"。**不完全对。**
- SortShuffle 的"sort"指的是**按 partitionId 排序**(决定字节区间),不一定按 key 排序;
- 只有指定了 `sortByKey` 或 `repartitionAndSortWithinPartitions` 这种带 Ordering 的算子,才会在分区内按 key 排序;
- Bypass 路径**完全不排序**。

### 4.3 reduceByKey vs groupByKey

```
reduceByKey:
  MapTask 内先聚合(combine on map side)
  → 网络传输的数据量 = unique keys × value 数
  
groupByKey:
  无 map 端合并,所有 <k, v> 都拷贝到 reducer
  → 数据量 = 全部 record
```

**⚠ 原笔记纠错(关键点 3):** 原笔记说"reduceByKey 一定会触发 Shuffle"。**有反例:**
- 如果上游 RDD 已经按相同 `Partitioner` 分区(如 `reduceByKey` 后再 `reduceByKey`),Spark 会识别为**OneToOneDependency**(窄依赖),不再 shuffle;
- `join` 也是同理:相同 Partitioner 下两个 RDD `join` 不会 shuffle。
判断方式:看 `rdd.dependencies` 是否为 `ShuffleDependency`。

---

## 五、广播变量(TorrentBroadcast)

```
Driver
  ├─ broadcast(value)
  │    └─> 切成 blockSize(默认 4MB)的 N 个 block
  │        每个 block 存到 BlockManager (Driver 一份)
  │
Executor 首次 access bcVar.value
  ├─ 本地 BlockManager 查无
  ├─ 向 Driver 询问 block 列表
  ├─ 随机从 Driver 或其他已有 Executor 拉取 block(BT 协议)
  ├─ 拉到的 block 也存进自己的 BlockManager,供后续 Executor 拉
  └─ 在本地拼装回完整对象
```

优点:大变量(几十 MB 维表)只在 Executor 间传一次,而不是每个 Task 都拷一份。

---

## 六、累加器(Accumulator)

```
Driver:
  val acc = sc.longAccumulator("count")
  
Executor 内 Task:
  acc.add(1)   // 本地累加(Task 内私有 copy)
  // Task 结束时连同结果 status 一起回传 Driver
  
Driver 端 SparkContext:
  收到 Task 完成事件 → DAGScheduler.taskEnded
    └─> Accumulators.add(updates)   合并到全局 acc
```

**⚠ 原笔记纠错(关键点 4):** 原文说"累加器在 Transformation 中也能正确累加"。**不可靠:**
- 累加器只在 Action 触发的 Task 完成时上报;
- 若 Stage 因失败重算(speculation 或 fetch failed),**累加器会被加多次**;
- 仅推荐用在 **Action 内的 RDD 操作**(`foreach`/`foreachPartition`),以保证 exactly-once 时机。
官方建议把累加器当成"调试/统计工具",不要用于业务正确性逻辑。

---

## 七、Spark On YARN 提交流程(cluster 模式)

```
Client                    YARN RM                 NodeManager
  │ spark-submit            │                        │
  │  --master yarn          │                        │
  │  --deploy-mode cluster  │                        │
  ├──► 申请提交 App ────────►│                        │
  │                          │ 分配 Container         │
  │                          ├──────────────────────►│
  │                          │                        │ 启动 AM 容器
  │                          │                        │   ├─ AM 内启动 Driver(运行用户 main)
  │                          │                        │   └─ Driver 创建 SparkContext
  │                          │                        │
  │                          │  AM 向 RM 申请 Executor│
  │                          │ ◄─────────────────────►│
  │                          │                        │ 启动 Executor 容器
  │                          │                        │   注册回 Driver(CoarseGrainedExecutorBackend)
  │                          │                        │   接收 LaunchTask
```

**client vs cluster:**
- `client`:Driver 在提交命令的本地机器,日志直接打到终端;Client 退出即 App 失败。适合调试。
- `cluster`:Driver 在 AM 容器内,Client 提交完就可断开。生产推荐。

---

## 八、Unified Memory(Spark 1.6+)

```
   Executor 堆内内存 (spark.executor.memory)
   ┌──────────────────────────────────────────────┐
   │ Reserved Memory     固定 300MB                │
   ├──────────────────────────────────────────────┤
   │ User Memory     (1 - spark.memory.fraction)   │
   │                  默认 25%, 存用户代码对象      │
   ├──────────────────────────────────────────────┤
   │ Spark Memory    spark.memory.fraction = 0.6   │
   │   ┌─────────────────────────────────────┐    │
   │   │ Storage Memory                       │    │
   │   │   spark.memory.storageFraction = 0.5 │    │
   │   ├─────────────────────────────────────┤    │
   │   │ Execution Memory                     │    │
   │   └─────────────────────────────────────┘    │
   └──────────────────────────────────────────────┘

动态占用:
  - Execution 不够 → 可以把 Storage 区域驱逐到磁盘借走
  - Storage   不够 → 可以借 Execution 空闲部分,但 Execution 一旦要用,
                    Storage 必须立刻让出(不可强行驱逐 Execution 的中间结果)
```

**⚠ 原笔记纠错(关键点 5):** 原文写"Execution 和 Storage 可互相驱逐"。**不对称:**
- Execution 区的中间数据丢失会让 Task 重算,**不允许被驱逐**;
- 只有 Storage 缓存可以被淘汰(LRU)。所以 Storage 借来的可被收回,Execution 借来的不可。

---

## 九、Cache / Persist / Checkpoint

| 机制 | 存储位置 | 生命周期 | 是否切断血缘 |
| --- | --- | --- | --- |
| cache() / persist(MEMORY_ONLY) | Executor 堆内 | SparkContext 关闭 | 否 |
| persist(MEMORY_AND_DISK) | 堆内 + 本地磁盘 | 同上 | 否 |
| checkpoint() | HDFS 等可靠存储 | 长期 | **是**(物化后血缘被截断) |

最佳实践:`rdd.cache(); rdd.count(); rdd.checkpoint();`
原因:`checkpoint()` 会再触发一次 Job 把 RDD 算出来写 HDFS,先 cache 一下避免重算。

---

## 十、关键纠错清单(汇总)

| # | 原笔记表述 | 正确表述 |
| --- | --- | --- |
| 1 | "Spark 全内存,比 MR 快 100 倍" | Shuffle 必落盘;快源自 DAG + 内存缓存 + Executor 复用;100x 仅特定场景 |
| 2 | "SortShuffle 每个 Map Task 按 key 排序" | 默认只按 partitionId 排序;按 key 排序仅在 sortByKey 等算子触发 |
| 3 | "reduceByKey 一定 Shuffle" | 上游已用相同 Partitioner 时是窄依赖,不 shuffle |
| 4 | "累加器在 Transformation 中可靠" | Stage 重算会被加多次,建议只在 Action 中使用 |
| 5 | "Execution/Storage 可互相驱逐" | Execution 不可被驱逐,Storage 可被(LRU)淘汰 |
| 6 | "HashShuffle 是当前默认" | 默认是 SortShuffle,HashShuffle 在 2.0 已移除 |
| 7 | "广播变量 Driver 直接群发到所有 Executor" | TorrentBroadcast:按 block 切分,Executor 互相拉(P2P) |

---
