# Redis 高性能分布式缓存 - 系统梳理

## 核心概念

### Redis 高性能的四大原因

#### 1. 单线程模型
- **定义**：Redis在处理网络请求和K/V读写操作时采用单线程
- **优势**：避免多线程上下文切换开销，无并发竞争，简化设计
- **其他工作**：持久化、异步删除、集群同步采用额外线程

#### 2. IO多路复用（Multiplexing）
- **核心机制**：单线程通过epoll/select监听多个客户端连接
- **实现原理**：
  ```
  主线程 ─→ epoll监听 ─→ 有事件 ─→ 处理请求 ─→ 循环
  ```
- **效果**：支持数万并发连接（Linux epoll）

#### 3. 内存数据结构
- **数据存储位置**：全部在内存中，无磁盘I/O开销
- **高效数据结构**：
  - String（SDS简单动态字符串）：O(1)获取长度
  - Hash（数组+链表）：O(1)查询
  - List（双向链表）：两端O(1)操作
  - Set/ZSet（哈希表+跳跃表）：快速查找和有序操作

#### 4. Reactor事件循环模型
```
┌─────────────────────────────────────┐
│    Client连接                        │
└────────────────┬────────────────────┘
                 │
                 ▼
         ┌──────────────┐
         │ 事件监听     │(epoll_wait)
         │ (IO多路复用) │
         └──────┬───────┘
                │
        ┌──────┴────────┐
        │               │
        ▼               ▼
    ┌────────┐    ┌────────┐
    │读事件  │    │写事件  │
    └────┬───┘    └────┬───┘
         │             │
         └──────┬──────┘
                ▼
        ┌──────────────────┐
        │ 命令执行         │(单线程)
        │ processCommand   │
        └────────┬─────────┘
                 │
                 ▼
        ┌──────────────────┐
        │ 返回结果写回     │
        │ Socket发送缓冲区 │
        └──────────────────┘
```

---

## 主从复制架构

### 核心问题分析

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 单点故障 | 仅一个Master | 引入主从复制 |
| 容量瓶颈 | 单机内存限制 | 水平扩展，分片存储 |
| 并发写限制 | 一个Master | Cluster多Master |

### 主从复制原理

#### 全量同步（初次连接或repl_backlog溢出）

```
Master                            Slave
   │                              │
   │◄─────── PSYNC ? -1 ──────────┤
   │    (尝试全量同步)             │
   │                              │
   │────── FULLRESYNC ────────────►│(返回replicationId)
   │                              │
   ├─────── BGSAVE ──┐            │
   │(保存RDB快照)     │ Fork子进程 │
   │                 │            │
   │◄────────────────┘            │
   │                              │
   │────── RDB文件 ──────────────►│(清空内存)
   │      (大量数据)              │
   │                              │
   └──── 期间写命令 ────────────►│(缓冲)
        (存入repl_backlog)
```

**关键配置**：
```
repl_backlog_size=1mb    # 环形缓冲区大小（默认1M）
```

#### 增量同步（短时连接断开）

```
Master                            Slave
   │                              │
   │◄─────── PSYNC <replId> <offset>──┤
   │    (发送已知的replicationId和offset)
   │                              │
   │────── CONTINUE ─────────────►│(增量同步可行)
   │                              │
   └──── offset→end的命令 ──────►│(只发送差异部分)
```

**判断条件**：
1. replicationId必须匹配（Master数据集版本）
2. offset在repl_backlog中（环形缓冲区未溢出）

#### PSYNC命令机制

```
PSYNC命令处理流程：

Master端：
├─ 检查replId是否匹配
├─ 检查offset是否在repl_backlog范围内
├─ YES → 返回CONTINUE + 从offset到end的命令
└─ NO  → 返回FULLRESYNC + 触发bgsave

关键参数：
├─ master_replid：Master的伪随机字符串（记录数据集版本）
├─ master_repl_offset：当前写操作偏移量
└─ repl_backlog：环形缓冲区，记录[offset-size, offset)的写命令
```

### 配置与命令

```bash
# 从节点配置（三种方式）
# 方式1：配置文件
slaveof <master-ip> <master-port>

# 方式2：启动参数
redis-server --slaveof <master-ip> <master-port>

# 方式3：运行时命令
SLAVEOF <master-ip> <master-port>

# 查看复制信息
INFO replication

# 取消主从关系
SLAVEOF NO ONE
```

### 易混淆点：全量同步 vs 增量同步

| 特性 | 全量同步 | 增量同步 |
|------|---------|---------|
| 触发条件 | 初次连接或repl_backlog溢出 | 连接断开，offset在backlog范围 |
| 数据量 | 整个RDB | 缺失部分命令 |
| 网络开销 | 大 | 小 |
| Slave端处理 | 清空内存后加载RDB | 直接执行命令 |
| Master端操作 | BGSAVE（fork子进程） | 无额外操作 |

