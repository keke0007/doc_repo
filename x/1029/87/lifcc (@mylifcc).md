---
author: "lifcc"
handle: "@mylifcc"
source: "https://x.com/mylifcc/status/2114296157501260123"
date: 2026-10-25T10:01:00.000Z
type: thread
likes: 132
reposts: 14
replies: 6
bookmarks: 108
views: 6610
---
# lifcc (@mylifcc)

给 PostgreSQL 做实时同步，终于不用在「装第三方插件污染源库」和「搭一套吃掉几 GB 内存的 Debezium 集群」之间二选一了。  
PGRun 团队刚用 Rust 开源了 pgsync——一个把内存控制到极致的单二进制 PostgreSQL 实时逻辑复制（CDC）引擎。实测将 1GB 数据库做全量一致性快照时，峰值内存（RSS）仅 33MB；日常增量流式同步稳态仅 15MB，单机跑 20 个并发任务每进程才 9.5MB。  
它最狠的不是快，而是死磕「绝对正确性（Correct before fast）」的三个架构设计：  
1️⃣ 零侵入全量 + 增量无缝切换：基于 PG 原生 pgoutput 协议（源库无需装 wal2json 等任何扩展），先通过导出快照（Exported Snapshot）完成一致性全量拷贝，再精准从对应 LSN 位点切入实时 CDC 流。  
2️⃣ 1:1 事务边界与原子 Checkpoint：源库每个事务在目标库严格作为唯一事务按序提交，且同步位点（pgsync.job）直接包在目标库同一个事务里落盘。哪怕遭遇 15 轮连续随机 SIGKILL 强杀，重启后依靠 LSN 幂等去重，数据 Hash 依然 100% 严丝合缝。  
3️⃣ 22 项前置体检与零静默丢数：启动前一句 pgsync check 自动排查 22 项权限与配置隐患；同步中途一旦遇到目标库缺列或异常，立即显式停机报错，绝不静默丢弃一行数据。  
你们目前在生产或测试环境做 PG 实时同步，主要用哪套方案？踩过最深的一个坑是什么？

![Image](<lifcc%20(@mylifcc).assets/image-01.jpg>)

---

开源仓库与 90 秒本地 Docker 体验指南：

> ![Link card preview](<lifcc%20(@mylifcc).assets/image-02.jpg>)
> 
> 🔗 [**GitHub - pgrundev/pgsync: PGSync — Real-time PostgreSQL replication**](https://t.co/VRx9s9psfi)
> 
> _From github.com_