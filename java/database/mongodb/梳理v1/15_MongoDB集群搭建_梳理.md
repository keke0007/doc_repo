# MongoDB 集群搭建完全梳理

## 第一章 MongoDB 分布式架构概述

### 1.1 核心概念

| 组件 | 说明 | 职责 |
|------|------|------|
| **Config Server** | 配置服务器 | 存储分片集群的元数据和配置信息，采用副本集部署 |
| **Shard** | 分片数据节点 | 存储实际数据，每个分片是独立的副本集 |
| **Mongos** | 路由层 | 查询和写入的统一入口，无状态节点，负责转发请求 |
| **Replica Set** | 副本集 | 提供高可用性，包含 Primary、Secondary、Arbiter |
| **Arbiter** | 仲裁节点 | 参与投票不存储数据，用于奇数个节点配置 |

### 1.2 集群拓扑架构

```
                    ┌─────────────────┐
                    │   Client App    │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Mongos Router  │
                    │  (Port 27017)   │
                    └────────┬────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
      ┌─────▼──────┐  ┌─────▼──────┐  ┌─────▼──────┐
      │  Shard 1   │  │  Shard 2   │  │  Shard N   │
      │ Replica Set│  │ Replica Set│  │ Replica Set│
      └────────────┘  └────────────┘  └────────────┘
            │                │                │
            └────────────────┼────────────────┘
                             │
                    ┌────────▼────────┐
                    │ Config Server   │
                    │  Replica Set    │
                    │  (3 nodes)      │
                    └─────────────────┘
```

---

## 第二章 副本集搭建步骤详解

### 2.1 Config Server 副本集搭建

#### 步骤 1：创建目录结构

```bash
# 创建配置文件目录
mkdir -p /opt/mongodb/mongo-cluster/config-server/conf

# 创建数据文件目录（3 个节点）
mkdir -p /opt/mongodb/mongo-cluster/config-server/data/{1..3}

# 创建日志文件目录
mkdir -p /opt/mongodb/mongo-cluster/config-server/logs/{1..3}
```

#### 步骤 2：生成并配置密钥文件

```bash
# 创建 base64 编码的密钥文件
openssl rand -base64 756 > /opt/mongodb/mongo-cluster/config-server/conf/mongo.key

# 设置权限为 600（仅所有者可读写）
chmod 600 /opt/mongodb/mongo-cluster/config-server/conf/mongo.key
```

#### 步骤 3：创建共享配置文件

所有三个节点使用同一配置文件：

```yaml
# 日志文件配置
storage:
    dbPath: /data/db                    # 数据存储目录

systemLog:
    destination: file
    logAppend: true
    path: /data/logs/mongo.log         # 日志文件路径

# 网络设置
net:
    port: 27017                        # 服务端口
    # bindIp: 127.0.0.1               # 绑定 IP（注释掉允许任何 IP）

# 副本集配置
replication:
    replSetName: configsvr             # Config Server 副本集名称

# 分片配置
sharding:
    clusterRole: configsvr             # 标记为配置节点

# 安全认证
security:
    authorization: enabled
    keyFile: /data/configdb/conf/mongo.key
```

#### 步骤 4：启动三个 Config Server 容器

```bash
# 启动 config-server1
docker run --name config-server1 -d \
    --net=mongo-cluster \
    --privileged=true \
    -v /opt/mongodb/mongo-cluster/config-server:/data/configdb \
    -v /opt/mongodb/mongo-cluster/config-server/data/1:/data/db \
    -v /opt/mongodb/mongo-cluster/config-server/logs/1:/data/logs \
    mongo --config /data/configdb/conf/mongo.conf

# 启动 config-server2 和 config-server3（仅改变数据和日志目录编号）
docker run --name config-server2 -d \
    --net=mongo-cluster \
    --privileged=true \
    -v /opt/mongodb/mongo-cluster/config-server:/data/configdb \
    -v /opt/mongodb/mongo-cluster/config-server/data/2:/data/db \
    -v /opt/mongodb/mongo-cluster/config-server/logs/2:/data/logs \
    mongo --config /data/configdb/conf/mongo.conf
```

