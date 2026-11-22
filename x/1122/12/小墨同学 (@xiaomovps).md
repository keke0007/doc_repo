---
author: "小墨同学"
handle: "@xiaomovps"
source: "https://x.com/xiaomovps/status/2123807628161916952"
date: 2026-10-01T04:33:42.000Z
type: tweet
likes: 288
reposts: 50
replies: 19
bookmarks: 297
views: 22652
---
# 小墨同学 (@xiaomovps)

研究了这么多 AI Agent，我越来越觉得，Harness 才是最值得深入研究的方向  
  
最近看到 Anthropic 分享的一篇关于长任务 Agent 的文章，里面的一些思路让我挺有感触的  
  
因为我自己不管是研究和使用都折腾过不少Harness，包括 Pi Agent、DeepSeek Harness、Claude Code、Codex，还有 Hermes Agent  
  
在这个过程中发现每一个Harness其实本质都很像，但是表现出来的结果却各不相同，有可能同一个模型在不同的Harness上表现就不一样  
  
我总结下来，一般在这几个问题上  
  
1、首先是上下文管理  
  
因为 Agent 在执行复杂任务的时候，会不断积累上下文，如何保留重要信息、压缩无用内容，以及在上下文切换之后继续理解之前的任务，其实很考验 Harness 的设计  
  
2、其次是长任务的执行能力  
  
这方面我自己踩过不少坑，有时候一个任务运行几十分钟突然卡死，或者中途出现问题只能重新开始  
  
所以我觉得一个成熟的 Harness，不应该只是能够执行任务，还需要考虑任务状态、异常恢复，以及怎么从中断的位置继续执行  
  
3、最后是结果验证  
  
这也是我觉得很容易被忽略的地方，很多时候 Agent 告诉你任务已经完成了，但实际结果可能并不符合预期  
  
如果缺少可靠的验证机制，模型再强，也很难保证复杂任务的最终质量  
  
我希望市面上的Harness越来越好，尤其是国产的Harness 不管是DeepSeek Harness还是WorkBuddy，只有强力模型搭配强力Harness才是唯一出入  
  
如果你也在研究这个方向，推荐看看：  
  
[anthropic.com/engineering/ef…](https://anthropic.com/engineering/effective-harnesses-for-long-running-agents)

![Image](<小墨同学%20(@xiaomovps).assets/image-01.jpg>)
![Image](<小墨同学%20(@xiaomovps).assets/image-02.jpg>)

> **小墨同学 (@xiaomovps)**
> 
> ![Article cover](<小墨同学%20(@xiaomovps).assets/image-03.jpg>)
> 
> 📝 **从零开始理解 Agent Harness（What is a Harness）【译】**
> 
> 本文译自 Earendil Product 的 What is a Harness?，原文发表于 2026 年 8 月 20 日，由小墨同学翻译整理。下文的“我”和“我们”沿用原作者视角，封面、章节要点、解释示意图和文末对照表为整理时补充。...