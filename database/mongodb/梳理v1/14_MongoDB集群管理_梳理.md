# MongoDB 集群管理知识梳理

> 原文档: WM_MongoDB集群管理（三）.md
> 整理日期: 2026-05-24
> 内容覆盖: 副本集、分片架构、集群搭建、数据路由

---

## 一、核心概念导览

### 1.1 集群使用意义

**为什么需要集群?**
- 解决单机故障风险：数据冗余备份，多服务器存储副本
- 提高可用性 (HA, High Availability)
- 提升读写性能：分散负载、扩展容量

**高可用性定义**：通过缩短计划停机和非计划停机时间，提高系统可用性。

### 1.2 关键组件

| 组件 | 角色 | 特点 |
|------|------|------|
| **Mongos** | 数据路由、请求入口 | 无物理存存储，缓存元数据在内存，分发请求 |
| **Config Server** | 配置元数据存储 | 存储分片路由信息、集群拓扑，通常部署3个 |
| **Shard (Mongod)** | 分片数据存储 | 存储应用数据，支持副本集 |
| **Replica Set** | 副本集（复制集） | 数据冗余、高可用，单个Shard的副本 |
| **Arbiter** | 仲裁者节点 | 不存储数据，参与投票，选举主节点 |

---

## 二、副本集 (Replica Set)

### 2.1 定义与目的

**什么是副本集?**
- 一组 MongoDB 进程维护同一数据集
- 数据在多台服务器间同步

**副本集目的**
1. 保证数据冗余和可靠性
2. 提高数据可用性（主节点故障可切换）
3. 提高读取能力（从节点可读）
4. 支持离线维护

### 2.2 副本集特点

- N个节点的集群
- 任何节点可作为主节点（除仲裁节点）
- 所有写操作仅在主节点上执行
- 自动故障迁移
- 自动恢复

### 2.3 成员角色

#### Primary（主节点）
- 接收所有写入操作
- 包含所有操作日志（oplog）
- 从节点从主节点同步数据

#### Secondary（从节点）
- 可参与主节点选举
- 从主节点同步最新数据，保持一致
- 可配置为 Read Preference，提供读服务
- 提升副本集读服务能力和可用性

#### Arbiter（仲裁节点）
- **不存储数据**，不能被选为Primary
- 仅参与投票，决定哪个从节点晋升为主节点
- 占用资源少（轻量级）
- 当副本集成员为偶数时，建议添加以提升可用性

### 2.4 副本集架构方案

#### 方案A：3个数据节点（无Arbiter）
```
Primary (主库)
├── Secondary (从库1) - 可选为主
└── Secondary (从库2) - 可选为主

优点: 两个完整数据副本，主故障时两个从库都可竞选
```

#### 方案B：2个数据节点+1个仲裁节点
```
Primary (主库)
├── Secondary (从库) - 可选为主
└── Arbiter (仲裁者) - 仅投票

优点: 资源节省（arbiter不存储数据）
缺点: 仅一个完整数据副本，冗余度相对较低
```

### 2.5 Primary 选举机制

#### 大多数概念
- 投票成员总数: N，则大多数 = **N/2 + 1**
- 复制集必须有**大多数**成员存活才能选举出Primary
- 存活成员不足大多数 → 复制集无法提供写服务（只读状态）

**投票成员表**
| 投票成员数 | 大多数 | 容忍失效数 |
|-----------|--------|----------|
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 2 | 1 |
| 4 | 3 | 1 |
| 5 | 3 | 2 |
| 6 | 4 | 2 |
| 7 | 4 | 3 |

**建议**：副本集成员数设置为**奇数**
- 3个和4个节点容忍失效数相同（都是1个）
- 3个节点相比4个节点消耗资源少、部署简单
- 5个节点最优平衡：充足冗余度 + 容错能力强

#### 选举过程
1. 节点启动后，相互发送心跳信息
2. 获得**大多数**成员投票支持的节点成为Primary
3. 其余节点成为Secondary

