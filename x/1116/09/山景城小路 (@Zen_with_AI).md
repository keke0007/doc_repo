---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2121957640775647548"
date: 2026-11-15T13:25:00.000Z
type: tweet
likes: 34
reposts: 5
replies: 4
bookmarks: 22
views: 2068
---
# 山景城小路 (@Zen_with_AI)

📐 ML 铅笔手稿 第 26期 📷✏️  
  
上期的问题：剪枝是把吃白饭的权重剪掉，让模型瘦下来。但推理时还有个大头开销不在权重上——每生成一个 token，都要把前面所有 token 的 K、V 重新算一遍吗？这就是 KV Cache 要解决的事。  
  
第一幕：KV Cache 是什么。自回归解码是逐个 token 往外吐：每出一个新 token，attention 都要看前面所有 token。如果每次都从头算，算量随长度平方增长，越生成越慢。KV Cache 的想法很朴素：K 和 V 一旦算出来就不变，存下来复用。每步只算新 token 的 Q、K、V，再和缓存里的 K、V 做 attention：  
  
Attention(Q_t, K_≤t, V_≤t) = softmax(Q_t·K_≤tᵀ / √d)·V_≤t  
  
新 token 只算自己的那一份，旧的全部查缓存——算量从 O(n²) 降到每步 O(n)，推理速度直接起飞。  
  
第二幕：显存账。缓存不是免费的，每存一个 token 的 K、V 都要占显存：  
  
每 token KV 显存 = 2·L·h·d·bytes  
  
2 是 K 和 V 两份，L 是层数，h 是头数，d 是头维度。算个例子，Llama-2-7B：32 层、32 头、头维 128、FP16（2 字节）：  
  
2×32×32×128×2 = 524,288 字节 = 512KB / token  
  
4k 上下文的一条序列，KV 就要 4096×512KB ≈ 2GB。batch 一大，显存先被 KV 吃光，权重反而是小头。更糟的是碎片化：朴素实现按最大长度预分配连续显存，每条序列预留 4k 位置，实际只用到几百个 token——大半显存空着；不同长度的序列混在一起，空闲块拼不成整块，GPU 明明有空显存，却装不下新请求。  
  
第三幕：PagedAttention（vLLM，OSDI 2023）。借操作系统的虚拟内存思想：KV 不再按序列连续存，切成固定大小的块（比如一块 16 个 token），用一张块表（block table）做映射：逻辑块 → 物理块。按需分配，用多少拿多少；不同序列可以共享同一块物理块——prompt 前缀一样就不用存两份；用完的块回收复用。效果：显存碎片几乎清零，同样显存能多跑几倍的 batch，吞吐量跟着涨。  
  
工程上的提醒：KV Cache 是推理加速的标配，几乎所有推理引擎都有；长上下文时代 KV 显存比权重还贵，8k、32k、128k 上下文先算 KV 账再谈 batch；KV 量化（INT8 / INT4）和 PagedAttention 常一起上；前缀缓存（prefix caching）是 PagedAttention 的直接红利，系统提示词只存一份。  
  
一句话：KV Cache 是“算过的别重算”，PagedAttention 是“存的时候别浪费”——把显存当分页内存管，推理才能又快又省。

![Image](<山景城小路%20(@Zen_with_AI).assets/image-01.jpg>)