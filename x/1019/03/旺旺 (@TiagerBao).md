---
author: "旺旺"
handle: "@TiagerBao"
source: "https://x.com/TiagerBao/status/2112064032345604189"
date: 2026-10-19T06:11:20.000Z
type: tweet
likes: 6
replies: 5
bookmarks: 5
views: 557
---
# 旺旺 (@TiagerBao)

Google Research 在试一件很直接的事：模型权重不动，只让 Agent 自己反复改 Prompt、工具、Memory 和 Skills。  
  
项目叫 Harness²。  
  
它把 Agent 外面的 Harness 拆成 8 个可优化部分：  
  
System Prompt  
Guardrails  
Memory  
Tool Guidance  
Subagents  
Skills  
Scripts  
Plugins  
  
核心流程是：  
  
Propose → Probe → Compose  
  
先提出几套 Harness 修改方案，再跑任务看行为差异，最后组合出更好的版本。  
  
而且支持两种模式：并行探索多个方案，或者连续多轮迭代。  
  
论文结果里，单轮优化最高带来：  
  
WorkBuddy-Bench：+11.5 分  
JobBench：+4.5 分  
LAB：+11.8 个百分点  
  
现在支持 OpenCode、Codex，以及 JobBench、LAB、WorkBuddy-Bench。  
  
这个方向挺值得关注：以后 Agent 的提升，可能不只是换更强模型，Harness 本身也会变成一个持续优化对象。  
  
GitHub：[github.com/google-researc…](https://github.com/google-research/harness2)  
  
#AI #Agent #Harness #Codex #OpenSource

> ![Link card preview](<旺旺%20(@TiagerBao).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - google-research/harness2**](https://t.co/xJuYATbBov)
> 
> _From github.com_