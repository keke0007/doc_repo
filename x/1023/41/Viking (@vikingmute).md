---
author: "Viking"
handle: "@vikingmute"
source: "https://x.com/vikingmute/status/2113615617958002698"
date: 2026-10-22T13:22:08.000Z
type: tweet
likes: 52
reposts: 7
replies: 11
bookmarks: 101
views: 6920
---
# Viking (@vikingmute)

谈谈我在实践中目前使用的类似软件工厂一样的流程，在 Grok Bot 中的实践，目前我在开发 [tinyship.cn](http://tinyship.cn) 中使用的这套流程，目前是半自动，就是最早的需求和最后的推送需要我确认一下：  
  
1 Ticket tracker：Bot 每日定期搜索 issue，每周一定期进行项目规划，和我头脑风暴提出新的需求  
  
2 Orchestrator： 经过我确认后开始，grok bot 作为 Orchestrator 开始编排，主bot总调度 + 各专属 bot  
  
3 Runner：Cursor CLI，按照任务类型，由 bot 决定采用什么模型，普通编码任务是 grok 4.7  
  
4 Code Review，简单任务 bot 直接 review，复杂任何 bot 会启动 codex 和 cursor 使用多 agents 交叉 review，使用我 review forge 的方式进行 review，注意这里 review 采用 A\ 的一个收敛方式：agent 写 agent 审，有可能可以互相折腾个没完，最多三轮，超过就让作者退一步重新想方案 几轮之后只审新增的 diff 已经提过的意见不再重复。  
  
5 Test：全量 E2E，三个框架，两种数据库，六轮  
  
6 失败就修 - 必要时再 review - 再测 - 以此循环 直到通过  
  
7 全绿了问你我推送 / 开 PR  
  
这样做了两个星期了，感觉很舒服，现在交付的快多了

> **Viking (@vikingmute)**
> 
> Matt 新的这个 skill 叫 software factory， 软件工厂https://github.com/mattpocock/skills/pull/1316…  
> 很值得一看了，这是他设计的一个术语，就是说现在在 AI Coding 的场景下，则是把 agent 嵌进一套流水线里，完全没有人的干预 自行完成任务  
>   
> 他设想的流程如下：  
> * 有 ticket / issue 作为输入  
> * trigger