#### 步骤 5：初始化 Config Server 副本集

```bash
# 进入第一个容器
docker exec -it config-server1 bash
mongo -port 27017

# 在 MongoDB 客户端执行初始化命令
rs.initiate({
    _id: "configsvr",
    members: [
        { _id: 1, host: "config-server1:27017" },
        { _id: 2, host: "config-server2:27017" },
        { _id: 3, host: "config-server3:27017" }
    ]
})
```

#### 步骤 6：创建管理员用户

```javascript
use admin
db.createUser({
    user: "root",
    pwd: "root",
    roles: [{ role: 'root', db: 'admin' }]
})
```

### 2.2 Shard 分片副本集搭建

#### 步骤 1：创建目录结构

以 Shard1 为例：

```bash
# 创建配置文件目录
mkdir -p /opt/mongodb/mongo-cluster/shard1-server/conf

# 创建数据文件目录（可多于 3 个）
mkdir -p /opt/mongodb/mongo-cluster/shard1-server/data/{1..4}

# 创建日志文件目录
mkdir -p /opt/mongodb/mongo-cluster/shard1-server/logs/{1..4}
```

#### 步骤 2：复制密钥文件

```bash
cp /opt/mongodb/mongo-cluster/config-server/conf/mongo.key \
   /opt/mongodb/mongo-cluster/shard1-server/conf/
```

#### 步骤 3：创建 Shard 配置文件

```yaml
storage:
    dbPath: /data/db

systemLog:
    destination: file
    logAppend: true
    path: /data/logs/mongo.log

net:
    port: 27017

replication:
    replSetName: shard1                # 分片的副本集名称

sharding:
    clusterRole: shardsvr              # 标记为分片节点

security:
    authorization: enabled
    keyFile: /data/configdb/conf/mongo.key
```

#### 步骤 4：启动分片容器

```bash
# 启动 3 个节点，第 4 个数据节点用于扩容演示
docker run --name shard1-server1 -d \
    --net=mongo-cluster \
    --privileged=true \
    -v /opt/mongodb/mongo-cluster/shard1-server:/data/configdb \
    -v /opt/mongodb/mongo-cluster/shard1-server/data/1:/data/db \
    -v /opt/mongodb/mongo-cluster/shard1-server/logs/1:/data/logs \
    mongo --config /data/configdb/conf/mongo.conf
```

#### 步骤 5：初始化分片副本集（含仲裁节点）

```javascript
docker exec -it shard1-server1 bash
mongo -port 27017

# 初始化：使用 arbiterOnly:true 标记第 3 个节点为仲裁
rs.initiate({
    _id: "shard1",
    members: [
        { _id: 0, host: "shard1-server1:27017" },
        { _id: 1, host: "shard1-server2:27017" },
        { _id: 2, host: "shard1-server3:27017", arbiterOnly: true }
    ]
})
```

#### 步骤 6：创建用户

```javascript
use admin
db.createUser({
    user: "root",
    pwd: "root",
    roles: [{ role: 'root', db: 'admin' }]
})
```

---

## 第三章 Mongos 路由层搭建

### 3.1 Mongos 特性

- **无状态节点**：可水平扩展，每个 mongos 独立工作
- **不存储数据**：仅负责路由转发
- **动态配置**：从 Config Server 实时获取元数据
- **无认证责任**：认证在数据节点完成，mongos 用 keyFile 通信

### 3.2 Mongos 搭建步骤

#### 步骤 1：创建目录和密钥

```bash
mkdir -p /opt/mongodb/mongo-cluster/mongos-server/conf
mkdir -p /opt/mongodb/mongo-cluster/mongos-server/data/1
mkdir -p /opt/mongodb/mongo-cluster/mongos-server/logs/1

cp /opt/mongodb/mongo-cluster/config-server/conf/mongo.key \
   /opt/mongodb/mongo-cluster/mongos-server/conf/
```

