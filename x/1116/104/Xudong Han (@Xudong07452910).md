---
author: "Xudong Han"
handle: "@Xudong07452910"
source: "https://x.com/Xudong07452910/status/2121553660358365333"
date: 2026-11-14T10:39:43.000Z
type: tweet
likes: 133
reposts: 23
replies: 10
bookmarks: 163
views: 8372
---
# Xudong Han (@Xudong07452910)

之前推荐过几篇 LangChain 的 Agent Eval 文章，今天再推荐一篇 Anthropic 新发的《Building strong safety evaluations》，讲怎么把安全评测做扎实，做 Eval 的朋友都值得看看。  
  
LangChain 那几篇关心 Agent 有没有把任务做成，这篇关心模型在某个情境下有没有做对的事，两类 Eval 正好互补。  
  
有两点特别实用：  
  
1. 先想清楚到底在测什么。安全评测最顺手的指标是拒绝率，可用户处在危机里时，直接拒绝本身就是失败。Anthropic 改测「恰当回应率」，由领域专家写好模拟用户的目标、手法和语气，再让另一个 Claude 扮演用户多轮往下聊，整段对话只要有一轮出错就算不通过。同一主题还会成对放有害请求和正常请求，「帮了不该帮的」和「拒了不该拒的」一起测。  
  
2. 别太信一个数字。LLM 评分器要先和人工标注对齐，一致率要接近专家彼此之间的水平；分析方案要在看结果前写好；复杂的多轮评测至少 100 条，单轮至少 1000 条，还要报置信区间。文中的例子是，Opus 5 和 Opus 5.5 在一项多轮防诈骗评测上分别是 98% 和 94%，但这个差异在统计上并不显著。  
  
文末给了一个最低要求：先写下要测的概念，先拿人工标注校准评分器，报告结果时带上不确定性。  
  
📎

> ![Link card preview](<Xudong%20Han%20(@Xudong07452910).assets/image-01.jpg>)
> 
> 🔗 [**Building strong safety evaluations**](https://t.co/EYa0KnN87j)
> 
> _From anthropic.com_