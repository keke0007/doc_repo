---
author: "九原客"
handle: "@9hills"
source: "https://x.com/9hills/status/2122600789420232907"
date: 2026-11-17T08:00:38.000Z
type: thread
likes: 320
reposts: 60
replies: 25
bookmarks: 422
views: 21488
---
# 九原客 (@9hills)

本体论应用到 Data Agent（国内一般叫智能问数、数据智能体）的项目推荐：  
  
EvoOntology 是唯一做了Bench、消融对比，而且实现 Ontology 进化（就是用 Agent读 Trace，迭代 Ontology 结构），毕竟要发论文，实验很重要。  
  
本体落地 Data Agent 落地的办法：1. 建设评估集 2. 自动构建本体结构 3. 人工 Review 优化 4. 根据Bad case 自动进化

![Image](<九原客%20(@9hills).assets/image-01.jpg>)

---

1. **[microsoft/Ontology-Playground](Ontology-Playground/)** —— 微软面向 **Microsoft Fabric IQ** 打造的纯静态可视化本体探索、设计与教学试验场  
  
2. **[fuyuxiang/ontology](ontology/)（元枢本体 · Yuanshu Ontology）** —— 对标 Palantir Foundry/AIP 理念的**企业级可运行本体（Operational Ontology）全栈平台**  
  
3. **[NVIDIA/auto-ontology](auto-ontology/)** —— 英伟达开源（BEAVER 榜单 #1 / BIRD-Interact #2）的**企业级自动化数据本体编译与高可靠 Text-to-SQL 平台**  
  
4. **[gogoymh/auto-ontology-mcp](auto-ontology-mcp/)** —— 衍生自 `NVIDIA/auto-ontology` 的**无模型托管（BYO-Client-LLM）轻量级 MCP 语义层与 Obsidian 结构化知识库服务器**  
  
5. **[ruc-datalab/EvoOntology](EvoOntology/)**（配套论文：[EvoOntology-2609.15779.pdf](EvoOntology-2609.15779.pdf)） —— 中国人民大学 Datalab 开源的**面向异构数据 Agent 的轨迹驱动自进化本体层（Self-Evolving Ontology Layer）**