---
author: "头雁"
handle: "@alacheng"
source: "https://x.com/alacheng/status/2122247047881400738"
date: 2026-11-16T08:35:00.000Z
type: tweet
likes: 411
reposts: 57
replies: 19
bookmarks: 327
views: 15578
---
# 头雁 (@alacheng)

就在大家都还在讨论黄仁勋事件时，英伟达技术团队悄悄的开源了Molt，一个面向agentic RL（智能体强化学习）研究的框架，定位是Hugging Face原生、极简、可扩展，代码量仅仅9.2K 行。  
  
运行时是完全异步的：rollout、训练、权重同步可重叠，慢的 episode 只影响自己。采用 token-first 合约（token ids、log-probs、action ranges、rewards、多模态张量全程对齐）  
  
支持多种算法：REINFORCE、GRPO、Dr.GRPO、RLOO、GAE（PPO）、FlashREINFORCE、On-policy distillation 等  
  
-Agentic-first：Agent 就是程序本身。奖励完全由你写的 Python 决定（Env 或 ChatAgent），支持打分器、多轮工具调用、VLM环境、LLM-as-judge等。  
  
-Hugging Face-native + PyTorch端到端：模型保持普通的 PyTorch 模块（HF布局），使用 NeMo AutoModel + FSDP2 训练。同一份 checkpoint 同时给 vLLM 和训练器用，无需权重转换或桥接。  
  
-极简 + 可扩展：单 Actor（可选 KL reference）设计，整个 RL 图很清晰。同一套入口、算法和 Agent 可从 4B 扩展到 1T 级 MoE（如 GLM-5.2 753B，--fsdp.ep_size 256），支持 TP / EP / CP 和 Adam CPU offload。  
  
[github.com/NVIDIA-NeMo/la…](https://github.com/NVIDIA-NeMo/labs-molt)  
  
[nvidia-nemo.github.io/labs-molt/](https://nvidia-nemo.github.io/labs-molt/)  
  
文档：[nvidia-nemo.github.io/labs-molt/guid…](https://nvidia-nemo.github.io/labs-molt/guide/)  
  
论文：Molt: A Scalable PyTorch-Native Training Framework for Agentic Reinforcement Learning（arXiv:2607.21653）

> ![Link card preview](<头雁%20(@alacheng).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - NVIDIA-NeMo/labs-molt: A scalable, agentic-first, and HuggingFace-native RL framework for...**](https://t.co/l6FOFAHXG4)
> 
> _From github.com_