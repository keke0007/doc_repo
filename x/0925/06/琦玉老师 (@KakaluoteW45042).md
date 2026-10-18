---
author: "琦玉老师"
handle: "@KakaluoteW45042"
source: "https://x.com/KakaluoteW45042/status/2102935226326630723"
date: 2026-09-24T01:11:16.000Z
type: thread
---
# 琦玉老师 (@KakaluoteW45042)

给 agent 项目写测试的关键：把副作用收口到少数原语（浏览器、文件、网络都算），对原语做回归测试，比给每个分支补单测划算得多。行为变更先更新用例再动手，这条深有同感。

> **meng shao (@shao__meng)**
> 
> AI Agent 要怎么写测试，还不会过度测试、为测试而测试 ？  
>   
> @anshnanda 分享了自己写在 AGENTS.md 顶部的三条规则，值得一看！  
>   
> 1. 严禁在写完代码之后再补单元测试。 事后补写的单测（尤其是 Agent
> 
> ![Image](<琦玉老师%20(@KakaluoteW45042).assets/image-01.jpg>)

---

多 agent 值钱的地方不是并行干活，是上下文隔离。搜索、探索这类容易污染主上下文的脏活丢给子 agent，干完只交结论回来，主对话保持干净。  
  
重复加载浪费额度、隔离麻烦，是 harness 没做好共享和分层，不是多 agent 本身不行。  
  
场景想不到，多半是单上下文还没被撑爆过。

> **RedpoQ｜AI × 独立开发 (@Redpoq_ai)**
> 
> 多agent协助都是应用到什么场景?除了独立review,并行工作区.我还想不到.多agent总是加载重复的上下文,浪费额度不说,还要进行隔离,在同一个worktree工作更是重灾区