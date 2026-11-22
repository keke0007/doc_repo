---
author: "Ruiteng Huang"
handle: "@huangruiteng"
source: "https://x.com/huangruiteng/status/2121837181249704445"
date: 2026-11-15T04:49:16.000Z
type: tweet
likes: 194
reposts: 33
replies: 17
bookmarks: 311
views: 17665
---
# Ruiteng Huang (@huangruiteng)

Anthropic 于 11 月教大家如何从零搭自己的 7*24 Agent，LoopX 自 6 月起，已帮用户和开发者搭好了一切通用组件，可开箱即用：五大原语均有能力对应，经过企业级采用和学术级 benchmark 验证打磨。  
  
1. Event Inbox（事件收件箱）→ Source State / Projection 分离 + CAS Claim & Lease Epoch 原子事务  
  
2. Timers（计时器与主动跟进）→ Idle-Aware Guarded Heartbeat + Quota 防空转机制  
  
3. Memory & Compaction（跨运行记忆）→ Context Delivery 渐进查书 + Compaction 周期无损 Delta  
  
4. Permissions & Deferred Calls（权限隔离与延后调用）→ 可继承 Decision Scope / Gate + 默认拒绝四级 ACL  
  
5. Outbox（低噪发件箱）→ Evidence-Backed Acceptance + 跨周期 Reward Memory

![Image](<Ruiteng%20Huang%20(@huangruiteng).assets/image-01.jpg>)

> **Ruiteng Huang (@huangruiteng)**
> 
> ![Article cover](<Ruiteng%20Huang%20(@huangruiteng).assets/image-02.jpg>)
> 
> 📝 **搭 7×24 在线 Agent 不用从零手搓：从 Anthropic 五大原语到 LoopX 语义控制面**
> 
> 1. 当前判断
> Anthropic（William Hu）在《Building always-on agents》里给出了一个非常关键的工程判断：“7×24 常驻在线（always-on）”是一种用户体验，而不是让模型进程一直挂着空烧 token。真正可落地的常驻...