---

## 三、Oplog（操作日志）

### 3.1 Oplog 概念
- Operation Log，记录 Primary 的所有写操作
- Secondary 通过同步 Oplog 保持与 Primary 数据一致
- Oplog 是一个**特殊的有界集合**（capped collection）

### 3.2 核心特性
- 记录写操作的完整历史（带时间戳）
- Secondary 按顺序重放 Oplog，实现异步复制
- Oplog 大小可配置（配置示例：oplogSize=4096MB）
- Oplog 大小影响副本集故障恢复时间

---

## 四、读写关注 (Read Preference & Write Concern)

### 4.1 Read Preference（读偏好）

允许配置从哪里读取数据，以分散读负载。

| 模式 | 说明 | 使用场景 |
|------|------|---------|
| **primary** | 仅从主节点读 | 默认，保证读最新数据 |
| **primaryPreferred** | 优先主节点，主不可用则从节点 | 大多数场景 |
| **secondary** | 仅从从节点读 | 分散读负载（可能读延迟数据） |
| **secondaryPreferred** | 优先从节点，无可用从节点则读主 | 读多场景 |
| **nearest** | 选择延迟最小的节点 | 地理分布式部署 |

**关键命令**：`db.setSecondaryOk()` 
- 允许从Secondary读取数据
- 不设置此参数时从Secondary查询报错：`NotPrimaryNoSecondaryOk`

### 4.2 Write Concern（写关注）

控制写操作的确认策略。

| 等级 | 说明 | 可靠性 | 性能 |
|------|------|--------|------|
| **0** | 不等待确认 | 最低 | 最快 |
| **1** | 等待主节点确认 | 中等 | 中等 |
| **majority** | 等待大多数节点确认 | 最高 | 最慢 |

---

## 五、分片架构 (Sharding)

### 5.1 分片的定义与目的

**什么是分片?**
- 将大型集合分割到多个服务器（分片）的方法
- 数据分散存储，提高存储容量和读写吞吐量

**分片目的**
- **存储扩展**：单机磁盘限制 → 分散多机
- **性能扩展**：单机CPU、内存、网卡限制 → 分散负载
- **自动均衡**：Balancer自动迁移数据保持均衡

### 5.2 两种扩展方向

| 方向 | 说明 | 缺点 |
|------|------|------|
| **垂直扩展** | 增加单机资源（CPU、内存、存储） | 成本高，有上限 |
| **水平扩展** | 数据分散多机（分片） | 架构复杂，但可无限扩展 |

### 5.3 分片集群架构

```
应用层
  ↓
[Mongos 1] [Mongos 2] [Mongos 3]  (路由层，多个做高可用)
  ↓    ↓    ↓
Config Server(3个，存储元数据)
  ↓    ↓    ↓
[Shard1副本集] [Shard2副本集] [Shard3副本集] ...  (分片层)
```

#### Config Server（配置服务器）
- 存储所有集群元数据（路由、分片信息）
- 通常部署**3个**（奇数，选举优选）
- Mongos启动时从ConfigServer加载配置
- ConfigServer变化时，通知所有Mongos更新状态

#### Mongos（分片路由）
- 数据库集群请求的**唯一入口**
- 无物理存储，仅在内存缓存元数据
- 根据分片键将请求路由到对应Shard
- 生产环境通常**多个Mongos**做高可用
- 负责chunk迁移和自动平衡协调

#### Shard（分片节点）
- 实际存储应用数据
- 每个Shard通常部署为副本集（数据冗余）
- 只负责其分片范围内的数据

### 5.4 分片集群的三大优势

1. **自动路由**
   - Mongos自动将请求路由到相应Shard
   - 应用层透明访问，无需修改代码

2. **保证高可用**
   - 分片+副本集结合：数据分散 + 数据备份
   - 单个Shard宕机不影响其他Shard

