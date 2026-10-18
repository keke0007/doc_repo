---
author: "老金"
handle: "@freeman1266"
source: "https://x.com/freeman1266/status/2059584416348037402"
date: 2026-05-27T10:36:04.000Z
type: tweet
likes: 49
reposts: 5
replies: 35
bookmarks: 45
views: 3373
---
# 老金 (@freeman1266)

RAG 系统效果差，90% 的锅在文本分割器。  
  
大多数人搭 RAG 时，随便选个分割器、设个 chunkSize 就跑了。结果检索出来的内容语义断裂、上下文丢失，最终答案一塌糊涂——不是模型不行，是数据就没切好。  
  
RecursiveCharacterTextSplitter 为什么是默认标配？  
  
核心在三个参数：  
  
separators（分隔符优先级）：不是一刀切，而是递归降级。先按换行切，切不干净再按句号，再不行按逗号，最后才硬截断。每一刀都尽量落在自然停顿点，而不是把一句话腰斩。  
  
chunkSize（目标上限）：不是精确等于这个长度，是"尽量不超过"。优先保证语义完整，长度其次。  
  
chunkOverlap（重叠区）：相邻 chunk 共享一小段内容（一般是 chunkSize 的 10%）。用户的问题如果恰好跨在两个 chunk 的交界处，overlap 能让关键语义同时出现在两个块里，大幅降低检索漏掉的概率。  
  
执行顺序：先按分隔符切 → 再收敛到 chunkSize → 最后用 overlap 缝合上下文。  
  
还有一个容易踩的坑：chunkSize 计的是字符数，不是 token 数。200 个中文字符可能对应 300+ token，生产环境里要用 tiktoken 实际测量，否则账单会给你惊喜。  
  
RAG 的质量从分割开始。搞清楚 RecursiveCharacterTextSplitter 的原理，才能调对参数。

![Image](<老金%20(@freeman1266).assets/image-01.jpg>)