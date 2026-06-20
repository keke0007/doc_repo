# 08 · MongoDB 副本集故障自动切换

## 一、知识点梳理

### 1. 切换原理
- PRIMARY 失联后，剩余成员发起选举。
- 当前实例需要获得 **多数派（majority）** 投票才能晋升 PRIMARY。
- 三节点副本集：宕一台还剩 2 票仍可达多数派，集群可写；宕两台只剩 1 票，无法形成多数派，集群进入「只读 → 最终全部 SECONDARY」状态。

### 2. 演示步骤
```bash
# 关掉当前主
/usr/local/mongodb/bin/mongo 127.0.0.1:27019
> use admin
> db.shutdownServer()        # 退出后会观察到 27017 或 27018 升主，写入正常

# 再关一台
/usr/local/mongodb/bin/mongo 127.0.0.1:27018
> use admin
> db.shutdownServer()        # 仅剩 27017 一票，无法选主，写入失败

# 重新拉起
/usr/local/mongodb/bin/mongod -f /data/mongodb/27017/mongodb.conf
/usr/local/mongodb/bin/mongod -f /data/mongodb/27018/mongodb.conf
```

### 3. 现象
- 自动切换的窗口期内（默认 ~10s），客户端会出现 `not master` 等异常，pymongo 这类驱动会自动重连到新主。
- 若想固定某个节点优先成为主，需要调整优先权重（详见 09 章）。

### 4. 选主条件回顾
| 条件 | 说明 |
|------|------|
| 多数派可达 | 投票成员中存活数 > 一半 |
| 优先权 priority > 0 | 0 表示永不参选 |
| 数据足够新 | 候选人 oplog 必须不落后于其它存活成员 |
| 隐藏/延迟节点不会主动竞选 | `hidden:true` 或 `slaveDelay > 0` 时通常 `priority = 0` |

### 5. 多数派与仲裁节点
- 当只有 2 个数据节点时，可加入一个 ARBITER（仲裁节点）凑成奇数票，例如：
  ```js
  rs.addArb('192.168.237.130:27020')
  ```
  仲裁不持有数据，只参与投票。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "挑选其中一台 secondary 升级为 primary 的条件是剩下的集群台数 ≥ 2" | 表述粗糙 | 真正条件是**存活的可投票成员超过半数**；3 成员副本集中存活 2 即可选主，宕到只剩 1 时无法选主 |
| 2 | "如果集群只剩下一个实例的话，会有异常" | 描述不准 | 准确说法：剩 1 实例时该节点会**变为 SECONDARY** 且写入失败，只能读旧数据；不是简单的"异常" |
| 3 | 未给出超时与切换耗时 | 缺细节 | `electionTimeoutMillis` 默认 10s，超过后 SECONDARY 才发起选举；新主上任前期写入会被驱动重试 |
| 4 | 没有提到驱动层的故障转移 | 缺细节 | pymongo / Java 驱动应当传 **副本集 URI**（`mongodb://h1,h2,h3/?replicaSet=shijiange`），驱动会自动跟踪新主 |
| 5 | "通过优先级指定 primary"是模糊提到 | 没给完整命令 | 见 09 章：`conf.members[i].priority = N; rs.reconfig(conf)` |

## 三、自动切换时序图（ASCII）

```
  时间 →
                    电源/进程 异常
   PRIMARY :27019  ──×── 失联
        │
        │ 心跳超时 (~10s)
        ▼
  SECONDARY :27017  ──┐
                       │ 发起选举
  SECONDARY :27018  ──┤   (要求多数派)
        │              │
        ▼              ▼
   投票汇总：2/3 同意 → :27018 成为新 PRIMARY
        │
        ▼
   客户端 (副本集 URI 模式)：
     ── 旧连接报 "not master"
     ── 自动发现新拓扑，重连 :27018
        │
        ▼
   写入恢复 ✔

  ── 二次故障：再宕一台 ──
   :27017 / :27019 都不可达
        │
        ▼
   :27018 失去多数派 → 自降为 SECONDARY
        │
        ▼
   集群只读，写入返回错误
```