3. **易于扩展**
   - 按需添加新Shard，自动均衡数据
   - 支持动态扩容和缩容

---

## 六、Chunk 和均衡 (Balancer)

### 6.1 Chunk 概念

**Chunk 是什么?**
- 单个Shard内部的数据分块单位
- 默认大小: **64MB**
- 每个Chunk通过**分片键范围**标识

**Chunk 的两个主要用途**

#### Splitting（分裂）
- 当Chunk大小超过配置的ChunkSize时触发
- MongoDB后台进程自动执行
- 目的：避免Chunk过大导致迁移困难

#### Balancing（均衡）
- Balancer是后台进程，负责Chunk跨Shard迁移
- 目的：保持各Shard数据均衡
- 过程：从Chunk数量最多的Shard → 数量最少的Shard

### 6.2 Chunk 特点

- 存储需求 > 64MB → Chunk分裂
- 自动分裂和迁移（后台执行）
- Chunk分裂频繁，消耗IO资源
- **分裂只在写入和更新时触发，读操作不触发**

### 6.3 ChunkSize 的选择

**ChunkSize影响**
- 分裂频率、迁移频率、数据分布均匀度

| ChunkSize | 分裂频率 | 迁移速度 | 路由消耗 | 使用场景 |
|-----------|---------|---------|---------|---------|
| **小** (16-32MB) | 频繁 | 快 | 高 | 数据要求均衡分布 |
| **默认** (64MB) | 中等 | 中等 | 中等 | 通用 |
| **大** (100-200MB) | 少 | 集中消耗 | 低 | 写入集中，IO限制 |

**建议**：生产环境通常选择 **100-200MB**

**ChunkSize 改变的影响**
- ChunkSize只会分裂，**不会合并**
- 改小ChunkSize → 需时间分裂到目标大小
- 改大ChunkSize → 现有分块不减少，新写入才会达到新目标大小

---

## 七、分片键 (Shard Key)

### 7.1 分片键定义

**什么是分片键?**
- 集合中的一个字段（或字段组合）
- MongoDB按分片键值范围分割数据
- **分片键必须有索引**，通常由 `sh.shardCollection()` 自动创建

### 7.2 分片键注意事项

1. **一经设置，不可修改、不可删除**
2. **必须有索引**（分片键用于路由）
3. **大小限制**：512 Bytes
4. **不支持空值插入**
5. **用于路由查询**：根据分片键查询可直接定位Shard
6. **集合级分片**后，不能再次分片

### 7.3 分片键分类

#### 范围分片 (Range Sharding)

```
按分片键值范围分割数据

Shard1: [MinKey ... 1000)
Shard2: [1000 ... 2000)
Shard3: [2000 ... MaxKey]
```

**优点**
- 范围查询效率高，Mongos快速定位目标Shard
- 支持基于分片键的范围查询

**缺点**
- 容易产生热点：某个值范围数据集中
- 数据分布可能不均（如自增ID导致新数据总在最后一个Shard）
- 写操作集中，无法分散

#### 哈希分片 (Hash Sharding)

```
对分片键计算哈希值，按哈希值范围分割

Shard1: [hash($minKey) ... hash(N1))
Shard2: [hash(N1) ... hash(N2))
Shard3: [hash(N2) ... hash($maxKey)]
```

**优点**
- 数据分布**更加均匀**（相近值的文档不在同一Chunk）
- 写操作均衡分散到各Shard
- 充分扩展写能力

**缺点**
- **不支持范围查询**（哈希后顺序改变）
- 范围查询需分发到所有Shard（散-聚查询）
- 性能低于范围分片的范围查询

### 7.4 分片键选择标准

**好的分片键应具备**

1. **分布足够离散** (Sufficient Cardinality)
   - 字段取值范围广，避免重复值过多
   - 反例：数据中心作为分片键（取值少）

2. **写请求均匀分布** (Evenly Distributed Write)
   - 避免某个值的文档特别多（Jumbo Chunk）
   - 哈希分片更容易达到此目标

