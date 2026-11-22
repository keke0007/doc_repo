---
author: "yibie"
handle: "@yibie"
source: "https://x.com/yibie/status/2122918712160535032"
date: 2026-11-17T20:09:46.000Z
type: thread
likes: 46
reposts: 11
replies: 3
bookmarks: 61
views: 3969
---
# yibie (@yibie)

给一个 coding agent 单独训练模型  
  
Hugging Face 给开源模型 Qwen3.8-27B 配了一个 LoRA，只在 pi 这个 coding agent 里用强化学习训练，不动基座权重。  
  
做法上有两个值得看的点。训练在没改过的 pi 里跑，模型每发一次工具调用，中间的 safeagents 代理先用规则查一遍，明显在钻奖励空子的调用直接拦掉、记 0 分。任务是从公开 agent 会话重建的可验证环境，分修 bug、问答、从零搭建、扩展代码四类，对错由任务自带的验证器判，不靠人打分。奖励里还专门加了一项：答对的前提下工具调用越少加分越多。  
  
效果是准确率和效率一起涨。SWE-bench Pro 最难的 51 题，基座模型 pass@1 是 29.4%，适配器到 62.7%，工具调用少 55%。Terminal-Bench 2.1 从 39.5% 到 55.8%，工具调用少 62%。评测是同任务配对、温度 0.8、关思考，显著性也给了。  
  
模型卡里有个真实例子：同一个 SWE-bench 任务，基座反复搜索、重跑片段，撞到 100 步上限没过；适配器找到函数、改掉、跑测试、补最后一个用例，一共 8 次调用。  
  
我觉得真正的信号是 LoRA 把适配成本压到了小团队够得着的程度：不用重新训一个 agent 模型，给自家工具和 harness 单独训一个小适配器就行。权重已经开放，vLLM、llama.cpp 都能挂。  
  
两点保留。数字都是 Hugging Face 自测，没有第三方复现；最终检查点是看着评测子集的分数挑的，而且训练计划 1000 步、第 191 步就提前停了。  
  
原文链接在评论区  
#评测

![Image](<yibie%20(@yibie).assets/image-01.jpg>)

> **clem (@ClementDelangue)**
> 
> Introducing Qwen3.8-27B-pi-adapted 🤗  
>   
> We adapted Qwen's model specifically for the pi coding agent using reinforcement learning and a LoRA adapter.  
>   
> The results, measured as pass@1:  
> 🚀 SWE-bench Pro (Hard 51): 29.4% → 62.7%  
> 💻 Terminal-Bench 2.1: 39.5% → 55.8%  
> ⚡ 55–62% fewer
> 
> ![Image](<yibie%20(@yibie).assets/image-02.jpg>)

---

原文：  
[x.com/ClementDelangu…](https://x.com/ClementDelangue/status/2122784279604383914)  
[huggingface.co/harness-hub/Qw…](https://huggingface.co/harness-hub/Qwen3.8-27B-pi-adapted)  
[huggingface.co/blog/harness-h…](https://huggingface.co/blog/harness-hub/harness-adapters)

> **clem (@ClementDelangue)**
> 
> Introducing Qwen3.8-27B-pi-adapted 🤗  
>   
> We adapted Qwen's model specifically for the pi coding agent using reinforcement learning and a LoRA adapter.  
>   
> The results, measured as pass@1:  
> 🚀 SWE-bench Pro (Hard 51): 29.4% → 62.7%  
> 💻 Terminal-Bench 2.1: 39.5% → 55.8%  
> ⚡ 55–62% fewer
> 
> ![Image](<yibie%20(@yibie).assets/image-02.jpg>)