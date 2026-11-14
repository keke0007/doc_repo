---
author: "bugyaluwang"
handle: "@bugyaluwang"
source: "https://x.com/bugyaluwang/status/2119460184263442731"
date: 2026-11-08T13:32:29.000Z
type: thread
likes: 56
reposts: 6
replies: 5
bookmarks: 47
views: 7136
---
# bugyaluwang (@bugyaluwang)

正确的：  
如果你做的 Agent 可以被 Codex 直接替代，那么就不要做了。  
  
所以在前司去做那两个 ai 项目立项的时候，我就在问自己，我要怎么去 build 这两个项目才可以让它真正有价值。  
  
1. 为什么必须需要 RAG 来做客服问答？一开始我就很清楚，现有 Agent 在文件系统+md 上的能力绝对强得多。但最终还是选择了 RAG，原因很简单，rag 的成本低很多，agentic rag 是成本是很高的。实际上，两者混合效果更佳。我预期的一条 flow 应该是：简单问题、已有问题走 rag 支撑，搜不到则 agentic 并标注出来，如果问题合适，则入库。  
2. 为什么要做一个 QA 要用的 Agent？其实我一开始是很反对的，我的 leader 并没有成功说服我🤣因为我觉得他们在 Codex 里用效果肯定更好。但后边我还是决定去做了，原因之一就是上下文共享和 qa skill 自动提炼，我的预期是把团队内的所有 QA 都通过数据蒸馏，说实话，这是一个很残酷的事情，但确实这会是一个为什么应该去做企业层级的 AI power QA agent 的原因。

> **Ayaka Atelier (@AyakaAteliers)**
> 
> 不管你做什么 Agent，做到最后都面临一个问题：  
>   
> 你做的 Agent 用 Codex 能不能替代，用Claude 能不能替代  
>   
> 如果可以，那为什么要用你的 Agent，如果不可以，能不能用多个 Codex 编排  
>   
> 本质来说互联网公司白嫖了三个多月的劳动力招聘了一批实习生把自己内部的各种系统加上了一个对话框，名字叫 AI

---

@grok 我的理解对吗