3. **避免散-聚查询** (Targeted Read)
   - 尽量支持基于分片键的查询
   - 非分片键查询时需分散到所有Shard

### 7.5 Jumbo Chunk 问题

**Jumbo Chunk 定义**
- 单个分片键值的文档特别多，导致Chunk过大
- 无法再分裂（限制：单Chunk最多250000文档）
- 无法迁移

**原因**
- 分片键基数太小（某值出现频率很高）
- ChunkSize设置过小

---

## 八、分片键设计案例

### 8.1 案例数据结构
```javascript
{
  "title": "文章标题",
  "by": "作者名称",
  "url": "http://example.com",
  "tags": ["语文", "数学"],
  "likes": 10000
}
```

数据量：百万 ~ 千万级

### 8.2 方案对比

#### 方案1：likes范围分片
```
分片键: likes (1表示升序/范围分片)
sh.shardCollection("test.blog", {likes: 1})
```
**问题**
- 新插入的文档likes通常连续增长
- 新数据总是写入同一个Shard（热点）
- 写分布不均，无法扩展写能力
- 按作者(by)查询需分发所有Shard

#### 方案2：likes哈希分片
```
分片键: likes (hashed)
sh.shardCollection("test.blog", {"likes": "hashed"})
```
**优点**
- 写入均分到多个Shard

**问题**
- 按作者(by)查询需分发所有Shard
- 范围查询(likes)需分发所有Shard

#### 方案3：by哈希分片
```
分片键: by ("hashed")
sh.shardCollection("test.blog", {"by": "hashed"})
```
**优点**
- 写入均分到多个Shard
- 按作者(by)查询直接定位单Shard

**问题**
- 同一作者的所有文档在一个Chunk（Jumbo Chunk风险）
- 按likes范围查询需全表扫描

#### 方案4：组合分片 (by+likes) ✓ 最优
```
分片键: (by, likes)，范围分片
sh.shardCollection("test.blog", {"by": "hashed", "likes": 1})
```

**优点**
- 写入均分到多个Shard（by哈希分散）
- 同一by的数据按likes进一步分散到多Chunk
- 按likes时间范围查询可利用复合索引
- 数据分布最均衡

**推荐此方案**

---

## 九、分片集群容量规划

### 9.1 存储型集群规划

**场景**：数据量大，访问量不高

**公式**
```
numberOfShards = N / M / 0.75

其中：
- N = 需要存储总量 (如 75GB)
- M = 单个Shard能存储 (如 1GB)
- 0.75 = 容量水位线（保留25%冗余）

numberOfMongos = 2+ (至少2个做高可用，访问不多)
```

**示例**
- 数据量: 75GB，单Shard容量: 1GB
- numberOfShards = 75 / 1 / 0.75 = 100 (需100个Shard)
- numberOfMongos = 2+

### 9.2 计算型集群规划

**场景**：并发量大，数据量相对小

**公式**
```
numberOfShards = Q / M / 0.75
numberOfMongos = Q / Ms / 0.75

其中：
- Q = 需要总QPS (Query Per Second)
- M = 单个Shard最大QPS
- Ms = 单个Mongos最大QPS
- 0.75 = 负载水位线
```

**示例**
- 总QPS需求: 100000
- 单Shard QPS: 1000
- 单Mongos QPS: 10000
- numberOfShards = 100000 / 1000 / 0.75 ≈ 134
- numberOfMongos = 100000 / 10000 / 0.75 ≈ 14

### 9.3 混合场景

若同时需解决存储和性能问题，**按需求更高的指标预估**。

---

## 十、集群部署流程

### 10.1 副本集搭建

