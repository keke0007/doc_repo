---
author: "洛克船长"
handle: "@slashui"
source: "https://x.com/slashui/status/2114135503750500838"
date: 2026-10-24T23:22:37.000Z
type: tweet
likes: 10
replies: 13
bookmarks: 2
views: 1039
---
# 洛克船长 (@slashui)

Anthropic 建议企业把 AI 系统里不必要的子智能体删掉。  
  
这是智能体架构上一个相当大的转变。文章说，一个配备了技能（skills）的单一智能体，往往能胜过复杂的多智能体系统；目前的经验是：  
  
→ 技能（Skills）：教智能体怎么做某件事，包括流程、专业知识和操作说明。  
  
→ 工具 / MCP：让智能体有能力去做事情，通过 API、操作和实时数据。  
  
→ 子智能体（Subagents）：用于并行工作、需要隔离上下文的场景，或者单个智能体搞不定的任务。  
  
最有意思的是"渐进式披露"（progressive disclosure）。不用再去生成各种专门的智能体，也不用把所有内容都塞进一个巨大的提示词里，而是让一个智能体在需要的时候自己去发现并加载技能和工具。  
  
他们的建议是：把现有的子智能体并入主循环，把它们的指令改写成技能，把它们的操作开放成工具，然后跑评测（evals），看看效果能不能保持住。  
  
随着模型越来越聪明，改进智能体架构越来越不在于你能加什么，而在于你能删掉什么。

> ![Link card preview](<洛克船长%20(@slashui).assets/image-01.png>)
> 
> 🔗 [**Building agents: When to fold subagents into your main agent loop**](https://t.co/c7L7KdNUuQ)
> 
> _From claude.com_