---

## 哨兵（Sentinel）架构与故障转移

### 核心问题
主从复制中，Master宕机后从节点无法自动晋升为主节点，需要人工干预。

### Sentinel架构设计

```
┌──────────────────┐
│   Client请求     │
└────────┬─────────┘
         │
    ┌────▼─────────────────────────────┐
    │     Sentinel哨兵集群（推荐3个+）  │
    │  ┌─────────┐  ┌─────────┐        │
    │  │Sentinel1│  │Sentinel2│        │
    │  └────┬────┘  └────┬────┘        │
    │       │            │             │
    │  ┌────▼────────────▼──┐          │
    │  │  监控Master状态    │          │
    │  │  PING/heartbeat   │          │
    │  └────┬───────────────┘          │
    │       │(Master宕机)               │
    │  ┌────▼───────────────┐          │
    │  │ 故障检测&选主      │          │
    │  │ (Raft投票)        │          │
    │  └────┬───────────────┘          │
    │       │                          │
    │  ┌────▼───────────────┐          │
    │  │ 选出新Master      │          │
    │  │ Slave→Master晋升  │          │
    │  └────┬───────────────┘          │
    │       │                          │
    │  ┌────▼───────────────┐          │
    │  │ 其他Slave复制新M  │          │
    │  │ 旧Master重启→Slave│          │
    │  └────────────────────┘          │
    └──────────────────────────────────┘
         │
    ┌────▼─────────────────────────────┐
    │   Redis Master-Slave集群         │
    │  ┌──────────┐  ┌──────────┐      │
    │  │ Master   │  │ Slave1   │      │
    │  └──────────┘  └──────────┘      │
    │  ┌──────────┐                    │
    │  │ Slave2   │                    │
    │  └──────────┘                    │
    └──────────────────────────────────┘
```

### Sentinel故障转移完整流程

```
1️⃣  主观下线检测
   └─ Sentinel发送PING命令→无响应(down-after-milliseconds)
     → Sentinel标记Master为SDOWN(Subjectively Down)

2️⃣  客观下线检测
   └─ 当多个Sentinel都标记为SDOWN → ODOWN(Objectively Down)

3️⃣  选举Leader Sentinel
   └─ Sentinel之间使用Raft算法选举：
     ├─ quorum个Sentinel投票支持 → 成为Leader
     └─ 负责故障转移

4️⃣  选举新Master
   └─ 从活跃Slave中选择：
     ├─ 优先级最高（slave-priority配置）
     ├─ offset最大（复制进度最新）
     ├─ runId最小（启动顺序）
     └─ 选出一个Slave

5️⃣  故障转移执行
   └─ SLAVEOF NO ONE    (新Master：取消从属)
     SLAVEOF new-master (其他Slave：复制新Master)
     旧Master重启 → 自动变为新Master的Slave

6️⃣  更新订阅者
   └─ 客户端通过Sentinel提供的API重定向到新Master
```

### Sentinel配置与启动

```bash
# sentinel.conf 配置示例
sentinel monitor mymaster 127.0.0.1 6379 1
#           └─ 被监控名称  └─┬────┘  └─ 需要多少投票

# Sentinel的其他关键配置
sentinel down-after-milliseconds mymaster 30000  # 30s无响应→SDOWN
sentinel parallel-syncs mymaster 1               # 故障转移时并行复制数
sentinel failover-timeout mymaster 180000        # 180s转移超时

# 启动哨兵（监听26379端口）
redis-sentinel sentinel.conf
# 或
redis-server sentinel.conf --sentinel
```

### 与主从模式的区别

| 特性 | 主从复制 | 哨兵模式 |
|------|---------|---------|
| 故障检测 | 无 | Sentinel自动检测 |
| 故障转移 | 手工干预 | 自动转移 |
| Master切换 | 需人工操作 | 自动选出新Master |
| 高可用 | 否 | 是 |
| 架构复杂度 | 简单 | 中等 |
| 生产环境 | 不推荐 | 推荐 |

### 易混淆点：哨兵 vs 主从 vs 集群

| 维度 | 主从复制 | 哨兵 | 集群 |
|------|---------|------|------|
| Master数量 | 1个 | 1个 | 多个 |
| 写能力 | 单Master写 | 单Master写 | 多Master写 |
| 自动故障转移 | 否 | 是 | 是 |
| 解决写压力 | 否 | 否 | 是 |
| 解决容量限制 | 否 | 否 | 是 |
| 槽分片 | 无 | 无 | 16384槽 |

---

## Cluster集群架构与槽分片

### 核心设计

**哨兵模式的问题**：
- 仍然只有一个Master（写能力单点）
- 每个节点保存全量数据（冗余）
- 无法横向扩展写性能

**Cluster解决方案**：
- 多Master架构，多点写
- 数据分片存储，节省内存
- 16384个槽位分布式管理

