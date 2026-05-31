# 10 · MongoDB 副本集的伸缩

## 一、知识点梳理

### 1. 加节点
1. 按已有节点的相同 `replSetName` 和优化参数启动新实例（例如 27020）。
2. 在 PRIMARY 执行：
   ```js
   use admin
   rs.add('192.168.237.129:27020')
   ```
3. 新成员加入后默认状态：
   - `STARTUP2 → SECONDARY`，期间会全量克隆 PRIMARY 的数据，再追 oplog。
   - `priority = 1`、`votes = 1`。
4. 高级用法：
   ```js
   rs.add({ host:'h:27020', priority:0, hidden:true, votes:0 })  // 隐藏备份节点
   rs.addArb('h:27021')                                          // 仲裁节点
   ```

### 2. 减节点
- 不能直接 `rs.remove(<primary>)`：必须先让其降级或换主。
  ```js
  rs.stepDown(60)      // 当前主主动让位 60s
  ```
- 移除：
  ```js
  use admin
  rs.remove('192.168.237.128:27019')
  ```

### 3. 加减后 members 顺序会乱
- `rs.add/rs.remove` 都会改变 `rs.config().members` 数组的下标。
- 调整 `priority` 时务必依据最新 `rs.config()`，不要凭记忆使用 `members[0]`。

### 4. 数据全量同步说明
- 新节点初始同步：从 PRIMARY 拉取所有数据库 → 重放 oplog 直到追平 → 进入 SECONDARY。
- 数据量大、网络紧张时，初始同步会很久；可以选择物理拷贝 dbPath（关停后），缩短首次同步时间。

### 5. oplog 大小
- 副本集复制依赖 PRIMARY 的 `local.oplog.rs`（capped collection）。
- 写入压力大时建议适当调高 oplogSize，避免新节点同步过程中 oplog 被回卷导致同步失败：
  ```yaml
  replication:
    oplogSizeMB: 10240
  ```

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "不可移除 primary" | 实际上是**不能直接移除当前主**，但可以先 `rs.stepDown()` 再 `rs.remove()` | 流程：`rs.stepDown(60)` → 等切主完成 → 在新主上 `rs.remove(...)` |
| 2 | "rs.add 的优先权重默认为 1" | 正确，但漏写默认 votes=1、healthy 之后才参与选举 | 默认 `priority=1, votes=1`，状态从 `STARTUP2 → SECONDARY` |
| 3 | 未介绍仲裁节点 | 缺关键场景 | `rs.addArb('h:port')` 可加仲裁节点，仅投票不存数据 |
| 4 | 没提初始同步与 oplog 关系 | 容易踩坑 | 初始同步过程中 oplog 必须覆盖整个克隆窗口，否则同步会失败 |
| 5 | "副本集经过添加删除后顺序会乱" | 表述对，但未给"应对方法" | 调整 priority 等参数前先 `cfg = rs.config()`，按 `cfg.members` 真实顺序操作 |

## 三、加节点与初始同步流程（ASCII）

```
  rs.add('h:27020') 在 PRIMARY 执行
            │
            ▼
   ┌──────────────────────────────┐
   │ 把新成员写入 replset 配置    │
   │ 同步给所有成员的 local 库    │
   └──────────┬───────────────────┘
              │
              ▼
   新节点从 STARTUP2 开始：
   ┌──────────────────────────────┐
   │ 阶段一：克隆数据库           │
   │  PRIMARY ── 全量数据 ─▶ NEW  │
   └──────────┬───────────────────┘
              │
              ▼
   ┌──────────────────────────────┐
   │ 阶段二：追 oplog             │
   │  PRIMARY.oplog.rs ─tail─▶ NEW│
   │  直到 lag ≈ 0                │
   └──────────┬───────────────────┘
              │
              ▼
        新节点变 SECONDARY，参与心跳/选举
```

## 四、减节点流程（ASCII）

```
  目标：移除 192.168.237.128:27019
            │
   该节点是 PRIMARY ?
   ┌──────┴───────┐
   是              否
   │               │
   ▼               │
 rs.stepDown(60)   │
   │               │
   ▼               ▼
   等到新主上任 ──▶ 在 PRIMARY 执行
                    rs.remove("192.168.237.128:27019")
                            │
                            ▼
                   该节点从 replset 配置剔除
                   仍可作为 standalone 启动 / 关停
```
