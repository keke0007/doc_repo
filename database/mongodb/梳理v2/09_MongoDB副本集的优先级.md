# 09 · MongoDB 副本集的优先级

## 一、知识点梳理

### 1. priority 的作用
- 每个成员都有 `priority`，默认值 `1`。
- **较高 priority 的成员更倾向于成为 PRIMARY**：副本集会主动让低优先级的现任主退位（step down）以满足偏好。
- `priority = 0` 表示该节点永不参选 PRIMARY，但仍可投票（除非 `votes = 0`）。

### 2. 何时使用
- 想让性能最强、网络最近、机房合规的那台节点稳定为主。
- 需要做"灾备节点"——只复制不主，可设 `priority = 0`。

### 3. 修改示例
只能在 PRIMARY 上执行：
```js
conf = rs.config()                  // 取出当前配置
conf.members[0].priority = 10        // 数组下标对应 members 顺序
conf.members[1].priority = 5
conf.members[2].priority = 2
rs.reconfig(conf)                    // 应用新配置
rs.config()                          // 校验
```
- 调整完成后，最高优先级实例若未在 PRIMARY 位置，会触发一次主备切换。
- 重启实例后，priority 设置依旧生效。

### 4. 与权重相关的字段
| 字段 | 含义 |
|------|------|
| `priority` | 选主权重，越大越优先；0 表示不参选 |
| `votes` | 投票数，0 或 1；0 表示有数据但不投票 |
| `hidden` | 是否对客户端隐藏（通常配合 priority=0 使用） |
| `slaveDelay`/`secondaryDelaySecs` | 延迟同步秒数，做误删恢复 |
| `tags` | 自定义标签，用于 `readPreference` / `writeConcern` |

### 5. 注意事项
- 数组下标与初始化时 `members` 的顺序一致；**经过 `rs.add` / `rs.remove` 后顺序可能变化**，务必先 `rs.config()` 看清。
- `rs.reconfig()` 期间会有短暂选举窗口，对在线业务有抖动。
- 修改 `votes` 需要谨慎，过多投票成员（>7）会导致选举超时。

## 二、原文勘误

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | "复本集" | 拼写不一 | 副本集 |
| 2 | "权重一样的无法控制谁为主" | 不严谨 | 权重相同时，按 oplog 最新、心跳延迟、_id 顺序综合裁定，仍是确定的算法，并不是"无法控制" |
| 3 | "只有 primary 可以更改权重配置" | 表述简化 | 实际上 `rs.reconfig()` 必须在 PRIMARY 执行；特殊情况下用 `{force:true}` 可在没有主时强制下发，但有数据回滚风险 |
| 4 | "索引号从 0 开始，每次递增 1，类似数组" | 不准确 | members 数组的索引号 `members[i]` 与 `_id` 字段不一定相等；rs.add/remove 后**索引顺序与 _id 完全可能不同**，应据 `rs.config()` 实际位置修改 |
| 5 | 未提及 priority=0 与 hidden | 知识点缺失 | 灾备 / 数据备份节点应使用 `priority:0, hidden:true`，避免被选为主或被客户端读到 |

## 三、调整 priority 触发的事件链（ASCII）

```
  PRIMARY 上执行 rs.reconfig(conf)
            │
            ▼
   ┌───────────────────────┐
   │ 写入新的 replica set  │
   │   配置到 local.system │
   │   .replset            │
   └────────┬──────────────┘
            │ 同步给所有成员
            ▼
   ┌───────────────────────┐
   │ 各节点比较 priority   │
   └────────┬──────────────┘
            │ 当前主非最高 priority
            ▼
   ┌───────────────────────┐
   │ 现任 PRIMARY step down│
   │      (可写状态结束)   │
   └────────┬──────────────┘
            │ 触发选举
            ▼
   ┌───────────────────────┐
   │ priority 最高且数据   │
   │ 足够新的成员胜出      │
   └────────┬──────────────┘
            ▼
        新 PRIMARY 上任
        客户端自动重定向

   关闭并重新启动后，priority 配置依然存在
   下次故障切换仍按权重选主
```