### 槽分片原理

```
┌─────────────────────────────────────────────────────────┐
│         Redis Cluster 16384槽位分配示意图               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Master1     │  │  Master2     │  │  Master3     │ │
│  │ Slot 0-5460  │  │Slot 5461-10922│ │Slot10923-16383│ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘ │
│         │                 │                 │          │
│    ┌────▼─┐           ┌───▼────┐       ┌───▼────┐    │
│    │Slave1│           │Slave2  │       │Slave3  │    │
│    └──────┘           └────────┘       └────────┘    │
│                                                     │
└─────────────────────────────────────────────────────────┘

CRC16哈希槽计算：
slot = CRC16(key) % 16384
```

### 请求路由与重定向

```
客户端请求KEY的完整流程：

1️⃣  客户端计算slot
   └─ slot = CRC16(key) % 16384

2️⃣  节点检查槽位归属
   ├─ 在自己管理的槽 → 执行命令，返回结果
   └─ 不在自己管理的槽 → 返回MOVED重定向

3️⃣  处理重定向
   ├─ MOVED 15495 127.0.0.1:9003  (永久重定向)
   │  └─ 槽位已迁移到新节点，更新路由表
   │
   └─ ASK 15495 127.0.0.1:9004     (临时重定向)
      └─ 槽位迁移中，仅本次请求重定向

4️⃣  客户端重新发送请求到目标节点
```

### 数据迁移与扩容缩容

```
扩容场景：新增Master节点

步骤1：创建新节点（无槽位）
       ├─ redis-server --port 9004 --cluster-enabled yes
       └─ CLUSTER NODES → 无slot分配

步骤2：加入集群
       ├─ redis-cli --cluster add-node 127.0.0.1:9004 127.0.0.1:9001
       └─ 建立握手，加入集群

步骤3：重新分片（reshard）
       ├─ redis-cli --cluster reshard 127.0.0.1:9001
       ├─ 选择源Master（如Master1）
       ├─ 选择目标Master（新Master）
       └─ 指定迁移槽位数量
       
步骤4：数据迁移流程
       ├─ 源Master：CLUSTER SETSLOT <slot> MIGRATING <target>
       ├─ 目标Master：CLUSTER SETSLOT <slot> IMPORTING <source>
       ├─ Redis-cli执行MIGRATE指令逐个迁移key
       │  └─ 源→目标：原子转移，包括TTL保留
       └─ 完成后CLUSTER SETSLOT <slot> NODE <target>

步骤5：验证（cluster slots/nodes）
```

### Cluster命令速查

```bash
# 集群管理
redis-cli --cluster create <node1> <node2> ... --cluster-replicas 1
redis-cli --cluster add-node <new-node> <existing-node>
redis-cli --cluster del-node <node> <node-id>
redis-cli --cluster reshard <node>           # 手动重新分片
redis-cli --cluster rebalance <node>         # 自动均衡所有槽

# 客户端连接（需-c参数自动重定向）
redis-cli -p 9001 -c
127.0.0.1:9001> set a a    # 自动重定向
127.0.0.1:9001> cluster nodes    # 查看集群拓扑
127.0.0.1:9001> cluster slots    # 查看槽分配

# 故障转移
CLUSTER FAILOVER [FORCE|TAKEOVER]   # 手动转移Master角色
```

### 与Sentinel区别

| 维度 | 哨兵 | 集群 |
|------|------|------|
| 架构 | Master-Slave+监控 | P2P分布式 |
| 高可用 | Sentinel监控 | 集群内置 |
| 写扩展 | 否（单Master） | 是（多Master） |
| 数据分片 | 否（全量保存） | 是（槽分片） |
| 通信开销 | 低 | 中等（节点间通信） |
| 应用场景 | 高可用+读写分离 | 高可用+大数据分片 |

---

## 持久化策略

### RDB（快照持久化）

```
RDB保存流程：

save命令（同步）            bgsave命令（异步）
    │                            │
    ▼                            ▼
┌─────────────┐          ┌──────────────┐
│主进程处理   │          │Fork子进程    │
│ 阻塞其他请求│          │继续处理请求  │
└──────┬──────┘          └────────┬─────┘
       │                          │
       ▼                          ▼
  暂存内存                   快照RDB文件
  快照                      保存到磁盘
       │                          │
       ▼                          ▼
   dump.rdb                  dump.rdb
  （阻塞期间）             （异步，无阻塞）

缺点：
├─ 最后一次快照后的数据可能丢失（取决于触发间隔）
└─ fork子进程会短时阻塞主线程
```

