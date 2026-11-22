---
author: "苏茶"
handle: "@Suercha007"
source: "https://x.com/Suercha007/status/2122161964365541792"
date: 2026-11-16T02:56:54.000Z
type: tweet
likes: 34
reposts: 1
replies: 11
bookmarks: 40
views: 10482
---
# 苏茶 (@Suercha007)

前段时间 RAGFlow 1.0 ，把 Python 底座整个用 Go 重写了。  
  
我一直有个疑问，既然 Rust这么火，他们为什么选 Go 不选 Rust。  
  
今天官方发了个文章解释了原因：Rust 留给更底层的 Infra，Go 留给编排层。  
  
因为 RAGFlow 的核心是任务编排，编排 ingestion 流水线、编排 Agent Harness、任务调度和状态管理。这些东西不需要极致的内存占用和零运行时开销，那是底下检索引擎 Infinity 才该考虑的。而 goroutine 和 channel 天然契合 Pipeline 的并发编排。  
  
还有个很现实的原因。Go 编译快，所以他们能一边继续迭代 Python 版，一边重写底座。用他们自己的话说，这叫给奔跑的汽车换轮子。  
  
他们坚决不做断点续跑，进程被杀、发布重启，任务直接退回队列从头重投。听着很浪费，但他们的分段缓存标识是由内容、模型和关键配置一起决定的，重跑全部命中缓存，实际算过的东西一次都不重复算。  
  
等于拿重跑很便宜，换掉了维护复杂断点状态的成本。  
  
原文：[mp.weixin.qq.com/s/UOpDkAU0dJNu…](https://mp.weixin.qq.com/s/UOpDkAU0dJNuH7K-8X-JWw)