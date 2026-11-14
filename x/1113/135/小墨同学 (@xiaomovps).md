---
author: "小墨同学"
handle: "@xiaomovps"
source: "https://x.com/xiaomovps/status/2119312964486352971"
date: 2026-10-11T01:12:56.000Z
type: tweet
---
# 小墨同学 (@xiaomovps)

自己开发一个 Personal Agent，其实可以从 Pi Durable 开始🔥  
  
看到使用GrokBot的人越来越多，大家也越来越多Personal Agent越来越感兴趣，我也想看看能不能自己写一个呢？  
  
我自己研究的时候就已经发现了这几个项目，可以从这 4 个项目开始，循序渐进地学习  
  
1、pi-durable-hello｜从零理解 Pi Durable  
  
通过 5 个简单的例子，学习任务持久化、崩溃恢复、请求去重和实时监听  
  
不需要 API Key，适合先搞懂 Durable Agent 的运行原理  
  
[github.com/avparkhi/pi-du…](https://github.com/avparkhi/pi-durable-hello)  
  
2、Batty｜开发自己的私人 AI 助理  
  
基于 Pi Durable，已经整合了长期记忆、定时任务、Subagent、Skills、MCP，还有移动端 PWA  
  
最值得研究的是它的长期记忆系统，怎么让 Agent 记住之前的事情，又不需要每次都把所有聊天记录塞进上下文  
  
[github.com/jeffton/batty](https://github.com/jeffton/batty)  
  
3、Pots｜开发一个会主动工作的 Agent  
  
有点像开源版 Grok Bot，支持 Telegram、Slack、Discord，可以主动处理消息，通过定时任务和外部事件唤醒自己  
  
比较有意思的是，它会先用普通脚本检查任务，确定需要处理才调用大模型，减少不必要的 Token 消耗  
  
[github.com/Crokily/pi-dur…](https://github.com/Crokily/pi-durable-bot)  
  
4、Hearth Pi｜让 Agent 控制智能家居  
  
把 Pi Durable 接进 Home Assistant，让 Agent 读取设备状态，再通过人工审批控制灯光、开关等设备  
  
虽然还处于实验阶段，但很适合学习 Agent 怎么连接现实设备，以及权限和审批机制怎么设计  
  
[github.com/cosmyo/ha-pi-d…](https://github.com/cosmyo/ha-pi-durable)  
  
从任务恢复、长期记忆，到主动执行和现实设备控制，这几个项目基本能串起一条 Personal Agent 的学习路线  
  
Pi是你的Agent启蒙，也可以成为你Personal Agent路上的好帮手，真就那句话  
  
万物基于Pi 😂

![🎥 Video](<小墨同学%20(@xiaomovps).assets/image-01.jpg>)
![Image](<小墨同学%20(@xiaomovps).assets/image-02.jpg>)

> **小墨同学 (@xiaomovps)**
> 
> ![Article cover](<小墨同学%20(@xiaomovps).assets/image-03.jpg>)
> 
> 📝 **Pi Durable 从入门到精通：从一个开源小助手，到你自己的 Hermes**
> 
> 你有没有遇到过这种情况：让 Agent 去办一件稍微长一点的事，它查了资料、列了步骤，正等着一个结果，这时候电脑合上了，或者服务器重启了。...