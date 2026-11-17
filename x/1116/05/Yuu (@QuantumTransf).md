---
author: "Yuu"
handle: "@QuantumTransf"
source: "https://x.com/QuantumTransf/status/2121968281968144703"
date: 2026-11-15T14:07:17.000Z
type: tweet
likes: 345
reposts: 56
replies: 31
bookmarks: 433
views: 19737
---
# Yuu (@QuantumTransf)

关于 Agent Memory，最近几篇论文（arXiv 2610.30755 / Memory Tax / Context as Infrastructure）验证了三个反直觉的事实：  
  
- 在召回率方面，是 The bitter lesson 的胜利：原始历史上的朴素检索在 7 组 benchmark 里拿了 6 个第一。  
- 最好的复杂 Memory 系统，表现还不如朴素 RAG 追加一轮：把原文抽成摘要或原子事实，往往带来负收益（negative net value）。  
- 只用 BM25 和 grep 的 harness，跟经典 memory 结构无明显差异，已经基本解决了准确率问题。  
此前我们给 agent 做「memory」，只是一种类比：想到什么好就加什么结构，原文反而被忽视了，没有人从第一性原理推过一遍  
  
如果简单一点呢？ 直接在原始历史上建立检索层，时间是自然标度，让 agent 在检索层上进行检索  
这就是我们开源的 Obelisk。 在我最新的 talk 《Agent 如何用历史做 research：为什么 agent history 应该成为 memory infra》 里，我先展示了只做这一层检索，agent 就能做到什么：  
「上次为什么这么写」，agentic research 可以直接得到答案：直接丢给 agent 去查，比如：「上次 auth bug 最后到底改了哪些文件，为什么这么改」  
跨整月历史的自主复盘，且结论都带着出处：我让 Codex 用 Obelisk 回顾了 Obelisk 自己一个月的开发过程，它生成的复盘里，每一条结论都直接定位到了具体的 session、message 和 workflow  
  
而 Obelisk 本身的实现极其克制：  
接入 11 个 Harness 的原始历史：覆盖 Claude Code、Codex、GitHub Copilot、Kiro、OpenCode、Kimi Code、DeepSeek Harness、Hermes Agent、OMP、Pi、ZCode。  
SQLite + FTS5 + JS Query Runtime（CodeAct）：不把历史整理成静态文档或纯自然语言摘要，也不设计固定的查询 API，而是以 skill 为载体，让 agent 现场写 JS 查询脚本去跑  
  
在这个 talk 的后半段，我也顺着第一性原理继续往下推了一步：对朴素检索继续 Scale 呢？  
当历史规模上来之后，单靠 full text search 命中范围有限、单次检索结果无法跨 session 复用、且第一轮命中会带偏后续检索倾向。 因此在检索层之上探索了第二阶段（Nowledge Mem Super Graph），一些结论：  
- View 的优势不是更准确的召回，而是赢在读得少（token -38%）：让 agent 先看到全局分布、知道信息是否完全，再花推理预算，往下又能随时落回 Obelisk 的原始历史。  
- 如果 Memory 会过期，不如没有 Memory：通过按时间频率分层的 live-update（新结论不直接覆盖旧结论、局部更新），让一部分理解活在上下文窗口之外，解决长程 research 的 context rot。  
  
归根结底是三件事：  
- 保留追溯回原始历史的路径，原文不能被丢掉  
- 值得花算力维护一层 graph view  
- Memory 应当更好有利于 agent 进行 research，no negative effect  
  
ppt: [github.com/tommy0103/talk…](https://github.com/tommy0103/talks/tree/main/2026-11-15-coscon26-magic)  
  
obelisk: [github.com/tommy0103/obel…](https://github.com/tommy0103/obelisk)  
nowledge-mem: [mem.nowledge.co](https://mem.nowledge.co)

![Image](<Yuu%20(@QuantumTransf).assets/image-01.jpg>)
![Image](<Yuu%20(@QuantumTransf).assets/image-02.jpg>)
![Image](<Yuu%20(@QuantumTransf).assets/image-03.jpg>)
![Image](<Yuu%20(@QuantumTransf).assets/image-04.jpg>)