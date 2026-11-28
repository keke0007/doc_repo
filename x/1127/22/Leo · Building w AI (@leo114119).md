---
author: "Leo · Building w/ AI"
handle: "@leo114119"
source: "https://x.com/leo114119/status/2125950214788047213"
date: 2026-11-26T13:50:04.000Z
type: tweet
likes: 255
reposts: 31
replies: 26
bookmarks: 249
views: 11971
---
# Leo · Building w/ AI (@leo114119)

今天 HF 热门榜上英伟达发了一篇非常值得做 Agent 的人细读的论文《Letting Models Manage Their Own Context》，直接挑战了现在所有 Coding Agent 的默认做法。  
目前几乎所有 Harness（Claude Code、Cursor、Antigravity）管的都是「框架驱动压缩」——等窗口快满了，外部脚本一刀切把早期历史总结成摘要。毛病很明显：框架根本不知道哪段旧报错或文件结构后面还要用，经常一刀把关键线索切废，或者切完导致整段前缀 KV Cache 全部失效。  
  
英伟达在开源框架 nooa 里试了「模型自管上下文」，直接给 LLM 暴露一个 Context API，让模型自己决定什么时候删、删哪段。结果在 32K 紧张预算下，GPT-6.1 在 OfficeQA Pro v2 上的准确率直接从 21% 飙到 56%，Kimi 用少 41% 的输入 token 反而考更高；更关键的是推理引擎层面可以复用编辑点之后未变动后缀的 KV Cache，把动态编辑上下文的 prefill 惩罚压下来。接下来 Agent Harness 的演进，大概率要从「外部硬截断」转向「把内存管理权下放给模型自己」。  
  
论文链接：

> ![Link card preview](<Leo%20·%20Building%20w%20AI%20(@leo114119).assets/image-01.jpg>)
> 
> 🔗 [**Letting Models Manage Their Own Context: Where We Stand**](https://t.co/ccxkzliPgS)
> 
> Agent harnesses today manage context on the model's behalf: when the window fills, the harness triggers a compaction that summarizes the history. We study the alternative, \emph{model-driven}...
> 
> _From arxiv.org_