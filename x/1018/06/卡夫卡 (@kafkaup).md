---
author: "卡夫卡"
handle: "@kafkaup"
source: "https://x.com/kafkaup/status/2111558117003612350"
date: 2026-10-17T20:41:00.000Z
type: tweet
likes: 12
reposts: 4
replies: 3
bookmarks: 11
views: 841
---
# 卡夫卡 (@kafkaup)

航空和国防业有一套强制的受控英语标准 ASD-STE100，存在的理由很直白：一条飞机维修指令被看错，是会死人的。规则就那几条——一个词只准一个意思、用主动语态、只用简单时态、一句话只给一条指令、不许省词。  
  
asd-ste100-skill 做的事，是把这套纪律搬给另一类读者：读上游 agent 输出、读 tool description、读报错信息的模型。它的理由也说得通，下游 agent 和当年那个非母语机械师处境一样，没有回传通道，没法反问"你指的是 X 还是 Y"。  
  
以前写 tool description，很容易写成 README 里那种句子：这个工具会尝试在各个已配置的后端之间同步状态，如果检测到冲突，可能会根据已设置的策略自动处理，否则就暴露出来人工审查。一句话里塞了三个条件分支，模型按哪条走全看运气。  
  
接上 skill 之后，它会逐句标出违规项：被动语态且施事不明、一句多指令、名词堆叠过长、现在完成时、分号、营销形容词、堆叠的模糊限定词，然后拆成四个短句重写，同时要求不丢掉任何事实、条件和范围限定。如果缩短会损失精度，它保留长句并标注取舍，而不是悄悄简化。输出默认只给改写后的正文，想看依据就问 show the diff，它才列出规则对照表。  
  
两种模式：Strict 用于流程、报错、tool description；STE-flavored 用于 README 和 PR 描述，保留句式纪律但不锁词表。  
  
代价要说清楚。仓库里的 scripts/ste-lint.py 只做结构检查，不比对原文和改写、不验证语义是否保留，零告警只代表结构规则没命中。短语动词也只认 spin up、reach out 这类固定一小串，take off the panel 就能混过去。它自己的 SKILL.md 都要带 baseline 41 才跑得干净，作者直接写在 README 里了。  
  
今日 +945 star · Python · 总计 4,450  
#ClaudeCode #AIAgents