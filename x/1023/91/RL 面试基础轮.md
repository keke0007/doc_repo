---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2112679456246735315"
date: 2026-10-20T22:56:48.000Z
type: article
---
# RL 面试基础轮

*By 山景城小路 (@Zen_with_AI)*

## 一、RL 基础：8 道必练（带答题要点）

Q1. Policy gradient 的直觉是什么？REINFORCE 的梯度长什么样？

- 目标：最大化期望回报 J(θ)；梯度 = E[ ∇log π(a|s) · G ]，G 是整条轨迹的回报
- 直觉：回报高的动作，把它的 log 概率往上推；回报低的往下压
- 口诀："好动作加概率，坏动作减概率，按回报加权"

Q2. REINFORCE 方差为什么大？baseline 怎么降方差？

- G 整条轨迹采样，噪声大，一次采样定生死
- 减 baseline b(s)：梯度 = E[ ∇log π · (G − b) ]，**不引入 bias**（因为 E[∇log π · b] = 0）
- 常用 baseline = value 函数 V(s)；这就自然过渡到 Actor-Critic
- 追问：为什么减一个常数不改变期望梯度？（score function 期望为零：E[∇log π] = 0）

Q3. Advantage 是什么？GAE 干嘛的？

- Advantage A(s,a) = Q(s,a) − V(s)：这个动作比平均水平好多少
- 只用单步 TD 误差 bias 大、方差小；用整轨迹 Monte Carlo 方差大、bias 小
- GAE(λ)：λ 在两者之间插值调权衡；λ=0 退化成 TD，λ=1 退化成 MC
- 工程默认值 λ=0.95，γ=0.99

Q4. PPO 的 clipped objective 为什么要 clip？

- 策略更新步子太大，一次更新直接崩（trust region 思想，TRPO 用 KL 硬约束）
- L = E[ min( r·A, clip(r, 1−ε, 1+ε)·A ) ]，r = 新/旧策略概率比，ε 通常 0.2
- 直觉：占便宜（r·A 很大）时掐住上限，保守更新；一阶优化就能训，不用 TRPO 的二阶
- 你 ML 手稿第 19 期讲过 GRPO——GRPO 就是 PPO 在 LLM 上的变体：**省掉 critic 网络**，用同组多个采样的相对奖励做基线

Q5. DPO 和 PPO 的本质区别？什么时候选 DPO？

- PPO：显式训 reward model，再在线 RL；贵、超参多、不稳定，但天花板高
- DPO（你第 18 期手稿）：跳过 reward model 和 RL，直接在偏好对上做分类式优化；轻、稳、便宜
- 选 DPO：资源紧、快速迭代；选 PPO/在线 RL：要榨上限、DPO 撞墙后
- 追问：DPO 的 implicit reward 长什么样？（β·log(π/π_ref) 相对参考模型的对数比）

Q6. Reward hacking / reward misspecification：举例 + 怎么防？

- 例子：reward model 给"长回答"高分 → 模型学会车轱辘话凑长度；代码题 reward 只看测试通过 → 模型写 hack 测试用例的代码
- 防：KL 散度约束（别离 SFT 模型太远）、reward model 集成、定期人工审计 bad case、保留 held-out 评估
- 面试官想听的：你承认"奖励 ≠ 真实目标"，并有成体系的缓解手段

Q7. RLHF 落地时最大的几个坑？（公开源原题：What challenges might arise?）

- Reward model 泛化：训练分布外 human preference 采不到，RM 外推瞎打分
- 分布偏移：policy 越训离 SFT 分布越远，RM 在新分布上不可靠
- 人类偏好本身噪声大、不一致（标注员分歧），RM 学的是"平均口味"
- 评估难：自动指标和人类真实满意度对不上，只能烧钱做人工评测

Q8. Exploration vs exploitation：三种经典手段的直觉

- ε-greedy：ε 概率随机探索，简单但低效
- 熵正则：在目标里加 entropy bonus，逼策略保持"不确定"、别太早收敛到局部最优（LLM RL 常用）
- UCB / 乐观探索：优先试"不确定性高"的动作，bandit 问题的标准答案
- 一句话：没有探索，RL 就是拿着噪声梯度在局部最优里打转