#### 步骤 2：创建 Mongos 配置文件

```yaml
systemLog:
    destination: file
    logAppend: true
    path: /data/logs/mongo.log

net:
    port: 27017

# 分片配置：指定 Config Server 地址
sharding:
    configDB: configsvr/config-server1:27017,config-server2:27017,config-server3:27017

security:
    keyFile: /data/configdb/conf/mongo.key
```

**关键点**：
- 不需要 `storage` 和 `replication` 配置
- 不需要 `authorization: enabled`（认证由 mongos 连接 config server 时处理）
- `configDB` 格式：`<副本集名>/<节点1>,<节点2>,<节点3>`

#### 步骤 3：启动 Mongos 容器

```bash
docker run --name mongos-server1 -d \
    -p 30001:27017 \
    --net=mongo-cluster \
    --privileged=true \
    --entrypoint "mongos" \
    -v /opt/mongodb/mongo-cluster/mongos-server:/data/configdb \
    -v /opt/mongodb/mongo-cluster/mongos-server/logs/1:/data/logs \
    mongo --config /data/configdb/conf/mongo.conf
```

**说明**：
- `--entrypoint "mongos"` 改变启动命令（不同于 mongod）
- `-p 30001:27017` 映射端口供外部连接

#### 步骤 4：添加分片到 Mongos

```javascript
docker exec -it mongos-server1 /bin/bash
mongo -port 27017

use admin
db.auth("root", "root")

# 添加 Shard1
sh.addShard("shard1/shard1-server1:27017,shard1-server2:27017,shard1-server3:27017")

# 添加 Shard2
sh.addShard("shard2/shard2-server1:27017,shard2-server2:27017,shard2-server3:27017")
```

---

## 第四章 分片集群配置

### 4.1 启用数据库分片

```javascript
use admin
db.auth("root", "root")

# 启用 test 数据库的分片功能
sh.enableSharding("test")
```

### 4.2 创建片键索引

```javascript
use test

# 创建片键索引（必须在分片之前）
db.blog.createIndex({ "by": "hashed", "likes": 1 })

# 查看索引
db.blog.getIndexes()
```

**片键规则**：
- 分片键必须有索引
- 一个集合只能有一个分片键（可以是复合键）
- 支持哈希分片（hashed）和范围分片（1 或 -1）

### 4.3 执行分片

```javascript
# 对 blog 集合分片
sh.shardCollection("test.blog", { "by": "hashed", "likes": 1 })
```

### 4.4 查看分片状态

```javascript
# 查看整个集群分片状态
sh.status()

# 查看数据分布
db.blog.getShardDistribution()
```

---

## 第五章 副本集运维操作

### 5.1 副本集状态查询

```javascript
# 查看主从信息（包含 PRIMARY、SECONDARY、ARBITER）
rs.isMaster()

# 查看详细配置
rs.conf()

# 查看节点状态
rs.status()
```

### 5.2 主从复制测试

#### 5.2.1 在主节点写入数据

```javascript
use test
db.blog.insert({
    "title": "MongoDB 教程",
    "description": "MongoDB 是一个 Nosql 数据库",
    "by": "我的博客",
    "url": "http://www.baiyp.ren",
    "tags": ["mongodb", "database", "NoSQL"],
    "likes": 100
})
```

#### 5.2.2 从节点读取数据

从节点默认不可读，需要执行以下步骤：

```javascript
use test
db.blog.findOne()                  # 失败：connection refused

use admin
db.auth("root", "root")            # 认证
use test

db.setSecondaryOk()                # 允许从节点读取
db.blog.findOne()                  # 成功
```

### 5.3 主从切换测试

```bash
# 停止主节点容器
docker stop shard1-server1

# 进入从节点，查看已变为主
docker exec -it shard1-server2 bash
mongo -port 27017
rs.isMaster()                      # 显示为 PRIMARY

# 重启原主节点
docker start shard1-server1
```

