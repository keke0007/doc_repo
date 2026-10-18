---
author: "Jason Zhu"
handle: "@GoSailGlobal"
source: "https://x.com/GoSailGlobal/status/2102745333167997403"
date: 2026-09-23T13:02:09.000Z
type: thread
---
# Jason Zhu (@GoSailGlobal)

发现了一个跟 Jev 对标的决策模型  
  
来自牛津 AI 团队研发的 this-that-model-1.0模型  
  
背后公司 [FLock.io](http://FLock.io) 做去中心化 AI 基建（联邦学习 + 区块链），拿过以太坊基金会学术资助，跟 UNDP、通义千问合作，融了 $11M  
  
开源，而且 API 完全免费  
  
它是一个 typed decision model：你给一段上下文 + n 个选项，它一次前向传播返回选择和校准过的置信度；31ms，零输出 token，格式错误在数学上不可能发生。  
  
跟 Jev 的区别：  
- Jev 是商业 API，按 token 收费  
- this-that-model 在官网API注册使用，input/output/cache 竟然全部免费，注册拿 API key 就能用  
- 你还能自己部署，MIT 开源  
  
用法跟 Jev 一模一样，任何「从 n 个选项里选一个」的场景，一次前向传播可以同时回答多个问题  
1️⃣ 意图分类  
2️⃣ 内容审核  
3️⃣ 风险分档  
4️⃣ 路由决策  
  
最近Jev又关闭注册了，this-that-model-1.0在官网即可获取免费API 👇

![🎥 Video](<Jason%20Zhu%20(@GoSailGlobal).assets/image-01.jpg>)

---

开源：[huggingface.co/flock-io/this-…](https://huggingface.co/flock-io/this-that-model-1.0)  
  
免费 API：[platform.flock.io](https://platform.flock.io)  
  
官网：[flock.io](https://flock.io)

> ![Link card preview](<Jason%20Zhu%20(@GoSailGlobal).assets/image-02.jpg>)
> 
> 🔗 [**flock-io/this-that-model-1.0 · Hugging Face**](https://t.co/njB3aCZQgA)
> 
> _From huggingface.co_