---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2097115650242248898"
date: 2026-09-07T17:43:41.000Z
type: article
---
# MIT 多模态 AI 课程 2026 版全套上线：视频免费在线学，从融合对齐一路讲到推理智能体与触觉嗅觉

*By meng shao (@shao__meng)*

![Banner](<MIT%20多模态%20AI%20课程%202026%20版全套上线：视频免费在线学，从融合对齐一路讲到推理智能体与触觉嗅觉.assets/image-01.jpg>)

![图像](<MIT%20多模态%20AI%20课程%202026%20版全套上线：视频免费在线学，从融合对齐一路讲到推理智能体与触觉嗅觉.assets/image-02.jpg>)

MIT《How to AI (Almost) Anything / Multimodal AI》2026 春季课程

课程地址：[mit-mi.github.io/mmai-course/spring2026/](https://mit-mi.github.io/mmai-course/spring2026/)

在线视频集合：[www.youtube.com/playlist?list=PLWzwL390pi04](https://www.youtube.com/playlist?list=PLWzwL390pi04)

> **9月8日 (@pliang279)**
> 
> Sharing lecture videos for the **How to AI (Almost) Anything/Multimodal AI** course I taught at MIT in spring 2026.  
>   
> This course became quite a hit the last time I shared it in spring 2025. Spring 2026's updated version contains updated topics on multimodal agents, reasoning,
> 
> ![Image](<MIT%20多模态%20AI%20课程%202026%20版全套上线：视频免费在线学，从融合对齐一路讲到推理智能体与触觉嗅觉.assets/image-03.jpg>)

## 课程介绍

这门课正式名称为 **Modeling: MultiModal AI**（课程号 MAS.S60 / 6.S985），由 [@medialab](https://x.com/@medialab) 的 **Paul Liang** [@pliang279](https://x.com/@pliang279) 主讲，2026 年春季学期授课。Liang 是多模态机器学习领域最具影响力的青年研究者之一，他和 Morency、Salakhutdinov 合著的《Multimodal Machine Learning: A Survey and Taxonomy》以及后续的 Multimodal Foundations 系列教程，几乎是这个领域每个人入门的必读材料。这门课可以看作他把多年研究体系（表示、融合、对齐、翻译、共学习六大挑战框架）整理成的一套完整教学。

课程副标题 "How to AI (Almost) Anything" 点明了它的目标：**不把 AI 局限在语言和视觉，要教学生如何把深度学习和基础模型的方法论迁移到任意数据形态上，**传感器、医疗数据、音乐、触觉、嗅觉、味觉。这正是课程区别于一般“视觉-语言大模型课”的核心定位。

授课阵容值得一提：除了 Liang，还有三位联合授课教师各带一个应用领域——运筹学名家 **Dimitris Bertsimas**（负责最优化与规范性建模）、城市交通学者 **Jinhua Zhao**（负责城市与交通）、制造工程专家 **Sang-Gook Kim**（负责制造）。

## 课程的技术主线

课程官网明确了六大核心主题，实际上是多模态学习的完整技术栈：

1. **异构数据的表示与融合：**不同模态的数据结构、噪声、时间尺度完全不同，如何统一建模
2. **跨模态对齐：**找到“这张图的那个区域”和“这句话的那个词”之间的对应关系
3. **多步推理：**在多模态之上做链式思考与决策
4. **多模态生成**：跨模态翻译、摘要与内容创作
5. **跨模态迁移**：把高资源模态（文本、图像）的知识迁移给低资源模态（如医疗、触觉数据）
6. **安全、伦理与对齐部署**

## 课程结构逐段解读

课程从 2 月初到 5 月中旬，每周两次课，结构上清晰地分为四个阶段：

**第一阶段：基础（第 1–3 周）。** 从课程引言、多模态数据集概览讲起，接着是 PyTorch 实操教程，然后进入“数据与异构性”；先建立“不同模态的数据在结构上到底差在哪”的直觉（这是 Liang 一贯的教学切入点：异构性是多模态一切困难的根源），再过一遍通用模型架构（序列、卷积、集合、图）。

**第二阶段：融合与对齐（第 3–5 周）。** 这是多模态学习的经典内核。从早期/晚期融合、可解释融合，讲到高阶交互与 Kosmos-2、Chameleon、MM1 等融合模型；对齐部分覆盖 CLIP、DINO，以及 2024 年很有影响力的 **Platonic Representation Hypothesis**（“柏拉图表示假说”，不同模态的模型最终会收敛到相似的底层现实表示），这个选材相当前沿。随后进入大型多模态模型（LMM）：Transformer 预训练与微调、LoRA、MoE、量化、LLaVA，并配有一节 Colab 实操课。

**第三阶段：生成与推理（第 6–7 周 + 第 9–10 周）。** 生成部分覆盖 VAE、扩散模型、流匹配（Flow Matching）、DiT、SD3，以及可控生成；也就是当下文生图/文生视频的主流技术路线。期中考试后进入本轮更新的重头：**多模态推理**（RLHF、DeepSeek-R1、DPO，后训练与推理时扩展的最新范式）、**可解释推理**​、**多模态交互与智能体**（VisualWebArena、Mind2Web、OpenVLA 等 GUI/具身智能体基准），以及**跨模态迁移**（PaLM-E、LLaVA-Med，展示如何用大模型撬动医疗等低资源领域）。

**第四阶段：应用与前沿（第 11–14 周）。** 四节应用课分别对应四位授课教师的研究领域：制造（多传感器融合）、设计（多模态智能体）、城市（时空融合）、交通（自动驾驶）；外加 Bertsimas 的“规范性建模”（预测 + 优化 + 因果反事实，把 ML 和运筹学接起来，这是 MIT 这门课很有特色的一讲）。最后两讲是最能体现“前沿”的地方：

- **Self-evolving AI（自我进化的 AI）**​：持续学习、自我进化与优化，阅读材料包括 AlphaEvolve 和 CORAL；对应 2025–2026 年“AI 自主改进 AI”的研究热潮。
- **AI for new senses（AI 与新感官）**​：触觉、嗅觉、味觉，阅读材料包括 SmellNet、AromaGen、OpenTouch。这一讲是课程标题 "Almost Anything" 的终极兑现；Liang 所在的 MIT 多感官智能组（MIT-MI）本身就在做嗅觉网络（GitHub 上就有 SmellNet 仓库），是建立在真实研究管线上的内容。

第 15 周以学生项目展示收尾。

## 2026 版相比 2025 版更新了什么

对比 2025 春季（当时课程号 MAS.S62，名为 AI4AI）的课程表，更新幅度是实质性的：

- **多模态推理成为独立模块**​。2025 版只在课程末尾用一讲带过强化学习与多步推理；2026 版在期中之后专门展开 RLHF/DPO/DeepSeek-R1 的后训练与推理范式，并新增了“可解释推理”一讲；这正对应推理模型（o 系列、R1）爆发这一年。
- **智能体（Agents）从一讲扩成一条线**​。2025 版只有 "Interactive agents" 一讲；2026 版新增了专门的 Agents 实操教程（智能体管线、智能体评测），交互课的阅读材料也换成了 VisualWebArena、Mind2Web、OpenVLA 这批智能体基准与具身模型。
- **新增 Self-evolving AI**​，2025 版完全没有这个主题。
- **新增触觉、嗅觉、味觉等新模态**​，2025 版止步于视觉、音频等常见模态。
- **应用课从无到有**​。2025 版基本是纯方法论课程；2026 版加入制造、设计、城市、交通四节应用课，以及“规范性建模”这种 ML × 运筹学的交叉内容；这也是课程从 "AI4AI" 更名为 "Modeling" 的原因，它现在是一门面向真实领域建模问题的课。

**不变的是其教学哲学**​：先讲数据的异构性，再讲融合与对齐，最后到大模型与生成；也就是“从第一性原理出发，而不是从某个具体模型出发”。

## 五、项目产出：这门课的“战绩”

期末项目是真实的研究起点。往届项目页上的报告可以说明质量：2025 年的三篇（ASL 情绪多模态数据集 EmoSign、两篇多模态推理诊断评估）发表在 NeurIPS 2025 Workshop；2024 年的 MMoE（多模态交互专家混合）发表在 EMNLP；更早的产出涵盖 ECCV、ICCV、ACL、AAAI 等。项目主题横跨手语理解、自杀意图的行为标记、教育演示理解、视觉问答的可解释性与对抗攻击；很多就是“把 AI 用到别人没想过的地方”的直接示范。

## 怎么学这门课

YouTube 播放列表公开了约十讲有视频的课程（引言、数据异构性、融合、对齐、大型多模态模型、多模态生成、现代生成式 AI、多模态推理、交互智能体、跨模态迁移、自我进化 AI、新感官），每讲都配公开的讲义 Slide；应用类的客座内容（健康、设计、制造、城市、交通）则只放讲义不放视频。全部材料和课程表见课程网站：[mit-mi.github.io/mmai-course/spring2026](https://mit-mi.github.io/mmai-course/spring2026/)。

**适合谁**​：它对学习者有一定要求，需要基本的机器学习和深度学习基础（会用到 PyTorch），但不需要多模态背景。最合适的定位是：**系统性地建立多模态 AI 的完整知识框架，并且了解 2025–2026 年最前沿的方向（推理模型、智能体、自进化、新模态）**​。相比零散读论文，这门课的价值在于 Liang 把整个领域组织成了一条有逻辑的主线：异构性 → 融合 → 对齐 → 大模型 → 生成 → 推理 → 智能体 → 迁移 → 新模态，每一环都配有该方向最经典的论文作为阅读材料。