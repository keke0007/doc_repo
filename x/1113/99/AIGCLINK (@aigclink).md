---
author: "AIGCLINK"
handle: "@aigclink"
source: "https://x.com/aigclink/status/2119630757375406263"
date: 2026-11-09T03:18:48.000Z
type: thread
likes: 204
reposts: 33
replies: 27
bookmarks: 205
views: 17622
---
# AIGCLINK (@aigclink)

蚂蚁最新开源了一个只有4B的专用AI决策模型：LemonDecider，超Jev 1.14，中位推理耗时仅50.7ms  
  
LemonDecider的思路是，让模型直接做判断，并不是先生成文字答案，再让程序从中提取结论  
  
基于Qwen3.5-4B专门针对决策任务训练，支持三种输出  
  
1、Choice，多选一，判断该调用哪个工具，进入哪条工作流  
2、Yes/No，二元判断，判断证据是否充分，请求是否符合条件  
3、Score，评分，评估检索相关性、内容质量等  
  
它还有一个特点，同一段上下文下可以一次提交多个决策问题，在长度允许时通过一次前向计算输出对应结果，减少重复处理上下文的开销  
  
在Decision Index 0.4.1 Public评测中，LemonDecider 4B 61.67分、Jev 1.14 60.70分  
  
完整约14万条测试请求中，中位推理耗时为50.7ms  
  
蚂蚁这次把120万训练样本连同训练代码一起开源了，覆盖864个任务标签，包含意图识别、工具路由、证据判断、检索相关性、内容安全等任务  
  
#LemonDecider #AI决策模型 #Jev

![Image](<AIGCLINK%20(@aigclink).assets/image-01.jpg>)

---

HF：

> ![Link card preview](<AIGCLINK%20(@aigclink).assets/image-02.jpg>)
> 
> 🔗 [**inclusionAI/LemonDecider · Hugging Face**](https://t.co/u9npXARNk2)
> 
> _From huggingface.co_