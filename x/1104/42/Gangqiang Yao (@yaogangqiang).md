---
author: "Gangqiang Yao"
handle: "@yaogangqiang"
source: "https://x.com/yaogangqiang/status/2117855803369787773"
date: 2026-10-27T03:01:34.000Z
type: tweet
likes: 136
reposts: 12
replies: 10
bookmarks: 174
views: 9539
---
# Gangqiang Yao (@yaogangqiang)

往 AGENTS.md 里塞满最佳实践和开发原则，效果其实非常有限，规则一多就会在上下文里被稀释。  
  
真正有效的工程化方案，是把「静态文档约束」变成至少 3 个周期巡检的后台 Agent：  
  
架构与重构 Agent：定时扫描状态，接口设计与测试覆盖，持续提交小步重构 PR  
  
CI Agent：监控构建稳定性与耗时变化，抓 Flaky 测试、压构建时长  
  
安全审计 Agent：持续扫描依赖漏洞与代码高危逻辑，发补丁  
  
别指望一个写功能的 Agent 同时做好所有事，用职责单一的巡检 Agent 做持续治理才是正解。

> **Gangqiang Yao (@yaogangqiang)**
> 
> 最后一条了，在 Vibe coding 时代人还应该 review 什么？应该 review 的是那些一旦被设计错，就修正起来成本高的事情。  
>   
> 哪些设计一旦出错，成本就会很高呢？我认为最核心的是「状态」。 x.com/yaogangqiang/s…