#### 步骤1：准备工作
```bash
# 创建挂载目录
mkdir -p /tmp/mongo/mongo{1..3}/data
mkdir -p /tmp/mongo/conf

# 下载镜像
docker pull mongo

# 生成密钥文件
cd /tmp/mongo/conf
openssl rand -base64 90 -out ./keyfile
chmod 600 keyfile

# 配置mongodb.conf
dbpath = /data/db
port = 27017
oplogSize=4096
directoryperdb=true
bind_ip=0.0.0.0
replSet=mongo-repliset
keyFile=/etc/mongo/mongo.key
```

#### 步骤2：启动容器（3个节点）
```bash
docker run --net mongo-cluster \
  --restart always --name mongo1 -p 30001:27017 \
  -v /tmp/mongo/mongo1/data:/data/db \
  -v /tmp/mongo/conf:/etc/mongo \
  -d mongo -f /etc/mongo/mongodb.conf
# ... mongo2, mongo3 类似
```

#### 步骤3：初始化副本集
```javascript
rs.initiate({
  "_id": "mongo-repliset",
  "members": [
    {"_id": 0, "host": "mongo1:27017"},
    {"_id": 1, "host": "mongo2:27017"},
    {"_id": 2, "host": "mongo3:27017"}
  ]
})
```

#### 步骤4：验证
```javascript
db.hello()        // 查看集群信息
rs.isMaster()     // 查看节点角色
```

### 10.2 分片集群搭建

#### 架构
```
Config Server (3个) ↓
Mongos (多个) → [Shard1-RS] [Shard2-RS] [Shard3-RS] ...
```

#### 步骤1：启用数据库分片
```javascript
sh.enableSharding("test")  // 开启test数据库分片
```

#### 步骤2：创建分片键索引
```javascript
// 必须先建索引
db.blog.createIndex({"by": "hashed", "likes": 1})
```

#### 步骤3：指定分片键
```javascript
// 格式: sh.shardCollection("<db>.<collection>", {shardKey})
// 1 = 范围分片，"hashed" = 哈希分片

sh.shardCollection("test.blog", {"by": "hashed", "likes": 1})
```

#### 步骤4：查看分片状态
```javascript
sh.status()                  // 查看全局状态
db.blog.getShardDistribution()  // 查看数据分布
```

### 10.3 扩容操作

#### 新增Shard节点
```javascript
// 在mongos执行
sh.addShard("shard4/shard4-server1:27017,shard4-server2:27017")
```

Balancer**自动迁移数据**到新Shard，达到均衡。

### 10.4 缩容操作

#### 删除Shard
```javascript
// 第一次删除：开始迁移数据
use admin
db.runCommand({removeShard: "shard4"})
// 状态: "state": "started" 或 "ongoing"

// 等待迁移完成，再次执行
db.runCommand({removeShard: "shard4"})
// 状态: "state": "completed" 表示真正删除
```

Balancer自动将该Shard上的Chunk迁移到其他Shard。

---

## 十一、ASCII 流程图

### 11.1 副本集选举流程

```
初始化副本集 (rs.initiate)
        ↓
    各节点发送心跳消息
        ↓
    投票选举 (需获得N/2+1支持)
        ↓
    ┌─────────────────────────────┐
    ↓                             ↓
[Primary] 获胜                   [Secondary] × (N-1)
├─ 接收所有写操作        ├─ 同步Oplog
├─ 记录操作日志(Oplog)   ├─ 保持数据一致
└─ 通知所有Secondary    ├─ 参与下次选举
                        └─ 可配置为只读

Primary宕机 → Secondary竞选 → 新Primary产生
```

### 11.2 副本集写操作复制流程

```
应用 → Primary(主库)
        ↓
    写入数据 + 记录Oplog
        ↓
    发送Oplog给Secondary
        ↓
  ┌─────────────────┬──────────────────┬────────────────┐
  ↓                 ↓                  ↓                ↓
Secondary1      Secondary2         Secondary3      Arbiter
(应用Oplog)     (应用Oplog)        (应用Oplog)     (仅投票)
  ↓                 ↓                  ↓
 同步完成          同步完成           同步完成

应用可选择ReadPreference从Secondary读取
```

