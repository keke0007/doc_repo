---
author: "Jerry Fang"
handle: "@Jerry_fsy"
source: "https://x.com/Jerry_fsy/status/2120828559111676004"
date: 2026-11-12T10:38:26.000Z
type: tweet
likes: 123
reposts: 19
replies: 6
bookmarks: 107
views: 5514
---
# Jerry Fang (@Jerry_fsy)

如果你想转 AI Agent 方向，或者在准备相关面试，下面这30个术语是我认为优先需要理解的。  
  
我按“基础 → Agent → 工程落地”做了整理：  
  
基础层  
1.LLM：用海量文本训练的大语言模型  
2.Token：模型处理文字的最小单位，计费按它算  
3.上下文窗口：模型一次能看到的最大 token 数  
4.Prompt：你给模型的输入和指令  
5.幻觉：模型一本正经地胡乱输出  
6.Embedding：把文字变成向量，语义近的向量也近  
7.多模态：同时处理文字、图片、音频  
8.推理模型：先一步步想（CoT）再回答  
9.微调：用自己的数据继续训练模型  
10.RLHF：用人类反馈让模型更符合偏好  
  
Agent 层  
11. Agent：能自己规划、调工具、循环执行直到完成任务  
12. Tool Calling：模型输出调用指令，程序去执行  
13. MCP：接入工具和数据的统一协议  
14. Workflow vs Agent：一个走固定流程，一个自己决定下一步  
15. Multi-Agent：多个 Agent 分工协作  
16. Memory：让 Agent 记住历史和用户信息  
17. Planning：把大任务拆成步骤  
18. Agent Skills：把能力打包成说明加脚本，按需加载  
19. Computer Use：像人一样操作电脑和浏览器  
20. A2A：Agent 之间通信协作的协议  
  
工程落地层  
21. RAG：先检索资料，再基于资料回答  
22. 向量数据库：存和查 Embedding，RAG 的底座  
23. Context Engineering：设计上下文放什么、怎么放  
24. Prompt Caching：缓存重复前缀，省钱又提速  
25. Sandbox：给 Agent 跑代码的隔离环境  
26. Eval：用一组 case 系统衡量效果  
27. LLM-as-Judge：让大模型给输出打分  
28. Tracing：记录 Agent 每一步，方便定位问题  
29. Guardrails：输入输出的安全护栏  
30. Harness：模型外面那层工程，管工具、上下文、循环和重试。同一个模型配不同 Harness，效果能差很远