---
author: "victor-wu.eth"
handle: "@victor_wu"
source: "https://x.com/victor_wu/status/2112841337695543458"
date: 2026-10-20T12:45:56.000Z
type: tweet
---
# victor-wu.eth (@victor_wu)

补充说明，我个人对哪种方案没有特殊喜好，我只是反感无论什么时候都把 RAG 类为第一选项。其实我自己在几个月前设计的知识库方案并不是原版 llm-wiki，甚至有种【劣化版】味道。  
1. 文档入库统一经过处理成 md  
2. 丢给 llm 进行分析，结合我们上传时的设置，给文章文章生成frontmatter，包括但不限于 topic、visibility、allowed_groups，大体分为文章的章节信息和权限信息，再把章节信息丢到 index 里  
3. 用户输入问题时，第一步先根据用户的问题去 index 找信息，第二步直接把找到的对应 topic 和对应的 md 直接丢给 llm，再据此回答。（当然前提已经检验过权限属性了）  
---  
为何用这种基于 index 的知识库方案，很简单，因为我们知识库就是给企业内部用的，每份文档内容时完整且清晰的，而且文档总量和单个文档的内容都不大，1m 的上下文完全吃得下。我当时想把文章根据 topic 直接切成若干文件，但是跑了 eval 发现正确率不如不切直接完整一篇塞给 llm。对我们来说，在内容有限地情况下，准确率和权限才是第一指标。  
  
arXiv 上有一篇SearchWiki: Learning to Build and Navigate Knowledge Wikis for Active Information Seeking [arxiv.org/abs/2608.29953](https://arxiv.org/abs/2608.29953) 就是我那时候这个方案地威力加强版  
它把语料自动编译成一个三层层次化、可导航的 wiki：  
Document overviews（文档概览层）：每篇文档的摘要 + 可选的章节子目录  
Topics（跨文档主题页）：把相关概念/主题抽成页面，并做跨文档链接  
Sources（源页面层）：细粒度的页面级记录（用于最终验证和引用）  
  
结论是哪怕SearchWiki 不进行后训练，也要比同等模型地 RAG 准确率更高  
  
仅以此贴强调，条条大路通罗马。

> **victor-wu.eth (@victor_wu)**
> 
> 来更新 AI 程序员面试心得  
>   
> 1. 看简历已经无法进行有效筛选了，全是 ai 润色过的，很难拉开差距  
> 2. 90% 从业者干的是知识库和智能客服，心疼客服行业三秒钟  
> 3. 基于 1 和 2，我们其实更看重一个人如何使用 agent 和对 agent 行业的追踪，我天天面试看到那些简历写 RAG RAG