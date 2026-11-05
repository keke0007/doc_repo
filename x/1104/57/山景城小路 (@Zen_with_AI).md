---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2115686218688319950"
date: 2026-10-29T06:04:36.000Z
type: tweet
likes: 109
reposts: 11
replies: 4
bookmarks: 118
views: 6104
---
# 山景城小路 (@Zen_with_AI)

OpenAI 机器学习ML 常考题：  
  
1. 找出噪声标注员并重训分类器  
给你一个 pandas DataFrame，含特征列、观测标签和标注员 ID。部分标注员乱标导致标签错误，找出这些标注员，剔除后重训分类器。  
  
2. 找出小型 Transformer 训练脚本里的 bug  
给你一个小型 decoder-only Transformer 的 PyTorch 实现、训练脚本和几个测试。里面藏了几个 bug，有的直接崩，有的能跑但 loss 不降或变 NaN，找出并修复。修好后，再给自回归解码加上 KV cache。  
  
3. 矩阵乘积扫描 + 手写反向  
给你 N 个 D×D 方阵 A[0..N-1]，算出每个 inclusive 前缀积 Y[i] = A[0]@…@ A[i]。先写 forward，再手写 backward，最后用 Hillis-Steele 扫描同时算正反两遍。  
  
4. 向量化 1-NN 分类器  
用 NumPy 实现 1 最近邻分类器，不用 Python 循环，按欧氏距离取最近训练点的标签。再把同一分类器写成单层线性层加 softmax 的前向，并改成 L1 距离版本。  
  
5. 从 logits 算熵  
先对原始 logits 稳定地算熵，再对分块到达的 logits 用常数额外空间流式算熵。  
  
6. 带 mask 和 label smoothing 的交叉熵  
实现带 masking 和 label smoothing 的交叉熵。  
  
7. 调试 GRPO 训练循环  
调试并改进一个语言模型的 GRPO 强化学习训练循环（PyTorch）。  
  
8. NumPy MLP 分片调试  
一个 NumPy 实现的 MLP，分片里藏了 5 个 bug，找出并修复。