### 5.4 节点扩缩容

#### 5.4.1 添加节点（扩容）

```bash
# 创建新数据目录
mkdir -p /opt/mongodb/mongo-cluster/shard1-server/data/4
mkdir -p /opt/mongodb/mongo-cluster/shard1-server/logs/4

# 启动新节点
docker run --name shard1-server4 -d \
    --net=mongo-cluster \
    --privileged=true \
    -v /opt/mongodb/mongo-cluster/shard1-server:/data/configdb \
    -v /opt/mongodb/mongo-cluster/shard1-server/data/4:/data/db \
    -v /opt/mongodb/mongo-cluster/shard1-server/logs/4:/data/logs \
    mongo --config /data/configdb/conf/mongo.conf

# 在主节点添加新节点
docker exec -it shard1-server1 bash
mongo -port 27017
use admin
db.auth("root", "root")
rs.add("shard1-server4:27017")
rs.isMaster()                      # 验证新节点已加入
```

#### 5.4.2 删除节点（缩容）

```javascript
# 在主节点执行删除
use admin
db.auth("root", "root")
rs.remove("shard1-server4:27017")
rs.isMaster()                      # 验证节点已删除
```

---

## 第六章 关键配置文件详解

### 6.1 配置文件层次结构

```
mongo-cluster/
├── config-server/          # Config Server 副本集
│   ├── conf/
│   │   ├── mongo.conf
│   │   └── mongo.key
│   ├── data/{1,2,3}/
│   └── logs/{1,2,3}/
├── shard1-server/          # Shard1 副本集
│   ├── conf/
│   │   ├── mongo.conf
│   │   └── mongo.key
│   ├── data/{1,2,3,4}/
│   └── logs/{1,2,3,4}/
├── shard2-server/          # Shard2 副本集（可选）
│   ├── conf/
│   ├── data/{1,2,3}/
│   └── logs/{1,2,3}/
└── mongos-server/          # Mongos 路由层
    ├── conf/
    │   ├── mongo.conf
    │   └── mongo.key
    ├── data/1/
    └── logs/1/
```

### 6.2 三种配置文件对比

| 参数 | Config Server | Shard Server | Mongos |
|------|---------------|--------------|--------|
| `storage.dbPath` | ✓ | ✓ | ✗ |
| `replication.replSetName` | ✓ | ✓ | ✗ |
| `sharding.clusterRole` | configsvr | shardsvr | ✗ |
| `sharding.configDB` | ✗ | ✗ | ✓（必须） |
| `security.authorization` | ✓ | ✓ | ✗（用 keyFile） |
| `security.keyFile` | ✓ | ✓ | ✓ |

### 6.3 关键配置项说明

```yaml
# 副本集名称：同一副本集内必须相同
replication:
    replSetName: configsvr          # 配置服务器副本集名：configsvr
                                    # 分片副本集名：shard1, shard2, 等

# 集群角色
sharding:
    clusterRole: configsvr          # configsvr（配置节点）
                                    # shardsvr（分片节点）
                                    # 不设置（普通副本集）

# 认证密钥
security:
    keyFile: /path/to/mongo.key     # 路径必须正确，权限必须为 600
```

---

## 第七章 keyFile 鉴权机制

### 7.1 密钥文件的用途

- **节点间通信**：mongod 和 mongos 通过 keyFile 相互认证
- **集群一致性**：确保只有授权的节点才能加入集群
- **安全通道**：加密 replication 和 sharding 通信

### 7.2 密钥文件操作

