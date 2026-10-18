---
author: "Jerry Fang"
handle: "@Jerry_fsy"
source: "https://x.com/Jerry_fsy/status/2109622065443835940"
date: 2026-10-12T12:27:49.000Z
type: tweet
likes: 311
reposts: 51
replies: 29
bookmarks: 446
views: 22772
---
# Jerry Fang (@Jerry_fsy)

制定了一个Agent Infra学习路径：  
  
我参考了一些资料，然后为自己制定了一个Agent Infra学习计划（预计需要6个月左右的学习时间，大家可以根据自己的节奏来，重点是掌握各个部分而不是追求速度）。  
  
将之前列出来的Agent Infra学习清单总结成了6大主要部分，先主攻这6个，其他的遇到再深入学习。目前主要制定了前2个部分的具体学习计划，感兴趣的可以一起学起来：  
  
1.Agent Runtime基础  
不使用现有的Agent框架，从0实现一个Agent Loop，理解Agent的执行过程。  
这个我会跟着“三元同学”的“Super Agent实战课”从头做一遍，这一遍我会从代码的细节去理解Agent的每一步执行。考虑到代码的版权，这一阶段我只会分享每一步从宏观的角度应该怎么做，感兴趣的可以在AI的辅助下从0搭建一个属于自己的Agent。  
需要了解的知识点：  
Agent Loop；Tool Calling；Context Management；Streaming；Agent State；MCP；Multi-Agent。  
  
2.分布式系统与可靠性  
通过学习“任务队列”，“通信”，“状态”和“可靠性”，实现一个可恢复的任务执行系统。需要做到重启进程后，未完成的任务仍然可以继续。  
对于这一部分的入门，DDIA这本书比较困难。目前先考虑使用System Design 101这个开源学习资料，并辅以MIT 6.1800补一下计算机系统相关的知识，以及通过MIT 6.5840补充分布式方面的知识（这两门MIT的课程会做选择性学习）。  
需要了解的知识点：  
Process/Thread/Async；RPC/HTTP/gRPC；Queue/PubSub；State Machine；Timeout/Retry；幂等；Transaction/WAL等。  
  
3.Agent Orchestration  
4.Sandbox与隔离执行  
5.Observability 和 Evaluation  
6.Production Agent Platform