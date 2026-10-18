---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2107951349950185600"
date: 2026-10-06T15:49:50.000Z
type: tweet
likes: 29
reposts: 5
replies: 3
bookmarks: 24
views: 5855
---
# 山景城小路 (@Zen_with_AI)

第5期ML面试问题  
  
Q：注意力机制里的 Q、K、V 分别是什么？  
  
A：  
  
查询 Query  
"我想找什么"。每个位置发出一个查询向量，代表它在找哪类信息。比如翻译时，解码器正要生成的词就拿着 Query，去原文里找相关的词。  
键 Key  
"我有什么"。序列里每个位置都有一把"钥匙"，标着"我这里存的是哪类信息"。Query 跟所有 Key 算相似度（点积），得分高的位置更相关。  
值 Value  
"具体内容"。相似度经 softmax 变成权重，对所有 Value 加权求和，得到融合后的表示。Key 决定"看哪里"，Value 决定"拿什么"。  
（一句话记住：Query 提问，Key 报名，Value 交答卷。）  
追问加分：QK 点积后为什么要除以 √dₖ？（维度大了点积方差变大，softmax 会饱和、梯度消失，除一下把分布稳住。）  
更深度问题：Multi-Head Attention 为什么要用多个头，而不是把单个头的维度做大？

> **山景城小路 (@Zen_with_AI)**
> 
> 第4期ML面试问题  
>   
> Q：决策树分裂特征时，信息增益、信息增益率、基尼系数都是干嘛的？  
>   
> A：  
>   
> 信息增益 information gain（ID3 用它）  
> 分裂前后"不确定性"减少了多少。不确定性用熵 entropy x.com/Zen_with_AI/st…