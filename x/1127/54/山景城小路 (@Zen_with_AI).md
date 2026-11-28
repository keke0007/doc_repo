---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2125272223724585137"
date: 2026-11-24T16:55:58.000Z
type: thread
likes: 8
reposts: 1
bookmarks: 8
views: 1868
---
# 山景城小路 (@Zen_with_AI)

跟着 Raschka 手搓推理模型 第 2 期  
  
想让模型学会推理，先得有个能跑起来的模型。这期不练推理，先把基座搭起来。  
  
Raschka 不从头预训练，直接请一个现成的预训练模型当底子：Qwen3 0.6B，只有 6 亿参数，小到普通电脑都能跑。做法是把权重下载下来、加载到选好的设备上，再自己写文本生成：把文字切成 token，模型一次吐一个 token 的概率分布，挑出下一个词、拼回去、再喂进去，循环着把句子接长。  
  
选 0.6B 不是图省事：模型小，错得看得见、改得快，前面这些例子 CPU 就能跑；等后面真要训练了，再上 GPU 也不迟。视频里还带着把张量形状、logits 和下一个 token 的预测一步步拆开看——生成不是黑盒，每一步算的是什么，先看清楚。  
  
关键在两处提速。第一是贪心解码加流式生成：每步取概率最高的那个词，边生成边吐字，不用等整段写完；遇到结束符就停，还要顺手测一下生成速度，心里有个底。第二是 KV 缓存：注意力每算一步都要翻一遍前面所有词的 Key 和 Value，缓存就是把算过的存起来，下一步只算新词，越生成越省。再配上 torch.compile 把模型编译一下，同一份代码在不同硬件上跑得更快。  
  
这期结束，手里就有了能跑的基座和一把尺子：下期先给它装个数学判卷器，在 MATH-500 上量一量现在几斤几两，再谈怎么让它变聪明。

![Image](<山景城小路%20(@Zen_with_AI).assets/image-01.jpg>)

---

把我我炒股的一个技巧贡献给大家。看韩国股市杠杆率的网站：  
  
[en.macromicro.me/charts/152896/…](https://en.macromicro.me/charts/152896/south-korea-margin-loan-balance)

![Image](<山景城小路%20(@Zen_with_AI).assets/image-02.jpg>)