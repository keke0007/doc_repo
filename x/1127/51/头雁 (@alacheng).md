---
author: "头雁"
handle: "@alacheng"
source: "https://x.com/alacheng/status/2125537485400821860"
date: 2026-11-25T10:30:01.000Z
type: tweet
likes: 91
reposts: 23
replies: 7
bookmarks: 99
views: 6089
---
# 头雁 (@alacheng)

Hugging Face在的RL的新实践总结  
  
《Hugging Face 的强化学习新实践：让模型真正适配 Agent Harness》是Hugging Face 团队通过在目标 Agent Harness（如 pi、OpenCode、Hermes Agent）内部做强化学习，训练针对特定 harness 的 LoRA 适配器。  
  
主要结果（以 Qwen3.8-27B 为例）：  
-在对应 harness 下，性能提升明显（例如 Terminal-Bench、SWE-bench Pro 等）。  
-同时更省 token（工具调用更少、推理更高效）。  
-还提供了可复用的配方，以及把多个适配器合并成一个“超级适配器”的方法。

> ![Link card preview](<头雁%20(@alacheng).assets/image-01.jpg>)
> 
> 🔗 [**Introducing harness adapters: adapt your favorite model to your harness to improve performance and...**](https://t.co/KeCkwPZR15)
> 
> _From huggingface.co_