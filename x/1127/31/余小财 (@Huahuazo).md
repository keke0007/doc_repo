---
author: "余小财"
handle: "@Huahuazo"
source: "https://x.com/Huahuazo/status/2125920251481358718"
date: 2026-11-26T11:51:00.000Z
type: tweet
likes: 88
reposts: 14
replies: 8
bookmarks: 85
views: 4543
---
# 余小财 (@Huahuazo)

“4块钱两小时训出64M模型”，起初当营销吹，翻完MiniMind trainer目录才信。  
  
预训练、SFT、LoRA、DPO、PPO/GRPO、Tool Use、Agentic RL 全躺在里面，还都是原生 PyTorch 手写的，trl 或 peft 那层壳压根没套。  
  
结构是对着 Qwen3 来的。我在 3090 上粗粗估过，用 mini 数据组合跑完 1 epoch 差不多两个多小时，租卡成本确实压在个位数。回头看两年前的 nanoGPT，更像教学 demo；市面上多数教程又只停在推理和 LoRA 微调那一步。  
  
后训练整条链路能摊开讲的仓库，MiniMind 算少数几个。想弄明白大模型到底怎么练出来的，从这儿下手比刷视频课实在。  
🔗：[github.com/jingyaogong/mi…](https://github.com/jingyaogong/minimind)

![Image](<余小财%20(@Huahuazo).assets/image-01.png>)