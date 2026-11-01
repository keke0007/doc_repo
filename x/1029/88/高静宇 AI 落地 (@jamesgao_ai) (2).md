---
author: "高静宇 | AI 落地"
handle: "@jamesgao_ai"
source: "https://x.com/jamesgao_ai/status/2114293704009912351"
date: 2026-10-25T09:51:15.000Z
type: tweet
likes: 54
reposts: 5
replies: 27
bookmarks: 41
views: 3303
---
# 高静宇 | AI 落地 (@jamesgao_ai)

企业落地 Agent，要不要给每个部门、每个岗位都配一个智能体？  
  
客服一个，商品一个，订单一个，营销一个，再加一个总调度。架构看起来分工清晰，但也可能把原来部门之间的沟通成本，一起复制进系统。  
  
Anthropic 的两篇文章，讨论了这个问题。  
  
Commerce Agent 指南建议用一个 Agent 配合 Skills 和工具，贯穿搜索、比较、购买和售后。这些环节虽然属于不同业务功能，却需要共享用户偏好、订单和对话状态。  
  
另一篇文章则指出，一些企业让各团队分别建设 Agent，再搭一个中央编排层，结果 Agent 架构成了组织架构的复制。  
  
这给企业规划智能体提供了一个值得重视的原则：  
  
先看完整业务任务和上下文，再决定建几个智能体。  
  
比如，企业想用 AI 改善沉睡会员唤醒。  
  
按照现有岗位，可以建设数据分析 Agent、营销策划 Agent、文案 Agent、活动执行 Agent。  
  
但实际工作中，目标人群为什么被选中、预算是多少、优惠力度怎么确定、此前活动效果如何，这些信息需要贯穿分析、策划和执行。按岗位拆开以后，各个 Agent 都可能需要重新理解这套背景。  
  
另一种设计是：围绕会员唤醒这个完整场景建设智能体，把分析方法、策划规则、文案规范沉淀为 Skills，通过工具连接客群、优惠券和触达能力。  
  
业务能力仍然分工建设，用户的任务和上下文可以保持连续。  
  
所以，企业决定是否单独建设一个智能体时，值得先问：  
  
它是否服务一个清楚的业务目标？  
它需要的上下文，能否相对独立地组织起来？  
它能否承担一段完整工作，并交付可验收的结果？  
它与其他智能体之间，是否需要频繁传递背景、反复协调？  
  
这些问题，比“公司里有没有这个部门、这个岗位”更适合作为划分依据。  
  
岗位可以提供参考。如果岗位边界恰好与任务和上下文边界一致，当然可以采用。但不能因为企业有十个岗位，就默认需要十个 Agent。  
  
企业落地 AI，也有机会重新组织工作。照着现有组织结构配置一遍智能体，可能会错过这次机会。  
  
两篇原文：  
[claude.com/resources/arti…](https://claude.com/resources/articles/the-anatomy-of-effective-commerce-agents)  
  
[claude.com/resources/arti…](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop)

> ![Link card preview](<高静宇%20AI%20落地%20(@jamesgao_ai)%20(2).assets/image-01.png>)
> 
> 🔗 [**A guide to the anatomy of effective commerce agents**](https://t.co/y5vvB6SvTn)
> 
> _From claude.com_