**配置示例**：
```
# redis.conf
save 900 1      # 900秒内有1个写入 → 触发RDB
save 300 10     # 300秒内有10个写入 → 触发RDB
save 60 10000   # 60秒内有10000个写入 → 触发RDB

dbfilename dump.rdb              # RDB文件名
dir /var/lib/redis/              # 保存路径
rdbcompression yes               # 是否压缩（消耗CPU，不推荐）
rdbchecksum yes                  # 保存校验和
stop-writes-on-bgsave-error yes  # bgsave失败时停止写（保护一致性）
```

### AOF（追加写操作日志）

```
AOF持久化流程：

Redis执行写命令：
    │
    ▼
┌──────────────────┐
│AOF缓冲区         │
│（aof_buf）       │
└────────┬─────────┘
         │(appendfsync策略触发)
    ┌────┴──────────────┬──────────────┐
    │                   │              │
    ▼                   ▼              ▼
 always            everysec           no
(同步写回)       (每秒写回)        (系统决定)
    │              │                  │
    └──────┬───────┴──────────────┬───┘
           │                      │
           ▼                      ▼
      AOF文件                  系统缓冲区
   appendonly.aof         (fdatasync触发)
```

**appendfsync策略对比**：

| 策略 | 写回时机 | 性能 | 丢失数据量 |
|------|---------|------|----------|
| always | 每个命令后 | 最低 | 最少(0条) |
| everysec | 每秒 | 中等 | ≤1秒数据 |
| no | 系统决定 | 最高 | 系统缓冲区数据 |

**AOF重写机制**：

```
触发条件：
├─ auto-aof-rewrite-percentage 100
│  (当前AOF大小是上次重写后的100%触发)
│
└─ auto-aof-rewrite-min-size 64mb
   (文件大小达到64M触发)

重写流程：
主进程 ─┬─ Fork子进程
        │  ├─ 读取内存中所有key
        │  └─ 生成最少命令写入temp-aof
        │
        └─ 继续处理新的写命令
           └─ 写入重写缓冲区
              (aof_rewrite_buf)

        子进程完成 ─┬─ 信号处理函数
                  ├─ 将重写缓冲区数据追加到temp-aof
                  └─ rename temp-aof → appendonly.aof
```

### 持久化优先级与选择

```
加载优先级（启动时）：
1️⃣  AOF存在 ─→ 加载AOF
2️⃣  RDB存在 ─→ 加载RDB
3️⃣  都存在 ─→ AOF优先（数据更新）

选择建议：
├─ 追求数据安全 → AOF(appendfsync=always) + 定期RDB
├─ 追求高性能   → 关闭AOF，仅RDB
├─ 均衡方案    → AOF(appendfsync=everysec) + RDB
└─ 纯缓存场景   → 都关闭（容忍数据丢失）
```

---

## 过期删除与内存淘汰策略

### 过期删除策略

```
三种经典策略对比：

定时删除                惰性删除           定期删除
│                       │                   │
├─ 优点：               ├─ 优点：            ├─ 优点：
│  └─ 节省内存          │  └─ CPU消耗低      │  └─ 均衡
│                       │                   │
├─ 缺点：               ├─ 缺点：            ├─ 缺点：
│  └─ CPU消耗高         │  └─ 内存浪费      │  └─ 难精准控制
│    (为每个key建定时器) │    (过期键不删)   │
│                       │                   │
└─ 时间复杂度：O(1)    └─ 时间复杂度：O(N) └─ 时间复杂度：O(N)

Redis采用：惰性删除 + 定期删除 (组合策略)
```

**实现原理**：

```
惰性删除：
  ├─ 在expireIfNeeded函数中
  ├─ 所有读/写命令执行前
  └─ 检查key是否过期 → 删除

定期删除：
  ├─ activeExpireCycle函数
  ├─ 默认每秒执行10次 (hz配置)
  ├─ 每次执行逻辑：
  │  ├─ 采样ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP个key(默认20)
  │  ├─ 删除其中所有过期key
  │  └─ 若>25%的key过期 → 重复采样，直到<25%
  │
  └─ 配置：hz 10  (每秒运行次数)
```

### 内存淘汰策略

```
触发条件：
  ├─ maxmemory达到上限
  └─ maxmemory-policy决定淘汰方式

8种策略分类：

┌─────────────────────────────────────────────┐
│                 内存淘汰策略                  │
├──────────────────┬──────────────────────────┤
│ 不淘汰           │ noeviction(默认)         │
│                  │ └─ 内存满时拒绝写操作    │
├──────────────────┼──────────────────────────┤
│ 考虑过期时间     │ volatile-ttl             │
│ 的key            │ └─ 删除过期时间最早的key│
├──────────────────┼──────────────────────────┤
│ 考虑访问频率     │ volatile-lfu             │
│ 的key            │ └─ 删除访问频率最低的key│
│                  │ (Redis 4.0+)             │
├──────────────────┼──────────────────────────┤
│ 考虑访问时间     │ volatile-lru             │
│ 的key            │ └─ 删除最近最少使用的key│
├──────────────────┼──────────────────────────┤
│ 所有key          │ allkeys-lfu              │
│                  │ └─ 所有key中删频率最低  │
│                  │ allkeys-lru              │
│                  │ └─ 所有key中删最近少用  │
│                  │ allkeys-random           │
│                  │ └─ 随机删除              │
├──────────────────┼──────────────────────────┤
│ 指定key集合      │ volatile-random          │
│                  │ └─ 随机删除过期key      │
└──────────────────┴──────────────────────────┘

Redis的近似LRU/LFU：
  ├─ 随机采样maxmemory-samples个key(默认5)
  ├─ 从中选择最符合策略的key删除
  └─ 官方推荐5-10个采样即可
```