```bash
# 生成密钥（需要 768 位以上）
openssl rand -base64 756 > /opt/mongodb/mongo-cluster/config-server/conf/mongo.key

# 设置权限（必须 600）
chmod 600 /opt/mongodb/mongo-cluster/config-server/conf/mongo.key

# 验证权限
ls -l /opt/mongodb/mongo-cluster/config-server/conf/mongo.key
# 输出：-rw------- 1 root root 1000 ... mongo.key

# 复制到其他节点（保持相同内容和权限）
cp /opt/mongodb/mongo-cluster/config-server/conf/mongo.key \
   /opt/mongodb/mongo-cluster/shard1-server/conf/
chmod 600 /opt/mongodb/mongo-cluster/shard1-server/conf/mongo.key
```

### 7.3 认证流程（双层认证）

```
┌─────────────────────────────────────────────────────────┐
│             MongoDB 集群认证流程                        │
└─────────────────────────────────────────────────────────┘

第一层：节点间认证（keyFile）
  mongos/mongod <──── keyFile ────> mongod
  （副本集通信、分片通信使用 keyFile 自动认证）

第二层：用户认证（username/password）
  Application <──── auth ────> mongos
  mongos       <──── auth ────> shard1/config-server
  （应用需要执行 db.auth()）
```

---

## 第八章 Docker 网络与端口规划

### 8.1 Docker 网络创建

```bash
# 创建自定义网络（allows DNS resolution）
docker network create mongo-cluster

# 查看网络
docker network ls
docker network inspect mongo-cluster
```

### 8.2 端口分配规划

| 组件 | 容器内端口 | 宿主机映射 | 用途 |
|------|----------|----------|------|
| config-server1 | 27017 | - | 内部通信 |
| config-server2 | 27017 | - | 内部通信 |
| config-server3 | 27017 | - | 内部通信 |
| shard1-server1 | 27017 | - | 内部通信 |
| shard1-server2 | 27017 | - | 内部通信 |
| shard1-server3 | 27017 | - | 内部通信 |
| shard2-server1 | 27017 | - | 内部通信 |
| shard2-server2 | 27017 | - | 内部通信 |
| shard2-server3 | 27017 | - | 内部通信 |
| mongos-server1 | 27017 | 30001 | **外部入口** |
| mongos-server2 | 27017 | 30002 | 外部入口（可选） |

**容器内通信**：使用容器名称进行 DNS 解析（如 config-server1:27017）

---

## 第九章 常用运维命令速查表

### 9.1 Docker 容器操作

```bash
# 查看所有容器
docker ps -a

# 启动、停止、重启
docker start <container>
docker stop <container>
docker restart <container>

# 进入容器
docker exec -it <container> bash

# 查看日志
docker logs <container> | tail -100

# 删除容器
docker rm <container>
```

### 9.2 副本集命令

```javascript
// 初始化副本集
rs.initiate({
    _id: "setname",
    members: [
        { _id: 0, host: "host1:27017" },
        { _id: 1, host: "host2:27017" },
        { _id: 2, host: "host3:27017", arbiterOnly: true }
    ]
})

// 查看副本集状态
rs.status()                         // 详细状态
rs.isMaster()                       // 主从判定
rs.conf()                           // 当前配置

// 添加、删除节点
rs.add("host4:27017")
rs.addArb("host4:27017")            // 添加仲裁节点
rs.remove("host4:27017")

// 设置主节点优先级
rs.reconfig({
    _id: "setname",
    members: [
        { _id: 0, host: "host1:27017", priority: 10 },
        { _id: 1, host: "host2:27017", priority: 5 }
    ]
})
```

### 9.3 分片命令

```javascript
// 查看分片状态
sh.status()

// 添加删除分片
sh.addShard("shard1/host1:27017,host2:27017,host3:27017")
sh.removeShard("shard2")

// 启用数据库分片
sh.enableSharding("dbname")

// 对集合分片
sh.shardCollection("db.collection", { "shardKey": 1 })

// 查看数据分布
db.collection.getShardDistribution()
```

### 9.4 用户认证命令

```javascript
// 创建用户
db.createUser({
    user: "username",
    pwd: "password",
    roles: [
        { role: "root", db: "admin" },
        { role: "readWrite", db: "dbname" }
    ]
})

// 认证
db.auth("username", "password")

// 查看当前用户
db.getUser("username")

// 修改密码
db.changeUserPassword("username", "newpassword")

// 删除用户
db.removeUser("username")
```

