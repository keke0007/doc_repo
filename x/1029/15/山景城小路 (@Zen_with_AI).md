---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2115222007935885813"
date: 2026-10-27T23:20:00.000Z
type: thread
likes: 251
reposts: 32
replies: 11
bookmarks: 271
views: 11592
---
# 山景城小路 (@Zen_with_AI)

大模型后训练面试准备：17 道高频题，从 Attention 到 GRPO、FSDP  
  
(部分思路和题目是网友启发 感谢）  
如果面试 LLM Post-training / Training Infra / RL Engineer，建议按这条路线准备：  
  
Attention 架构 → SFT / RL / OPD → GRPO → KL 散度 → FSDP / ZeRO。  
  
每道题都按三个层次回答：解决什么问题、具体怎么做、有什么代价。

---

大模型后训练面试 01/17：标准 Attention 的瓶颈  
  
标准 Self-Attention 怎么工作？  
  
先把输入映射成 Query、Key、Value，计算 Q 和 K 的相似度，经 Softmax 得到权重，再对 V 加权求和。  
  
公式：  
Attention(Q,K,V) = Softmax(QKᵀ / √dₖ)V  
序列长度为 L 时，全注意力的计算量与 L²… [Show more](https://x.com/Zen_with_AI/status/2115222008271405539)

---

大模型后训练面试 02/17：MHA、MQA、GQA 有什么区别？  
  
三者的核心区别是 Query Head 与 Key/Value Head 的对应关系。  
  
MHA：每个 Query Head 都有独立的 Key 和 Value Head，表达能力强，但 KV Cache 较大。  
  
MQA：所有 Query Head 共享一组 Key 和 Value，KV Cache 最小。  
  
GQA：多个 Query Head… [Show more](https://x.com/Zen_with_AI/status/2115222008686624858)

---

大模型后训练面试 03/17：DeepSeek 的 MLA 为什么能省显存？  
  
MLA，全称 Multi-head Latent Attention，是 DeepSeek-V2 引入、DeepSeek-V3 延续使用的注意力架构。  
  
传统 MHA/GQA 会缓存每层的 Key 和 Value。MLA… [Show more](https://x.com/Zen_with_AI/status/2115222009177424257)

> ![Link card preview](<山景城小路%20(@Zen_with_AI).assets/image-01.jpg>)
> 
> 🔗 [**DeepSeek-V2: A Strong, Economical, and Efficient...**](https://t.co/0bxxVRlKhn)
> 
> We present DeepSeek-V2, a strong Mixture-of-Experts (MoE) language model characterized by economical training and efficient inference. It comprises 236B total parameters, of which 21B are...
> 
> _From arxiv.org_

---

大模型后训练面试 04/17：Qwen 为什么混合线性注意力和标准注意力？  
  
以 Qwen3-Next 和 Qwen3.5 为例，它们采用混合注意力架构，把 Gated DeltaNet 与 Gated Attention 结合起来。  
  
标准全注意力需要计算当前 Query 与历史 Key 的相关性。Gated DeltaNet 则属于线性注意力路线，利用递归状态更新和… [Show more](https://x.com/Zen_with_AI/status/2115222009659748810)

---

大模型后训练面试 05/17：DeepSeek DSA 和 Qwen QSA 有什么区别？  
  
两者都属于稀疏注意力：不让每个 Query 都完整计算与所有历史 token 的注意力，而是先筛选值得关注的内容。  
  
DeepSeek DSA 使用轻量级 Lightning Indexer 为历史 token 打分，再选择 Top-K token 参与后续注意力计算。… [Show more](https://x.com/Zen_with_AI/status/2115222010288836689)

> ![Link card preview](<山景城小路%20(@Zen_with_AI).assets/image-02.jpg>)
> 
> 🔗 [**DeepSeek-V3.2: Pushing the Frontier of Open Large Language Models**](https://t.co/F3Tie6BDHl)
> 
> We introduce DeepSeek-V3.2, a model that harmonizes high computational efficiency with superior reasoning and agent performance. The key technical breakthroughs of DeepSeek-V3.2 are as follows:...
> 
> _From arxiv.org_

---

大模型后训练面试 06/17：为什么做后训练也要懂 Attention？  
  
因为模型架构直接影响显存占用、推理吞吐量和 RL Rollout 成本。  
  
RL 训练通常需要针对同一个 Prompt 生成多条回答，再进行评估和策略更新。上下文越长、回答越长，Rollout 消耗的计算资源和缓存通常越多。  
  
MLA 可以减少 KV… [Show more](https://x.com/Zen_with_AI/status/2115222010733441256)

---

大模型后训练面试 07/17：SFT、RL、OPD 有什么区别？  
  
三者最大的区别，是学习信号不同。  
  
SFT：用人工标注或合成的标准答案做监督训练，通常使用 Token 级交叉熵，提高目标答案的生成概率。… [Show more](https://x.com/Zen_with_AI/status/2115222011152838682)

---

大模型后训练面试 08/17：GRPO 是什么？和 PPO 有什么区别？  
  
GRPO，全称 Group Relative Policy Optimization，是 DeepSeekMath 论文提出的策略优化方法。  
  
流程很直观：  
  
针对同一个 Prompt 采样 G 条回答；计算每条回答的奖励；根据组内奖励构造相对 Advantage；最后用策略比率和裁剪目标更新模型。… [Show more](https://x.com/Zen_with_AI/status/2115222011672957140)

> ![Link card preview](<山景城小路%20(@Zen_with_AI).assets/image-03.jpg>)
> 
> 🔗 [**DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open...**](https://t.co/JBZM3NHjWR)
> 
> Mathematical reasoning poses a significant challenge for language models due to its complex and structured nature. In this paper, we introduce DeepSeekMath 7B, which continues pre-training...
> 
> _From arxiv.org_

---

大模型后训练面试 09/17：GRPO 为什么需要 Probability Ratio 和 Clipping？  
  
GRPO 需要比较新策略和生成样本时所使用的旧策略。  
  
对第 i 条回答的第 t 个 Token，可以定义：  
  
ρᵢₜ = πθ(token | context) / πold(token | context)  
如果某个 Token 的 Advantage… [Show more](https://x.com/Zen_with_AI/status/2115222012121788454)

---

大模型后训练面试 10/17：GRPO 训练有哪些常见问题？  
  
第一，奖励信号失效。同一个 Prompt 的多条回答获得几乎相同的奖励，组内 Advantage 可能接近零，训练缺少有效的区分信号。  
  
第二，Reward Hacking。模型学会利用奖励函数的漏洞，而不是真正完成任务。  
  
第三，Rollout 成本高。每个 Prompt… [Show more](https://x.com/Zen_with_AI/status/2115222012566306989)

---

大模型后训练面试 11/17：OPD 和 GRPO 如何选择？  
  
核心区别是学习信号。  
  
GRPO 利用奖励信号，比较同一个 Prompt 下多条回答的相对表现，再根据 Advantage 更新策略。  
  
OPD 利用教师模型的监督。学生自己生成回答，教师再对这些轨迹提供指导，通常包括 Token 级概率信息。  
  
GRPO… [Show more](https://x.com/Zen_with_AI/status/2115222012964856256)

---

大模型后训练面试 12/17：什么是 KL 散度？  
  
KL 散度衡量两个概率分布的差异：  
  
D_KL(P || Q) = Σₓ P(x) log(P(x) / Q(x))  
它非负，但一般不对称，因此不是严格意义上的距离。  
  
在 RLHF 或 GRPO 中，KL 常用于约束当前策略不要偏离参考策略太远。一个简化目标是：  
  
目标 = 期望奖励 − β ×… [Show more](https://x.com/Zen_with_AI/status/2115222013493354953)

---

大模型后训练面试 13/17：Forward KL 和 Reverse KL 有什么区别？  
  
在 OPD 中，常用分布损失让学生模型学习教师模型的输出分布，但 KL 的方向很重要。  
  
Forward KL：  
  
D_KL(π_teacher || π_student)  
它要求学生覆盖教师分布中的概率质量，通常被直观描述为偏向 Mode Covering。  
  
Reverse KL：… [Show more](https://x.com/Zen_with_AI/status/2115222013883404712)

---

大模型后训练面试 14/17：为什么需要 FSDP 或 ZeRO？  
  
训练大模型时，GPU 不只要保存参数，还要保存梯度、优化器状态和前向传播的中间激活。  
  
以 Adam 为例，一阶和二阶动量也会占用大量显存。所以参数能放进 GPU，不代表完整训练状态也能放进去。  
  
FSDP 和 ZeRO… [Show more](https://x.com/Zen_with_AI/status/2115222014319566940)

---

大模型后训练面试 15/17：ZeRO-1、2、3 有什么区别？  
  
ZeRO 是 DeepSpeed 提供的大模型训练状态分片技术，三个阶段逐步增加被切分的状态。  
  
ZeRO-1：切分优化器状态，梯度和模型参数通常仍然复制。  
  
ZeRO-2：进一步切分梯度，模型参数通常仍然复制。… [Show more](https://x.com/Zen_with_AI/status/2115222014835482994)

> 🔗 [**Zero Redundancy Optimizer**](https://t.co/1I6jQ7IoDN)
> 
> DeepSpeed is a deep learning optimization library that makes distributed training easy, efficient, and effective.
> 
> _From deepspeed.ai_

---

大模型后训练面试 16/17：FSDP 和 ZeRO-3 有什么关系？  
  
FSDP 是 PyTorch 的 Fully Sharded Data Parallel 方案，与 ZeRO 的目标相似：把训练状态分片到多个 GPU，降低单卡显存需求。  
  
以 FSDP Full Shard 为例：  
  
每张卡长期保存自己负责的参数分片；前向计算前通过 All-Gather… [Show more](https://x.com/Zen_with_AI/status/2115222015431102934)

---

大模型后训练面试 17/17：FSDP / ZeRO-3 能解决所有 OOM 吗？  
  
不能。它们主要降低训练状态的显存占用，其他显存来源还有：  
  
Activation Memory：反向传播需要的中间激活。  
  
Attention Memory：长序列注意力计算及临时张量。  
  
Communication Buffers：参数收集和梯度归约的临时内存。  
  
Rollout KV… [Show more](https://x.com/Zen_with_AI/status/2115222015863071051)