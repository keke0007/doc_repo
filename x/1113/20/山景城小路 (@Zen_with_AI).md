---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2120948994541023602"
date: 2026-11-12T18:37:00.000Z
type: tweet
likes: 7
reposts: 1
replies: 1
bookmarks: 5
views: 1330
---
# 山景城小路 (@Zen_with_AI)

Meta：广告排序模型快赶上 LLM 的规模了，硬件却跟不上，Meta 的办法是每一单位算力都去它最值钱的地方  
  
Intelligent Delivery 是 ML 驱动的分配系统，覆盖全栈。Resource Core Model（60 亿稀疏参数加 2000 万稠密参数）在候选召回前先算每个请求值不值得加算力，离线测试里识别高价值请求的能力比上一代好约 20%。  
  
再分三层决策：哪个 region 接（分钟级跨 region 调流量）、落哪种硬件（Intel、AMD、Arm 的 CPU，NVIDIA、AMD、MTIA 各代加速器，按 workload 和 SKU 的实测效率分配）、host 内分多少（在尾延迟约束内优化）。  
  
2017 年以来累计带来约 10% 的排序性能提升。2026 上半年，agent 已贡献约 20% 的优化，自动化程度最高的机制上 agent 承担约 80% 的工程量，优化周期从数月压到数周。

> ![Link card preview](<山景城小路%20(@Zen_with_AI).assets/image-01.png>)
> 
> 🔗 [**Intelligent Delivery: Making Every Unit of Compute Serve Better Ads**](https://t.co/s5YBh2DNA8)
> 
> _From engineering.fb.com_