### 9.5 性能监控命令

```javascript
// 查看操作统计
db.getProfilingStatus()

// 查看当前操作
db.currentOp()

// 查看集合统计
db.collection.stats()

// 查看数据库统计
db.stats()
```

---

## 第十章 Mongos 到数据节点的请求路径

### 10.1 请求路由流程

```
┌────────────────┐
│   Client App   │
│  (port 30001)  │
└────────┬───────┘
         │ 1. 连接并认证
         ▼
  ┌─────────────┐
  │   Mongos    │
  │  (Router)   │
  └──────┬──────┘
         │ 2. 解析分片键
         │ 3. 从 Config Server 获取元数据
         ▼
    ┌────────────────────────────────┐
    │   Config Server 副本集         │
    │  (configsvr/replica)           │
    │  - shard1 chunk 分布            │
    │  - shard2 chunk 分布            │
    │  - 分片键范围                   │
    └────────────────────────────────┘
         │ 4. 根据分片键路由
         │
    ┌────┴────┬──────────┐
    │          │          │
    ▼          ▼          ▼
 ┌─────┐   ┌──────┐   ┌──────┐
 │Shard1│   │Shard2│   │ShardN│
 │(RS)  │   │ (RS) │   │(RS)  │
 └─────┘   └──────┘   └──────┘
    │          │          │
    └────┬─────┴────┬─────┘
         │ 5. 返回结果合并
         ▼
  ┌──────────────┐
  │  Client App  │
  │  返回结果    │
  └──────────────┘
```

### 10.2 关键交互点

| 交互 | 说明 |
|------|------|
| Client → Mongos | 应用连接 mongos，使用 db.auth() 认证 |
| Mongos → Config Server | 获取集群元数据（chunk 分布、分片键范围） |
| Mongos → Shard | 根据分片键将请求转发到对应分片 |
| Shard → Replica Set | 分片内部由副本集保证高可用 |
| Shard → Mongos | 返回数据给 mongos，由 mongos 合并返回给应用 |

### 10.3 Mongos 缓存机制

- Mongos 缓存 Config Server 的元数据
- 定期刷新元数据（配置可调）
- 如果遇到 StaleConfigException，则重新刷新元数据

---

## 第十一章 搭建流程时序图

```
时间轴
  │
  ├─► 1. 创建 Docker 网络
  │    docker network create mongo-cluster
  │
  ├─► 2. 搭建 Config Server 副本集
  │    ├── 创建目录结构
  │    ├── 生成 keyFile
  │    ├── 创建配置文件（mongo.conf）
  │    ├── 启动 3 个容器（config-server1/2/3）
  │    └── rs.initiate() 初始化副本集
  │
  ├─► 3. 搭建 Shard1 副本集
  │    ├── 创建目录结构
  │    ├── 复制 keyFile
  │    ├── 创建配置文件
  │    ├── 启动 3 个容器（shard1-server1/2/3）
  │    └── rs.initiate() + 设置仲裁节点
  │
  ├─► 4. 搭建 Shard2 副本集（可选）
  │    ├── 创建目录结构
  │    ├── 复制 keyFile
  │    ├── 创建配置文件
  │    ├── 启动 3 个容器
  │    └── rs.initiate() + 设置仲裁节点
  │
  ├─► 5. 搭建 Mongos 路由层
  │    ├── 创建目录结构
  │    ├── 复制 keyFile
  │    ├── 创建配置文件（指向 Config Server）
  │    ├── 启动 mongos 容器（--entrypoint mongos）
  │    └── sh.addShard() 添加所有分片
  │
  ├─► 6. 配置分片策略
  │    ├── sh.enableSharding("test") 启用分片
  │    ├── db.blog.createIndex() 创建片键索引
  │    └── sh.shardCollection() 执行分片
  │
  ├─► 7. 测试与验证
  │    ├── 主从复制测试
  │    ├── 主从切换测试
  │    └── 查看分片分布情况
  │
  └─► 8. 扩缩容演示
       ├── rs.add() 添加节点
       └── rs.remove() 删除节点
```