**LRU vs LFU对比**：

```
LRU (Least Recently Used)：
  ├─ 记录：最后访问时间(24bit)
  ├─ 场景：周期性热数据
  └─ 问题：偶尔访问的数据易被误认为热数据

LFU (Least Frequently Used)：
  ├─ 记录：访问频率(8bit) + 访问时间(16bit)
  ├─ 场景：长期热数据识别
  └─ 优势：准确识别真正热数据

在Redis中的字段实现：
  ├─ 24bit LRU字段拆分为：
  │  ├─ 前16bit：ldt (Last Decrement Time)访问时间戳
  │  └─ 后8bit：counter 访问计数器
  │
  └─ LFU模式下：
     ├─ counter >0时递增(单调性)
     └─ counter在一段时间无访问时递减
```

### Key设计规范

```
可读性与可管理性：
  ├─ 使用业务名前缀：ugc:video:1
  ├─ 用冒号分隔：业务:表:ID
  └─ 禁用特殊字符：空格、换行、引号

简洁性：
  ├─ 在保证语义前提下控制长度
  ├─ 长key占用内存（SDS结构元数据）
  │  ├─ 1~31字节：1字节元数据
  │  ├─ 32~255字节：3字节元数据
  │  ├─ 256~65535字节：5字节元数据
  │  └─ 65536+字节：9字节元数据
  │
  └─ 优化：u:{uid}:fr:m:{mid} 替代冗长名称

避免Big Key：
  ├─ String：控制<10KB
  ├─ Collection：元素个数<10000
  │
  ├─ 危害：
  │  ├─ 网络传输慢
  │  ├─ 内存消耗多
  │  ├─ 阻塞主线程
  │  └─ 从节点加载慢
  │
  └─ 检测方法：
     ├─ redis-cli --bigkeys
     └─ DEBUG OBJECT key (查看encoding和serializedlength)
```

---

## Redis性能瓶颈与优化

### 常见瓶颈分析

```
性能瓶颈识别：

1️⃣  网络I/O瓶颈
   ├─ 现象：单条大key请求耗时长
   ├─ 原因：序列化/网络传输
   └─ 优化：
      ├─ 减小value大小(分割大value)
      ├─ 使用管道(Pipeline)批量操作
      └─ 启用Cluster多键分散

2️⃣  命令延迟瓶颈
   ├─ 现象：某些命令响应慢
   ├─ 原因：O(N)操作(KEYS/SCAN/SORT)
   └─ 优化：
      ├─ 禁用KEYS，用SCAN迭代
      ├─ 分散大集合为多个小集合
      └─ 使用Lua脚本批量操作

3️⃣  内存瓶颈
   ├─ 现象：淘汰率高，命中率低
   ├─ 原因：热数据超过maxmemory
   └─ 优化：
      ├─ 扩容或分片(Cluster)
      ├─ 调整淘汰策略(LFU识别热数据)
      └─ 压缩数据结构(压缩列表)

4️⃣  持久化瓶颈
   ├─ 现象：写命令响应时间长
   ├─ 原因：RDB fork阻塞或AOF同步写
   └─ 优化：
      ├─ 使用bgsave，禁用同步save
      ├─ AOF改为everysec或no
      ├─ 降低RDB触发频率
      └─ 关闭压缩(rdbcompression=no)
```

### 优化清单

| 优化项 | 配置项 | 推荐值 | 原因 |
|--------|--------|--------|------|
| 内存 | maxmemory | 实际可用内存80% | 留余量避免OOM |
| 淘汰策略 | maxmemory-policy | allkeys-lfu | 准确识别热数据 |
| RDB | save | 调整采样参数 | 避免频繁保存 |
| AOF | appendfsync | everysec | 平衡安全性和性能 |
| 慢查询 | slowlog-log-slower-than | 10000µs | 记录>10ms的命令 |
| 客户端输出 | client-output-buffer-limit | 合理调整 | 防止OOM |

---

## 内存模型与数据结构

### String（字符串）- SDS简单动态字符串

