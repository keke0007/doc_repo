---
author: "旺旺"
handle: "@TiagerBao"
source: "https://x.com/TiagerBao/status/2120995600640684477"
date: 2026-11-12T21:42:12.000Z
type: tweet
likes: 53
replies: 5
bookmarks: 4
views: 711
---
# 旺旺 (@TiagerBao)

1.7B 参数的小模型，拒绝回答率从 87.3% 降到了 2.8%。  
  
OrcaRouter 发布了 LFM3-1.7B 的 Abliterated 版本，模型权重已上线 Hugging Face。  
  
几个关键数据：  
  
仅 17 亿参数，BF16 权重约 3.4GB  
采用混合卷积与注意力架构，面向本地和边缘设备  
通过消融技术削弱模型的拒答机制  
官方测试中，GSM8K 得分保持 86%  
MMLU 从 49% 降至 45.7%  
  
需要注意，拒答率来自项目方测试，不代表模型能力更强。安全限制减少，也意味着使用风险增加。  
  
我更关心它在本地 Agent 场景下的表现，尤其是工具选择、任务路由这些高频小任务。目前项目还没有提供相关评测。  
  
你愿意用这种 1.7B 小模型处理本地 Agent 的简单任务，还是更看重模型的稳定性和安全性？  
  
模型地址：  
[huggingface.co/orcarouter/LFM…](https://huggingface.co/orcarouter/LFM3-1.7B-abliterated)  
  
#AI #LocalLLM #Agent #开源

> ![Link card preview](<旺旺%20(@TiagerBao).assets/image-01.jpg>)
> 
> 🔗 [**orcarouter/LFM3-1.7B-abliterated · Hugging Face**](https://t.co/w0iWGUgDUg)
> 
> _From huggingface.co_