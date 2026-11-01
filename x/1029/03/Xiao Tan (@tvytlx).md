---
author: "Xiao Tan"
handle: "@tvytlx"
source: "https://x.com/tvytlx/status/2115319147919995345"
date: 2026-10-28T05:46:00.000Z
type: tweet
likes: 301
reposts: 25
replies: 11
bookmarks: 309
views: 15214
---
# Xiao Tan (@tvytlx)

OCR 准确率做到 95%，人还是得把 100% 的页面核一遍。  
  
因为你不知道错的那 5% 在哪。  
  
做过票据、合同、报表自动化的，应该都踩过这个坑。模型越来越准，审核的人一个没少。  
  
OpenBMB 刚开源了两个模型，我觉得第二个更有意思。  
  
第一个是 MiniCPM-OCR 1.0，1.3B，专门对付现实里的烂文档：手机拍的、弯的、歪的、翻拍的。  
Real5-OmniDocBench 93.68，一张 5090 每秒 1.2 页。  
  
第二个叫 Jev，2B，专门给 OCR 结果打分。  
  
做法挺巧的：  
把 OCR 认出来的文字，重新渲染成一张图  
跟原图那一小块放一起  
让模型判断这俩是不是一回事  
最后给一个 0 到 1 的分  
  
有点像小学做完除法，再乘回去验算。  
  
判断对错的 AUROC 是 93.6。  
Qwen3.5 那个 397B 的大模型是 93.8。  
差 0.2，参数量差了快 200 倍。  
  
而且它不挑 OCR。GLM-OCR、PaddleOCR-VL、小米 OCR 的输出拿给它判，都在 91.9 到 93.4 之间。  
  
也就是说现有流水线不用动，后面挂一个它就行。  
  
用法就两种：  
高分自动放行，低分转人工。  
或者几个 OCR 一起跑，每一块挑分最高的拼起来。  
  
边界也说一下：  
它只能在已有结果里挑，几个模型全错了它也救不回来。  
阈值得拿自己的业务样本去调。  
JevOCRBench 是他们自己出的榜单，真要上生产还是得自己测。  
  
但我觉得这个思路比 OCR 本身值钱。  
  
很多自动化卡住，是因为人不敢信。  
每个结果都配一个“这次靠不靠谱”的分数，人只去看低分那一小撮，账就完全变了。  
  
AI 写的代码、AI 填的表，缺的其实都是这么一个便宜的验算员。  
  
两个都是 Apache 2.0，vLLM、llama.cpp、Ollama 都能跑。  
  
先有了干活的模型，现在又有了质检的模型，照这个进度，下一步估计就是专门开会的模型了。

> ![Link card preview](<Xiao%20Tan%20(@tvytlx).assets/image-01.jpg>)
> 
> 🔗 [**From the LocalLLaMA community on Reddit: A 1.3B OCR model just hit 93.68 on real-world document...**](https://t.co/klE6OJvY9M)
> 
> _From reddit.com_