---
author: "纳米AI"
handle: "@antiAIvo"
source: "https://x.com/antiAIvo/status/2114222781479133673"
date: 2026-10-25T05:09:26.000Z
type: tweet
likes: 55
reposts: 4
replies: 13
bookmarks: 34
views: 2140
---
# 纳米AI (@antiAIvo)

用claude做了一个新栏目：Agent入门实践  
[nano-ai.tech/agent/](https://nano-ai.tech/agent/)  
  
很多人觉得 Agent 很复杂  
其实拆开看，它就是一个 while 循环：  
调模型，执行工具，把结果写回上下文，再调模型  
难懂的地方在于  
你看不到上下文在循环里是怎么变的  
这个栏目就是把它画出来，一格一格播给你看  
怎么用：  
  
左边栏选章节，一共 10 章  
从一次 LLM 调用讲到一个完整的 coding agent  
每章中间有一个播放器  
点「播放」自动走，点「下一格」手动走  
  
播放器左边是代码，黄色高亮的是当前执行到的那一行  
右边是此刻的上下文  
每种角色一个颜色：system、user、assistant、工具结果  
刚加进来的那一块会亮黄  
外框变成黄色，表示这一刻整个上下文正在发给模型  
  
播放器下面是 token 计量条，每调用一次模型就多一条。  
浅色是重发的历史，深色是这一圈新加的  
看完你就明白，为什么 Agent 跑久了又贵又慢  
  
10 章分别是：  
一次调用、工具调用、Agent Loop、上下文窗口  
规划、记忆、MCP 与 Skills、子 Agent、失败模式、实战拆解  
建议先看第 3 章 Agent Loop  
看懂这十几行代码，后面每一章都是在它上面加东西