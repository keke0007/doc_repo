---
author: "AYi"
handle: "@AYi_AInotes"
source: "https://x.com/AYi_AInotes/status/2097653239713788256"
date: 2026-09-09T11:47:59.000Z
type: thread
---
# AYi (@AYi_AInotes)

别再到处花钱买提示词大全了，Anthropic 内部工程师已经当面把底牌给掀了：  
  
“提示词工程（Prompting）基本已经进垃圾桶了。  
我们 90% 的工程师早就全面用上了自愈循环（Loops），现在全员正在往智能体图（Agentic Graphs）转。”  
  
视频里这位 Anthropic 工程师，直接面对一个完全空白的终端，  
只用了短短 10 分钟，现场徒手搓出了一整套完整的 Claude Code 核心系统。  
  
看完这段演示，你会瞬间明白为什么你手里的 Agent 永远像个弱智实习生：  
你还在指望用一段精妙的 Prompt 让 AI 别犯错，  
而顶级工程师的解法是：默认 AI 每一步都会写挂，通过构建状态图（Graphs）和反馈闭环（Loops），让 Agent 撞墙后自己看报错、自己改代码、自己跑测试重试。  
  
从玩具到生产力，其实就四步跳跃：  
Agents（单点执行）→ Loops（容错循环）→ Graphs（多智能体图工程）→ Self-Improving（自我进化系统）  
  
Prompt 只是过去式，图工程才是未来几年所有技术人必须重构的底层骨架。  
10 分钟全是干货，没有任何公关废话，建议所有做 AI 开发和自动化的朋友静下心看完。 [https://x.com/0xjmori/status/2097378778196394377/video/1…](/0xjmori/status/2097378778196394377/video/1)

![🎥 Video](<AYi%20(@AYi_AInotes).assets/image-01.jpg>)

---

1. 视频核心看点：注意看 04:30 处她如何用极简状态机接管 Claude 的报错信号，那是“自愈循环”最值钱的代码逻辑  
2. 术语纠偏：图工程（Graph Engineering）不是做知识图谱，而是用有向图（如 LangGraph、Google ADK）来管理 Agent 的决策分支与跨会话记忆；