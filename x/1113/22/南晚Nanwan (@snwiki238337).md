---
author: "南晚Nanwan"
handle: "@snwiki238337"
source: "https://x.com/snwiki238337/status/2120859404262121738"
date: 2026-11-12T12:41:00.000Z
type: thread
likes: 17
reposts: 3
bookmarks: 12
views: 1367
---
# 南晚Nanwan (@snwiki238337)

推荐两个关于 agentic memory 的视频，讲的挺不错的，推荐看看，第一个是：[youtube.com/watch?v=9ta1ca…](https://youtube.com/watch?v=9ta1cawyVsU)，第二个是：[youtube.com/watch?v=9ta1ca…](https://youtube.com/watch?v=9ta1cawyVsU)  
  
内容包含了目前基本主流的 agent 记忆的方案，讲解的还是较为详细的适合看看学习。包括摘要记忆、缓冲记忆、向量记忆、实体记忆等等的几种主流的代表记忆范式。  
  
第二个视频主要是关于这个跨会话的长期记忆机制、记忆路由，以及目前的生产级开源框架相关的东西。  
  
挺不错的视频，可以学习一下。

> ![Link card preview](<南晚Nanwan%20(@snwiki238337).assets/image-01.jpg>)
> 
> 🔗 [**Agentic Memory Part 1: What is Agentic Memory & Types**](https://t.co/Oy9n6qQofC)
> 
> _From youtube.com_

---

AWS 发布了一套可插拔 agent 参考架构，有点像 dsh 的插件化理论。  
  
这篇文章要解决的核心工程是为什么很多团队做出来的多 agent 原型演示时很顺利，但是生产环境容易出问题。  
  
原因往往在于做原型时大家习惯把模型名称、子 agent 的调用地址、数据库查询逻辑全部写在业务代码里。  
  
等到上生产需要控制成本换小模型、过安全合规审查或者排查线上故障时，修改一些地方容易导致一些不可言说的问题出现。  
  
AWS 这套方案的核心思路，就是把智能体系统拆成七个插件化独立的层，层与层之间只通过固定的接口契约和配置中心通信，确保换掉任何一层的组件都不会影响其他模块。  
  
根据他们给出的电商售后参考实现，可以很清楚地看到这套解耦思路是怎么一步步落地的。  
  
第一步是把模型和业务逻辑拆开。在底层的数据和推理层，系统没有让后面的 Agent 直接去绑定某个厂商的 SDK，而是通过 LiteLLM 搭了一层统一推理接口，并把当前启用的模型 ID、退货政策知识库 ID 等全部注册到配置中心（SSM）。  
  
后续无论是想把简单任务切给便宜的轻量模型，还是把推理后端从 AWS 原生服务换成第三方托管平台，只需要在配置层改一个字符串，上层的智能体代码一行都不用动。  
  
第二步是在编排层解决总 agent 与子 agent之间的硬编码耦合。如果总 Agent 代码里写死了各个子 Agent 的地址，每次业务部门新加一个专职 Agent 都得去改总 agent 代码。  
  
这里的做法是引入一个注册表和统一工具网关。各个专职子 Agent 部署后自己往注册表里登记能力，总 Agent 在收到用户请求时，通过运行时 harness 动态去注册表里发现可用的子 Agent 并通过 A2A 协议派发任务。  
  
而子 Agent 去查数据库或知识库的底层工具调用，则全部收口到统一网关做精细的权限校验。这样一来，子 agent 的增删改就变成了热插拔。  
  
第三步是怎么在不污染业务代码的前提下，把隐私和一些比如故障排查的善后工作加上去。  
  
在真实售后场景里，底层工具从数据库查出订单详情后，里面往往带着用户的真实邮箱、电话和家庭住址，如果直接把这些原始数据塞回给总控大模型，很容易引发隐私泄露或提示词注入。  
  
这套架构是在统一网关上挂了一个拦截器，就是说工具返回的数据在送达大模型之前，先在网关被拦截下来，自动过一遍把敏感隐私字段打码脱敏，再交给上层继续推理。  
  
同时，由于所有跨层调用都遵循统一的接口，分布式追踪能把一次对话从前端入口、总控分发、网关拦截到数据库查询的耗时和 Token 账单完整链接起来。  
  
一旦出现了某些问题的话，对于问题的排查就会很简单。顺着链路一眼就能看清到底是知识库没召回、模型算错了日期，还是说是别的问题。  
  
github：[github.com/aws-samples/sa…](https://github.com/aws-samples/sample-pluggable-agentic-ai-framework)  
  
原文：[aws.amazon.com/cn/blogs/machi…](https://aws.amazon.com/cn/blogs/machine-learning/pluggable-agentic-ai-framework-a-seven-layer-composable-enterprise-agentic-ai-reference-architecture-on-amazon-bedrock-agentcore/)

> ![Link card preview](<南晚Nanwan%20(@snwiki238337).assets/image-02.jpg>)
> 
> 🔗 [**GitHub - aws-samples/sample-pluggable-agentic-ai-framework: Pluggable Agentic AI Framework is a...**](https://t.co/TZgLku2wNo)
> 
> _From github.com_