```
数据结构：

┌──────────────────────────────────────┐
│ SDS (Simple Dynamic String)          │
├──────────────────────────────────────┤
│ len: 5        (已用长度)             │
│ alloc: 10     (分配长度)             │
│ flags: 1      (字符串类型)           │
│ buf: "hello\0____"  (缓冲区)        │
│      ├─实际内容     ├─预留空间      │
│      └─可快扩展     (避免频繁扩展) │
└──────────────────────────────────────┘

优势：
├─ O(1)获取长度(直接读len字段)
├─ 二进制安全(支持任意字符)
└─ 空间预留(降低扩容开销)

应用：
├─ 缓存(单值缓存)
├─ 计数器(INCR/DECR)
└─ 分布式锁(SETNX)
```

### Hash（哈希表）- 数组+链表

```
数据结构：

┌─────────────────────────────────────┐
│         Hash表                       │
├─────────────────────────────────────┤
│ 数组(hash bucket):                  │
│  ┌─┐  ┌─┐  ┌─┐  ┌─┐  ┌─┐          │
│  │●┼──●┼──●┼──●┼──●┼─┐            │
│  └─┘  └─┘  └─┘  └─┘  └─┘  NULL      │
│   │    │    │    │    │             │
│   ▼    ▼    ▼    ▼    ▼             │
│  Entry Entry Entry Entry Entry      │
│  (key) (key) (key) (key) (key)      │
│  value value value value value      │
│
│  渐进式rehash：
│  ├─ 触发条件：负载因子(used/size) > 1
│  ├─ 维护双哈希表：old_table, new_table
│  └─ 每个命令执行时增量迁移
│     (避免一次性阻塞)
└─────────────────────────────────────┘

应用：
├─ 对象缓存(user:1:name, user:1:age)
├─ 购物车(cart:uid → field:productId, value:count)
└─ 会话存储(session:sessionId)

时间复杂度：
├─ HGET/HSET/HDEL: O(1)
├─ HGETALL: O(N)   (遍历所有field)
└─ HSCAN: O(1)     (迭代方式)
```

### List（列表）- 双向链表

```
数据结构：

┌──────────────────────────────────┐
│  List (双向链表)                  │
├──────────────────────────────────┤
│                                  │
│  head◄──┬──►┌──────┐◄──┬──►     │
│        │  │ │Node 1│  │        │
│        │  │ │prev: ◄──┘        │
│        │  │ │next: ──┐         │
│        │  │ │value:10│         │
│        │  │ └──────┘ │         │
│        │  └──────────┤         │
│        │             │         │
│        └─────────────►┌──────┐◄─┘
│                      │Node 2│
│                      │prev: ◄──┐
│                      │next: ──►│
│                      │value:20│
│                      └──────┘ │
│                               ▼
│                             tail
│
│  内部优化：
│  ├─ Redis 3.2+使用quicklist
│  ├─ 链表+压缩列表组合
│  └─ 减少内存碎片
└──────────────────────────────────┘

操作复杂度：
├─ LPUSH/RPUSH/LPOP/RPOP: O(1)  (两端操作)
├─ LINDEX: O(N)                 (中间查找)
├─ LRANGE: O(N)                 (范围返回)
└─ LTRIM: O(N)

应用：
├─ 消息队列(LPUSH+RPOP)
├─ 阻塞队列(BLPOP)
├─ 关注流(LPUSH+LRANGE)
└─ 排行榜(LRANGE)
```

### Set（集合）与ZSet（有序集合）

```
Set数据结构：

┌─────────────────────────────┐
│   Set (哈希表)               │
├─────────────────────────────┤
│   哈希表存储member           │
│   ┌──────┐  ┌──────┐        │
│   │member1  │member2  ...   │
│   └──────┘  └──────┘        │
│                             │
│  操作复杂度：                │
│  ├─ SADD/SREM/SISMEMBER: O(1)│
│  ├─ SMEMBERS: O(N)          │
│  └─ SINTER/SUNION: O(N+M)   │
│                             │
│  应用：                      │
│  ├─ 标签系统(去重)          │
│  ├─ 共同关注(SINTER)        │
│  └─ 抽奖(SPOP)              │
└─────────────────────────────┘

ZSet数据结构：

┌──────────────────────────────────┐
│  ZSet (哈希表 + 跳跃表)            │
├──────────────────────────────────┤
│                                  │
│  哈希表：O(1)查询score           │
│  ┌─────────────────────────┐     │
│  │member → score映射表      │     │
│  └─────────────────────────┘     │
│                                  │
│  跳跃表：O(logN)范围查询         │
│  ┌────┐  ┌────┐  ┌────┐        │
│  │    ├──►│    ├──►│    │       │
│  ├────┤  ├────┤  ├────┤        │
│  │ m1 ├──►│ m2 ├──►│ m3 │      │
│  ├────┤  ├────┤  ├────┤        │
│  │    ├──►│    │  │    │       │
│  └────┘  └────┘  └────┘        │
│   ↑       ↑       ↑             │
│  score: score: score:          │
│    10     20      30            │
│                                  │
│  操作复杂度：                     │
│  ├─ ZADD/ZREM: O(logN)          │
│  ├─ ZRANGE: O(logN+count)       │
│  └─ ZRANGEBYSCORE: O(logN+count)│
│                                  │
│  应用：                          │
│  ├─ 排行榜(score=分数)          │
│  ├─ 热搜(score=热度)            │
│  └─ 时间序列(score=时间戳)      │
└──────────────────────────────────┘
```