### 11.3 分片路由查询流程

```
应用层发起查询
  ↓
[Mongos] 解析查询条件
  ↓
    根据分片键值 查询ConfigServer元数据
  ↓
    ┌──────────────────────────────────────┐
    │ 确定数据所在Shard范围                 │
    └──────────────────────────────────────┘
  ↓
  ┌─────────────────────────────────────────────────┐
  │ 分片键查询 (Targeted Query)                      │
  │ → 直接转发到1个Shard，返回结果                   │
  └─────────────────────────────────────────────────┘
            或
  ┌─────────────────────────────────────────────────┐
  │ 非分片键查询 (Scatter-Gather)                     │
  │ → 分发到所有Shard并行查询                        │
  │ → Mongos对结果进行合并、排序                      │
  │ → 返回最终结果给应用                             │
  └─────────────────────────────────────────────────┘
```

### 11.4 Chunk分裂和迁移流程

```
新数据写入
  ↓
Chunk大小超过配置值 (默认64MB)
  ↓
[后台分裂进程] 自动分裂
  ↓
┌─────────────────────────────────────┐
│ Chunk分裂:                           │
│ Chunk A → Chunk A' + Chunk A''      │
└─────────────────────────────────────┘
  ↓
[Balancer检测] 各Shard Chunk数量
  ↓
  ┌──────────────────────────────────────────┐
  │ Chunk数量分布不均:                        │
  │ Shard1: 3个Chunk                         │
  │ Shard2: 5个Chunk (最多)                   │
  │ Shard3: 2个Chunk (最少)                   │
  └──────────────────────────────────────────┘
  ↓
[Balancer启动迁移]
  ↓
Shard2 → Shard3 迁移1个Chunk
  ↓
均衡完成
```

---

## 十二、⚠️ 原文勘误

### 勘误1：Mongodb.conf 配置项错误

**原文出现位置**: 第622行

**原文错误**:
```ini
systemctlLog:     # 错误写法
    destination: file
    logAppend: true
    path: /data/logs/mongo.log
```

**正确写法**:
```yaml
systemLog:        # 正确（不是systemctlLog）
  destination: file
  logAppend: true
  path: /data/logs/mongo.log
```

**影响**: 配置文件语法错误，MongoDB启动失败

---

### 勘误2：Java代码语法错误

**原文出现位置**: 第1417行

**原文错误**:
```java
blog Shelby(names[r.nextInt( bound: names.length - 1)]);
```

**正确写法**:
```java
blog.setBy(names[r.nextInt(names.length - 1)]);
```

**问题**:
- `Shelby` 是无效方法名
- `r.nextInt()` 参数应直接传整数，不用 `bound:` 键值对

---

### 勘误3：数据库命令拼写错误

**原文出现位置**: 第2057行

**原文错误**:
```javascript
db微信公众({user:"root",pwd:"root",roles:[{role:'root',db:'admin'}]})
```

**正确写法**:
```javascript
db.createUser({user:"root", pwd:"root", roles:[{role:'root', db:'admin'}]})
```

**问题**: `db微信公众` 是乱码/垃圾文本

---

### 勘误4：数据库命令拼写错误

**原文出现位置**: 第2063行

**原文错误**:
```javascript
shard4:PRIMARY> dbcreateUser(...)
```

**正确写法**:
```javascript
shard4:PRIMARY> db.createUser(...)
```

**问题**: 缺少`.` 分隔符

---

### 勘误5：Shard Key 定义表述不准确

**原文出现位置**: 第1028-1030行

**原文表述**:
```
分片键shard key

MongoDB中数据的分片是、以集合为基本单位的
```

**更准确表述**:
```
分片键 (Shard Key)

MongoDB 中对集合的分片是以集合为基本单位的。
集合中的数据通过分片键被分成多部分。
分片键是集合中选定的一个字段（或字段组合），
MongoDB按该字段的值范围或哈希值将数据分散到不同Shard。
```

