---
author: "山景城小路"
handle: "@Zen_with_AI"
source: "https://x.com/Zen_with_AI/status/2126024669077926016"
date: 2026-11-26T18:45:55.000Z
type: tweet
likes: 30
reposts: 5
replies: 8
bookmarks: 29
views: 2361
---
# 山景城小路 (@Zen_with_AI)

4 个 Qwen 0.6B nano swarm，成功率 6.6%→82.3%  
  
4 个 0.6B 的小模型，居然能学会互相配合。  
4 个 agent 共用一个 Qwen3-0.6B，任务是把各自手里的数字排好序，谁也不知道别人的数，不说话就解不出来。  
75 轮训练下来，成功率从 6.6% 涨到 82.3%，每次尝试的 token 还少了 37%。  
  
agent 沟通到底是真帮忙，还是只是在多烧钱。这轮的答案是真帮忙：广播从每队 2.38 次涨到 5.73 次，token 反而少了。博客附了完整复现教程，两块 H200 跑 5 小时，约 50 美元。

> ![Link card preview](<山景城小路%20(@Zen_with_AI).assets/image-01.jpg>)
> 
> 🔗 [**Swarms, Flocks, Gangs: How to build, train, and evaluate multi-agent systems**](https://t.co/qyVcstLCa9)
> 
> _From huggingface.co_