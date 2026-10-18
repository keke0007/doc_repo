---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2108707426564477181"
date: 2026-10-08T16:28:50.000Z
type: tweet
---
# 山景城小路 (@Zen_with_AI)

📐 ML 铅笔手稿 第 12期 📷✏️  
  
上期的问题：RNN 一步步读句子，离得远的词互相"够不着"——信息要穿过几十个时间步，传到后面就稀释了。2017 年一篇论文干脆把循环扔了，换成 Attention（注意力机制）。  
  
核心就一个动作：查字典。  
  
每个位置手里有三样东西：查询 q（query，我在找什么）、键 k（key，我这里有什么）、值 v（value，我的实际内容）。拿 q 去和所有位置的 k 算相似度（点积），分数越高越"相关"；softmax 把分数归一成权重 α，再按权重把 v 加权求和：  
  
αᵢ = softmax(q·kᵢ/√dₖ)，输出 = Σαᵢvᵢ  
  
矩阵形式一步写完：Attention(Q,K,V) = softmax(QKᵀ/√dₖ)V  
  
为什么除以 √dₖ？点积的方差随维度 dₖ 增长，维度一大分数就拉得很开，softmax 直接饱和、梯度消失——除以 √dₖ 把方差压回 1，训练才稳定。  
  
多头（multi-head）：一组 Q/K/V 只能看到一种"相关"，开 h 个头并行查、结果拼起来——有的头看语法搭配，有的头看指代关系，各管一摊。  
  
自注意力（self-attention）：Q、K、V 都来自同一句话，自己跟自己算相关——"它"指的到底是"猫"还是"狗"，靠的就是这个权重矩阵。  
  
2017 年 Vaswani 等人的《Attention Is All You Need》用纯 Attention 搭出 Transformer，机器翻译成绩直接刷新纪录——RNN 时代从此结束。  
  
一句话：Attention = 拿查询去"查字典"，按相关度加权求和，让模型自己决定该看哪里。  
  
下一期：Transformer——把 Attention 堆成整台机器。

![Image](<山景城小路%20(@Zen_with_AI).assets/image-01.jpg>)

> **山景城小路 (@Zen_with_AI)**
> 
> 📐 ML 铅笔手稿 第 11期 📷✏️  
>   
> 上期的问题：MLP 参数成千上万，隐藏层的"正确输出"是多少没人知道，w 到底怎么调？答案：反向传播（backpropagation）。  
>   
> 前向传播（forward pass）就是上期的公式：  
>   
> z⁽ˡ⁾ = W⁽ˡ⁾a⁽ˡ⁻¹⁾ + b⁽ˡ⁾，a⁽ˡ⁾ = σ(z⁽ˡ⁾)  
>   
> 一层层算出预测 ŷ，再用损失 L x.com/Zen_with_AI/st…
> 
> ![Image](<山景城小路%20(@Zen_with_AI).assets/image-02.jpg>)