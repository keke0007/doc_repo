---
author: "Ruiteng Huang"
handle: "@huangruiteng"
source: "https://x.com/huangruiteng/status/2121963877600784573"
date: 2026-11-15T13:49:47.000Z
type: thread
likes: 39
reposts: 5
replies: 4
bookmarks: 27
views: 3298
---
# Ruiteng Huang (@huangruiteng)

长程 Agent 连续跑几十轮时，上下文交付有一对硬矛盾：每轮全量重塞，重复状态会迅速稀释注意力；只传一段有损摘要，又容易把关键验收约束和历史负证据截丢。  
  
我们在 LoopX 里做了一套确定性的 Context Delivery 机制，在不删改一条原约束的前提下，分三步逐层减负（从 100% 完整包起算）：  
  
1. 按需选读（Progressive）→ 先压到 32.5%  
完整控制包落盘存证并锁死 SHA-256，默认只展开当前决策必需的视图，需要细节再按指针回读原文（八题 213 次唤醒 22.41 → 7.29 MB，先砍掉 67.5% 无关长尾）。  
  
2. 周期内无损差分（Epoch Delta）→ 再压到原包 ~20% 量级  
只要还在同一个未压缩的上下文周期内，续跑只传相对上一轮的结构增删改：在按需包基础上，全周期 46 轮平均再砍 38.5%（3.09 → 1.90 MB），其中 24 个纯增量轮再砍 74.3%（压进原包 10% 以内）。一旦宿主发生上下文压缩（Compaction）、重启或基线哈希失配，立刻原子切回完整包重建，绝不悬空叠加旧差分。  
  
3. 首屏目录入口（--initial-read）→ 首读正文压至 3.0%  
续跑首屏先读变更目录与指针索引（含 CLI 封装 5.0%）；算上后续按指针补读，ANN 18h 整场未缓存输入 Token 净省 12.1%。  
  
固定同一套 Explore + Stride 在 EdgeBench 十题只切 Context ON vs OFF，10 题里 7 题占先：等时长 ~18h 下 Borden 45 → 80（+77.8%）、Grid 代价 2,311 → 2,197（−114）、Rust 微先；历史同构下 ANN 3,746 → 7,159 QPS（+91.1%）、FFmpeg +35.9%、VLIW −55 周期；同时原样公开 Portfolio / Codec / OpenTTD 3 个 OFF 占优的反例。  
  
不靠全量硬塞，也不靠摘要碰运气，靠确定性合同守住长程信噪比。完整设计与十题数据见评论区：

![Image](<Ruiteng%20Huang%20(@huangruiteng).assets/image-01.jpg>)
![Image](<Ruiteng%20Huang%20(@huangruiteng).assets/image-02.jpg>)
![Image](<Ruiteng%20Huang%20(@huangruiteng).assets/image-03.jpg>)

---

- 中文技术博客（双合同细节 + 分层字节与 Token 账本 + 十题全表）：  
[loopx-project.github.io/loopx/blog/zh/…](https://loopx-project.github.io/loopx/blog/zh/context-delivery/)  
  
- Context Delivery 使用指南：  
[github.com/loopx-project/…](https://github.com/loopx-project/loopx/blob/main/docs/guides/context-delivery.zh-CN.md)  
  
- EdgeBench 十题控制变量对照 PR：  
[github.com/loopx-project/…](https://github.com/loopx-project/loopx/pull/8760)