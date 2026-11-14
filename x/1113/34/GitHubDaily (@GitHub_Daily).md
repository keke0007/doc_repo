---
author: "GitHubDaily"
handle: "@GitHub_Daily"
source: "https://x.com/GitHub_Daily/status/2120781150507631016"
date: 2026-11-12T07:30:03.000Z
type: tweet
likes: 65
reposts: 13
replies: 13
bookmarks: 77
views: 5152
---
# GitHubDaily (@GitHub_Daily)

Coding Agent 看着复杂，拆开来大多就是一个模型加几个工具的循环。那不同产品之间的差距到底出在哪？  
  
《Jev 工程学：为 coding agent 而作》这份笔记给的回答是，差距不在循环里，在循环每一轮喂给模型的东西。  
  
内容综合自 TypeSafe 创始人 Diogo Almeida 的设计笔记，这个仓库把它完整翻成了中文，保留了原结构和 7 张插图。  
  
GitHub：[github.com/yibie/jev-engi…](http://github.com/yibie/jev-engineering-zh)  
  
全文从一个问题切入：如果语言模型没有 KV cache，Coding Agent 该怎么设计？由此拆出了现有 Agent 默认继承的 6 个设计选择。  
  
模型路由为什么省不下钱、上下文压缩为什么盲目、子 Agent 为什么很少被触发，这几块都讲到了，还有一节专门算 Token 的账。  
  
仓库里另附作者的原始笔记，可以对照着看从草稿到成文的加工过程。在做 Agent 的，我觉得可以花一个晚上读完。

![Image](<GitHubDaily%20(@GitHub_Daily).assets/image-01.jpg>)