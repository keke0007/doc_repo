---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2095330467000328632"
date: 2026-09-02T16:49:01.000Z
type: tweet
likes: 169
reposts: 36
replies: 23
bookmarks: 204
views: 72032
---
# meng shao (@shao__meng)

CS 329Z: Engineering AI Agents  
Stanford / Fall 2026 @stanfordnlp  
[cs329z.stanford.edu](https://cs329z.stanford.edu)  
  
课程定位：从"模型"到"系统"的工程学  
覆盖：简单 LLM 流水线 → 复合 AI 系统 → 自主 Agent。三位讲师的背景也高度互补：  
@Diyi_Yang（斯坦福 NLP 教授，人机交互与社会计算方向）  
@michaelryan207（DSPy 核心贡献者，自动评估 AutoMetrics 作者）  
@jyangballin（SWE-agent / SWE-bench / SWE-smith 作者，软件工程 Agent 领域最重要的研究者之一）  
  
# 课程主线：三大工程挑战  
贯穿全课的三个核心问题——分解（decomposition）、数据（data）、评估（evaluation）。11 周的内容基本围绕这三条线展开，可以分为五个模块：  
  
模块 1：构建基元（Week 2–3）  
· LLM 作为构建材料：API/SDK（litellm）、结构化输出、约束生成、解码策略、test-time compute、上下文工程、模型选型与成本/延迟权衡  
· RAG：embedding、向量库、分块策略、混合检索、cross-encoder 与 ColBERT 后期交互  
· 工具调用：函数调用 API、MCP（Model Context Protocol）、工具设计、代码沙箱、错误处理与重试  
  
模块 2：框架与设计模式（Week 3–5）  
· 框架层：DSPy（signature / module / optimizer）、LangChain/LangGraph、LlamaIndex，重点是"框架抽象了什么 vs. 你手写了什么"  
· 设计模式：workflow vs. agent 的分类学，五种可组合 workflow 模式，ReAct / plan-and-execute / reflection，"scaffold（脚手架）本身就是设计决策"  
· 记忆架构：短期/长期记忆、记忆作为工具动作、文件系统作为外化记忆、跨 Agent 记忆（MemGPT、Mem0、Generative Agents）  
· 多 Agent 系统：编排模式、handoff 与状态传递，以及一个很有态度的对照阅读——既读 AutoGen，也读《Why Do Multi-Agent LLM Systems Fail?》和 Neubig 的《Don't Sleep on Single-agent Systems》  
  
模块 3：优化（Week 5）  
· 从提示词到微调的全景：GEPA、MIPROv2、OPRO、TextGrad（提示优化）；LoRA/QLoRA、蒸馏、RLHF/DPO（权重优化）；test-time scaling（推理算力）  
· 核心问题是决策框架：什么时候优化 prompt、什么时候优化 weights、什么时候堆推理算力  
  
模块 4：数据与评估（Week 6–8）——最有分量的部分  
· 数据：trace、demonstration、feedback 三类数据；训练数据 vs. 评估数据；数据飞轮；合成数据；从 Agent 轨迹构建数据集（SWE-smith）  
· 评估基础：为什么 eval 难；4 元组框架（request / environment / stopping criteria / scorer）；好 benchmark 的性质；tinyBenchmarks  
· 评估基础设施：三类 grader、LLM-as-judge 的 prompt 设计与已知偏差、pairwise vs. pointwise、非确定性指标 pass@k vs. pass^k、harness 设计  
  
模块 5：安全与前沿（Week 8–11）  
· 安全：工具访问的隐私风险、prompt injection（含间接注入）、红队、沙箱与权限模型、输出护栏、human-in-the-loop  
· Coding Agent：SWE-agent、Claude Code、OpenHands 的端到端架构对比  
· 主动式 Agent：从 reactive 到 proactive，General User Models（GUM）、Next Action Prediction，以及"Agent 何时应主动、何时应等待"的 mixed-initiative 问题  
· 开放问题：多模态/web/计算机使用 Agent、科学 Agent、长时运行架构、生产可观测性（tracing、monitoring、成本管理）  
  
# 作业设计：一手建、一手评  
  
HW1：从零构建 Agent 系统（10%） 给定论文库，构建能检索并推理回答科学问题的 Agent。  
· Part A：只用 litellm 手写 RAG + 工具调用 + ReAct 式 Agent 循环  
· Part B：用 DSPy 重建关键组件，并反思框架抽象了什么  
  
HW2：评估一个 Agent（10%） 给定一个预构建 Agent，设计完整评估套件：代码型 grader、至少一个 LLM-as-judge、用 4 元组框架构建 benchmark 任务、错误分析。

![Image](<meng%20shao%20(@shao__meng).assets/image-01.jpg>)

> **Diyi Yang (@Diyi_Yang)**
> 
> This fall @michaelryan207 @jyangballin and I are teaching a new course CS329Z "Engineering AI Agents" on how to build AI Agents from scratch. Come join us and learn how to build them 🤖
> 
> ![Image](<meng%20shao%20(@shao__meng).assets/image-02.jpg>)