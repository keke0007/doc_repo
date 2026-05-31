# Kafka 完整梳理

## 目录
1. [Kafka 架构总览](#kafka-架构总览)
2. [核心概念](#核心概念)
3. [架构详解](#架构详解)
4. [消息存储机制](#消息存储机制)
5. [生产者机制](#生产者机制)
6. [消费者机制](#消费者机制)
7. [副本同步与高可用](#副本同步与高可用)
8. [性能优化](#性能优化)
9. [Spring Kafka 应用](#spring-kafka-应用)
10. [业务实战](#业务实战)
11. [易混淆点速查](#易混淆点速查)
12. [原文勘误](#原文勘误)
13. [Kafka vs 其他MQ](#kafka-vs-其他mq)

---

## Kafka 架构总览

### ASCII 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                   Kafka 分布式消息系统                       │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Producer (生产端)     Broker Cluster          Consumer (消费端)
│     │                      │                        │
│     │  send(key,val)   ┌─────────┐               │
│     ├──────────────→  │ Broker1 │              │
│     │                  │ Topic P0│            ┌─→ ConsumerGroup1
│     │  重试/幂等/      ├─────────┤            │    Consumer1
│     │  事务/ACK   │ Broker2 │            │    Consumer2
│     │                  │ Topic P1│            │
│     └────────────────→ ├─────────┤            └─→ ConsumerGroup2
│                        │ Broker3 │                 Consumer1
│                        └─────────┘
│                             │
│                    ZooKeeper/KRaft
│                 (Metadata & Controller)
│
└─────────────────────────────────────────────────────────────┘
```

---

## 核心概念

### 1. 主要角色

| 角色 | 定义 | 职责 |
|------|------|------|
| **Broker** | Kafka 集群节点（机器） | 存储消息、处理读写、副本同步 |
| **Producer** | 生产者 | 发送消息到 Topic |
| **Consumer** | 消费者 | 从 Topic 消费消息 |
| **Controller** | 控制器（某个 Broker） | 集群协调、Leader 选举、分区重分配 |
| **ZooKeeper/KRaft** | 元数据管理 | 存储集群状态（新版可用 KRaft 替代 ZK） |

### 2. 主题与分区

| 概念 | 定义 | 作用 |
|------|------|------|
| **Topic** | 消息主题/通道 | 逻辑分组，生产消费的入口 |
| **Partition** | 分区 | 物理分割，提升并发吞吐（可独立消费） |
| **Replicas** | 副本集合 | 冗余机制，每个分区可有多个副本备份 |
| **Leader** | 分区主副本 | 处理所有读写请求 |
| **Follower** | 分区从副本 | 从 Leader 同步数据，不处理读写 |

### 3. 副本管理三元组

```
AR (Assigned Replicas)   = 所有副本集合
├── ISR (In-Sync Replicas) = 与 Leader 同步的副本（可参与选主）
└── OSR (Out-Sync Replicas) = 不同步的副本（追赶 Leader）

触发 ISR → OSR 条件（二者满足其一）：
• replica.lag.time.max.ms：同步延迟超时（默认 10s）
• replica.lag.max.bytes：消息数量落差过大
```

### 4. 偏移量与水位值

```
offset（偏移量）
  ├─ 消费偏移量：消费者当前消费到的位置
  ├─ Log Start Offset：日志起始位置
  └─ Log End Offset (LEO)：日志末端位置（下一条待写消息）

HW (High Watermark - 高水印)
  └─ 消费者可见的最高位置
  └─ HW = min(Leader.LEO, min(所有ISR副本的LEO))
  └─ 消费者最多消费到 HW 位置（HW 之后不可见）

LEO (Log End Offset)
  └─ 当前日志末端位置（未提交的消息存储位置）
  └─ Leader 和 Follower 都有 LEO
  └─ HW <= LEO（总是成立）

可见消息范围 = [0, HW)
未提交消息 = [HW, LEO)
```

ASCII 示意：

```
消息序号：0  1  2  3  4  5  6  7 | 8  9  10 11 12
        ┌──────────────────────┬──────────────┐
        │   对消费者可见        │   对消费者不可见  │
        │  消息 offset[0,8)    │ 消息offset[9,12]│
        └──────────────────────┴──────────────┘
         ↑                      ↑
        offset=0              HW=8

Leader: LEO=12  HW=8
Follower B: LEO=10  HW=8
Follower C: LEO=9   HW=8

HW = min(12, 10, 9) = 9 (ISR中最小的LEO)
```

---

## 架构详解

### 1. 版本演进

| 版本 | 时间 | 关键特性 | 建议 |
|------|------|---------|------|
| 0.7 | - | 基础消息队列功能 | ✗ |
| 0.8 | - | 副本机制（真正高可用） | ✗ |
| 0.9 | - | 权限认证、新 Consumer API | ⚠️ 不建议 |
| 0.10 | - | Kafka Streams、流处理 | ✓ 0.10.2.2 |
| 0.11 | - | 幂等 Producer、事务、消息格式重构 | ✓ 0.11.0.3 |
| 1.0/2.0 | - | Streams 改进 | ✓ 2.0+ |
| 3.0+ | - | KRaft 可用（无 ZK） | ✓ 最新 |

### 2. 元数据管理：ZooKeeper vs KRaft

```
ZooKeeper 架构（传统）
  Broker1 ──┐
  Broker2 ──┼─→ ZooKeeper → /controller (Leader选举)
  Broker3 ──┤              → /brokers (节点信息)
             │              → /topics (主题分配)
             └→ 每个 Broker 监听 ZK 变化

KRaft 架构（3.0+，无 ZK 依赖）
  Broker1(Controller)  ← 元数据 Leader
  Broker2             ← 元数据 Follower
  Broker3             ← 元数据 Follower
  
  优势：
  • 减少依赖，部署更简单
  • 元数据吞吐更高
  • 跨集群 Replica 可行
```

### 3. 集群启动配置关键参数

```properties
# server.properties 配置示例

# 基础配置
broker.id=1                           # 集群内唯一ID
listeners=PLAINTEXT://:9092           # 监听地址（必须）
advertised.listeners=PLAINTEXT://IP:9092  # 对外宣告地址

# ZooKeeper 配置
zookeeper.connect=zk1:2181,zk2:2181   # ZK 地址（多个逗号分隔）

# 存储配置
log.dirs=/var/kafka/logs              # 日志存储路径

# 副本配置
min.insync.replicas=2                 # ISR 最小副本数（acks=all 时用）
default.replication.factor=3          # 默认副本数

# 清理配置
log.retention.hours=168               # 日志保留时间（小时）
log.retention.bytes=1073741824        # 日志保留大小（字节）
log.segment.bytes=1073741824          # 单个 segment 大小
```

---

## 消息存储机制

### 1. 分段存储架构

```
Topic: orders (2 个分区)
├── Partition 0
│   ├── 00000000000000000000.log       [offset 0-999)
│   ├── 00000000000000000000.index     (稀疏索引)
│   ├── 00000000000000001000.log       [offset 1000-1999)
│   ├── 00000000000000001000.index
│   └── 00000000000000002000.log       [offset 2000-...)
│       (持续增长，旧日志按规则清理)
│
└── Partition 1
    ├── 00000000000000000000.log
    ├── 00000000000000000000.index
    └── ...

Segment 命名规则：
  文件名 = 该段首条消息的 offset
  例：00000000000000001000.log 表示包含 offset 1000+ 的消息
```

### 2. 日志索引详解

```
稀疏索引（Sparse Index）
  00000000000000000000.index 内容：
  ┌──────────┬──────────┐
  │ offset   │ position │  位置存储在文件中的字节偏移
  ├──────────┼──────────┤
  │ 0        │ 0        │  第 0 条消息在文件第 0 字节
  │ 5        │ 1234     │  第 5 条消息在文件第 1234 字节
  │ 10       │ 4567     │  第 10 条消息在文件第 4567 字节
  │ ...      │ ...      │
  └──────────┴──────────┘

消息查找步骤（查找 offset=6）：
  1. 文件名排序 → 确定在 00000000000000000000.log
  2. 二分查找索引 → offset 5 对应位置 1234
  3. 从位置 1234 开始读取 → 读到 offset 10 停止（下一条）
  4. 解析 offset 6 的消息

时间索引 (*.timeindex)：
  └─ 同样结构，根据时间戳快速定位消息
```

### 3. 日志清理策略

```
Kafka 清理单位：整个 segment 文件（不是单条消息）

触发清理条件（满足任一）：
├─ 按时间：log.retention.ms (默认 7 天)
│  └─ 超过该时间的 segment 被删除
├─ 按大小：log.retention.bytes (默认无限制)
│  └─ 当日志总大小超过此值，删除旧 segment
└─ 按 cleanup.policy
   ├─ delete（默认）：直接删除过期 segment
   └─ compact：保留最新版本的每个 key

清理频率：
  log.retention.check.interval.ms = 5min（默认5分钟检查一次）
```

---

## 生产者机制

### 1. 消息发送流程

```
Producer.send(topic, key, value)
    │
    ├─→ ① Serializer 序列化 (key + value)
    │
    ├─→ ② Partitioner 选择分区
    │    ├─ 有 partition 指定 → 直接用
    │    ├─ 无 partition 有 key → hash(key) % partitionNum
    │    └─ 都无 → 轮询 (round-robin)
    │
    ├─→ ③ Accumulator 积累批次
    │    └─ 存入内存缓冲区 (buffer.memory)
    │       ├─ batch.size：一批的大小 (16KB)
    │       └─ linger.ms：最多等待时间 (0ms)
    │
    ├─→ ④ Sender 线程发送
    │    ├─ 构造 ProduceRequest
    │    └─ 发送到对应 Broker
    │
    ├─→ ⑤ Broker 处理 (根据 acks 参数)
    │
    └─→ ⑥ Callback 返回结果 (异步或同步)

时间线示意：
┌────────┬───────────────┬────────┐
│ send() │ 等待 (batch)  │ 发送   │
└────────┴───────────────┴────────┘
  t=0      t=[0,linger]   t=max(batch.size,linger)

两种发送模式：
  异步 send()           同步 send().get(timeout)
  └─ 立即返回          └─ 阻塞等待结果
  └─ 通过 Callback     └─ 或超时异常
```

### 2. 关键配置参数

```yaml
spring.kafka.producer:
  # 网络与重试
  bootstrap-servers: localhost:9092,localhost:9093
  retries: 3                    # 重试次数（新版：delivery.timeout.ms）
  request-timeout-ms: 30000     # 单次请求超时
  
  # 序列化
  key-serializer: StringSerializer
  value-serializer: StringSerializer
  
  # 吞吐优化
  batch-size: 16384             # 一批的最大字节数 (16KB)
  linger-ms: 10                 # 最多等待时间 (10ms)
  buffer-memory: 33554432       # 总缓冲区大小 (32MB)
  
  # 可靠性
  acks: 1                       # 应答级别 (0/1/all)
  compression-type: snappy      # 压缩算法
  
  # 特殊特性
  enable-idempotence: true      # 幂等性 (自动 acks=all)
  transactional-id: producer-1  # 启用事务
```

### 3. ACK 应答机制详解

```
acks 参数决定何时认为消息"已发送"

┌─────────┬──────────────────────┬──────────┬────────┐
│ acks    │ 详细解释               │ 可靠性   │ 延迟   │
├─────────┼──────────────────────┼──────────┼────────┤
│ 0       │ Producer 不等待 ACK  │ 最低     │ 最低   │
│ (None)  │ 消息发送即返回         │ 丢消息   │        │
│         │ Broker 可能未收到      │          │        │
├─────────┼──────────────────────┼──────────┼────────┤
│ 1       │ Leader 写入本地磁盘   │ 中等     │ 中等   │
│ (Leader)│ Replica 可能未同步     │ 宕机丢   │        │
│         │ 立即返回 ACK           │          │        │
├─────────┼──────────────────────┼──────────┼────────┤
│ all     │ ISR 副本都写入磁盘    │ 最高     │ 最高   │
│ (-1)    │ 必须满足 min.insync.  │ 基本不丢 │ 慢     │
│         │ replicas 个副本       │          │        │
│         │ 所有副本返回 ACK      │          │        │
└─────────┴──────────────────────┴──────────┴────────┘

选择建议：
  acks=0   → 日志收集（可丢）
  acks=1   → 常见业务（平衡）
  acks=all → 金融支付（绝不能丢）
```

### 4. 幂等与事务

```
幂等性（Idempotent Producer）
  问题：网络抖动导致重试，可能重复发送
  方案：Producer ID + 序列号
  
  启用：enable.idempotence = true（自动设 acks=all）
  
  原理：
  ┌─────────────────────────────────────┐
  │ Producer                             │
  │ ├─ ProducerID (唯一标识)            │
  │ └─ SequenceNumber (递增计数)         │
  └─────────────────────────────────────┘
           ↓
  ┌─────────────────────────────────────┐
  │ Broker (去重检测)                   │
  │ ├─ 记录每个 ProducerID 的最大 seq   │
  │ ├─ 若 seq <= 记录值 → 重复，丢弃    │
  │ └─ 若 seq > 记录值 → 新消息，保存   │
  └─────────────────────────────────────┘
  
  限制：只能保证同一 Producer 内幂等
       重启后 ProducerID 变更，需要人工去重

事务（Transactional Producer）
  enable.idempotence = true           # 依赖幂等
  transactional.id = "unique-id"      # 事务 ID（必须）
  transaction.state.log.replication.factor = 3
  
  API 使用：
    producer.beginTransaction()
    producer.send(msg1)
    producer.send(msg2)
    producer.commitTransaction()      # 全部成功或全部失败
    // 或 abortTransaction()
  
  原理：
  ├─ 消费端需设 isolation.level = read_committed
  ├─ 只有提交的消息对消费者可见
  └─ 保证"恰好一次"语义 (Exactly Once)
```

### 5. 分区策略

```java
// 默认分区器：org.apache.kafka.clients.producer.internals.DefaultPartitioner

策略优先级：
1. 有 partition 参数 → 直接使用
   producer.send(new ProducerRecord(topic, partition=0, key, value))
   
2. 有 key 参数 → hash(key) % numPartitions
   producer.send(new ProducerRecord(topic, key="order_123", value))
   → 同一 key 总是路由到同一分区（顺序性保障）
   
3. 都无 → 轮询 (Round-Robin)
   producer.send(new ProducerRecord(topic, null, value))
   → 均匀分散到各分区

自定义分区器：
public class MyPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes,
                        Cluster cluster) {
        if (key == null) return 0;
        String keyStr = (String) key;
        // 例：0开头的 key 去分区0，其他去分区1
        if (keyStr.startsWith("0")) {
            return 0;
        }
        return 1;
    }
}

配置：
spring.kafka.producer.properties.partitioner.class = 
    com.example.MyPartitioner
```

---

## 消费者机制

### 1. 消费者组与负载均衡

```
Consumer Group（消费者组）
  └─ 同一 group 的消费者共同消费一个 topic
  └─ 不同 group 可独立消费同一 topic
  
消息分配规则（Rebalance）：
┌─────────────────────────────────────────┐
│ Topic: orders (3 个分区)                 │
│ P0    P1    P2                           │
├─────────────────────────────────────────┤
│ 场景1：1个消费者                         │
│ Consumer1 ← P0, P1, P2 (独享全部分区)   │
│                                          │
│ 场景2：2个消费者                         │
│ Consumer1 ← P0, P2                       │
│ Consumer2 ← P1                           │
│                                          │
│ 场景3：3个消费者                         │
│ Consumer1 ← P0                           │
│ Consumer2 ← P1                           │
│ Consumer3 ← P2                           │
│                                          │
│ 场景4：4个消费者（消费者>分区）         │
│ Consumer1 ← P0                           │
│ Consumer2 ← P1                           │
│ Consumer3 ← P2                           │
│ Consumer4 ← 闲置（浪费资源！）          │
└─────────────────────────────────────────┘

最佳实践：
  消费者数量 = 分区数量（充分利用）
```

### 2. Rebalance（重新平衡）流程

```
触发 Rebalance 事件：
  • 消费者加入/退出
  • 消费者心跳超时
  • topic 分区数变更
  • 消费者订阅 topic 变更

完整流程：
①─→ 某消费者加入（或宕机）
②─→ Coordinator (Broker) 发现变化
③─→ 通知 group 中所有消费者
④─→ 所有消费者 revoke（撤销）当前分区
    ├─ 提交已消费偏移量
    └─ 清理本地资源
⑤─→ Rebalancer 重新分配分区
    └─ 使用 AssignmentStrategy (RangeAssignor/RoundRobinAssignor)
⑥─→ 所有消费者 assign（分配）新分区
    └─ 开始从新分区消费
⑦─→ 消费继续

时间损耗：
  Rebalance 过程中，所有消费者停止消费（STW）
  → 需最小化频率和时间

配置调优：
  session.timeout.ms = 10s              # 心跳超时
  heartbeat.interval.ms = 3s            # 心跳发送频率
  max.poll.interval.ms = 300s           # 处理消息最长时间
  
  session.timeout > heartbeat.interval（心跳有冗余）
  max.poll.interval > 处理一批消息的时间

ASCII 时间线：
┌──────┬──────┬──────┬──────┬──────┐
│收消息│处理  │心跳  │收消息│处理  │
└──────┴──────┴──────┴──────┴──────┘
←─ max.poll.interval ─→（超过触发 revoke）
      ↑
    heartbeat_interval
```

### 3. 偏移量提交策略

```yaml
spring.kafka.consumer:
  bootstrap-servers: localhost:9092
  
  # 关键配置
  enable-auto-commit: true              # 自动提交（默认）
  auto-commit-interval: 100              # 提交间隔 (100ms)
  auto-offset-reset: latest              # 无偏移时策略
  
  group-id: my-group
  key-deserializer: StringDeserializer
  value-deserializer: StringDeserializer

自动提交流程：
┌──────────────────────────────────────────┐
│ poll() ─→ 消费 ─→ 处理 ─→ auto-commit   │
│          消息      业务      offset      │
└──────────────────────────────────────────┘

风险：处理失败但 offset 已提交 → 消息丢失

手动提交（更安全）：
enable-auto-commit: false

// 代码示例
@KafkaListener(topics = "orders", groupId = "mygroup")
public void consume(ConsumerRecord<String, String> record,
                   Acknowledgment ack) {
    try {
        // 处理消息
        processMessage(record.value());
        
        // 业务成功后再提交
        ack.acknowledge();  // 同步提交
        // 或 ack.nack();   // 提交失败，Rebalance 重新消费
    } catch (Exception e) {
        log.error("处理失败", e);
        // 异常时不提交，Rebalance 后重新消费
    }
}

同步 vs 异步提交：
  commitSync()   → 阻塞，重试，保证提交
  commitAsync()  → 非阻塞，不重试，可能失败
  
  建议组合：
  commitAsync()  处理消息时异步提交
  commitSync()   消费者关闭前同步提交（保证最后一次成功）
```

### 4. 偏移量重置规则

```
auto-offset-reset（无已提交 offset 时）

earliest
  ├─ 无提交 offset → 从头开始消费 (offset=0)
  └─ 有提交 offset → 从提交点开始

latest (默认)
  ├─ 无提交 offset → 从末尾开始消费 (只消费新消息)
  └─ 有提交 offset → 从提交点开始

none
  ├─ 任何分区无 offset → 抛异常
  └─ 所有分区都有 offset → 从提交点开始

⚠️ 重要：有提交 offset 时，auto-offset-reset 失效！
   即使设 earliest，也会从提交点继续消费
```

---

## 副本同步与高可用

### 1. 副本同步原理详解

```
三个角色的 LEO 和 HW：

Leader:
  ├─ LEO (Log End Offset)：自己的日志末端
  ├─ HW (High Watermark)：当前可提交的位置
  └─ RemoteLEO：每个 Follower 的 LEO 记录

Follower (多个):
  ├─ LEO：自己的日志末端
  └─ HW：从 Leader 返回的值

初始化：
  Leader.LEO = 0, HW = 0
  Follower.LEO = 0, HW = 0
  Leader.RemoteLEO[follower] = 0

消息同步流程（伪代码）：

// 场景：3 副本 (Leader A, Follower B, C)
// Producer 发送 3 条消息到 A

① Producer 发送消息
   A.LEO += 3  → A.LEO = 3
   
② Follower B 发起 Fetch 请求 (报告自己的 LEO)
   B.LEO = 0
   
③ Leader A 处理 Fetch
   Leader:
     A.RemoteLEO[B] = 0 (来自 B 的报告)
     A.HW = min(A.LEO, A.RemoteLEO[B], A.RemoteLEO[C])
     A.HW = min(3, 0, 0) = 0
   
   返回给 B：新消息 + A.HW
   
④ Follower B 接收并更新
   B.messages.addAll(新消息)
   B.LEO = 3
   B.HW = min(B.LEO, Leader.HW) = min(3, 0) = 0
   
⑤ Follower C 类似操作
   C.LEO = 3
   C.HW = 0
   
⑥ 下一轮 Fetch，B 和 C 报告 LEO=3
   A.RemoteLEO[B] = 3
   A.RemoteLEO[C] = 3
   A.HW = min(3, 3, 3) = 3
   返回给 B、C：A.HW = 3
   
⑦ B、C 更新自己的 HW
   B.HW = 3
   C.HW = 3
   
消费者可见范围：[0, HW) = [0, 3)
即 3 条消息都可以被消费

时间轴示意：
┌────────┬────────┬────────┬────────┐
│ 轮1    │ 轮2    │ 轮3    │ 轮4    │
├────────┼────────┼────────┼────────┤
│A: 0,0 │A: 3,0 │A: 3,0 │A: 3,3 │
│B: 0,0 │B: 0,0 │B: 3,0 │B: 3,3 │
│C: 0,0 │C: 0,0 │C: 3,0 │C: 3,3 │
└────────┴────────┴────────┴────────┘
         (HW 延迟一轮更新)
```

### 2. Leader Election（领导者选举）

```
触发选举：
  • Leader 宕机
  • 分区离线（ISR 为空）

选举规则（Kafka 0.11+）：

从 ISR 副本集合中选择
  ⚠️ 不是选 LEO 最大的，而是选 ISR 中的任意一个
  → 防止数据丢失（ISR 都是同步的）

第一阶段：Broker 发现 Leader 失败
  ├─ ZK watch 触发（Leader 临时节点消失）
  └─ Controller 被通知

第二阶段：Controller 执行选举
  ├─ 从 ISR 中选择新 Leader
  ├─ 更新 /brokers/topics/.../partitions/.../state
  └─ 通知所有 Follower

第三阶段：新 Leader 初始化
  ├─ 立即可接收消息
  ├─ 旧 Follower 清理多余数据（truncate）
  └─ 从新 Leader 同步（LEO 回退到 HW）

高可用保障：
  min.insync.replicas = 2
  → 即使 1 个副本宕机，ISR 中还有 1 个
  → Leader 不会宕机（因为 ISR 不为空）

完整示意：
┌──────────────────────────────────────┐
│ Leader A (在线)                       │
│ ├─ LEO: 100                          │
│ └─ RemoteLEO[B]: 98, [C]: 95        │
├──────────────────────────────────────┤
│ Follower B (在线，ISR 中)            │
│ ├─ LEO: 98                           │
│ └─ HW: 95 (min(100,98,95))          │
├──────────────────────────────────────┤
│ Follower C (在线，ISR 中)            │
│ ├─ LEO: 95                           │
│ └─ HW: 95                            │
└──────────────────────────────────────┘

A 宕机 → B 成为新 Leader
  B.LEO = 98 (原来的值)
  新 Leader 的 LEO 成为新 HW
  → 消费到 offset 95-98 的消息可能丢失
    (原因：未在 ISR 中被确认)
```

### 3. Leader Epoch（解决 HW 缺陷）

```
问题场景（0.11 前，仅用 HW）：

场景A：数据丢失
┌────────────────────────────────────────┐
│ Leader A (LEO=5, HW=3)                │
│ Follower B (LEO=5, HW=2, offline)     │
│                                        │
│ ① B 宕机                               │
│ ② 待恢复后：B.LEO = B.HW = 2（后退！）│
│ ③ 同时 A 也宕机了                     │
│ ④ B 被选为新 Leader                   │
│ ⑤ A 恢复变成 Follower，从 B 同步      │
│ ⑥ A.LEO = 2（丢失了 offset 2-5）     │
└────────────────────────────────────────┘

场景B：数据不一致
┌────────────────────────────────────────┐
│ Leader A (HW=5, 消息内容1-5)          │
│ Follower B (HW=4, 消息内容1-4, offline)│
│                                        │
│ ① A B 同时宕机                        │
│ ② B 先恢复成新 Leader（不幸！）       │
│ ③ 新消息到达 B，offset=4 的消息不同   │
│ ④ B.HW = 5                           │
│ ⑤ A 恢复变成 Follower，从 B 同步      │
│ ⑥ A.offset[4] 被覆盖成新值            │
│ ⑦ 数据不一致！                        │
└────────────────────────────────────────┘

解决方案：Leader Epoch
  └─ 给 Leader 加版本号，选主时通过 epoch 协商

实现：

Leader Epoch 记录：
  epoch: 0, offset: 0    ← Leader 第一次上任的位置
  epoch: 1, offset: 10   ← Leader 第二次上任的位置
  epoch: 2, offset: 25   ← ...

恢复流程：

① Follower B 宕机恢复
   B.epoch_file = [epoch: 0, offset: 0]
   
② B 向当前 Leader A 发起 Fetch LeaderEpoch 请求
   B 报告：my_epoch = 0
   
③ A 处理请求
   A.epoch_file = [epoch: 0, ..., epoch: 1, ...]
   发现：B.epoch (0) <= A.latest_epoch (1)
   返回：truncate 到 epoch 0 对应的 offset
   
④ B 不再 truncate 到 HW，而是 truncate 到返回值
   → 保留更多数据，减少丢失

优势：
  • 防止数据丢失（保留 HW 之外的数据）
  • 防止数据不一致（通过 epoch 协商）
  • Kafka 0.11+ 默认启用
```

---

## 性能优化

### 1. 零拷贝（Zero-Copy）

```
传统文件读写（4 次拷贝）：
┌──────────────────────────────────────────┐
│ 传统 I/O 流程                             │
├──────────────────────────────────────────┤
│ ① 磁盘 → 内核 Buffer                     │
│ ② 内核 Buffer → 用户 Buffer              │
│ ③ 用户 Buffer → 内核 Socket Buffer       │
│ ④ 内核 Socket Buffer → 网卡             │
└──────────────────────────────────────────┘
  CPU 负责转移数据（低效）

零拷贝优化（DMA，0 次 CPU 拷贝）：
┌──────────────────────────────────────────┐
│ sendfile() 系统调用                       │
├──────────────────────────────────────────┤
│ 磁盘 ─→ 内核 Buffer ─→ 网卡              │
│  │                       │                │
│  └─ DMA 直接传输 ────────┘                │
│    （CPU 无参与）                        │
└──────────────────────────────────────────┘

实现方式：

Kafka Broker 使用 FileChannel.transferTo()
  └─ 底层调用操作系统 sendfile
  └─ 在 Linux/Unix 上有硬件支持

Java 实例：
  File file = new File("0.log");
  RandomAccessFile raf = new RandomAccessFile(file, "r");
  FileChannel fileChannel = raf.getChannel();
  
  SocketChannel socketChannel = ...;
  
  // 零拷贝传输
  fileChannel.transferTo(0, fileChannel.size(), socketChannel);

性能提升：
  传统 I/O：~200MB/s（CPU 100%）
  零拷贝：~4GB/s（CPU <1%）
  → 10-20 倍性能提升
```

### 2. 顺序写与 PageCache

```
磁盘 I/O 策略：

随机写（查询数据库常见）
  磁盘头频繁移动 → 寻道延迟大
  性能：~100 IOPS

顺序写（Kafka 采用）
  数据追加到文件末尾
  磁盘头无需移动
  性能：~5000+ IOPS（50 倍提升！）

Kafka 顺序写示意：
┌──────────────────────────┐
│ 消息到达顺序             │
│ msg1 → msg2 → msg3 → ... │
└──────────────────────────┘
            ↓
┌──────────────────────────┐
│ 写入 0.log 文件           │
│ [msg1][msg2][msg3]...    │
│ ↑                        │
│ 追加（顺序写）           │
└──────────────────────────┘

PageCache（操作系统页缓存）

Kafka 写入流程：
  Producer 消息 → Broker 内存 PageCache → 定期 flush → 磁盘

PageCache 优势：
  • 内存速度，接近 RAM 延迟
  • 操作系统自动管理（无需应用层维护）
  • Consumer 读取时直接从 PageCache 读（二次读很快）

配置参数：
  log.flush.interval.messages = 10000   # 消息数达到时 flush
  log.flush.interval.ms = 1000          # 时间达到时 flush
  
  建议：依赖操作系统自动 flush
        不主动配置（减少 I/O）

性能对比：
  普通数据库（写磁盘） ← 几十 ms
  Kafka（PageCache） ← 几微秒
  → Kafka 延迟极低的原因之一
```

### 3. 批量与压缩

```
批处理（Batching）

原理：
  多个消息合并成一批再发送 → 减少网络往返

配置：
  batch.size = 16384 (16KB)
  linger.ms = 10 (10ms)
  
  何时发送一批：
  ├─ 缓存大小 >= batch.size → 立即发
  ├─ 等待时间 >= linger.ms → 发
  └─ close() 时 → 发

性能差异：
  无批处理：linger.ms=0
    1000 消息 = 1000 次网络请求 → 延迟高
    吞吐：~1000 msg/s
  
  批处理：batch.size=16KB, linger.ms=10
    1000 消息分 10 批 → 10 次网络请求 → 延迟低
    吞吐：~100000 msg/s
  → 100 倍性能提升

压缩（Compression）

算法对比：
┌───────┬────────┬────────┬────────┐
│ 算法  │ 压缩率 │ CPU 占 │ 使用场景│
├───────┼────────┼────────┼────────┤
│ none  │ 0      │ 0      │ 禁用   │
│ gzip  │ 40-50% │ 高     │ 通用   │
│ snappy│ 20-30% │ 低     │ 高吞吐 │
│ lz4   │ 30-40% │ 最低   │ 实时   │
│zstd   │ 50-60% │ 中等   │ 最优   │
└───────┴────────┴────────┴────────┘

配置：
  compression.type = snappy

压缩流程：
  Producer:
    消息 → 批量 → [压缩] → 网络 → Broker
  
  Broker:
    收到压缩消息
    ├─ 保存压缩形式（节省磁盘）
    └─ 不解压
  
  Consumer:
    从 Broker 拉取
    ├─ 接收压缩消息
    └─ 自动解压

性能权衡：
  snappy：推荐，平衡压缩率和 CPU
  lz4：超高吞吐场景
  zstd：磁盘存储优化（3.0+）
```

### 4. 网络优化

```
参数调优：

send.buffer.bytes = 102400          # Socket 发送缓冲区
receive.buffer.bytes = 102400       # Socket 接收缓冲区
requests.max.bytes = 1048576        # 单个请求最大大小

TCP 优化：
  tcp_wmem_max / tcp_rmem_max 增大（系统参数）
  
连接优化：
  connections.max.idle.ms = 540000   # 连接空闲时间
  请求复用（减少连接建立）
  
Nagle 算法禁用：
  tcp_nodelay = true                 # 避免小数据包延迟
```

---

## Spring Kafka 应用

### 1. 基础配置

```yaml
# application.yml

spring:
  kafka:
    bootstrap-servers: localhost:9092,localhost:9093
    
    producer:
      # 基础
      retries: 3
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      
      # 吞吐
      batch-size: 16384
      linger-ms: 10
      buffer-memory: 33554432
      
      # 可靠性
      acks: 1
      compression-type: snappy
      
      # 特殊特性
      enable-idempotence: true
      properties:
        transactional.id: kafka-producer-1
    
    consumer:
      # 基础
      group-id: mygroup
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      
      # 位移提交
      enable-auto-commit: true
      auto-commit-interval: 100
      auto-offset-reset: latest
      
      # 消息处理
      max-poll-records: 500
      session-timeout: 10000
      heartbeat-interval: 3000
      
      # 性能
      fetch-min-size: 1024
      fetch-max-wait-ms: 500
    
    listener:
      ack-mode: MANUAL                # 手动提交模式
      type: batch                     # 批量消费
      poll-timeout: 3000
```

### 2. 消息发送

```java
// 异步发送（默认）
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

public void sendAsync(String topic, String message) {
    kafkaTemplate.send(topic, message)
        .thenAccept(result -> 
            log.info("消息发送成功: {}", result.getRecordMetadata().offset())
        )
        .exceptionally(ex -> {
            log.error("消息发送失败", ex);
            return null;
        });
}

// 同步发送（需要结果）
public void sendSync(String topic, String key, String message) {
    try {
        SendResult<String, String> result = kafkaTemplate
            .send(topic, key, message)
            .get(3, TimeUnit.SECONDS);
        
        log.info("发送成功，offset: {}", 
            result.getRecordMetadata().offset());
    } catch (TimeoutException e) {
        log.error("发送超时");
    } catch (Exception e) {
        log.error("发送失败", e);
    }
}

// 指定分区
public void sendToPartition(String topic, int partition, 
                           String key, String message) {
    kafkaTemplate.send(
        new ProducerRecord<>(topic, partition, key, message)
    );
}

// 事务发送
public void sendWithTransaction(String topic, List<String> messages) {
    try {
        messages.forEach(msg -> kafkaTemplate.send(topic, msg));
        // 自动提交事务（Spring 管理）
    } catch (Exception e) {
        log.error("事务失败，自动回滚");
    }
}

// 监听发送结果
@Bean
public KafkaTemplate<String, String> kafkaTemplate(
        ProducerFactory<String, String> pf) {
    KafkaTemplate<String, String> template = new KafkaTemplate<>(pf);
    
    template.setProducerListener(new ProducerListener<String, String>() {
        @Override
        public void onSuccess(ProducerRecord<String, String> record,
                            RecordMetadata recordMetadata) {
            log.info("消息发送成功: partition={}, offset={}",
                recordMetadata.partition(),
                recordMetadata.offset());
        }
        
        @Override
        public void onError(ProducerRecord<String, String> record,
                          RecordMetadata recordMetadata,
                          Exception exception) {
            log.error("消息发送失败", exception);
        }
    });
    
    return template;
}
```

### 3. 消息消费

```java
// 单条消费
@KafkaListener(topics = "orders", groupId = "group1")
public void listen(ConsumerRecord<String, String> record) {
    log.info("消息内容: {}, offset: {}, partition: {}",
        record.value(),
        record.offset(),
        record.partition());
}

// 批量消费（推荐用于高吞吐）
@KafkaListener(topics = "orders", groupId = "group1")
public void listenBatch(List<ConsumerRecord<String, String>> records) {
    records.forEach(record -> {
        log.info("处理消息: {}", record.value());
    });
}

// 指定分区
@KafkaListener(
    topics = "orders",
    topicPartitions = @TopicPartition(
        topic = "orders",
        partitions = {"0", "1"}
    ),
    groupId = "partition-group"
)
public void listenPartition(ConsumerRecord<String, String> record) {
    log.info("分区: {}, 消息: {}", 
        record.partition(),
        record.value());
}

// 手动提交偏移量
@KafkaListener(topics = "orders", groupId = "group1")
public void listenManual(ConsumerRecord<String, String> record,
                        Acknowledgment ack) {
    try {
        log.info("处理消息: {}", record.value());
        
        // 业务处理成功后提交
        ack.acknowledge();  // 同步提交
    } catch (Exception e) {
        log.error("处理失败，不提交 offset", e);
        // 异常时不提交，下次重新消费
    }
}

// 监听容器错误
@Bean
public ConcurrentKafkaListenerContainerFactory<String, String>
        kafkaListenerContainerFactory(
        ConsumerFactory<String, String> cf) {
    ConcurrentKafkaListenerContainerFactory<String, String> factory
        = new ConcurrentKafkaListenerContainerFactory<>();
    
    factory.setCommonErrorHandler(new DefaultErrorHandler(
        new LinearBackOffWithMaxRetries(3, 1000)
    ) {
        @Override
        public void handle(Exception thrownException,
                          ConsumerRecord<?, ?> data,
                          Consumer<?, ?> consumer,
                          MessageListenerContainer container) {
            log.error("消费异常，offset: {}", 
                data.offset(),
                thrownException);
        }
    });
    
    return factory;
}

// 获取消费位置
@Bean
public ApplicationRunner runner(KafkaTemplate<String, String> kt) {
    return args -> {
        // 发送消息
        kt.send("orders", "test");
        
        // 查询消费进度（通过 AdminClient）
        AdminClient admin = AdminClient.create(
            Collections.singletonMap(
                AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG,
                "localhost:9092"
            )
        );
        
        ListConsumerGroupOffsetsResult result = admin.listConsumerGroupOffsets("group1");
        // 获取每个分区的 offset
    };
}
```

### 4. 自定义序列化与分区

```java
// 自定义序列化器
public class Order {
    private String id;
    private String status;
    // getters/setters...
}

public class OrderSerializer implements Serializer<Order> {
    @Override
    public byte[] serialize(String topic, Order data) {
        if (data == null) return null;
        // 使用 JSON 序列化
        return JSON.toJSONString(data).getBytes(StandardCharsets.UTF_8);
    }
}

public class OrderDeserializer implements Deserializer<Order> {
    @Override
    public Order deserialize(String topic, byte[] data) {
        if (data == null) return null;
        return JSON.parseObject(
            new String(data, StandardCharsets.UTF_8),
            Order.class
        );
    }
}

// 配置
@Configuration
public class KafkaConfig {
    @Bean
    public ProducerFactory<String, Order> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, OrderSerializer.class);
        return new DefaultProducerFactory<>(config);
    }
    
    @Bean
    public KafkaTemplate<String, Order> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}

// 自定义分区器
public class OrderPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                        Object value, byte[] valueBytes,
                        Cluster cluster) {
        if (value == null) return 0;
        
        Order order = (Order) value;
        // 金牌用户去分区 0，普通用户去分区 1
        if (order.getId().startsWith("VIP")) {
            return 0;
        }
        return 1;
    }
}

// 使用自定义分区器
@Bean
public ProducerFactory<String, Order> producerFactory() {
    Map<String, Object> config = new HashMap<>();
    config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    config.put(
        ProducerConfig.PARTITIONER_CLASS_CONFIG,
        OrderPartitioner.class
    );
    return new DefaultProducerFactory<>(config);
}
```

---

## 业务实战

### 1. 顺序性保障

```
场景：订单状态流转（下单 → 支付 → 确认 → 发货）
      不能乱序

级别选择：
  全局有序：单分区单消费者，吞吐低，实际少用
  局部有序：同一订单有序，不同订单并行 ← 推荐

实现方案：

① 生产端：使用订单 ID 作为 key
@PostMapping("/order")
public void placeOrder(Order order) {
    // 同一订单始终路由到同一分区
    kafkaTemplate.send(
        new ProducerRecord<>(
            "orders",
            order.getId(),  // key = order_id（保证有序）
            JSON.toJSONString(order)
        )
    );
}

② 消费端：需要在单消费者处理一个分区，或用内存队列

方案 A：单线程单分区（吞吐低，不推荐）
@KafkaListener(
    topics = "orders",
    topicPartitions = @TopicPartition(
        topic = "orders",
        partitions = {"0"}
    ),
    groupId = "single-consumer"
)
public void consume(ConsumerRecord<String, String> record) {
    processOrder(record.value());
}

方案 B：分区内二级队列 + 线程池（推荐）
@Component
public class OrderConsumer {
    // 分区数 = BlockingQueue 数量
    private List<BlockingQueue<Order>> queues = 
        Arrays.asList(
            new LinkedBlockingQueue<>(),
            new LinkedBlockingQueue<>(),
            new LinkedBlockingQueue<>()
        );
    
    @PostConstruct
    public void init() {
        // 为每个队列分配一个处理线程
        for (int i = 0; i < queues.size(); i++) {
            int queueIndex = i;
            executorService.submit(() -> {
                while (true) {
                    try {
                        Order order = queues.get(queueIndex).take();
                        processOrder(order);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            });
        }
    }
    
    @KafkaListener(topics = "orders", groupId = "order-group")
    public void consume(ConsumerRecord<String, String> record) 
            throws InterruptedException {
        Order order = JSON.parseObject(
            record.value(),
            Order.class
        );
        
        // 二次分发：同一订单总是进同一队列
        int queueIndex = Math.abs(
            order.getId().hashCode() % queues.size()
        );
        queues.get(queueIndex).put(order);
    }
    
    private void processOrder(Order order) {
        log.info("处理订单: {}", order.getId());
        // 订单处理逻辑
    }
}

ASC 流程图：
Kafka 分区
  ├─ P0: order_1  → 分发到Q1 → Thread1 处理
  ├─ P1: order_2  → 分发到Q2 → Thread2 处理
  └─ P2: order_1  → 分发到Q1 → Thread1 处理（同订单同线程）
           
订单 1 处理顺序被保证！
订单 1 和 订单 2 可并行处理！
```

### 2. 海量数据同步（CDC）

```
场景：MySQL 订单数据 → Kafka → 大数据分析平台

架构：
MySQL (binlog) ─→ Canal ─→ Kafka ─→ Druid/Flink
         │              │
         └─数据变化监听─┘
         
关键配置：

1. MySQL 启用 binlog（8.0 默认开启）
   验证：SHOW VARIABLES LIKE 'log_bin';
   
2. Canal 部署（中间件）
   功能：监听 MySQL binlog → 转换为 Kafka 消息
   
   docker-compose.yml:
   services:
     canal:
       image: canal/canal-server
       environment:
         canal.instance.master.address: mysql:3306
         canal.instance.dbUsername: canal
         canal.instance.dbPassword: canal
         canal.mq.topic: order_changes
         
3. Kafka Topic
   创建 order_changes topic 接收 CDC 数据

4. 消费端处理
   @KafkaListener(topics = "order_changes", groupId = "analytics")
   public void processCDC(ConsumerRecord<String, String> record) {
       // Canal 消息格式（JSON）
       // {
       //   "data": [...],          # 变更的数据
       //   "database": "...",      # 库名
       //   "table": "...",         # 表名
       //   "type": "INSERT/UPDATE",# 操作类型
       //   "ts": 1234567890       # 时间戳
       // }
       
       String canalMessage = record.value();
       JSONObject obj = JSON.parseObject(canalMessage);
       
       String type = obj.getString("type");
       String table = obj.getString("table");
       
       if ("INSERT".equals(type) && "orders".equals(table)) {
           List<JSONObject> dataList = obj.getJSONArray("data")
               .toJavaList(JSONObject.class);
           
           // 写入数据湖/数据仓库
           dataList.forEach(this::writeToDataWarehouse);
       }
   }

数据流转示意：
MySQL orders 表
  │
  ├─ INSERT order_123  → binlog
  │
  ├─ UPDATE order_123 (status=paid)
  │
  └─ DELETE order_123
  
        ↓ (Canal 监听)
  
Kafka order_changes topic
  {"data":[...], "type":"INSERT", "ts":1611234567}
  {"data":[...], "type":"UPDATE", "ts":1611234568}
  {"data":[...], "type":"DELETE", "ts":1611234569}
  
        ↓ (消费端处理)
  
数据分析平台
  ├─ Druid：实时 OLAP
  ├─ Flink：实时流处理
  └─ Hive：离线分析
```

---

## 易混淆点速查

### 1. 三种一致性语义

| 语义 | 原因 | 场景 | 配置 |
|------|------|------|------|
| **At Most Once（最多一次）** | 不重试，message 可能丢失 | 日志收集、非关键数据 | `acks=0` 或`retries=0` |
| **At Least Once（至少一次）** | 重试导致可能重复 | 大多数业务 | `acks=1/all`, `retries>0` |
| **Exactly Once（恰好一次）** | 幂等 + 事务 | 金融支付、库存扣减 | `enable.idempotence=true` + 事务 |

### 2. HW vs LEO

```
┌─────────┬──────────────────────┬─────────────────────┐
│ 概念    │ 含义                 │ 作用                │
├─────────┼──────────────────────┼─────────────────────┤
│ HW      │ 高水印（同步的位置） │ 消费可见边界        │
│ (High   │ = min(ISR 中所有副本  │ 消费者最多消费到 HW │
│ Watermark)│ 的 LEO)            │ 之前的消息           │
│         │                      │ 保证消息被多副本复制 │
├─────────┼──────────────────────┼─────────────────────┤
│ LEO     │ 日志末端位移          │ 写入边界            │
│ (Log    │ = 下一条待写消息位置  │ 表示已写入的消息数  │
│ End     │ 每个副本独自维护      │ LEO 包含未提交消息   │
│ Offset) │                      │                     │
└─────────┴──────────────────────┴─────────────────────┘

关系：
  HW <= LEO（总是成立）
  
  HW 落后 LEO 的原因：
  Follower 同步有延迟
  等待所有 ISR 同步后才能提高 HW

示例：
  Leader:   LEO=10  HW=8  (消息 0-7 被确认)
  Follower: LEO=9   HW=8  (未同步 9 号消息)
  
  消费者可见消息：offset [0, 8)
  消息 8、9 不可见（还在 HW 到 LEO 之间）
```

### 3. acks 参数详解

```
┌────────┬──────────┬──────────┬────────┬──────────────────┐
│ acks   │ 什么时候 │ 数据安全 │ 延迟   │ 失败情况         │
│        │ 返回 ACK │ 程度     │        │                  │
├────────┼──────────┼──────────┼────────┼──────────────────┤
│ 0      │ 发送即返 │ 最低     │ 最低   │ Broker 宕机消息  │
│        │ 回（无等 │ 不可靠   │ (<1ms) │ 丢失             │
│        │ 待）     │          │        │ 网络中断消息丢失 │
│        │          │          │        │ 无重试           │
├────────┼──────────┼──────────┼────────┼──────────────────┤
│ 1      │ Leader 写│ 中等     │ 中等   │ Leader 写入后宕机│
│ (Leader)│ 入本地   │ 较可靠   │ (5-10ms)│ 副本未同步 消息 │
│        │ 磁盘即返 │          │        │ 丢失             │
│        │ 回       │          │        │                  │
│        │          │          │        │ Replica 宕机无影│
│        │          │          │        │ 响               │
├────────┼──────────┼──────────┼────────┼──────────────────┤
│ all    │ ISR 中全 │ 最高     │ 最高   │ Leader 失败但副本│
│ (-1)   │ 部副本写 │ 极可靠   │ (10-50 │ 成功 → 选新 Leader│
│        │ 入磁盘   │ 基本不丢 │ ms)    │ 无消息丢失       │
│        │ 再返回   │          │        │                  │
├────────┼──────────┼──────────┼────────┼──────────────────┤
│ min.   │ 必须同步 │          │        │ 需要与 acks=all  │
│ insync │ 的副本数│ (配合    │        │ 配合使用         │
│ .      │ (影响    │ acks)    │        │ 默认值 = 1       │
│ replicas│ acks=all)│          │        │ (只有 Leader)    │
└────────┴──────────┴──────────┴────────┴──────────────────┘

选择建议：
  日志：acks=0（丢消息无关紧要）
  业务：acks=1（平衡安全和性能）
  金融：acks=all + min.insync.replicas=2（绝对安全）

代码示例：
@Bean
public ProducerFactory<String, String> producerFactory() {
    Map<String, Object> config = new HashMap<>();
    config.put(ProducerConfig.ACKS_CONFIG, "all");          # acks=-1
    config.put(ProducerConfig.RETRIES_CONFIG, 3);
    config.put(ProducerConfig.MIN_INSYNC_REPLICAS_CONFIG, 2);
    return new DefaultProducerFactory<>(config);
}
```

### 4. Consumer Rebalance 触发条件

```
会导致 STW（Stop The World）停止消费

条件 1：消费者加入/移除
  ├─ 新消费者加入 group
  ├─ 消费者显式离开 close()
  └─ 消费者心跳超时 (session.timeout.ms)

条件 2：订阅改变
  └─ consumer.subscribe() 改变订阅 topic

条件 3：Topic 分区数变化
  └─ 新增分区（最常见）

条件 4：消费者处理消息超时
  └─ poll() 调用间隔超过 max.poll.interval.ms

Rebalance 影响：
  ┌─────────────────────────────────────────┐
  │ Rebalance 期间（通常 10-30s）           │
  ├─────────────────────────────────────────┤
  │ • 所有消费者停止消费                     │
  │ • 消息处理中断                           │
  │ • 系统吞吐下降到 0                      │
  │ • 延迟上升                               │
  └─────────────────────────────────────────┘

优化建议：
  session.timeout.ms = 10s         # 心跳超时（不要太短）
  heartbeat.interval.ms = 3s       # 心跳间隔（< session.timeout）
  max.poll.interval.ms = 300s      # 处理超时（足够大）
  
  # 调整消费速度
  max.poll.records = 500            # 单次最多拉取 500 条
  
  # 处理逻辑要快
  业务处理耗时 << max.poll.interval.ms
```

### 5. auto-offset-reset 无效场景

```
常见误区：

❌ 错误理解：
  "我设了 auto-offset-reset=earliest，就一定能从头消费"

✓ 正确理解：
  auto-offset-reset 仅在 "无已提交 offset" 时生效
  一旦消费过一次，就有了已提交 offset
  再启动消费者，会从该 offset 继续消费

验证场景：

场景 1：第一次消费（无 offset 记录）
  ├─ auto-offset-reset 生效
  ├─ earliest → 从 offset=0 开始
  └─ latest → 从最新消息开始

场景 2：第二次启动同一 group 消费者
  ├─ 已有 offset=100 记录
  ├─ auto-offset-reset 失效！
  └─ 从 offset=100+ 继续消费
  
  ⚠️ 要重新消费旧消息，需要：
  # 重置 offset
  kafka-consumer-groups.sh \
    --bootstrap-server localhost:9092 \
    --group mygroup \
    --reset-offsets \
    --to-earliest \
    --topic orders \
    --execute

场景 3：只有部分分区有 offset
  ├─ auto-offset-reset=none
  ├─ 会抛异常！
  └─ （不允许部分分区有差异）
```

---

## 原文勘误

### ⚠️ 发现的硬错误或版本差异

| 位置 | 原文 | 问题 | 更正 |
|------|------|------|------|
| 版本命名 | "现在的版本号是3位" | 表述不清 | 应说 "1.0+版本采用3位" (e.g., 2.7.0 = 2.7.0) |
| OSR 翻译 | "Out-Sync Relipcas" | 拼写错误 (Relipcas) | 应为 "Out-Sync Replicas" |
| Consumer API | "不建议使用 consumer API" (0.9版) | 已过时，应明确版本 | 0.10+ 推荐使用新 Consumer API |
| Coordinator | 未明确说明 | 容易混淆 | Coordinator = 某个 Broker，负责消费者组协调 |
| KRaft | 文档未提及 KRaft | 3.0+ 后 ZK 可选 | 新版应提及 KRaft 架构 |
| min.insync.replicas | 未明确说明与 acks=all 的关系 | 参数独立 | 应说明需与 acks=all 配合使用 |
| Offset 与消息序号 | "第几条消息 = 偏移量 + 1" | 表述容易混淆 | 应说 "消息下标 = offset, 消息序号(第几条) = offset + 1" |
| Log Segment 大小 | segment.bytes = 1000 示例 | 生产环境不实用 | 生产通常 1GB, 示例仅为演示 |
| HW 更新延迟 | "HW 一般会小于 LEO" | 对但未说明原因 | 应说 "HW 更新延迟一轮 Fetch" |

---

## Kafka vs 其他 MQ

### 完整对比表

```
┌──────────┬─────────────┬─────────────┬──────────────┬──────────────┐
│ 特性     │ Kafka       │ RabbitMQ    │ RocketMQ     │ 选择建议     │
├──────────┼─────────────┼─────────────┼──────────────┼──────────────┤
│ 吞吐量   │ 极高(百万+) │ 中等(万级)  │ 高(十万+)    │ Kafka最强    │
│ 延迟     │ 低(<10ms)   │ 极低(<1ms) │ 低(几ms)     │ RabbitMQ最优 │
│ 可靠性   │ 高(副本机制)│ 高(持久化) │ 极高(多副本) │ RocketMQ最好 │
│ 部署     │ 复杂+ZK/KRaft│ 简单      │ 复杂         │ RabbitMQ简单 │
│ 功能     │ 基础+流处理 │ 功能丰富   │ 功能丰富     │ 功能对比     │
│ 消息顺序 │ 支持(key)   │ 不保证     │ 支持(queue)  │ Kafka/RMQ好  │
│ 事务     │ 支持(0.11+) │ 支持       │ 支持         │ 都支持       │
│ 消费模式 │ Pull        │ Push/Pull  │ Pull         │ Kafka灵活    │
│ 社区     │ Apache大    │ RabbitMQ大 │ 阿里开源     │ Kafka最活跃  │
│ 成熟度   │ 生产级      │ 生产级     │ 生产级       │ 都成熟       │
├──────────┼─────────────┼─────────────┼──────────────┼──────────────┤
│ 场景     │ 日志/大数据 │ 业务解耦   │ 金融交易     │ 如下分析     │
│         │ 流处理      │ 微服务     │ 可靠传输     │             │
│         │ 实时分析    │            │              │             │
└──────────┴─────────────┴─────────────┴──────────────┴──────────────┘

选型决策树：

需要日志收集、大数据处理、流处理？
  ├─ YES → Kafka（专为此设计）
  
业务消息队列，需要简单可靠？
  ├─ YES → RabbitMQ（部署简单，功能完整）
  
金融级别可靠性，需要极高可用？
  ├─ YES → RocketMQ（阿里双11验证，开源社区好）
  
单纯消息解耦，低延迟 + 高可靠？
  ├─ YES → RabbitMQ（业界标准）

性能优先（百万 TPS 级别）？
  ├─ YES → Kafka（分布式设计）

个人建议：
  • 初创公司：RabbitMQ（学习曲线低）
  • 中等公司：Kafka（为大数据时代设计）
  • 大厂/金融：RocketMQ（可靠性至上）
```

### 架构对比

```
Kafka 分布式架构：
  多个 Broker ← 天然分布式，水平扩展容易
  
RabbitMQ 架构：
  单 Broker（可集群，但通常一主多从）
  → 扩展不如 Kafka 灵活

RocketMQ 架构：
  Broker 集群 + NameServer（类似 Kafka Controller）
  → 架构受 Kafka 启发，优化了某些特性
```

---

## 核心知识点速查表

### 参数速查

```yaml
# Producer 关键参数
bootstrap.servers                    # 集群地址
acks: [0, 1, all]                   # ACK 级别（可靠性）
retries: N                          # 重试次数（新版用 delivery.timeout.ms）
batch.size: 16384 bytes             # 批量大小
linger.ms: 10                       # 最长等待
buffer.memory: 33MB                 # 缓冲区
compression.type: [none, gzip, snappy, lz4, zstd]
enable.idempotence: true/false      # 幂等（自动 acks=all）
transactional.id: "..."             # 事务 ID

# Consumer 关键参数
bootstrap.servers                    # 集群地址
group.id                            # 消费者组
enable.auto.commit: true/false      # 自动提交
auto.commit.interval.ms: 100        # 提交间隔
auto.offset.reset: [earliest, latest, none]
session.timeout.ms: 10000           # 心跳超时
heartbeat.interval.ms: 3000         # 心跳间隔
max.poll.interval.ms: 300000        # 处理超时
max.poll.records: 500               # 单次拉取数

# Broker 关键参数
broker.id: 0                        # Broker 唯一 ID
listeners/advertised.listeners      # 监听和宣告地址
log.retention.hours: 168            # 保留时间（小时）
log.retention.bytes: 1GB            # 保留大小
log.segment.bytes: 1GB              # Segment 大小
min.insync.replicas: 2              # ISR 最小副本数
default.replication.factor: 3       # 默认副本数
zookeeper.connect                   # ZK 地址
```

### 常见问题速查

```
Q: 消息重复了怎么办？
A: ① acks=1，Leader 宕机
   ② Producer 重试
   启用幂等：enable.idempotence=true
   或应用层去重

Q: 消息丢失了怎么办？
A: ① acks=0，没有 ACK
   ② Broker 宕机，副本未同步
   解决：acks=all + min.insync.replicas=2

Q: 消费者 Rebalance 频繁？
A: ① 消费处理太慢 → 增加 max.poll.interval.ms
   ② 心跳超时 → 增加 session.timeout.ms
   ③ 消费者崩溃 → 检查异常日志

Q: 消息积压怎么办？
A: ① 增加消费者数量（<= 分区数）
   ② 增加消费吞吐：max.poll.records, 批量处理
   ③ 增加分区数 + 重新消费
   ④ 如果数据无关可清理 Topic

Q: 怎么保证消息顺序？
A: ① 同一 key 路由到同一分区
   ② 单线程消费（吞吐低）
   ③ 或多线程 + 内存队列保证顺序

Q: 消费进度卡住了？
A: ① 查看 LAG = LOG-END-OFFSET - CURRENT-OFFSET
   ② 排查消费端业务逻辑
   ③ 检查异常日志
   ④ 重置 offset 重新消费

Q: 如何监控 Kafka？
A: ① Kafka Eagle（UI，推荐）
   ② Prometheus + Grafana（成熟方案）
   ③ 自定义 metrics
   ④ JMX 监控
```

---

## 总结

Kafka 是一个**高吞吐、低延迟、高可用**的分布式消息系统，核心优势：

1. **架构优势**
   - 分布式设计，天然支持横向扩展
   - 副本机制保证高可用
   - ISR 机制平衡一致性和可用性

2. **性能优势**
   - 顺序写 + PageCache → 磁盘速度接近内存
   - 零拷贝 (sendfile) → 降低 CPU 占用
   - 批量处理 → 吞吐翻倍

3. **功能优势**
   - 支持幂等性和事务 (0.11+)
   - 支持流处理 (Kafka Streams)
   - 丰富的消费模式

4. **应用前景**
   - 大数据处理（日志收集、数据仓库）
   - 实时流处理（Flink on Kafka）
   - 微服务通信（轻量级）

学习路线建议：
```
基础概念 → 架构设计 → 生产消费 → 副本同步 → 性能优化 → 生产应用
  ↓         ↓         ↓         ↓         ↓         ↓
Broker   Topic/     Producer/ Rebalance 零拷贝   监控告警
Partition Consumer  ACK      HW/LEO    批量      故障恢复
```

---

**最后更新**: 2026-05-24  
**基于版本**: Kafka 2.7.0+  
**难度等级**: 中级（理解生产消费和副本机制）
