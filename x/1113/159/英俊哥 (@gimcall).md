---
author: "英俊哥"
handle: "@gimcall"
source: "https://x.com/gimcall/status/2121189282203439313"
date: 2026-11-13T10:31:49.000Z
type: tweet
---
# 英俊哥 (@gimcall)

想转 AI Agent 开发，或者准备相关面试，这 30 个术语最好先搞明白。  
  
只知道 Agent、RAG、MCP 这些名字还不够，至少要能解释清楚它们分别解决什么问题。  
  
我按照基础、Agent、工程落地三个层次整理了一遍。  
  
基础层：先搞懂模型是怎么工作的  
  
LLM：用海量文本训练的大语言模型  
Token：模型处理文字的基本单位，API 通常按它计费  
上下文窗口：一次请求中模型能够处理的最大 Token 数  
Prompt：输入给模型的内容与指令  
幻觉：模型输出看起来很合理，内容却可能完全错误  
Embedding：把文字转换成向量，让语义相近的内容更容易被检索到  
多模态：让模型处理文本、图像、音频等不同类型的信息  
推理模型：能够通过多步骤推理处理复杂问题  
微调：拿特定数据继续训练已有模型  
RLHF：用人类反馈优化模型行为  
  
Agent 层：让模型从回答问题变成执行任务  
  
Agent：自己规划、调用工具，并根据执行结果继续完成任务  
Tool Calling：模型决定调用什么工具，程序负责真正执行  
MCP：让 Agent 连接工具和数据的标准协议  
Workflow vs Agent：一个按固定流程走，一个可以动态决定下一步  
Multi-Agent：多个 Agent 分工合作  
Memory：保存历史信息和用户偏好，供后续任务使用  
Planning：将复杂目标拆成可执行的步骤  
Agent Skills：把操作说明和脚本打包成可复用能力，按需加载  
Computer Use：让 Agent 直接操作桌面、网页和浏览器  
A2A：让不同 Agent 之间能够通信、交接和协作  
  
工程落地层：决定 Agent 能不能真正用起来  
  
RAG：先从外部资料中检索，再结合检索内容回答  
向量数据库：负责存储、查询 Embedding，是 RAG 常用组件  
Context Engineering：决定上下文里装哪些信息，以及如何组织  
Prompt Caching：重复输入部分直接利用缓存，省 Token 成本和时间  
Sandbox：隔离 Agent 执行代码的环境  
Eval：用测试案例衡量系统的实际表现  
LLM-as-Judge：让大模型对生成结果进行评价  
Tracing：追踪 Agent 每一步执行过程，方便定位故障  
Guardrails：限制不安全或不符合要求的输入输出  
Harness：包在模型外面的工程系统，负责工具、上下文、执行循环、异常处理和重试  
  
尤其要理解 Harness。  
  
同一个模型，底层的工具调用、上下文管理和重试机制不同，最终表现可能差得很远。