---

## 分布式锁实现

### 问题分析

```
场景：秒杀场景中的库存超卖

┌─────────────┐     ┌──────────┐     ┌──────────┐
│ Client 1    │     │ Client 2 │     │ Client N │
└──────┬──────┘     └────┬─────┘     └────┬─────┘
       │                 │                 │
       └────────┬────────┴────────┬────────┘
                │                │
                ▼                ▼
            ┌────────────────────────┐
            │  GET库存数量(库存=100) │
            └────────────────────────┘
                │
       ┌────────┴─────────┬──────────┐
       ▼                  ▼          ▼
   ┌───┐            ┌───┐      ┌───┐
   │99 │            │99 │      │99 │
   │SET│            │SET│      │SET│
   └───┘            └───┘      └───┘
   
 结果：库存减1三次，但实际只卖了1件！(超卖)

根本原因：
  ├─ GET和SET非原子操作
  └─ 多个客户端并发执行导致RaceCondition
```

### 基础分布式锁实现

```
版本演进：

【V1】简单SETNX + EXPIRE
────────────────────────
问题：非原子性，容易死锁
    SETNX lock 1    ─┐
                     ├─ 之间宕机→死锁
    EXPIRE lock 5   ─┘

【V2】SETNX with EX (原子操作)
──────────────────────────────
SET lock <requestId> EX 5 NX
  └─ 原子性：获取锁+设置过期时间
  
问题：没有锁续期，业务超时会丢锁

【V3】SETNX + Lua释放 + Watchdog续期
──────────────────────────────────────
获取：SET lock <uniqueId> EX 5 NX
      ├─ uniqueId = UUID + ThreadId
      └─ 防止解错锁

续期：Watchdog线程定时检查
      ├─ 若持有锁则EXPIRE续期
      └─ 防止业务超时导致锁失效

释放：Lua脚本检查uniqueId再删除
      └─ 保证"解铃还须系铃人"

【V4】可重入锁 (Redisson实现)
──────────────────────────────
使用Hash数据结构：
  ├─ key: lock_name
  ├─ field: threadId (线程唯一标识)
  └─ value: 重入计数

加锁：
  ├─ HSET lock <threadId> 1   (首次)
  ├─ HINCRBY lock <threadId> 1 (重入)
  └─ EXPIRE lock 30            (续期)

释放：
  ├─ HINCRBY lock <threadId> -1 (重入次数-1)
  ├─ 若counter > 0 → 延长过期时间
  └─ 若counter = 0 → 删除key并通知等待者
```

### Lua脚本原子性保证

```
加锁Lua脚本：
──────────────
if redis.call('exists', KEYS[1]) == 0 then
    redis.call('hset', KEYS[1], ARGV[2], 1)
    redis.call('pexpire', KEYS[1], ARGV[1])
    return nil
end
if redis.call('hexists', KEYS[1], ARGV[2]) == 1 then
    redis.call('hincrby', KEYS[1], ARGV[2], 1)
    redis.call('pexpire', KEYS[1], ARGV[1])
    return nil
end
return redis.call('pttl', KEYS[1])

解锁Lua脚本：
──────────────
if redis.call('exists', KEYS[1]) == 0 then
    redis.call('publish', KEYS[2], ARGV[1])
    return 1
end
if redis.call('hexists', KEYS[1], ARGV[3]) == 0 then
    return nil
end
local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1)
if counter > 0 then
    redis.call('pexpire', KEYS[1], ARGV[2])
    return 0
else
    redis.call('del', KEYS[1])
    redis.call('publish', KEYS[2], ARGV[1])
    return 1
end

优势：
├─ 原子性：Lua脚本作为整体执行
├─ 无竞态条件：中间不被其他命令中断
└─ 复用：脚本存储在服务端，客户端只需发送脚本SHA
```

### 阻塞锁实现（Redisson）

```
非阻塞 vs 阻塞：

【非阻塞】
获取失败 ─┐
          ├─ 直接返回false
          └─ 业务层需轮询重试(消耗CPU)

【阻塞】(pub/sub模式)
获取失败 ─┬─ 订阅解锁消息
          │  (redisson_lock_channel:{key})
          │
          ├─ 等待解锁通知
          │  (BLPOP或CountDownLatch)
          │
          ├─ 收到通知 ─┬─ 再次尝试获取锁
          │           └─ 成功则继续
          │
          └─ 超时 → 放弃，返回获取失败
```