**问题**: 原文表述有标点错误和冗余

---

### 勘误6：Chunk 迁移触发条件表述错误

**原文出现位置**: 第1001行

**原文表述**:
```
chunk分裂的时机：在插入和更新，读数据不会分裂。
```

**更准确表述**:
```
Chunk 分裂的触发条件：在插入和更新操作时触发
读数据不会触发分裂，无论读多频繁
```

**问题**: 原文缺少"触发"、"条件"等关键词，表述不够清晰

---

## 十三、核心知识点速查表

| 知识点 | 关键内容 | 备注 |
|--------|---------|------|
| **副本集选举** | N/2+1投票，奇数最优 | 3和4节点容错能力相同 |
| **Primary故障** | Secondary自动竞选 | 无需人工干预 |
| **Read Preference** | db.setSecondaryOk()允许从Secondary读 | 默认仅从Primary读 |
| **Oplog** | Secondary通过同步Oplog保持数据一致 | 时间戳标记，有界集合 |
| **Mongos** | 无物理存储，缓存元数据，分发请求 | 多个做高可用 |
| **Config Server** | 存储分片元数据，通常3个 | 宕机影响整个集群 |
| **分片键选择** | 离散度高，写均分，避免散-聚查询 | 一经设置不可修改 |
| **Chunk默认大小** | 64MB | 生产建议100-200MB |
| **范围分片** | 支持范围查询，易产生热点 | 新数据可能集中 |
| **哈希分片** | 数据分布均匀，不支持范围查询 | 范围查询需分散所有Shard |
| **组合分片** | (by, likes)最优平衡 | 联合哈希和范围优势 |
| **Balancer** | 自动均衡Chunk分布 | 可配置启用/禁用 |
| **Chunk分裂** | 写入时触发，后台自动 | 消耗IO资源 |
| **Jumbo Chunk** | 单分片键值文档过多，无法迁移 | 基数太小时易发生 |
| **扩容** | sh.addShard()，自动迁移 | Balancer自动重均衡 |
| **缩容** | removeShard分两步（开始→完成） | 等待Chunk迁移完成后确认 |

---

## 十四、学习路线建议

### 初级（理解概念）
1. 理解为什么需要集群（可用性、容量、性能）
2. 掌握副本集的三种角色（Primary, Secondary, Arbiter）
3. 理解Mongos、Config Server、Shard的分工

### 中级（搭建和使用）
1. 搭建3节点副本集，测试故障转移
2. 搭建简单分片集群（3个Shard）
3. 选择合理的分片键并验证数据分布

### 高级（优化和运维）
1. 分析ChunkSize对性能的影响
2. 优化分片键选择，避免Jumbo Chunk
3. 实施分片集群的监控和告警
4. 实践扩容、缩容、故障恢复流程

---

## 十五、常用命令速查

### 副本集命令
```javascript
rs.initiate()         // 初始化副本集
rs.isMaster()         // 查看主从信息
rs.status()           // 查看副本集状态
rs.add("host:port")   // 添加节点
rs.remove("host:port")// 删除节点
db.setSecondaryOk()   // 允许从Secondary读
```

### 分片命令
```javascript
sh.enableSharding("dbName")                    // 启用数据库分片
sh.shardCollection("db.col", {key:1})         // 指定分片键
sh.status()                                    // 查看分片状态
sh.addShard("shard/host1,host2,host3")       // 添加分片
db.collection.getShardDistribution()           // 查看数据分布
sh.getBalancerState()                          // 查看均衡器状态
db.runCommand({removeShard: "shardName"})     // 删除分片
```

### 数据库命令
```javascript
use dbName              // 切换数据库
db.auth(user, pwd)      // 认证
db.createUser({...})    // 创建用户
db.hello()              // 查看集群信息
```

---

**文档完成** ✓

总结：
- **章节数**: 15个主要章节
- **ASCII图数**: 4个流程图
- **勘误数**: 6处
