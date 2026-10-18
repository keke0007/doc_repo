---
author: "老金"
handle: "@freeman1266"
source: "https://x.com/freeman1266/status/2055508050040242225"
date: 2026-05-16T04:38:03.000Z
type: tweet
---
# 老金 (@freeman1266)

Zero-Shot vs Few-Shot Prompting  
   
Zero-Shot 和 Few-Shot，大多数人以为只是"给不给例子"的区别。  
   
实际上这两种方式触发了 LLM 完全不同的工作模式：  
   
Zero-Shot：只下指令，不给例子。 模型靠预训练知识直接推理，适合通用任务、格式清晰的问题。  
  
Few-Shot：先给示范，再下指令。 模型从例子中提取模式，然后类比执行——本质是在 prompt 里做了一次"临时微调"。  
   
关键区别不是例子数量，是激活路径不同：  
   
Zero-Shot 激活的是模型的"通用理解"  
Few-Shot 激活的是模型的"模式匹配"  
   
什么时候用 Few-Shot？  
  
输出格式非常特殊（JSON结构、特定语气）。任务边界模糊，语言描述说不清楚。Zero-Shot 结果不稳定，需要锚定  
   
一个反直觉的发现：Few-Shot 例子质量比数量重要。3个高质量例子，比10个随意例子效果好得多。