### 与其他分布式锁对比

| 方案 | 优点 | 缺点 | 场景 |
|------|------|------|------|
| Redis | 高性能、简单 | 单点、无强一致 | 对一致性要求不高 |
| Zookeeper | 强一致、可靠 | 性能低、复杂 | 金融级别系统 |
| 数据库 | 强一致 | 性能低 | 小规模、一致性要求高 |
| etcd | 一致性强 | K8s生态专属 | Kubernetes应用 |

---

## ⚠️ 原文勘误

1. **槽位计算错误**
   - 原文：`把hash结果对16383进行取余`
   - 正确：应为`对16384进行取余`(0-16383共16384个槽)

2. **Redis多线程误解**
   - 原文：描述略显笼统
   - 澄清：Redis 6.0+多线程仅用于网络IO读写，命令执行仍为单线程

3. **Sentinel投票数配置**
   - 原文：`sentinel monitor mymaster 127.0.0.1 6379 1`中的1
   - 澄清：这个1是quorum票数(需要多少个Sentinel同意)，非Master个数

---

## 核心知识点速查表

### 三种部署模式对比

| 特性 | 主从复制 | 哨兵(Sentinel) | 集群(Cluster) |
|------|---------|-----------------|----------------|
| **Master个数** | 1个 | 1个 | 多个(≥3) |
| **故障检测** | 无 | Sentinel监控 | 集群内置 |
| **自动转移** | 否 | 是 | 是 |
| **写扩展能力** | 否 | 否 | 是 |
| **数据分片** | 否 | 否 | 是(16384槽) |
| **高可用性** | ✗ | ✓ | ✓ |
| **内存冗余** | 高(全量) | 高(全量) | 中等(分片) |
| **应用场景** | 备份、读写分离 | 高可用+读写分离 | 大数据+高并发 |

### 关键配置速查

| 配置项 | 含义 | 推荐值 |
|--------|------|--------|
| **maxmemory** | 最大内存 | 实际可用的80% |
| **maxmemory-policy** | 淘汰策略 | allkeys-lfu |
| **appendfsync** | AOF同步策略 | everysec |
| **slowlog-log-slower-than** | 慢查询阈值 | 10000(µs) |
| **hz** | 定期删除频率 | 10(次/秒) |
| **repl_backlog_size** | 主从缓冲区 | 1mb(可根据网络调整) |
| **cluster-enabled** | 集群开启 | yes/no |
| **cluster-node-timeout** | 节点超时 | 15000(ms) |

### 持久化选择清单

```
【追求最强一致性】
├─ AOF: appendfsync=always
├─ RDB: save规则保守
└─ 缺点：性能最低(~30k QPS)

【推荐生产配置】
├─ AOF: appendfsync=everysec
├─ RDB: 定期手动bgsave
└─ 优点：丢失≤1秒数据，性能可接受

【性能优先(容忍丢失)】
├─ 关闭AOF
├─ RDB: 仅定期手动触发
└─ 注意：宕机丢失最后一次快照后数据

【纯缓存(无需持久化)】
├─ 关闭所有持久化
└─ 好处：性能最高
```

### 性能优化清单

- [ ] 禁用rdbcompression(减少CPU)
- [ ] 调整save规则(避免频繁RDB)
- [ ] 使用Pipeline(批量操作)
- [ ] 启用Cluster(分散热点)
- [ ] 定期使用SCAN替代KEYS
- [ ] 分割大value(避免Big Key)
- [ ] 使用Lua脚本(原子操作)
- [ ] 设置合理的超时时间
- [ ] 监控slowlog(发现慢查询)
- [ ] 采用LFU策略(识别热数据)

---

## 常见面试题速查

**Q1: Redis为什么单线程这么快？**  
A: (1)内存存储无磁盘IO (2)高效数据结构 (3)IO多路复用(epoll) (4)避免多线程上下文切换

**Q2: Redis6.0为什么引入多线程？**  
A: 瓶颈不在CPU而在网络I/O，多线程仅用于读写Socket，命令执行仍单线程

**Q3: 主从、哨兵、集群怎么选？**  
A: 小规模读多→主从；需高可用→哨兵；大数据多Master写→集群

**Q4: 如何实现分布式锁？**  
A: SET lock <uniqueId> EX 5 NX + Lua脚本释放 + Watchdog续期

**Q5: 缓存穿透、击穿、雪崩的区别？**  
A: 穿透(穿过缓存数据库都没有) 击穿(单key热点过期) 雪崩(多key集中失效)

---

**文档完成时间**: 2026/05/24  
**Redis版本参考**: 3.2 ~ 6.2  
**涵盖重点**: 单线程模型、主从复制、哨兵、集群、持久化、锁、优化
