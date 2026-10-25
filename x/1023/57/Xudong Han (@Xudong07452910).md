---
author: "Xudong Han"
handle: "@Xudong07452910"
source: "https://x.com/Xudong07452910/status/2113228875040010566"
date: 2026-10-22T11:20:00.000Z
type: tweet
---
# Xudong Han (@Xudong07452910)

一个错误如果被 Agent 总结成 Skill，下一次它可能会更自信地再犯一遍。  
  
《More Memory, Worse Agents》研究的就是这种 Error Reproduction。  
  
很多 Agent Memory 会把过去轨迹总结成规则和经验，再用于后续任务。问题在于，一旦总结本身出了偏差，错误也会跟着长期保存，并继续影响之后的决策。  
  
作者因此提出 Anti-Persistence：  
  
真实轨迹和结果可以长期保存，但抽象出来的规则和策略只在当前任务临时生成，用完就丢。下一次再根据新的上下文重新总结。  
  
实验覆盖 ALFWorld、WebShop 和 ScienceWorld。Anti-Persistence 在 9 个模型×任务组合里拿到 7 个最好结果，平均成功率 59.9%，同时 LLM 调用成本最低。  
  
长期保存，并不天然等于长期学习。我的判断是：该长期存的是事实（轨迹和结果），而不是结论（规则和策略）。结论随时可以重新算，存错了却很难发现。  
  
做 Agent Memory 或 Skill 自进化的可以先收藏。我会先检查自己的 Memory 里存的是原始轨迹还是总结出的规则；如果是规则，至少给每条记上来源轨迹，出错时能追回去。  
  
如果某类错误在接入 Memory 之后反而更频繁了，先怀疑的就该是那几条总结出来的规则。  
  
📎 arxiv: