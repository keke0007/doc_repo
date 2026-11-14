---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2120412352864784853"
date: 2026-11-11T07:04:34.000Z
type: tweet
likes: 232
reposts: 45
replies: 6
bookmarks: 245
views: 7492
---
# 山景城小路 (@Zen_with_AI)

Alisa 的《Alisa's book of LLMs》。一个人亲手写给自己的复习笔记：从神经网络基础讲起，一个概念配一段手推。  
如果您正在准备 ML 面试，这个 Notion 笔记值得收藏。  
先讲“多层感知机”（MLP）。一个神经元怎么算，写成矩阵是什么样，一批样本一起算又是什么样，每一步都带着形状推。PyTorch 里 W 为什么存成 n_out × n_in、前向为什么要转置，也写明白了。  
  
然后是“反向传播”。Z = XW + b 的三个梯度一个一个推，还给了通用法则：先对单个样本求雅可比，共享的张量把 batch 维求和，不共享的保留。  
  
最后是“激活函数”。sigmoid、tanh、softmax、ReLU、Leaky ReLU、Swish、GLU、SwiGLU，每个都讲了长处和毛病，比如 ReLU 为什么会“死掉”。  
  
面试问到的 MLP 和反向传播手推，这里都有现成的推导思路，照着走一遍，比背公式管用。  
  
[alisawuffles.notion.site/alisa-s-book-o…](https://alisawuffles.notion.site/alisa-s-book-of-llms)

![Image](<山景城小路%20(@Zen_with_AI).assets/image-01.jpg>)
![Image](<山景城小路%20(@Zen_with_AI).assets/image-02.jpg>)
![Image](<山景城小路%20(@Zen_with_AI).assets/image-03.jpg>)