---

## 第十二章 Docker 容器启动命令详解

### 12.1 启动 mongod（数据节点通用模板）

```bash
docker run --name <容器名> -d \
    --net=mongo-cluster \              # 加入 Docker 自定义网络
    --privileged=true \                # 授予特权（访问系统资源）
    -v <主机配置路径>:/data/configdb \ # 挂载配置文件和 keyFile
    -v <主机数据路径>:/data/db \       # 挂载数据目录
    -v <主机日志路径>:/data/logs \     # 挂载日志目录
    mongo --config /data/configdb/conf/mongo.conf
```

### 12.2 启动 mongos（路由节点专用）

```bash
docker run --name <容器名> -d \
    -p <宿主端口>:27017 \              # 只有 mongos 需要端口映射
    --net=mongo-cluster \
    --privileged=true \
    --entrypoint "mongos" \            # 改变启动命令！
    -v <主机配置路径>:/data/configdb \
    -v <主机日志路径>:/data/logs \
    mongo --config /data/configdb/conf/mongo.conf
```

### 12.3 关键参数说明

| 参数 | 说明 |
|------|------|
| `--name` | 容器名称，用于 DNS 解析（同一网络内） |
| `--net=mongo-cluster` | 加入 Docker 自定义网络 |
| `--privileged=true` | 运行特权容器（允许系统调用） |
| `-p 30001:27017` | 端口映射（仅 mongos 需要） |
| `-v host:container` | 挂载卷（配置、数据、日志都需要） |
| `--entrypoint "mongos"` | 改变容器启动命令（只有 mongos 需要） |
| `--config` | MongoDB 配置文件路径 |

---

## 附录 A：原文勘误

### A.1 配置文件中的语法错误

**原文第 180 行**：
```javascript
// 错误
db微信公众({user:"root",pwd:"root",roles:[{role:'root',db:'admin'}]})

// 正确
db.createUser({user:"root",pwd:"root",roles:[{role:'root',db:'admin'}]})
```

**勘误说明**：`db微信公众` 应为 `db.createUser`，是 OCR 识别错误。

---

### A.2 配置文件中的注释不一致

**原文第 269 行**：
```yaml
replication:
    replSetName: shard1 #复制集名称是 shardsvr
```

**正确说法**：
```yaml
replication:
    replSetName: shard1 #复制集名称应为 shard1 或其他自定义名称，不是 shardsvr
```

**勘误说明**：shardsvr 是 `sharding.clusterRole` 的值，不是 `replSetName` 的值。

---

### A.3 命令中的空格缺失

**原文第 644 行**：
```bash
sh.addShard("shard1/shard1-server1:27017, shard1-server2:27017, shard1-server3:27017")
```

**问题**：格式中的逗号两边有空格，可能导致解析失败。

**建议格式**（无空格）：
```bash
sh.addShard("shard1/shard1-server1:27017,shard1-server2:27017,shard1-server3:27017")
```

---

### A.4 方法名大小写错误

**原文第 494 行和 861 行**：
```javascript
// 错误
db.setSecondaryok()
sh.addshard()

// 正确
db.setSecondaryOk()
sh.addShard()
```

**勘误说明**：MongoDB 方法名遵循 camelCase 命名规则，注意大小写。

---

### A.5 不完整的 JSON 块

**原文第 864-898 行**：JSON 插入命令有嵌套错误，缺少闭合括号。

