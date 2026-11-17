---
author: "VerySmallWoods"
handle: "@verysmallwoods"
source: "https://x.com/verysmallwoods/status/2122008167517135247"
date: 2026-11-14T15:47:49.000Z
type: tweet
likes: 157
reposts: 19
replies: 3
bookmarks: 172
views: 7872
---
# VerySmallWoods (@verysmallwoods)

结合这篇文章，整理了一套用 Pi Durable + Pi AI SDK 组装 always-on Agent 的架构。  
  
Agent 平时等待，收到消息、Webhook 或定时事件后唤醒，推进任务、保存进度，再决定何时继续，以及是否通知用户。  
  
各个组件可以这样组合：  
- Conversation 保存讨论，Submission 接收输入，用稳定 requestId 关联重复请求。  
- Harness 调度 GenerationTask 和 ToolTask，完成模型生成、工具调用与后续执行。  
- Durable Storage 保存历史、检查点和结果，进程重启后重新打开存储并恢复。  
- Pi AI SDK 统一接入模型服务。  
  
应用层负责事件收件箱、长期任务、记忆、审批和通知；定时唤醒可结合持久任务实现，周期规则由应用管理。外部操作还需要幂等或结果核对。  
  
配图分三层：全局架构、主动调度循环、可恢复执行核心。  
  
或许我会尝试在 ChatOllama 中实现一下。

![Image](<VerySmallWoods%20(@verysmallwoods).assets/image-01.jpg>)
![Image](<VerySmallWoods%20(@verysmallwoods).assets/image-02.jpg>)
![Image](<VerySmallWoods%20(@verysmallwoods).assets/image-03.jpg>)

> **九原客 (@9hills)**
> 
> Claude 这个 Building Always on Agents，实际是 Personal Agent的最基本的设计模式。  
>   
> 之前在 raft build 上就看到类似的设计。  
>   
> https://claude.com/resources/articles/building-always-on-agents…
> 
> ![Image](<VerySmallWoods%20(@verysmallwoods).assets/image-04.jpg>)
> ![Image](<VerySmallWoods%20(@verysmallwoods).assets/image-05.png>)