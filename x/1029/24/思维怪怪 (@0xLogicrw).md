---
author: "思维怪怪"
handle: "@0xLogicrw"
source: "https://x.com/0xLogicrw/status/2115003558773084339"
date: 2026-10-26T02:19:00.000Z
type: tweet
likes: 74
reposts: 10
replies: 3
bookmarks: 79
views: 9032
---
# 思维怪怪 (@0xLogicrw)

Anthropic 平台工程负责人 Katelyn Lesse 上周透露，她领导的团队已让 Claude 编写全部代码，工程师主要负责系统设计和验证。  
  
10 月 26 日，她又发表长文，认为软件工程师接下来有一批更难的问题要解决。未来可能有上千个 Agent 持续数周协作，现有的软件基础设施、权限和安全机制还不足以支撑这种规模。  
  
比如 Kubernetes 可以运行 Agent，但不擅长管理大量需要保存状态、频繁休眠和恢复的任务。Google 已经推出 Agent Substrate，让闲置的 Agent 暂停运行，需要时再快速恢复，减少算力浪费。  
  
权限也不能简单照搬现有做法。Agent 可能代替用户工作几周，还会向其他 Agent 分派任务。系统需要单独识别每个 Agent、限制它能做什么，并允许用户随时撤销授权。  
  
Anthropic 自己已经遇到过越权事故。今年 9 月，公司披露了 4 起安全测试事件。由于测试环境错误接通公网，模型访问了真实的第三方系统。其中一个甚至获取了管理员权限，继续收集密码、修改系统设置，还读取了个人信息。  
  
这些事故都发生在单个模型独立执行任务时。多 Agent 协作还会增加另一层风险：一个 Agent 被恶意指令控制后，可能把指令传给其他 Agent，甚至写入共享记忆，让后续任务继续受到影响。  
  
Lesse 认为，Agent 必须在严格隔离的环境中运行，操作记录不能被它们自行篡改。系统还需要真正有效的紧急开关，能够一次停止所有相关 Agent，并撤销它们的访问权限。

> **Katelyn Lesse (@katelyn_lesse)**
> 
> ![Article cover](<思维怪怪%20(@0xLogicrw).assets/image-01.png>)
> 
> 📝 **The new set of problems for engineers to solve**
> 
> A lot of software engineers are going through what can only be described as an identity crisis. The job has changed a lot. Some are depressed, others are having the time of their lives. There's a...
> 
> software engineers are going to have to tackle a new set of problems to unlock powerful multiagent systems to solve the world's hardest problems - right now is one of the most interesting moments in history to be an engineer.