**正确格式**：
```javascript
use test

// 批量插入 500 条数据到 blog
for (var i = 0; i < 500; i++) {
    db.blog.insert({
        "_id": i,
        "title": "张三的文章" + i,
        "by": "李四" + i,
        "url": "http://www.baidu.com",
        "tags": ["语文", "数学"],
        "likes": i
    });
}

// 批量插入 500 条数据到 blog2
for (var i = 0; i < 500; i++) {
    db.blog2.insert({
        "_id": i,
        "title": "张三的文章" + i,
        "by": "李四" + i,
        "url": "http://www.baidu.com",
        "tags": ["语文", "数学"],
        "likes": i
    });
}
```

---

## 附录 B：核心知识点速查表

### B.1 快速对照表

| 需求 | 命令/配置 |
|------|---------|
| 创建密钥 | `openssl rand -base64 756 > mongo.key` |
| 设置密钥权限 | `chmod 600 mongo.key` |
| 初始化副本集 | `rs.initiate({...})` |
| 查看副本集状态 | `rs.status()` |
| 添加数据节点 | `rs.add("host:27017")` |
| 添加仲裁节点 | `rs.addArb("host:27017")` |
| 删除节点 | `rs.remove("host:27017")` |
| 添加分片 | `sh.addShard("replicaset/host1,host2,host3")` |
| 启用分片 | `sh.enableSharding("dbname")` |
| 分片集合 | `sh.shardCollection("db.coll", {key: 1})` |
| 查看分片状态 | `sh.status()` |
| 认证用户 | `db.auth("user", "pwd")` |
| 创建用户 | `db.createUser({user, pwd, roles})` |
| 启用从节点读 | `db.setSecondaryOk()` |
| 查看数据分布 | `db.collection.getShardDistribution()` |

### B.2 Config 文件快速模板

**Config Server 配置**：
```yaml
storage:
    dbPath: /data/db
systemLog:
    destination: file
    path: /data/logs/mongo.log
net:
    port: 27017
replication:
    replSetName: configsvr
sharding:
    clusterRole: configsvr
security:
    authorization: enabled
    keyFile: /data/configdb/conf/mongo.key
```

**Shard Server 配置**：
```yaml
storage:
    dbPath: /data/db
systemLog:
    destination: file
    path: /data/logs/mongo.log
net:
    port: 27017
replication:
    replSetName: shard1
sharding:
    clusterRole: shardsvr
security:
    authorization: enabled
    keyFile: /data/configdb/conf/mongo.key
```

**Mongos 配置**：
```yaml
systemLog:
    destination: file
    path: /data/logs/mongo.log
net:
    port: 27017
sharding:
    configDB: configsvr/config-server1:27017,config-server2:27017,config-server3:27017
security:
    keyFile: /data/configdb/conf/mongo.key
```

### B.3 故障排查清单

| 问题 | 排查步骤 |
|------|---------|
| 节点无法连接 | 1. 检查容器是否运行（docker ps）<br/>2. 检查网络连通性（docker network inspect）<br/>3. 查看日志（docker logs） |
| keyFile 权限错误 | 运行 `chmod 600 mongo.key` |
| 副本集初始化失败 | 1. 确认所有节点已启动<br/>2. 检查 replSetName 拼写<br/>3. 检查主机名解析 |
| 分片转向错误 | 1. 确认 mongos 已添加所有分片<br/>2. 执行 `sh.addShard()` 重新添加<br/>3. 查看 Config Server 日志 |
| 认证失败 | 1. 检查用户是否存在（db.getUser()）<br/>2. 检查密码是否正确<br/>3. 确认 authorization: enabled |

---

## 总结

本文档完整覆盖了 MongoDB 分片集群搭建的全流程：

1. **架构理解**：掌握 Mongos、Config Server、Shard、Replica Set 的角色
2. **搭建步骤**：从创建目录、生成密钥、编写配置、启动容器到初始化
3. **配置文件**：三种不同配置文件的差异与关键参数
4. **运维操作**：副本集管理、分片配置、用户认证、扩缩容
5. **故障诊断**：常见问题的排查方法

通过 Docker 容器化搭建，可以快速体验完整的分布式数据库集群。
