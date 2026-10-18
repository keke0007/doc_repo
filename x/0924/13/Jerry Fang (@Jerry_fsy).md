---
author: "Jerry Fang"
handle: "@Jerry_fsy"
source: "https://x.com/Jerry_fsy/status/2102699787082698758"
date: 2026-08-13T15:43:29.000Z
type: thread
---
# Jerry Fang (@Jerry_fsy)

对于想入门AI领域的朋友们，我筛选出一些适合入门的资料，我会附上领域以及对应的资料：  
  
Agent闭环迭代：  
1.[engineering.fb.com/2026/09/02/ml-…](https://engineering.fb.com/2026/09/02/ml-applications/organizational-second-brain-ai-learns-from-experts/)  
Meta的这一篇讲了对于历史case，经过专家纠正后，系统深入调研Agent执行/回答有偏差的原因，然后进行Agent的迭代。  
我认为是Agent迭代的一个很重要的组成。  
我自己有在做这块的试验，后面可以分享一下效果  
cc: @AIatMeta  
  
2.[mp.weixin.qq.com/s/5VDN-T9K8Wr-…](https://mp.weixin.qq.com/s/5VDN-T9K8Wr-DaQ15-I6CA?poc_token=HMWWs2qjZrx5OjySGOfgpNvZEGXXFWJf6ZAdudBB)  
腾讯的这篇分享系统讲了Agent闭环迭代飞轮的4要素：评测->知识（历史case算其中的一部分）->Harness改进->人review。  
是一篇很好的闭环迭代的启蒙文章。  
cc: @TencentGlobal  
  
Agent上下文工程：  
1.[medium.com/@peakji/contex…](https://medium.com/@peakji/context-engineering-for-ai-agents-lessons-from-building-manus-71883f0a67f2)  
Manus对上下文的理解，在当时确实很领先的，在这篇文章中能看到很多对于Agent工程的思考，尤其是对KV Cache命中率的深入理解。  
cc: @peakji  
  
AI Infra：  
1.[aleksagordic.com/blog/vllm](https://aleksagordic.com/blog/vllm)  
这篇是看到@Kay2289123 的推荐，确实是适合作为Infra的第一篇入门文章。主要围绕vLLM讲了很多推理的概念。感谢推荐。  
  
2.还有下面这篇，深入浅出讲GPU如何工作：

> **Akshay (@akshay_pachaar)**
> 
> ![Article cover](<Jerry%20Fang%20(@Jerry_fsy).assets/image-01.jpg>)
> 
> 📝 **How a GPU Actually Works**
> 
> The intuition an LLM engineer needs, without the hardware manual.
> By the end, techniques like quantization, speculative decoding, and continuous batching should stop looking like a list of tricks to...

---

Manus这篇文章很值得看看，它算比较早提出并采用code mode的agent产品。以及文章中的重点，agent要围绕着输入缓存命中率做文章。