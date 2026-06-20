# Redis 应用与原理 - 完整梳理

## 目录
1. [缓存基础](#缓存基础)
2. [缓存技术方案对比](#缓存技术方案对比)
3. [Redis 基础](#redis-基础)
4. [数据结构详解](#数据结构详解)
5. [持久化机制](#持久化机制)
6. [复制与高可用](#复制与高可用)
7. [缓存问题处理](#缓存问题处理)
8. [底层数据结构](#底层数据结构)
9. [核心知识速查](#核心知识速查)
10. [原文勘误](#原文勘误)

---

## 缓存基础

### 什么是缓存

**定义**: 缓存是高速存储设备,用于调节速度不一致的两个或多个物体的速度差异,加速访问速度较慢的一方。

**关键概念**:
- **命中率**: 查询数据存在于缓存时称为"命中",可直接由缓存提供;未命中时需穿过缓存层到数据库获取
- **缓存 vs 内存**: 缓存是内存的一种应用,但不等同于内存本身

**缓存在系统架构中的作用**:
- 减少客户端访问次数
- 减少服务端请求次数
- 减少数据库压力
- 减少磁盘IO次数

### 缓存加入带来的挑战

1. 应用中哪类数据应使用缓存?
2. 应用数据何时、如何写入缓存?
3. 应用缓存数据如何保证命中率?
4. 缓存如何保证实时性?
5. 如何保证缓存数据不丢失?

---

## 缓存技术方案对比

### 本地缓存
**方案**: Java 程序使用 ThreadLocal、List、Map 等保存应用数据

**生命周期**: 同 JVM 生命周期

**缺点**: 无分布式支持

### Ehcache
**特点**:
- 基于 Java 开发,简单轻量
- 纯进程内缓存,非分布式
- 被广泛用于 Hibernate 等开源项目

**局限**: 多节点部署时存在数据不一致问题(如用户登录信息)

### Memcache
**特点**:
- C 语言编写,高性能分布式缓存
- 支持多核 CPU 并发工作
- 基于纯内存,数据格式为 KV HashTable

**局限**: 不支持持久化,宕机重启会压垮数据库

### Redis (推荐)

**对比 Memcache**:

| 维度 | Memcache | Redis |
|------|----------|-------|
| **编程语言** | C | ANSI C (C) |
| **数据结构** | 仅 KV | KV + List/Hash/Set/ZSet 等 |
| **过期设置** | SET 时指定 | 支持后期指定(EXPIRE) |
| **集群模式** | 客户端分片 | 原生支持集群 |
| **持久化** | 无 | RDB/AOF |
| **高可用** | 无 | 支持主从/哨兵/集群 |
| **性能** | 大数据(>100k) 更优 | 小数据、复杂操作更优 |
| **应用场景** | 多读少写、大数据量 | 读写兼需、业务复杂、数据安全要求高 |

---

## Redis 基础

### Redis 简介

**官方定义**: Remote DIctionary Server (Redis) - 跨平台的非关系型数据库

**作者**: Salvatore Sanfilippo

**本质**: Key-Value 存储系统 + 内存数据库

**特性**:
- 开源,ANSI C 编写,BSD 协议
- 支持网络访问
- 基于内存 + 可选持久化
- 提供多种编程语言 API
- 分布式支持

**官方资源**:
- 英文文档: https://redis.io/commands/
- 中文文档: http://www.redis.cn/commands.html#string

### Redis 环境搭建

#### 基于 Docker

```bash
# 启动
docker run --name some-redis -d daocloud.io/library/redis:6.0.6

# 进入容器
docker exec -it some-redis sh

# 启动 CLI
redis-cli

# 测试连接
127.0.0.1:6379> info
```

#### 物理机安装

```bash
# 切换目录
cd /opt/redis

# 下载
wget https://download.redis.io/releases/redis-6.2.0.tar.gz

# 解压
tar zxvf redis-6.2.0.tar.gz
cd redis-6.2.0

# 编译
make

# 启动
src/redis-server
```

**常见编译错误**:
- `make[3]: cc: 命令未找到` → `yum install -y gcc`
- `jemalloc/jemalloc.h: 没有那个文件` → `make MALLOC=libc`

#### 关键配置

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `port` | 监听端口 | 6379 |
| `protected-mode` | 保护模式开关 | yes (需改 no 以允许远程连接) |
| `bind` | 绑定 IP 地址 | 127.0.0.1 (学习环境注释为 #) |
| `daemonize` | 守护进程启动 | no |
| `requirepass` | 连接密码 | 无 |

**启动命令示例**:
```bash
redis-server redis.conf --port 9010 --protected-mode no
```

---

## 数据结构详解

### 0. Redis 数据库

**概念**: Redis 内置 16 个数据库(可配置 dbnum 属性调整)

**默认数据库**: 0

**切换命令**: `SELECT database_index` (从 0 开始)

**特点**: 
- 每个客户端有自己的目标数据库
- 数据库间数据完全独立,不能跨库操作

**例**:
```
数据库 0: name=zhangsan
数据库 1: name=zhangsan, age=14
数据库 15: name=zhangsan, age=16
```

### 1. String (字符串)

**底层实现**: 简单动态字符串 (Simple Dynamic String, SDS)

#### SDS 结构演进

**3.2 版本前**:
```
+--------+--------+--------+
| free   | len    | buf[]  |
+--------+--------+--------+
- free: 未使用空间
- len: 已使用字符串长度
- buf: char 数组,以 \0 结尾
```

**3.2 版本后** (按类型划分头部):
```
+-------+-------+-------+-------+-------+
|flags  | alloc | len   | buf[] |
+-------+-------+-------+-------+-------+
- flags: 占 1 字节,低 3 位表示头部类型,后 5 位预留
- alloc: 最大容量
- len: 已使用长度
- 剩余空间: free = alloc - len
```

**SDS 类型范围**:

| 类型 | 范围 | 最大值 |
|------|------|--------|
| SDS_TYPE_5 | [0, 2^5) | 31 |
| SDS_TYPE_8 | [2^5, 2^8) | 255 |
| SDS_TYPE_16 | [2^8, 2^16) | 65,535 |
| SDS_TYPE_32 | [2^16, 2^32) | 4,294,967,295 |
| SDS_TYPE_64 | [2^32, 2^64) | 2^64-1 |

**优势**: 
- 动态扩容,减少重新分配
- 字符串长度 O(1) 获取
- 二进制安全

#### 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `SET` | 设置键值,覆写旧值,清除 TTL | `SET name zhangsan` |
| `GET` | 获取值 | `GET name` |
| `APPEND` | 追加字符串 | `APPEND name san` |
| `GETRANGE` | 获取子字符串(包括首尾) | `GETRANGE name 0 3` |
| `SETRANGE` | 设置子字符串 | `SETRANGE name 0 zha` |
| `STRLEN` | 获取字符串长度 | `STRLEN name` |

**SET 扩展选项**:
```bash
# EX - 秒级过期 (等价 SETEX)
SET key value EX 3600

# PX - 毫秒级过期 (等价 PSETEX)
SET key value PX 3600000

# NX - 仅键不存在时设置 (等价 SETNX)
SET key value NX

# XX - 仅键存在时设置
SET key value XX
```

**运算命令**:
```bash
INCR counter           # 自增 1
INCRBY counter 5       # 增加 5
INCRBYFLOAT counter 0.5  # 增加浮点数
DECR counter           # 自减 1
DECRBY counter 3       # 减少 3
MSET k1 v1 k2 v2      # 批量设置
MGET k1 k2            # 批量获取
```

**易混淆点**:
- `APPEND` 后如果 key 不存在,等同于 `SET`
- `GETRANGE` 支持负数偏移(-1 为最后一个字符)
- `SET` 会清除旧的 TTL 设置

---

### 2. Hash (哈希表)

**定义**: 类似 Java HashMap,用于存储多属性对象

**应用场景**: 热点数据(多属性记录),便于单个属性修改

**对比 String+JSON**:
- String: 适合频繁读操作,整体序列化
- Hash: 适合频繁写操作,单个属性修改不需重新序列化整个对象

**注意**: Redis 过期功能仅作用于 key,不能针对 field 设置过期

#### 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `HSET` | 设置字段值(新增或覆盖) | `HSET user:1 name zhangsan` |
| `HGET` | 获取字段值 | `HGET user:1 name` |
| `HMSET` | 批量设置字段(已过时,推荐 HSET) | `HMSET user:1 name zhangsan age 14` |
| `HMGET` | 批量获取字段值 | `HMGET user:1 name age` |
| `HGETALL` | 获取所有字段和值 | `HGETALL user:1` |
| `HEXISTS` | 判断字段是否存在(存在返 1,否则 0) | `HEXISTS user:1 name` |
| `HSETNX` | 字段不存在时设置(已存在无效) | `HSETNX user:1 name zhangsan` |
| `HDEL` | 删除一个或多个字段 | `HDEL user:1 name age` |
| `HLEN` | 获取字段数量 | `HLEN user:1` |
| `HINCRBY` | 字段值增加增量(可负数) | `HINCRBY user:1 age 1` |
| `HINCRBYFLOAT` | 字段值增加浮点增量 | `HINCRBYFLOAT user:1 score 0.5` |
| `HKEYS` | 获取所有字段名 | `HKEYS user:1` |
| `HVALS` | 获取所有字段值 | `HVALS user:1` |

**使用场景示例**:
```bash
# 商品对象 (价格、库存、关注数、评价数经常变动)
HSET product:1001 name "iPhone 15" price 5999 stock 100 likes 5000 reviews 150

# 用户对象
HSET user:1 name zhangsan email zhangsan@qq.com age 30 city beijing
```

---

### 3. List (列表)

**定义**: 双端链表,提供高效节点重排和顺序访问

**底层实现**: Redis 自建链表结构(C 语言无内置链表)

**链表节点结构**:
```c
typedef struct listNode {
    struct listNode *prev;  // 前置节点
    struct listNode *next;  // 后置节点
    void *value;            // 节点值
} listNode;
```

**特点**: 双向链表,支持两端快速增删

#### 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `LPUSH` | 在表头插入一个或多个值 | `LPUSH list:1 a b c` |
| `RPUSH` | 在表尾插入一个或多个值 | `RPUSH list:1 x y z` |
| `LPOP` | 移除并返回表头元素 | `LPOP list:1` |
| `RPOP` | 移除并返回表尾元素 | `RPOP list:1` |
| `BLPOP` | 阻塞式 LPOP(无元素时等待) | `BLPOP list:1 30` (超时 30s) |
| `BRPOP` | 阻塞式 RPOP(无元素时等待) | `BRPOP list:1 30` |
| `LLEN` | 获取列表长度 | `LLEN list:1` |
| `LINDEX` | 获取指定下标元素 | `LINDEX list:1 0` |
| `LRANGE` | 获取范围内元素 | `LRANGE list:1 0 -1` (全部) |
| `LSET` | 设置指定下标元素 | `LSET list:1 0 newvalue` |
| `LTRIM` | 保留范围内元素,删除其他 | `LTRIM list:1 0 2` |
| `LREM` | 移除指定个数的值 | `LREM list:1 2 value` |
| `LINSERT` | 在指定值前/后插入 | `LINSERT list:1 BEFORE pivot value` |

**下标说明**:
- 0: 第一个元素
- 1: 第二个元素
- -1: 最后一个元素
- -2: 倒数第二个元素

**应用场景**:
- 微型消息队列
- 阻塞队列
- 栈结构
- 活动流(Timeline)

---

### 4. Set (集合)

**定义**: 类似 Java HashSet,无序,自动去重

**特点**: 相比 List 提供高效的成员存在性判断,相比 Hash 缺少 value

**优势**: 支持集合运算(交集、并集、差集)

#### 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `SADD` | 添加一个或多个成员 | `SADD set:1 a b c` |
| `SREM` | 移除一个或多个成员 | `SREM set:1 a b` |
| `SMEMBERS` | 获取所有成员 | `SMEMBERS set:1` |
| `SISMEMBER` | 判断成员是否存在(存在返 1) | `SISMEMBER set:1 a` |
| `SCARD` | 获取成员数量 | `SCARD set:1` |
| `SPOP` | 移除并返回随机成员 | `SPOP set:1` |
| `SRANDMEMBER` | 返回随机成员(不移除) | `SRANDMEMBER set:1` (1 个) / `SRANDMEMBER set:1 3` (3 个) |
| `SMOVE` | 移动成员到另一集合 | `SMOVE source dest member` |
| **集合运算** | | |
| `SINTER` | 返回交集 | `SINTER set:1 set:2` |
| `SUNION` | 返回并集 | `SUNION set:1 set:2` |
| `SDIFF` | 返回差集 | `SDIFF set:1 set:2` |
| `SINTERSTORE` | 交集存储到新集合 | `SINTERSTORE dest set:1 set:2` |
| `SUNIONSTORE` | 并集存储到新集合 | `SUNIONSTORE dest set:1 set:2` |
| `SDIFFSTORE` | 差集存储到新集合 | `SDIFFSTORE dest set:1 set:2` |

**应用场景**:
- 标签系统(共同标签)
- 权限系统(共同权限)
- 社交网络(共同关注、共同好友)
- 去重计数

---

### 5. Sorted Set (有序集合)

**定义**: Set + score 分值,自动按分值排序

**差异**: 每个成员必须自定分值,按 score 排序

**应用**: 排序场景(点赞排序、排行榜、延时任务等)

#### 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `ZADD` | 添加成员及分值 | `ZADD zset:1 100 "member1" 200 "member2"` |
| `ZREM` | 移除成员 | `ZREM zset:1 member1` |
| `ZSCORE` | 获取成员分值 | `ZSCORE zset:1 member1` |
| `ZRANK` | 获取排名(分值升序,0 开始) | `ZRANK zset:1 member1` |
| `ZREVRANK` | 获取反向排名(分值降序) | `ZREVRANK zset:1 member1` |
| `ZRANGE` | 按排名范围获取成员 | `ZRANGE zset:1 0 -1` (全部) / `ZRANGE zset:1 0 -1 WITHSCORES` (含分值) |
| `ZREVRANGE` | 反向范围获取 | `ZREVRANGE zset:1 0 -1` |
| `ZRANGEBYSCORE` | 按分值范围获取 | `ZRANGEBYSCORE zset:1 100 200` / `ZRANGEBYSCORE zset:1 100 (200` (不含上界) |
| `ZREVRANGEBYSCORE` | 反向分值范围 | `ZREVRANGEBYSCORE zset:1 200 100` |
| `ZCARD` | 获取成员数量 | `ZCARD zset:1` |
| `ZCOUNT` | 统计分值范围内成员数 | `ZCOUNT zset:1 100 200` |
| `ZINCRBY` | 增加成员分值 | `ZINCRBY zset:1 50 member1` (增加 50) |
| `ZRANGEBYLEX` | 按字典序范围获取 | `ZRANGEBYLEX zset:1 - +` |
| `ZREMRANGEBYRANK` | 按排名范围移除 | `ZREMRANGEBYRANK zset:1 0 2` |
| `ZREMRANGEBYSCORE` | 按分值范围移除 | `ZREMRANGEBYSCORE zset:1 100 200` |

**排序说明**:
- 升序(score 从小到大): ZRANK、ZRANGE、ZRANGEBYSCORE
- 降序(score 从大到小): ZREVRANK、ZREVRANGE、ZREVRANGEBYSCORE
- 分值相同时按字典序(member)排列

**应用场景**:
- 排行榜(游戏分数、销售排名、热度排序)
- 延时任务队列(score 为时间戳)
- 消息队列(score 为消息时间)
- 评分系统

---

### 6. Bitmap (二进制位数组)

**定义**: 位数组,每位存储 0 或 1,用于高效节省空间的计数

**底层**: 二进制位映射

**容量**: 最大 2^32 位(512MB),可保存 40 亿个结果

**优势**: 内存占用极低(1 bit 存 1 个 bool)

#### 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `SETBIT` | 设置指定偏移位的值(0 或 1) | `SETBIT bitmap:1 10 1` |
| `GETBIT` | 获取指定偏移位的值 | `GETBIT bitmap:1 10` |
| `BITCOUNT` | 统计被设置为 1 的位数 | `BITCOUNT bitmap:1` |
| `BITPOS` | 查找第一个 bit 值为 1/0 的位置 | `BITPOS bitmap:1 1` |
| `BITOP` | 位运算(AND/OR/XOR/NOT) | `BITOP AND dest src1 src2` |
| `BITFIELD` | 位域操作 | `BITFIELD key GET u4 0` |

**应用场景**:
- 用户签到(每日一位,365 天仅需 46 字节)
- 权限位标记(32 权限用 4 字节)
- 活跃用户统计
- IP 地址去重

---

### 7. HyperLogLog

**定义**: 概率数据结构,用于估算大集合的基数(不同元素数量)

**特点**: 
- 内存占用极低(标准 12KB)
- 统计结果存在误差(标准误差约 0.81%)
- 不可获取具体元素

**应用**: UV 统计、页面访问数估算

#### 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `PFADD` | 添加元素 | `PFADD hll:1 a b c` |
| `PFCOUNT` | 统计基数(不同元素数) | `PFCOUNT hll:1` |
| `PFMERGE` | 合并多个 HyperLogLog | `PFMERGE dest hll:1 hll:2` |

**vs Set**:
- Set: 精确统计,内存消耗大(百万级别以上)
- HyperLogLog: 近似统计,内存极小,误差可接受

---

### 8. Geo (地理位置)

**定义**: 存储地理坐标(经纬度)和距离计算

**底层**: 基于 Sorted Set,score 为 geohash

**精度**: 经纬度精度支持到 12 位小数

#### 常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `GEOADD` | 添加地理位置 | `GEOADD cities 13.361389 38.115556 "Palermo"` |
| `GEOPOS` | 获取位置坐标 | `GEOPOS cities "Palermo"` |
| `GEODIST` | 计算两位置距离 | `GEODIST cities "Palermo" "Catania" km` |
| `GEORADIUS` | 半径范围查询 | `GEORADIUS cities 15 37 200 km` |
| `GEORADIUSBYMEMBER` | 以成员为中心的范围查询 | `GEORADIUSBYMEMBER cities "Palermo" 100 km` |
| `GEOHASH` | 获取位置的 geohash | `GEOHASH cities "Palermo"` |

**应用场景**:
- 附近的人/店铺
- 地理围栏(Geofencing)
- 位置打卡

---

### 9. Stream (流)

**定义**: Redis 5.0+ 新增,消息流数据结构

**特点**: 时间序列消息队列,支持消费者组

**常用命令**:
```bash
# 添加消息
XADD stream:1 * field1 value1 field2 value2

# 读取消息范围
XRANGE stream:1 - +

# 阻塞读取(消费模式)
XREAD BLOCK 0 STREAMS stream:1 0

# 消费者组
XGROUP CREATE stream:1 group1 0
XREADGROUP GROUP group1 consumer1 STREAMS stream:1 >
```

**应用场景**:
- 事件日志
- 实时数据流处理
- 消息队列(优于 List 的 BRPOP)

---

## 持久化机制

### 缓存持久化的意义

**问题**: 缓存数据都在内存,Redis 宕机后数据丢失

**解决**: 持久化 - 将数据写入磁盘,重启时恢复

**场景**:
- 数据从数据库一次加载,只查询不修改 → 可容忍非持久化
- 缓存为实时业务数据,涉及新增/修改/删除 → 必须持久化

### RDB (Redis DataBase)

**定义**: 基于数据快照的持久化,将内存中的数据集以二进制形式写入磁盘

**工作原理**:
1. Redis 进程 fork 子进程
2. 子进程遍历内存数据,写入临时文件
3. 写入完成后原子性重命名为 dump.rdb
4. 下次启动时读取 rdb 文件恢复数据

**触发机制**:

配置文件设置保存条件:
```
save <seconds> <changes>
```

**默认配置**:
```
save 900 1      # 900秒(15分钟)内有 1 个更改
save 300 10     # 300秒(5分钟)内有 10 个更改
save 60 10000   # 60秒内有 10000 个更改
```

**特点**:
- 只持久化有效数据(过期数据不被保存)
- 文件紧凑,恢复快
- 全量快照,可能丢失最后的修改

**手动触发**:
```bash
SAVE      # 阻塞式保存(不推荐)
BGSAVE    # 后台保存(推荐)
LASTSAVE  # 获取上次保存时间戳
```

#### RDB 流程图

```
┌─────────────────────────────────────────────────┐
│ Redis 主进程                                      │
├─────────────────────────────────────────────────┤
│ 1. 接收 BGSAVE 或满足 save 条件                  │
│                                                   │
│ 2. Fork 子进程 ──────────────┐                   │
│                               │                   │
│ 3. 主进程继续服务客户端      │ 子进程:          │
│    - 读请求正常              │ - 遍历全部 KEY   │
│    - 写请求: 同时更新内存    │ - 序列化写入     │
│      和 AOF/RDB              │ - 生成 dump.rdb  │
│                               │ - 原子重命名     │
└───────────────┬───────────────┴──────────┬───────┘
                │                          │
           主进程内存                   dump.rdb 文件
           保持一致                     (磁盘持久化)
                │
          ┌─────▼──────┐
          │ Redis 启动  │
          │ 读取 RDB    │
          │ 恢复数据    │
          └─────────────┘
```

---

### AOF (Append Only File)

**定义**: 基于操作日志的持久化,记录 Redis 执行过的所有写命令

**工作原理**:
1. 每个写命令执行后,追加到 AOF 文件
2. Redis 重启时,重放 AOF 文件中的命令恢复数据

**配置**:
```
appendonly no              # 开启/关闭 AOF(默认 no)
appendfilename "appendonly.aof"  # AOF 文件名
```

**启用 AOF**:
```
# 修改 redis.conf
appendonly yes
```

**优点**:
- 丢失数据少(可配置每秒/每条命令同步)
- 易读,便于调试和恢复

**缺点**:
- 文件体积大
- 恢复速度慢(需重放每条命令)

**AOF 重写**:
```bash
# 手动触发
BGREWRITEAOF

# 自动触发(配置)
auto-aof-rewrite-percentage 100   # AOF 文件大小超过上一次重写的 100%
auto-aof-rewrite-min-size 64mb    # AOF 文件最小为 64MB 时才重写
```

**重写流程**:
1. Fork 子进程
2. 子进程读取内存数据,生成新的 AOF 文件
3. 主进程继续接收客户端请求,同时写入缓冲
4. 子进程重写完成后,主进程将缓冲追加到新 AOF
5. 原子性用新 AOF 替换旧文件

#### RDB vs AOF 对比

| 维度 | RDB | AOF |
|------|-----|-----|
| **持久化形式** | 二进制快照 | 文本日志 |
| **文件大小** | 小(压缩) | 大 |
| **恢复速度** | 快 | 慢(需重放命令) |
| **数据丢失风险** | 可能丢失两次快照间的数据 | 可最小化损失(可配置同步频率) |
| **写入性能** | 影响小(仅快照时) | 影响中等(每条命令涉及 IO) |
| **使用场景** | 对数据完整性要求中等 | 对数据安全要求高 |

#### RDB + AOF 混合持久化

**配置**:
```
aof-use-rdb-preamble yes  # AOF 文件前部分使用 RDB 格式
```

**优势**: 结合两者优点,快速恢复 + 数据安全

---

### 缓存过期与淘汰

#### 过期机制

**定义**: 为 key 设置生存时间,到期后自动删除

**设置过期时间**:
```bash
EXPIRE key 30              # 30 秒后过期
PEXPIRE key 30000          # 30000 毫秒后过期
EXPIREAT key 1609459200    # 指定时间戳过期(秒)
PEXPIREAT key 1609459200000  # 指定时间戳过期(毫秒)
```

**查询过期时间**:
```bash
TTL key        # 返回剩余秒数(-1 永不过期,-2 不存在)
PTTL key       # 返回剩余毫秒数
```

**取消过期**:
```bash
PERSIST key    # 移除过期时间
```

**底层实现 - 过期字典**:
- 过期字典键: 指向键空间的某个 key 对象指针
- 过期字典值: long 型整数,保存 key 的过期时间戳(毫秒精度)

**优势**:
- 修改过期时间时只操作过期字典,不影响原 key
- 判断或删除过期键时仅遍历过期字典,不需全库扫描

#### 过期键删除策略

**三种策略**:

1. **定时删除** (Lazy Deletion)
   - 在 key 被访问时检查是否过期
   - 过期则删除后返回 nil
   - **优点**: 节省 CPU,及时删除
   - **缺点**: 未被访问的过期键一直存在

2. **定期删除** (Active Deletion)
   - 定期(周期性)扫描过期字典删除过期键
   - **优点**: 平衡 CPU 和内存
   - **缺点**: 扫描间隔内的过期键仍占内存

3. **惰性删除** (Passive Deletion)
   - 在访问 key 时检查过期,未访问则不处理
   - **缺点**: 可能导致内存泄漏

**Redis 采用**: 定期删除 + 定时删除 的混合策略

#### 内存淘汰策略

**触发场景**: 内存使用达到 maxmemory 限制

**淘汰策略**(配置 maxmemory-policy):

1. **noeviction** (默认)
   - 内存满时返回错误,不删除任何数据
   - 适用: 数据完整性要求高的场景

2. **allkeys-lru**
   - 在所有键中,淘汰最久未使用的键
   - 适用: 缓存场景(热数据保留)

3. **volatile-lru**
   - 在设置过期时间的键中,淘汰最久未使用的
   - 适用: 缓存混合场景

4. **allkeys-random**
   - 在所有键中随机淘汰
   - 适用: 无业务差异的缓存

5. **volatile-random**
   - 在设置过期的键中随机淘汰

6. **volatile-ttl**
   - 优先删除剩余 TTL 最短的键
   - 适用: 快过期的键优先移除

7. **allkeys-lfu** (Redis 4.0+)
   - 在所有键中,淘汰最少使用频次的键
   - 适用: 热度差异大的场景

8. **volatile-lfu** (Redis 4.0+)
   - 在设置过期的键中,淘汰最少使用频次的

**配置示例**:
```
maxmemory 256mb              # 最大内存限制
maxmemory-policy allkeys-lru # 淘汰策略
```

---

## 复制与高可用

### 主从复制 (Replication)

**架构**: 一主多从

**配置从节点**:
```bash
# 命令方式
SLAVEOF master-ip master-port

# 配置文件方式
slaveof <masterip> <masterport>
```

**特点**:
- 主从数据完全一致
- 支持读写分离(主写从读)
- 从服务器故障不影响主
- 主服务器故障仍可读(需手动切换)

**复制流程**:
1. 从连接主,发送 SYNC 命令
2. 主执行 BGSAVE,生成 RDB 快照
3. 主将 RDB 和后续命令流发送给从
4. 从加载 RDB 并重放命令

---

### Sentinel (哨兵)

**定义**: Redis 高可用解决方案,自动故障检测和转移

**原理**: 多个 Sentinel 实例监控主从服务器,主故障时自动提升从为新主

**关键概念**:

**主观下线状态**:
- Sentinel 在 `down-after-milliseconds` 毫秒内连续收到服务器无效回复
- 该 Sentinel 标记服务器为主观下线

**客观下线状态**:
- 一个 Sentinel 判定主为下线后,询问其他 Sentinel
- 多数 Sentinel 确认下线,则为客观下线
- 触发故障转移(Failover)

**故障转移流程**:
```
主服务器宕机
    │
    ▼
Sentinel 检测 (down-after-milliseconds)
    │
    ▼
标记主观下线
    │
    ▼
询问其他 Sentinel
    │
    ▼
确认客观下线
    │
    ▼
从从服务器中选择一个
    │
    ▼
提升为新主服务器
    │
    ▼
其他从连接新主
```

**配置示例** (sentinel.conf):
```
port 26379
sentinel monitor mymaster 127.0.0.1 6379 2  # 监控 127.0.0.1:6379 的主,需 2 个 Sentinel 确认
sentinel down-after-milliseconds mymaster 30000  # 30 秒无响应视为下线
sentinel parallel-syncs mymaster 1  # 故障转移时最多多少个从同时复制新主
sentinel failover-timeout mymaster 180000  # 故障转移超时(毫秒)
```

---

### Redis 集群 (Cluster)

**定义**: 分布式数据库方案,通过分片实现数据共享

**关键概念 - 槽 (Slot)**:
- 集群分为 16384 个槽(0-16383)
- 每个节点处理 0~16384 个槽
- 所有槽都被处理时,集群处于上线状态

**槽分配机制**:

**键到槽的映射**:
```
slot_id = CRC16(key) % 16384
```

**为何选 16384 个槽?**
- CRC16 理论范围: 0~65535(2^16)
- 实际选择: 2^14 = 16384
- 原因: 节点数通常不超过 1000 个,通信成本权衡

**集群节点结构** (clusterNode):
```c
struct clusterNode {
    mstime_t ctime;              // 创建时间
    char name[40];               // 节点名称
    int flags;                   // 节点标识(主/从/故障等)
    char ip[46];                 // IP 地址
    int port;                    // 端口号
    clusterState *state;         // 集群状态视图
    char slots[16384/8];         // 二进制位数组,记录负责的槽
    int numslots;                // 负责的槽数量
    // ...
};
```

**集群状态结构** (clusterState):
```c
struct clusterState {
    clusterNode myself;              // 当前节点指针
    dict nodes;                      // 节点名单(HashMap)
    clusterNode *slots[16384];       // 每个槽指向对应的 clusterNode
    // ...
};
```

**槽查询优化**:

Q: 既然 clusterNode 记录了自己负责的槽,为何还需 clusterState.slots[]?

A:
- 查询"某槽由哪个节点负责": 遍历 clusterNode.slots[] 需 O(N) 复杂度(N=节点数),而 clusterState.slots[i] 直接返回为 O(1)
- 统计"某节点负责哪些槽": 遍历 clusterNode.slots[] 为 O(1),遍历 clusterState.slots[] 需 O(16384)
- 结论: 时间和空间的权衡

**启用集群**:
```
# redis.conf
cluster-enabled yes      # 必须为 yes
cluster-config-file nodes.conf  # 集群配置文件
cluster-node-timeout 15000      # 节点超时时间
```

---

## 缓存问题处理

### 缓存穿透 (Cache Penetration)

**定义**: 查询的 key 在缓存中不存在,数据库中也不存在,导致每次请求都穿过缓存查数据库

**表现**: 大量查询不存在的数据,数据库压力大

**成因**:
- 恶意请求(尝试注入非法 ID)
- 业务异常(业务删除了数据但缓存未清)
- 数据异常(并发修改导致数据不一致)

#### 解决方案

**方案 1: 布隆过滤器 (Bloom Filter)**

原理: 使用位数组和多个哈希函数判断元素是否可能存在
```
初始化: 将数据库中所有有效 ID 加入布隆过滤器
查询: 先检查布隆过滤器
  - 判定为"不存在" → 直接返回空(100%准确)
  - 判定为"可能存在" → 查缓存或数据库
```

**优点**: 节省内存,快速判断

**缺点**: 存在误判(判定存在但实际不存在),且难以删除元素

**方案 2: 缓存 NULL 值**

原理: 数据库查询为空时,也在缓存中存储 NULL/空值
```
if (cache.get(key) == NULL) {
    return "无数据";  // 直接返回,不查数据库
}

if (cache.get(key) == empty_value) {
    data = db.query(key);
}
```

**注意**: 设置合理的过期时间(如 5 分钟),避免一直占用空间

**在 Redis 中实现**:
```bash
# 数据库无数据时
SET cache:nonexist:123 "NULL" EX 300

# 查询逻辑
value = GET cache:nonexist:123
if (value == "NULL") {
    return 无数据
}
```

---

### 缓存雪崩 (Cache Avalanche)

**定义**: 大批量 key 同时过期,缓存一次性失效,导致所有请求压到数据库

**表现**: 缓存失效后,数据库瞬间流量暴增,可能导致数据库宕机

**成因**:
- 批量导入数据时设置相同过期时间
- 热点数据集中过期
- Redis 宕机(所有数据失效)

#### 解决方案

**方案 1: 错开过期时间**

原理: 避免大量 key 同时过期

```java
// 设置过期时间时,随机增加偏移量
int baseExpire = 3600;  // 1 小时
int randomOffset = random(0, 600);  // 0~10 分钟随机偏移
int finalExpire = baseExpire + randomOffset;

cache.set(key, value, finalExpire);
```

**方案 2: 热点数据不设置过期**

```bash
# 热点商品信息
SET hotproduct:1001 "..." # 不设置过期时间

# 定时更新
DUMP hotproduct:1001  # 定时刷新数据
```

**方案 3: 多级缓存**

```
请求
  │
  ▼
┌─────────────────┐
│ 本地缓存(L1)     │ 快速,小容量
└─────────────────┘
  未命中│
  ▼
┌─────────────────┐
│ Redis 缓存(L2)   │ 中速,中容量
└─────────────────┘
  未命中│
  ▼
┌─────────────────┐
│ 数据库(L3)       │ 慢速,大容量
└─────────────────┘
```

**方案 4: 缓存预热**

在 Redis 启动或流量高峰前,将热点数据提前加载到缓存

```java
@PostConstruct
public void cacheWarmup() {
    List<Product> hotProducts = db.findHotProducts();
    hotProducts.forEach(p -> 
        cache.set("product:" + p.getId(), p)
    );
}
```

---

### 缓存击穿 (Cache Breakdown)

**定义**: 某个热点 key 缓存过期,此时大量请求并发查询,都穿过缓存查数据库

**表现**: 单个热点 key 过期时,数据库短时间内压力陡增

**成因**: 热点数据过期 + 高并发访问时间重合

#### 解决方案

**方案 1: 加互斥锁 (Mutex Lock)**

原理: 第一个请求获得锁,去数据库查询并更新缓存;其他请求等待,然后读缓存

```java
public Object getData(String key) {
    Object value = cache.get(key);
    if (value == null) {
        synchronized(lock) {  // 获得锁
            value = cache.get(key);  // 再检查一次(Double Check Locking)
            if (value == null) {
                value = db.query(key);
                cache.set(key, value, expire);
            }
        }
    }
    return value;
}
```

**分布式环境实现** (使用 Redis 锁):
```bash
# 伪代码
def get_data(key):
    value = redis.get(key)
    if value == None:
        # 尝试获得分布式锁
        lock_key = "lock:" + key
        if redis.setnx(lock_key, "1"):  # 成功获得锁
            redis.expire(lock_key, 10)
            try:
                value = db.query(key)
                redis.set(key, value, expire)
            finally:
                redis.delete(lock_key)
        else:
            # 未获得锁,等待后重试
            time.sleep(0.1)
            return get_data(key)
    return value
```

**方案 2: 设置过期时间软过期**

原理: 逻辑过期时间 < 物理过期时间,定时更新热点数据

```java
public class CacheData {
    private Object value;
    private long logicalExpireTime;  // 逻辑过期时间
}

public Object getData(String key) {
    CacheData cacheData = cache.get(key);
    
    if (cacheData == null) {
        return null;
    }
    
    if (System.currentTimeMillis() < cacheData.logicalExpireTime) {
        return cacheData.value;  // 未逻辑过期,直接返回
    }
    
    // 逻辑过期,后台更新
    updateAsync(key);
    return cacheData.value;  // 返回旧数据,避免查库
}
```

**方案 3: 预加载和续期**

```bash
# 热点数据每 1 小时刷新一次
EVERY 1 HOUR:
  DUMP hotkey
  (或延长过期时间)
  SET hotkey newvalue EX 3600
```

#### 缓存穿透 vs 击穿 vs 雪崩对比

| 维度 | 穿透 | 击穿 | 雪崩 |
|------|------|------|------|
| **成因** | 数据不存在 | 热点过期 | 大批过期 |
| **特征** | 缓存和数据库都无数据 | 单个热点过期 | 大量 key 同时过期 |
| **影响** | 少量恶意请求影响 | 热点访问时数据库压力大 | 整体缓存失效 |
| **解决** | 布隆/NULL 缓存 | 互斥锁/软过期 | 错开过期/预热 |

---

## 底层数据结构

### SDS (简单动态字符串)

已在 String 数据结构部分详解。

### 字典 (Dict)

**定义**: Redis 中的哈希表实现,用于实现键空间和 Hash 数据结构

**结构**:
```c
typedef struct dict {
    dictType *type;       // 类型特定函数指针
    void *privdata;       // 类型私有数据
    dictht ht[2];         // 两个哈希表(用于 rehash)
    long rehashidx;       // rehash 索引(-1 表示未 rehash)
} dict;

typedef struct dictht {
    dictEntry **table;    // 哈希表数组
    unsigned long size;   // 哈希表大小
    unsigned long sizemask;  // size - 1,用于计算 hash 值对应索引
    unsigned long used;   // 已使用节点数
} dictht;

typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;  // 链表法解决哈希冲突
} dictEntry;
```

**哈希冲突处理**: 链表法(链地址法)

**Rehash 机制** (渐进式):
```
数据量增长 → size 不足 → 触发 rehash
  │
  ▼
分配新的 ht[1](大小为 ht[0] 的 2 倍)
  │
  ▼
每次操作时,将 ht[0] 的部分数据迁移到 ht[1]
  │
  ▼
所有数据迁移完成 → 释放旧 ht[0] → ht[1] 变为主表
```

**好处**: 避免一次性 rehash 导致的性能抖动

---

### 跳表 (Skip List)

**定义**: 高效的有序数据结构,用于 Sorted Set 实现

**原理**: 在有序链表基础上,增加多层索引加速查找

**结构**:
```
┌─────────┬─────────┬─────────┬──────────┐ level 2
│  nil    │   (10)  │   (30)  │   nil    │
└────┬────┴────┬────┴────┬────┴────┬─────┘
     │         │         │         │
┌────▼────┬───▼────┬────▼────┬───▼────┐ level 1
│  nil    │ (10)   │   (20)  │ (30)   │
└────┬────┴────┬───┴────┬────┴────┬───┘
     │         │        │         │
┌────▼────┬───▼─┬───┬───▼─┬──┬───▼─┬───┐ level 0
│  nil    │(10) │(15)│(20)│(25)│(30)│nil│
└─────────┴─────┴───┴────┴───┴────┴───┘
```

**查找流程**:
```
寻找 25:
  从顶层开始 → 10 < 25 → 30 > 25 → 下一层
  在 level 1: 10 < 25 → 20 < 25 → 30 > 25 → 下一层
  在 level 0: 10 → 15 → 20 → 25 (找到)
```

**时间复杂度**: O(log N) 查找 + O(1) 范围访问

**VS 红黑树**:
- 实现简单,易于并发
- 范围查询效率高(链表结构)
- 插删时无需大量旋转操作

---

### 压缩列表 (ZipList)

**定义**: 内存紧凑的列表,用于 List 和 Hash 在元素少时的编码

**特点**:
- 顺序存储,消除指针开销
- 节约内存(相比链表/哈希表)
- 元素过多时会转换为正常数据结构

**应用条件**:
- Hash: 元素少且字段名/值都较小
- List: 元素少且值较小

**缺点**:
- 添加/删除时可能需要重新分配内存
- 不适合频繁修改

---

### 快速列表 (QuickList)

**定义**: List 在 Redis 5.0+ 的新编码,综合链表和压缩列表的优点

**结构**: 多个压缩列表连接成一个双向链表

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ ZipList 1    │──│ ZipList 2    │──│ ZipList 3    │
│[1,2,3]      │  │[4,5,6]       │  │[7,8,9]      │
└──────────────┘  └──────────────┘  └──────────────┘
```

**优势**:
- 保留链表的灵活性(易于添加/删除节点)
- 使用压缩列表节约内存
- 平衡时间和空间

**配置**:
```
list-max-ziplist-size -2      # 单个 ziplist 最多 8KB
list-compress-depth 0         # 0=不压缩,1=两端不压缩中间压缩
```

---

## 缓存一致性与性能

### 缓存更新策略

#### Cache-Aside (旁路缓存)

原理: 应用程序负责缓存和数据库的一致性

```
读流程:
  查缓存 → 命中返回 → 未命中查数据库 → 写缓存 → 返回

写流程:
  先更新数据库 → 再删除缓存 (推荐)
  或
  更新缓存 + 更新数据库 (延迟删除缓存)
```

**优点**: 逻辑简单,适用大多数场景

**缺点**: 缓存失效窗口可能导致数据不一致

#### Write-Through (写穿)

原理: 写操作同时更新缓存和数据库

```
写流程:
  更新缓存 → 更新数据库 → 返回
```

**优点**: 缓存与数据库一致

**缺点**: 写操作性能受限,每次必须写库

#### Write-Behind (写回)

原理: 先写缓存,异步写数据库

```
写流程:
  更新缓存 → 返回 → 异步更新数据库
```

**优点**: 写操作速度快,可批量提交

**缺点**: 缓存宕机丢数据,数据一致性风险

---

### 分布式缓存的性能问题

**问题**: 网络 I/O 成本高,频繁往返浪费时间

**场景举例**:
```java
// N+1 问题
List<User> users = userService.findUserList(deptId);
for (User user : users) {
    // 每个用户都调用一次 Redis,造成 N 次网络往返
    String deptName = deptService.getDeptName(user.getDeptId());
    user.setDeptName(deptName);
}
```

**优化方案**:

**方案 1: 本地缓存字典**
```java
// 程序启动时加载字典
Map<String, String> deptCache = new HashMap<>();
for (Dept dept : deptService.findAll()) {
    deptCache.put(dept.getId(), dept.getName());
}

// 循环中直接使用本地缓存
for (User user : users) {
    user.setDeptName(deptCache.get(user.getDeptId()));
}
```

**方案 2: Redis 管道批量查询**
```bash
# Redis 管道,一次发送多个命令,减少网络往返
PIPELINE:
  GET dept:1
  GET dept:2
  GET dept:3
  ...
```

**方案 3: 扩展数据结构**
```bash
# 在 User Hash 中直接存储部门名称
HSET user:1 name "zhangsan" deptId 10 deptName "技术部"
```

---

## 命令执行流程

### Redis 命令执行链路

```
┌──────────────────────────────────────────────────────────┐
│ 1. 客户端 (Client)                                        │
│    - 本地缓冲命令参数                                      │
│    - 通过网络发送到 Redis Server                          │
└────────────────┬─────────────────────────────────────────┘
                 │ TCP 连接
┌────────────────▼─────────────────────────────────────────┐
│ 2. Redis 服务器接收 (Server)                             │
│    - 接收网络请求                                         │
│    - 解析命令协议 (RESP)                                  │
└────────────────┬─────────────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────────────┐
│ 3. 事件循环 (Event Loop)                                  │
│    - 多路复用 (epoll/kqueue)                             │
│    - 处理读写事件                                         │
│    - 周期性执行 Redis 自身任务                           │
└────────────────┬─────────────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────────────┐
│ 4. 命令处理器 (Command Processor)                         │
│    - 查找命令对应的处理函数                                │
│    - 参数类型检查                                         │
│    - 调用相应的命令实现                                    │
└────────────────┬─────────────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────────────┐
│ 5. 数据结构操作 (Data Structure)                          │
│    - Dict (键空间查找)                                     │
│    - SDS/List/Hash/Set/ZSet/etc (具体操作)              │
│    - 更新过期字典/淘汰策略                                │
└────────────────┬─────────────────────────────────────────┘
                 │
┌────────────────▼─────────────────────────────────────────┐
│ 6. 响应编码 (Response Encoding)                           │
│    - 将结果转换为 RESP 协议                              │
│    - 发送给客户端                                         │
└────────────────┬─────────────────────────────────────────┘
                 │ TCP 连接
┌────────────────▼─────────────────────────────────────────┐
│ 7. 客户端接收 (Client Receive)                            │
│    - 接收 RESP 格式响应                                   │
│    - 反序列化为对象                                       │
└──────────────────────────────────────────────────────────┘
```

### 单线程模型与性能

**Redis 单线程**: 命令执行阶段单线程处理,但网络 I/O 使用多路复用

**为何单线程?**
1. 避免线程同步和上下文切换开销
2. 内存操作极快,避免锁竞争
3. 代码逻辑简单,易于维护

**多线程操作**:
- Redis 4.0+: 支持异步删除(UNLINK)、异步重写 AOF
- Redis 6.0+: 网络 I/O 多线程(提升吞吐量)

---

## Lua 脚本与事务

### 事务

**定义**: 原子性执行一组命令

**命令**:
```bash
MULTI         # 开始事务
[command1]    # 命令入队
[command2]
...
EXEC          # 执行事务
DISCARD       # 放弃事务
WATCH key     # 监视 key,如果在 MULTI 前被修改则事务失败
UNWATCH       # 取消监视
```

**特点**:
- 命令入队,不立即执行
- EXEC 时一次性执行(不被中断)
- 不支持回滚(执行失败不会 rollback)
- 没有隔离级别概念

**示例**:
```bash
WATCH account:1
MULTI
DECRBY account:1 100    # 扣款
INCRBY account:2 100    # 入账
EXEC
```

### Lua 脚本

**定义**: 在 Redis 服务端执行 Lua 脚本,保证原子性

**优势**: 
- 原子执行,避免竞态条件
- 减少网络往返
- 支持复杂逻辑

**命令**:
```bash
EVAL "return redis.call('GET', KEYS[1])" 1 mykey
EVALSHA script_sha1 1 mykey
SCRIPT LOAD script_content   # 预加载脚本
SCRIPT EXISTS script_sha1
SCRIPT FLUSH                 # 清空脚本缓存
```

**示例 - 分布式锁**:
```lua
-- 加锁 (原子操作)
-- KEYS[1]: 锁 key, ARGV[1]: 过期时间, ARGV[2]: 唯一值
if redis.call('exists', KEYS[1]) == 0 then
    redis.call('set', KEYS[1], ARGV[2])
    redis.call('expire', KEYS[1], ARGV[1])
    return 1
else
    return 0
end
```

---

## 慢查询日志

**定义**: 记录执行时间超过阈值的命令,用于性能优化

**配置**:
```
slowlog-log-slower-than 10000    # 超过 10000 微秒(10ms)记录
slowlog-max-len 128              # 最多保存 128 条日志
```

**命令**:
```bash
SLOWLOG GET [count]           # 获取最新 N 条慢查询日志
SLOWLOG LEN                   # 获取日志条数
SLOWLOG RESET                 # 清空日志
```

**日志字段**:
```
1) 日志序列号
2) 时间戳
3) 执行耗时(微秒)
4) 命令及参数
5) 客户端地址
6) 客户端名称
```

---

## 典型应用场景

### 分布式锁

**需求**: 多个进程/服务竞争同一资源,需互斥访问

**实现方案**:

**方案 1: SET NX + Expire**

```lua
-- 加锁 (Lua 脚本保证原子性)
if redis.call('SET', KEYS[1], ARGV[1], 'NX', 'EX', ARGV[2]) then
    return 1
else
    return 0
end

-- 释放锁 (检查值是否匹配,防止误删)
if redis.call('GET', KEYS[1]) == ARGV[1] then
    redis.call('DEL', KEYS[1])
    return 1
else
    return 0
end
```

**流程**:
```
业务进程 1              Redis               业务进程 2
  │                     │                      │
  ├─ SET lock:x V1 NX ─→│                      │
  │                     ├─ OK ─→ 获得锁       │
  │                     │                      │
  │              (执行业务逻辑)                 │
  │                     │                      │
  │                     │ ← SET lock:x V2 NX ─┤
  │                     │                      │
  │                     ├─ NIL (失败) ────────→ 等待
  │                     │                      │
  │  (业务完成)         │                      │
  │                     │                      │
  ├─ DEL lock:x ────────→│                      │
  │                     │ ← SET lock:x V2 NX ─┤
  │                     │                      │
  │                     ├─ OK ─→ 获得锁       │
  └                     └                      └
```

**关键点**:
- 使用唯一值(UUID)防止误删他人的锁
- 使用 Lua 脚本保证"检查+删除"原子性
- 设置过期时间防止死锁

**方案 2: Redis SET + Mutex(互斥锁阻塞)**

```java
public class RedisDistributedLock {
    private static final long DEFAULT_TIMEOUT = 10 * 1000;  // 10秒
    
    public boolean lock(String key, long timeout) {
        long endTime = System.currentTimeMillis() + timeout;
        String value = UUID.randomUUID().toString();
        
        while (System.currentTimeMillis() < endTime) {
            Boolean locked = redis.setIfAbsent(key, value, Duration.ofSeconds(10));
            if (locked) {
                this.value = value;
                return true;
            }
            Thread.sleep(10);  // 等待 10ms 后重试
        }
        return false;
    }
    
    public void unlock(String key) {
        // Lua 脚本检查值是否匹配再删除
        redis.eval("""
            if redis.call('GET', KEYS[1]) == ARGV[1] then
                return redis.call('DEL', KEYS[1])
            else
                return 0
            end
        """, Collections.singletonList(key), value);
    }
}
```

### 限流

**需求**: 限制单位时间内的请求数

**方案 1: 滑动窗口(令牌桶)**

```lua
-- KEYS[1]: 限流 key, ARGV[1]: 最大请求数, ARGV[2]: 时间窗口(秒)
local key = KEYS[1]
local max_count = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local current_time = tonumber(ARGV[3])

local count = redis.call('GET', key)
if not count then
    redis.call('SET', key, 1)
    redis.call('EXPIRE', key, window)
    return 1
elseif tonumber(count) < max_count then
    redis.call('INCR', key)
    return tonumber(count) + 1
else
    return 0  -- 限流
end
```

**方案 2: 漏桶 (Leaky Bucket)**

```bash
# 初始化:每秒处理 10 个请求
# 使用 ZSet,score 为时间戳

ZADD rate_limit:user:123 1000 req1
ZADD rate_limit:user:123 2000 req2
...

# 检查: 移除 1 分钟外的请求,计算当前窗口内的请求数
ZREMRANGEBYSCORE rate_limit:user:123 -inf (now - 60000)
ZCARD rate_limit:user:123
```

### 排行榜

**需求**: 实时排序,支持增删改查

**实现**: Sorted Set

```bash
# 添加或更新分数
ZADD leaderboard 100 "user1"
ZADD leaderboard 200 "user2"
ZADD leaderboard 150 "user3"

# 获取前 10 名 (分数最高)
ZREVRANGE leaderboard 0 9 WITHSCORES

# 获取用户排名 (分数从高到低)
ZREVRANK leaderboard "user2"

# 获取分数范围内的用户
ZREVRANGEBYSCORE leaderboard 200 100

# 增加分数
ZINCRBY leaderboard 50 "user1"
```

**实时排行榜流程**:
```
游戏结束,玩家得分 +100
        │
        ▼
ZINCRBY leaderboard 100 user:123
        │
        ▼
查询排名
ZREVRANK leaderboard user:123
        │
        ▼
实时返回排名信息
```

### 会话缓存 (Session Cache)

**需求**: 存储用户登录信息,快速查询

**实现**:

**方案 1: String + JSON**
```bash
SET session:token:abc123 '{"userId":"1","username":"zhangsan","loginTime":1234567890}'
EXPIRE session:token:abc123 1800  # 30 分钟过期
```

**方案 2: Hash**
```bash
HSET session:token:abc123 userId 1 username zhangsan loginTime 1234567890
EXPIRE session:token:abc123 1800
```

**查询和更新**:
```bash
# 查询
HGETALL session:token:abc123

# 更新某个属性
HSET session:token:abc123 lastAccessTime 1234567999
```

---

## 核心知识速查

### 命令速查表

#### String 命令
| 命令 | 功能 |
|------|------|
| SET/GET | 设置/获取 |
| APPEND | 追加 |
| STRLEN | 长度 |
| INCR/DECR | 自增/自减 |
| INCRBY/DECRBY | 指定增减 |
| MSET/MGET | 批量设置/获取 |
| GETRANGE/SETRANGE | 子串获取/设置 |
| SETEX/PSETEX | 设置+过期(秒/毫秒) |
| SETNX | 不存在才设置 |

#### Hash 命令
| 命令 | 功能 |
|------|------|
| HSET/HGET | 设置/获取字段 |
| HMSET/HMGET | 批量设置/获取 |
| HGETALL | 获取所有字段和值 |
| HKEYS/HVALS | 获取所有字段名/值 |
| HEXISTS | 判断字段存在 |
| HDEL | 删除字段 |
| HLEN | 字段数量 |
| HINCRBY | 字段值增加 |

#### List 命令
| 命令 | 功能 |
|------|------|
| LPUSH/RPUSH | 左/右插入 |
| LPOP/RPOP | 左/右移除 |
| BLPOP/BRPOP | 阻塞左/右移除 |
| LLEN | 长度 |
| LRANGE | 范围查询 |
| LINDEX | 指定下标 |
| LSET | 设置指定下标 |
| LTRIM | 保留范围 |

#### Set 命令
| 命令 | 功能 |
|------|------|
| SADD/SREM | 添加/移除 |
| SMEMBERS | 获取所有 |
| SCARD | 成员数量 |
| SISMEMBER | 判断存在 |
| SINTER/SUNION/SDIFF | 交/并/差集 |
| SPOP | 随机移除 |
| SRANDMEMBER | 随机返回 |

#### Sorted Set 命令
| 命令 | 功能 |
|------|------|
| ZADD | 添加 member-score |
| ZREM | 移除 |
| ZSCORE | 获取分值 |
| ZRANK/ZREVRANK | 正/反排名 |
| ZRANGE/ZREVRANGE | 正/反范围 |
| ZRANGEBYSCORE | 分值范围 |
| ZINCRBY | 增加分值 |
| ZCARD/ZCOUNT | 成员数/范围数 |

#### Key 相关命令
| 命令 | 功能 |
|------|------|
| DEL | 删除 key |
| EXISTS | 判断存在 |
| EXPIRE/PEXPIRE | 设置过期(秒/毫秒) |
| TTL/PTTL | 查询剩余(秒/毫秒) |
| PERSIST | 移除过期 |
| KEYS | 模式匹配(不推荐生产) |
| SCAN | 游标扫描 |
| TYPE | 查看数据类型 |
| RENAME | 重命名 |

### 数据结构对照表

| 数据类型 | 底层实现 | 应用场景 | 命令示例 |
|---------|---------|---------|---------|
| String | SDS | 缓存对象/计数 | SET/GET/INCR |
| Hash | Dict/ZipList | 对象属性存储 | HSET/HGET |
| List | QuickList | 队列/栈/消息流 | LPUSH/RPOP |
| Set | Dict/IntSet | 去重/标签/权限 | SADD/SINTER |
| Sorted Set | ZipList/SkipList | 排行榜/延时队列 | ZADD/ZRANGE |
| Bitmap | 二进制数组 | 签到/权限位 | SETBIT/BITCOUNT |
| HyperLogLog | 专用结构 | UV 统计 | PFADD/PFCOUNT |
| Geo | ZSet(geohash) | 地理位置 | GEOADD/GEODIST |
| Stream | 专用结构 | 消息队列 | XADD/XREAD |

### 场景解决方案对照

| 场景 | 问题 | 解决方案 | 关键点 |
|------|------|---------|--------|
| 缓存穿透 | 数据不存在 | 布隆过滤器/NULL缓存 | 防止数据库压力 |
| 缓存雪崩 | 大量key同时过期 | 错开过期时间/预热/多级缓存 | 分散过期时间 |
| 缓存击穿 | 热点key过期 | 互斥锁/软过期/预加载 | 高并发保护 |
| 分布式锁 | 资源竞争 | SET NX + Expire + Lua | 唯一值+原子性 |
| 限流 | 请求过多 | 令牌桶/滑动窗口 | 时间窗口+计数 |
| 排行榜 | 实时排序 | Sorted Set | score自动排序 |
| 会话存储 | 用户信息 | Hash/String+JSON | TTL自动过期 |
| 消息队列 | 异步处理 | List BRPOP/Stream | 阻塞式消费 |

---

## 原文勘误

### ⚠️ 发现的错误与修正

#### 1. SDS 3.2 前版本 len 字段说明错误

**原文** (第 291 行):
```
假设 len = 10，代表这个 SDS 保留了一个 5 byte 长度的字符串
```

**错误**: len = 10 应表示 10 字节长度,不是 5 字节

**修正**:
```
假设 len = 10，代表这个 SDS 保存了一个 10 字节长度的字符串
```

---

#### 2. HMSET 命令已过时

**原文** (第 339-340 行):
```
hmset: 将哈希表 key 中的域 field 的值设为 value
```

**问题**: HMSET 在 Redis 4.0 后已过时,推荐使用 HSET

**修正**:
```
HSET: 设置一个或多个字段值(Redis 4.0+ 推荐,取代 HMSET)
HMSET: (已过时)批量设置多个字段值,现推荐用 HSET 替代
```

---

#### 3. 链表节点结构定义不完整

**原文** (第 371-381 行):
```c
typedef struct listNode{
    struct listNode prev;
    struct listNode next;
    void value;
}
```

**问题**: 指针符号 `*` 缺失

**修正**:
```c
typedef struct listNode {
    struct listNode *prev;  // 前置节点(指针)
    struct listNode *next;  // 后置节点(指针)
    void *value;            // 值(指针)
} listNode;
```

---

#### 4. BITFIELD 命令说明缺失

**原文**: 未提及 BITFIELD 命令

**补充**: Redis 3.2+ 支持位域操作,常用于紧凑存储多个位字段

```bash
# 获取无符号 4 位值(位置 0)
BITFIELD mykey GET u4 0

# 设置无符号 4 位值
BITFIELD mykey SET u4 0 15

# 自增操作
BITFIELD mykey INCRBY u4 0 1 OVERFLOW SAT
```

---

#### 5. 集群槽数量的注释不准确

**原文** (第 653-655 行):
```
理论上CRC16算法可以得到2^16个数值,其数值范围在0-65535之间,也就是最多可以有65535个虚拟槽,
取模运算key的时候,应该是CRC(key)%65535
```

**错误**: 
- 2^16 = 65536 个值(0~65535),但表述为"最多 65535 个虚拟槽"不准确
- 应为 `CRC16(key) % 16384` 而非 `% 65535`

**修正**:
```
理论上 CRC16 算法可以得到 2^16 = 65536 个值(范围 0-65535),最多可支持 65536 个虚拟槽。
实际中 Redis Cluster 采用 2^14 = 16384 个槽,计算公式为 CRC16(key) % 16384。
原因: 节点通信成本权衡,节点数通常不超过 1000 个。
```

---

#### 6. 缓存策略的集合操作命令说明

**原文** (Set 部分):
```
smove: 将 member 元素从 source 集合移动到 destination 集合
```

**补充**: SMOVE 在 Redis 2.4+ 支持,返回值为移动成功(1)或失败(0)

```bash
# 正确用法
SMOVE source dest member
# 返回 1: 成功
# 返回 0: 失败(源集合无此元素或目标已存在)
```

---

#### 7. Sorted Set ZRANGEBYSCORE 参数说明不全

**原文** (第 453 行):
```
zrangebyscore: 返回有序集 key 中,所有 score 值介于 min 和 max 之间(包括等于 min 或 max)的成员
```

**补充**: 支持开区间和特殊值

```bash
# 标准用法(闭区间)
ZRANGEBYSCORE myzset 100 200

# 开区间
ZRANGEBYSCORE myzset (100 200    # 不含 100,包含 200
ZRANGEBYSCORE myzset 100 (200    # 包含 100,不含 200

# 无穷大
ZRANGEBYSCORE myzset -inf +inf   # 所有成员
ZRANGEBYSCORE myzset (100 +inf   # > 100 的所有成员
```

---

#### 8. 缺少 String 类型的 GETDEL 和 GETEX 命令

**原文**: Redis 6.2+ 新增命令未提及

**补充**:
```bash
# GETDEL: 获取并删除(原子操作)
GETDEL key
# 返回 key 的值,同时删除 key

# GETEX: 获取并设置过期时间(原子操作)
GETEX key EX 3600     # 获取后设置 1 小时过期
GETEX key EXAT 1234567890  # 设置过期时间戳
GETEX key PERSIST     # 移除过期时间
```

---

### 总结

文档中发现 **8 处** 值得关注的错误或遗漏:
1. SDS len 说明错误
2. HMSET 过时未标注
3. C 结构体指针符号缺失
4. BITFIELD 命令缺失
5. 槽数量计算不准确
6. SMOVE 返回值说明不全
7. ZRANGEBYSCORE 开区间语法缺失
8. Redis 6.2+ 新命令缺失

---

## 总结

本梳理文档共包含 **9 个主章节**,详细涵盖:

1. **缓存基础** - 缓存的定义、作用、挑战
2. **缓存技术对比** - 本地/Ehcache/Memcache/Redis 选型
3. **Redis 基础** - 环境搭建、配置、数据库概念
4. **数据结构详解** - 9 种主要数据类型(String/Hash/List/Set/ZSet/Bitmap/HyperLogLog/Geo/Stream)
5. **持久化机制** - RDB/AOF/过期淘汰
6. **复制与高可用** - 主从/Sentinel/Cluster
7. **缓存问题处理** - 穿透/雪崩/击穿的原因与解决方案
8. **底层数据结构** - SDS/Dict/SkipList/ZipList/QuickList 的实现原理
9. **应用场景** - 分布式锁/限流/排行榜/会话缓存等

**流程图** 5 个:
- Redis 命令执行链路
- RDB 持久化流程
- Sentinel 故障转移流程
- 缓存穿透/击穿/雪崩解决方案流程
- 分布式锁实现流程

**勘误** 8 处已标注并修正。

---

*文档完成于 2026-05-24,基于原文系统梳理。*
