# GitHub 项目分类汇总

从 `tbd` 文件夹的全部笔记文件中提取到的 GitHub 项目，去重后共 **419** 个（另有 2 个链接已失效）。

每条的格式为：`仓库名 — 官方简介 · ⭐ Star 数 · 来源：提到它的笔记文件`。同一仓库在多个文件出现时只保留一条，来源处会列出全部文件。Star 数为抓取当天的数值。

| 分类 | 数量 | 一句话说明 |
| --- | --- | --- |
| [G. 学习教程、课程与书籍](#cat-g) | 58 | 从零实现、教程、课程、教材、方法论文章集 |
| [C. Agent Skills、插件与 MCP 扩展](#cat-c) | 52 | 技能库、技能管理器、插件市场、MCP server、给 agent 加能力的工具包 |
| [A. AI Agent 框架与运行时](#cat-a) | 38 | 构建 agent 的框架、harness、运行时、多智能体编排引擎、agent 平台 |
| [L. 开发工具与 DevOps / 基础设施](#cat-l) | 36 | 版本管理、终端、git 工具、系统工具、K8s、监控、自托管基础设施、运维脚本 |
| [E. Agent 记忆、上下文与知识管理](#cat-e) | 30 | 长期记忆、上下文数据库、RAG、代码知识图谱、第二大脑、笔记与 wiki |
| [B. Coding Agent 客户端与工作台](#cat-b) | 25 | Claude Code / Codex / Cursor 等编码 agent 的 CLI、桌面端、IDE、并行 agent 工作台 |
| [D. Loop Engineering、Harness 工程与规范驱动开发](#cat-d) | 20 | 长程 loop/harness 工程方法、Ralph playbook、spec-driven development、code review 工作流 |
| [P. 生产力、办公与协作](#cat-p) | 20 | Office/文档处理、Markdown 与笔记应用、书签、项目与团队管理、聊天协作 |
| [H. Awesome 清单与资源合集](#cat-h) | 16 | awesome-* 榜单、项目导航、资源合集 |
| [F. Agent 可观测性、评估与治理](#cat-f) | 15 | 会话查看、轨迹与用量分析、telemetry、审计、评测、权限治理、成本控制 |
| [R. 语言学习与个人成长](#cat-r) | 14 | 英语/雅思学习、翻译词典、自学与认知类内容 |
| [I. 大模型基础：训练、推理与多模态](#cat-i) | 13 | LLM 训练/推理、强化学习、语音合成与识别、OCR、端侧模型、图像视频生成 |
| [M. 网络、代理与自托管服务](#cat-m) | 12 | 反向代理、LLM API 网关、流量看板、网络扫描与拓扑、订阅转换 |
| [O. 设计与可视化](#cat-o) | 11 | 设计系统、白板与思维导图、流程图/架构图工具、动画与可视化引擎 |
| [K. 数据、数据库与数据分析](#cat-k) | 10 | 数据库客户端/IDE、text-to-SQL 与 GenBI、数据加载与数据工程 |
| [N. 安全、抓包与逆向](#cat-n) | 10 | 渗透测试、抓包分析、Wi-Fi/网络攻击、逆向工程、安全能力库 |
| [Q. 硬件、IoT 与机器人](#cat-q) | 10 | 物联网平台与设备、智能家居、ESP32、无人机、工业监测、CAD |
| [S. 垂直行业应用](#cat-s) | 10 | 金融交易、电商、招聘求职、新闻资讯、GIS、行业解决方案 |
| [T. 浏览器自动化与数据采集](#cat-t) | 10 | 浏览器 agent、网页抓取、真实浏览器控制、爬虫工具 |
| [J. 提示词工程与 AI 应用](#cat-j) | 9 | 提示词管理/优化、通用 AI 聊天与助手应用、AI 应用示例集 |
| [失效链接](#cat-broken) | 2 | 已无法访问的仓库 |

<a name="cat-g"></a>

## G. 学习教程、课程与书籍（58 个）

从零实现、教程、课程、教材、方法论文章集

- [rasbt/LLMs-from-scratch](https://github.com/rasbt/LLMs-from-scratch) — Implement a ChatGPT-like LLM in PyTorch from scratch, step by step ⭐ 104,865 · 来源：0819
- [ByteByteGoHq/system-design-101](https://github.com/ByteByteGoHq/system-design-101) — Explain complex systems using visuals and simple terms. Help you prepare for system design interviews ⭐ 89,131 · 来源：0905
- [shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) — Bash is all you need - A nano claude code–like 「agent harness」, built from 0 to 1 ⭐ 76,662 · 来源：0820、0821、0905
- [microsoft/ai-agents-for-beginners](https://github.com/microsoft/ai-agents-for-beginners) — 18 Lessons to Get Started Building AI Agents ⭐ 74,563 · 来源：agent开发
- [microsoft/AI-For-Beginners](https://github.com/microsoft/AI-For-Beginners) — 12 Weeks, 24 Lessons, AI for All! ⭐ 68,447 · 来源：article
- [shanraisshan/claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) — from vibe coding to agentic engineering - practice makes claude perfect ⭐ 65,884 · 来源：0826、0909、0911、0913、link-3.txt
- [rohitg00/ai-engineering-from-scratch](https://github.com/rohitg00/ai-engineering-from-scratch) — Learn it. Build it. Ship it for others ⭐ 54,395 · 来源：0820、0909、link-3.txt
- [bojieli/ai-agent-book](https://github.com/bojieli/ai-agent-book) — 《深入理解 AI Agent：设计原理与工程实践》（李博杰 著）开源主仓库：全书正文、编译版 PDF 与按章配套代码 ⭐ 46,109 · 来源：0810、link-4.txt、link-5.txt
- [luongnv89/claude-howto](https://github.com/luongnv89/claude-howto) — A visual, example-driven guide to Claude Code — from basic concepts to advanced agents, with copy-paste templates that bring immediate value ⭐ 41,458 · 来源：0820、0909、0911、0913、link-4.txt
- [HandsOnLLM/Hands-On-Large-Language-Models](https://github.com/HandsOnLLM/Hands-On-Large-Language-Models) — Official code repo for the O'Reilly Book - "Hands-On Large Language Models" ⭐ 29,054 · 来源：0905
- [systemdesign42/system-design-academy](https://github.com/systemdesign42/system-design-academy) — If you want to become good at AI engineering & system design, join this newsletter 👇 ⭐ 28,587 · 来源：link-3.txt
- [harvard-edge/cs249r_book](https://github.com/harvard-edge/cs249r_book) — Machine Learning Systems ⭐ 28,195 · 来源：0823、link-3.txt
- [Vonng/ddia](https://github.com/Vonng/ddia) — 《Designing Data-Intensive Application》DDIA 第一版 / 第二版 中文翻译 ⭐ 23,590 · 来源：0824
- [2025Emma/vibe-coding-cn](https://github.com/2025Emma/vibe-coding-cn) — Vibe Coding 指南 一个通过与 AI 结对编程，将想法变为现实的终极工作站 📚 相关文档 🚀 入门指南 ⚙️ 完整设置流程 📞 联系方式 ✨ 支持项目 🤝 参与贡献 本仓库的 AI 解读链接： zread.ai/tukuaiai/vibe-coding-cn 🖼️ 概览 Vibe Coding 是一个与 AI 结对编程的终极工作… ⭐ 22,980 · 来源：link-4.txt
- [liquidslr/system-design-notes](https://github.com/liquidslr/system-design-notes) — Notes of the book System Desgin Interview - An Insider's Guide ⭐ 19,528 · 来源：0817、0827、0909
- [ykdojo/claude-code-tips](https://github.com/ykdojo/claude-code-tips) — 45+ tips for getting the most out of Claude Code, from basics to advanced - includes a custom status line script and Claude Code running itself in a container. Also includes the dx plugin: skills for everyday dev workflows ⭐ 10,078 · 来源：0817
- [FareedKhan-dev/train-llm-from-scratch](https://github.com/FareedKhan-dev/train-llm-from-scratch) — A straightforward method for training your LLM, from downloading data to generating text ⭐ 9,606 · 来源：0819
- [KimYx0207/AI-Coding-Guide-Zh](https://github.com/KimYx0207/AI-Coding-Guide-Zh) — Claude Code + OpenClaw + Codex + WorkBuddy 中文教程 | 50篇完整教程 + 1张速查卡 | 80万+内容量 | 1500+实操示例 | AI Coding / Agent 四线学习路径（编程+助手+Agent+办公） ⭐ 5,994 · 来源：loopengineer.txt
- [pguso/ai-agents-from-scratch](https://github.com/pguso/ai-agents-from-scratch) — Demystify AI agents by building them yourself. Local LLMs, no black boxes, real understanding of function calling, memory, and ReAct patterns ⭐ 4,747 · 来源：0809、0810、0909
- [xdash/FDE-the-Guidance-Book-of-Forward-Deployed-Engineer](https://github.com/xdash/FDE-the-Guidance-Book-of-Forward-Deployed-Engineer) — FDE（前沿部署工程师）从零入门指南（基于范冰《增长黑客》原书框架） ⭐ 4,651 · 来源：article
- [didilili/ai-agents-from-zero](https://github.com/didilili/ai-agents-from-zero) — 🚀 2026 最系统的 AI Agent 速成指南｜智能体实战教程 · 完整学习路径 + 实战项目 + 面试题库 · 对标大模型应用开发工程师岗位 · 覆盖LangChain / LangGraph / Coze / Dify / MCP / skills / LLM / RAG / 提示词 · 企业级部署与微调 · 从0到企业级落地 + 从学习到上线项目 + 面试准备一体化 ⭐ 4,584 · 来源：0824
- [duoan/TorchCode](https://github.com/duoan/TorchCode) — 🔥 LeetCode for PyTorch — practice implementing softmax, attention, GPT-2 and more from scratch with instant auto-grading. Jupyter-based, self-hosted or try online ⭐ 4,580 · 来源：link-3.txt
- [FareedKhan-dev/all-agentic-architectures](https://github.com/FareedKhan-dev/all-agentic-architectures) — 35 production-grade agentic AI architectures (Reflexion, LATS, GraphRAG, MemGPT, Voyager, BrowserAgent, ...) — a Python library and runnable textbook with multi-provider LLM support and a 17-task benchmark leaderboard ⭐ 4,489 · 来源：0909
- [walkinglabs/hands-on-modern-rl](https://github.com/walkinglabs/hands-on-modern-rl) — 🚀 An open-source, hands-on curriculum bridging the gap from basic RL concepts to LLM alignment, RLVR, and advanced Agentic systems ⭐ 4,361 · 来源：0808
- [nas5w/interview-guide](https://github.com/nas5w/interview-guide) — An opinionated, actionable guide for software engineering interviews ⭐ 4,308 · 来源：link-5.txt
- [evoiz/Agentic-Design-Patterns](https://github.com/evoiz/Agentic-Design-Patterns) — 📚 Agentic Design Patterns - A Hands-On Guide to Building Intelligent Systems 📖 About This Repository This repository contains the complete materials for "Agentic Design P… ⭐ 3,543 · 来源：0819、0830、agent开发
- [Sumanth077/Hands-On-AI-Engineering](https://github.com/Sumanth077/Hands-On-AI-Engineering) — A curated collection of practical AI projects implementing OCR systems, RAG, AI agents, and other AI use cases ⭐ 3,483 · 来源：0819
- [buchidonggua/dg-ai-notes](https://github.com/buchidonggua/dg-ai-notes) — Pi源码解读和二次开发实战 🌐 在线阅读 ： dg-ai-notes.pages.dev —— 双轨教程 · 沉浸式阅读 · 深浅色主题 🤔 Pi-Agent 是什么？为什么值得学？ Pi-Agent 是 @earendil-works 开源的 Agent SDK，定位是 生产级 AI Agent 的运行时底座 ——也是 OpenClaw… ⭐ 2,780 · 来源：0823、link-5.txt
- [study8677/awesome-architecture](https://github.com/study8677/awesome-architecture) — 🧭 Architecture-first system design: 26 bilingual tutorials, 25 architecture templates, and 6 end-to-end cases covering distributed systems, AI-native systems, RAG, coding Agents, and production trade-offs ⭐ 2,311 · 来源：0819
- [langchain-ai/agents-from-scratch](https://github.com/langchain-ai/agents-from-scratch) — Build an email assistant with human-in-the-loop and memory ⭐ 2,216 · 来源：agent开发
- [datawhalechina/deepagents-in-action](https://github.com/datawhalechina/deepagents-in-action) — 📚 《Deep Agents 实战》—— LangChain 官方大使出品，基于 LangChain / LangGraph 生态，从零构建生产级 AI Agent 的完整指南 ⭐ 2,015 · 来源：link-5.txt
- [gavinkhung/machine-learning-visualized](https://github.com/gavinkhung/machine-learning-visualized) — ML algorithms implemented and derived from first-principles in Jupyter Notebooks and NumPy ⭐ 1,960 · 来源：0817、0904
- [Nicolepcx/ai-agents-the-definitive-guide](https://github.com/Nicolepcx/ai-agents-the-definitive-guide) — Repo for AI Agents The Definitive Guide ⭐ 1,833 · 来源：0913
- [Ramakm/ai-hands-on](https://github.com/Ramakm/ai-hands-on) — A group of notebooks and other files which can help you learn AI from scratch ⭐ 1,442 · 来源：0820
- [hahhforest/pi-textbook](https://github.com/hahhforest/pi-textbook) — 《动手学 Pi》：沿 15 个真实 checkpoint 从零构建 Pi-style Agent ⭐ 1,319 · 来源：link-5.txt
- [alchaincyf/deepseek-harness-orange-book](https://github.com/alchaincyf/deepseek-harness-orange-book) — DeepSeek Harness橙皮书《从开机到拆开》：完整系统提示词、129行启动清单、三份原始会话日志——官方文档没有的一手实测。PDF/EPUB/HTML免费下载 ⭐ 1,279 · 来源：0819
- [SaladDay/pi-from-scratch](https://github.com/SaladDay/pi-from-scratch) — 600 行 TypeScript 写成的超级迷你版 pi，让你轻松从 0 写出属于你的 pi-agent ⭐ 1,197 · 来源：0821
- [victordibia/designing-multiagent-systems](https://github.com/victordibia/designing-multiagent-systems) — Building LLM-Enabled Multi Agent Applications from Scratch ⭐ 1,171 · 来源：0911、0913
- [analyticalrohit/pytorch_fundamentals](https://github.com/analyticalrohit/pytorch_fundamentals) — Introduction to PyTorch, covering tensor initialization, operations, indexing, and reshaping ⭐ 1,057 · 来源：0821
- [rasbt/MachineLearning-QandAI-book](https://github.com/rasbt/MachineLearning-QandAI-book) — Machine Learning Q and AI book ⭐ 967 · 来源：link-5.txt
- [hardness1020/awesome-agent-architecture](https://github.com/hardness1020/awesome-agent-architecture) — Learn AI agents from scratch ⭐ 921 · 来源：0817
- [livialima/linuxupskillchallenge](https://github.com/livialima/linuxupskillchallenge) — Learn the skills required to sysadmin a remote Linux server from the commandline ⭐ 666 · 来源：0820、0830、0905
- [giswqs/intro-gispro](https://github.com/giswqs/intro-gispro) — Code examples for the book titled Introduction to GIS Programming ⭐ 621 · 来源：0823
- [DistSysCorp/ddia](https://github.com/DistSysCorp/ddia) — DDIA 逐章精读 ⭐ 556 · 来源：0907
- [antinomie-lab/pi-book](https://github.com/antinomie-lab/pi-book) — Source-backed architecture notes on building an agent ⭐ 405 · 来源：0821
- [GeostatsGuy/MachineLearningDemos](https://github.com/GeostatsGuy/MachineLearningDemos) — well-documented demonstration Python Jupyter workflows for many common machine learning workflows ⭐ 376 · 来源：0904
- [Moh4696/build-ai-agents-free](https://github.com/Moh4696/build-ai-agents-free) — How to Build AI Agents Completely Free in 2026 the ultimate beginner's guide $0 · no credit card · no prior experience · open-source building an AI agent shouldn't be som… ⭐ 334 · 来源：0822
- [chemark/ai-agent-book](https://github.com/chemark/ai-agent-book) — 深入理解 AI Agent：设计原理与工程实践（学习副本，upstream: bojieli/ai-agent-book） ⭐ 291 · 来源：autotest
- [TrenTorch/TrenTorch](https://github.com/TrenTorch/TrenTorch) — Learn PyTorch by building your own. (inspired from Harvard's TinyTorch) ⭐ 211 · 来源：0907
- [xiaomoBoy/pi-bluebook](https://github.com/xiaomoBoy/pi-bluebook) — Pi Coding Agent 中文学习蓝皮书：从安装与第一个可验收任务开始，逐步掌握 Session、Context、Skill、Extension、Subagent 与长期 Agent 工作流。 ⭐ 168 · 来源：0911
- [XianZS/PythonLearning](https://github.com/XianZS/PythonLearning) — A comprehensive Python learning repository!Easy reading of learning documents: ⭐ 96 · 来源：skills
- [agent-infra-foundation/agent-infra-book](https://github.com/agent-infra-foundation/agent-infra-book) — Technical books and research dossiers about the infrastructure that gives AI agents private workspaces, controlled execution, durable state, and safe publication ⭐ 73 · 来源：agent开发
- [Vink567/agent-orange-book](https://github.com/Vink567/agent-orange-book) — Agent 橙皮书：一份给普通人的 AI Agent 入门手册（非官方开源指南） ⭐ 43 · 来源：link-4.txt
- [adpanru/cordis-mini](https://github.com/adpanru/cordis-mini) — 用约 600 行纯 Python，从零实现 Cordis / deepseek-harness 的核心 Agent 插件架构：插件系统、Context 服务注册、依赖拓扑排序、事件分发、Waterfall 中间件与可逆副作用。无需真实 LLM API，内置 Fake LLM，通过完整 Agent Loop 直观理解 AI Agent 插件架构。 ⭐ 33 · 来源：0822
- [plamen5rov/vibe-coding-playbook](https://github.com/plamen5rov/vibe-coding-playbook) — The ultimate guide for vibe coding with AI tools like Claude and OpenCode ⭐ 25 · 来源：0907、0911、_repos.txt
- [dataPro-lgtm/production-grade-multi-agent-systems](https://github.com/dataPro-lgtm/production-grade-multi-agent-systems) — 《生产级多智能体系统：从架构判断到工程落地》中文开源书 ⭐ 24 · 来源：agent开发
- [wilwaldon/claude-code-cheat-sheet](https://github.com/wilwaldon/claude-code-cheat-sheet) — A single-page reference covering every keyboard shortcut, slash command, CLI flag, MCP server config, CLAUDE.md location, skill/agent frontmatter option, environment variable, and permission mode in Claude Code. Built for engineers who'd rather scan a table than dig through docs ⭐ 10 · 来源：0907、0911、_repos.txt
- [littlekelvin/tiny-claw-java](https://github.com/littlekelvin/tiny-claw-java) — 从0开始构建harness的java版本 ⭐ 2 · 来源：agent开发

<a name="cat-c"></a>

## C. Agent Skills、插件与 MCP 扩展（52 个）

技能库、技能管理器、插件市场、MCP server、给 agent 加能力的工具包

- [mattpocock/skills](https://github.com/mattpocock/skills) — Skills for Real Engineers. Straight from my .agents directory ⭐ 260,781 · 来源：0825、0911、link-4.txt、mattpocock、workflow.txt
- [msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents) — A complete AI agency at your fingertips - From frontend wizards to Reddit community ninjas, from whimsy injectors to reality checkers. Each agent is a specialized expert with personality, processes, and proven deliverables ⭐ 152,002 · 来源：link-3.txt
- [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) — Production-grade engineering skills for AI coding agents ⭐ 93,801 · 来源：0810、0820、0824、link-4.txt
- [ayghri/i-have-adhd](https://github.com/ayghri/i-have-adhd) — A skill to stop your coding agent from burying the answer. ADHD-friendly output ⭐ 43,742 · 来源：0817
- [emilkowalski/skills](https://github.com/emilkowalski/skills) — Skills for Designers and Engineers ⭐ 37,354 · 来源：link-4.txt、skills
- [virgiliojr94/book-to-skill](https://github.com/virgiliojr94/book-to-skill) — Turn any technical book PDF into a Claude Code skill — ready to study, reference, and use while you work ⭐ 30,356 · 来源：skills、tool
- [Nutlope/hallmark](https://github.com/Nutlope/hallmark) — Anti-AI-slop design skill for Claude Code, Cursor, and Codex ⭐ 28,511 · 来源：0911
- [phuryn/pm-skills](https://github.com/phuryn/pm-skills) — PM Skills Marketplace: 100+ agentic skills, commands, and plugins — from discovery to strategy, execution, launch, and growth ⭐ 26,274 · 来源：0810、link-4.txt
- [titanwings/colleague-skill](https://github.com/titanwings/colleague-skill) — Distilly — Distill how they think into reusable Skills for any Agent or Bot. Formerly Colleague Skill（原同事 Skill） ⭐ 24,668 · 来源：link-5.txt
- [microsoft/SkillOpt](https://github.com/microsoft/SkillOpt) — SkillOpt is a text-space optimizer that trains reusable natural-language skills for frozen LLM agents through trajectory-driven edits, validation-gated updates, and deployable best_skill.md artifacts ⭐ 16,972 · 来源：link-4.txt
- [earthtojake/text-to-cad](https://github.com/earthtojake/text-to-cad) — A library of agent skills for CAD, CAE and CAM ⭐ 15,493 · 来源：0909、harness
- [wonderwhy-er/DesktopCommanderMCP](https://github.com/wonderwhy-er/DesktopCommanderMCP) — This is MCP server for Claude that gives it terminal control, file system search and diff file editing capabilities ⭐ 9,565 · 来源：link-3.txt
- [cordiverse/cordis](https://github.com/cordiverse/cordis) — Meta-Framework of Spatiotemporal Composability ⭐ 8,421 · 来源：0817、0826
- [cursor/plugins](https://github.com/cursor/plugins) — Cursor plugin specification and official plugins ⭐ 7,569 · 来源：0824、0825
- [tw93/Waza](https://github.com/tw93/Waza) — 🥷 Engineering habits you already know, turned into skills Claude can run ⭐ 7,005 · 来源：0817、link-5.txt
- [uditgoenka/autoresearch](https://github.com/uditgoenka/autoresearch) — Claude Autoresearch Skill — Autonomous goal-directed iteration for Claude Code. Inspired by Karpathy's autoresearch. Modify → Verify → Keep/Discard → Repeat forever ⭐ 6,301 · 来源：0909、skills
- [callstack/agent-device](https://github.com/callstack/agent-device) — Mobile app automation and verification for AI coding agents. CLI, MCP server, and typed Node.js API for iOS, Android, HarmonyOS, TV, web, macOS, and Linux ⭐ 4,557 · 来源：link-4.txt
- [humanlayer/skills](https://github.com/humanlayer/skills) — skills Claude Code skills from HumanLayer . Available Skills improve-claude-md Rewrites your CLAUDE.md using <important if> blocks to improve instruction adherence. npx s… ⭐ 3,941 · 来源：0825
- [microsoft/skill-recorder](https://github.com/microsoft/skill-recorder) — Desktop app that records your on-screen work session and uses the GitHub Copilot CLI to reconstruct it as an intent + ordered steps, then builds a reusable Skill or Automation for Microsoft Scout, Microsoft Copilot Cowork, or Copilot Studio ⭐ 3,929 · 来源：skills
- [anthropics/claude-plugins-community](https://github.com/anthropics/claude-plugins-community) — Community plugin marketplace for Claude Cowork and Claude Code. Read-only mirror — submit plugins at clau.de/plugin-directory-submission ⭐ 3,893 · 来源：0825
- [mitsuhiko/agent-stuff](https://github.com/mitsuhiko/agent-stuff) — These are commands I use with agents, mostly Claude ⭐ 3,079 · 来源：0830
- [totec448-spec/chat-on-steroids](https://github.com/totec448-spec/chat-on-steroids) — Cross-platform local MCP capabilities for ChatGPT with Chrome integration, Goal, Compact & Resume, and durable multi-agent workflows ⭐ 2,038 · 来源：0824
- [Fenng/Tech-Doc-Style-Chinese](https://github.com/Fenng/Tech-Doc-Style-Chinese) — Yet another reusable writing skill for Chinese technical documentation and product copy ⭐ 1,115 · 来源：skills
- [agent-sh/agentsys](https://github.com/agent-sh/agentsys) — AI writes code. This automates everything else · 24 plugins · 49 agents · 44 skills · for Claude Code, OpenCode, Codex, Cursor, Kiro ⭐ 985 · 来源：agent开发、skills
- [luongnv89/asm](https://github.com/luongnv89/asm) — The universal skill manager for AI coding agents ⭐ 928 · 来源：0819、0820
- [AmazingAng/old-coder](https://github.com/AmazingAng/old-coder) — An old coder's strategy for the agent era: don't read the code — make it run the gauntlet. Evidence-first development skill for coding agents, inspired by Uncle Bob ⭐ 723 · 来源：agent开发
- [sandroandric/AgentHandover](https://github.com/sandroandric/AgentHandover) — What if OpenClaw, Claude Code, Codex etc. knew how to do your work without you saying it? AgentHandover observes you, learns and teaches your agents with self-improving skills ⭐ 700 · 来源：0830、0905
- [danielealbano/android-remote-control-mcp](https://github.com/danielealbano/android-remote-control-mcp) — An MCP Server for Android running on the phone, optmized for token usage, supports also files downloads and cloudflare and ngrok automated tunnelling ⭐ 617 · 来源：0816
- [warpdotdev/common-skills](https://github.com/warpdotdev/common-skills) — Common Skills This repository is where common agent skills that should be shared across repositories go. A skill belongs here when it captures a reusable workflow, conven… ⭐ 576 · 来源：0825、0830、0905
- [ayi-ai/nie-grassroots-logic](https://github.com/ayi-ai/nie-grassroots-logic) — 聂·基层运行逻辑 · Agent Skill：基于聂辉华《基层中国的运行逻辑》的方法论工具箱（不含原书全文） ⭐ 547 · 来源：0821
- [shaom/infocard-skills](https://github.com/shaom/infocard-skills) — Open-source agent skills for generating editorial-style information cards from natural-language input ⭐ 508 · 来源：link-5.txt
- [mcp2everything/mcp2mqtt](https://github.com/mcp2everything/mcp2mqtt) — 本项目通过将 MCP 协议转换为 MQTT 协议，我们能够利用强大的大型语言模型（LLMs），就能轻松操控您的智能家居、机器人或其他硬件设备。 ⭐ 370 · 来源：link-5.txt
- [shareAI-lab/shareAI-skills](https://github.com/shareAI-lab/shareAI-skills) — Skills distilled from the Lab's real work and collaboration practices ⭐ 314 · 来源：0808、skills
- [LeoYeAI/teammate-skill](https://github.com/LeoYeAI/teammate-skill) — Distill a teammate into an AI Skill. Auto-collect Slack/Teams/GitHub data, generate Work Skill + 5-layer Persona, with continuous evolution. Powered by MyClaw.ai. Works with Claude Code, OpenClaw, and any AgentSkills-compatible agent ⭐ 257 · 来源：link-5.txt
- [vikingmute/review-forge](https://github.com/vikingmute/review-forge) — review-forge is an Agent Skill for structured, auditable code review workflows ⭐ 219 · 来源：方法论
- [scarletkc/agents](https://github.com/scarletkc/agents) — Shared standards and reusable skills for Claude Code, Codex CLI, and other AI coding agents ⭐ 212 · 来源：0909
- [luoling8192/ai-coding-principles](https://github.com/luoling8192/ai-coding-principles) — A collection of Claude Code skills that enforce coding discipline and prevent common AI coding anti-patterns ⭐ 171 · 来源：0907
- [itshen/source-reading-methodology](https://github.com/itshen/source-reading-methodology) — 带 AI 精读大型开源仓库的方法论：四阶段流程、可复用模板、28 条踩坑清单，核心是让每个技术论断都可回溯到源码具体行 ⭐ 136 · 来源：0823
- [oil-oil/codex-dev-team](https://github.com/oil-oil/codex-dev-team) — 协调探索、执行和独立评审，让复杂任务按明确职责并行推进，由主 Agent 统一验收。 ⭐ 118 · 来源：link-4.txt
- [musoyangrigor/gitx-skill](https://github.com/musoyangrigor/gitx-skill) — GitX is a portable AI-agent skill for creating clean Git commits, tagged branches, and safe pushes across Codex, Claude Code, Cursor, and other skills-compatible agents ⭐ 71 · 来源：0824
- [robotbird/skillkit](https://github.com/robotbird/skillkit) — AI agent's skills manager ⭐ 61 · 来源：link-5.txt
- [reorx/envops](https://github.com/reorx/envops) — envops A single-file CLI to inspect and manipulate .env files, designed to be safe by default: values that look like secrets are masked in all output unless you explicitl… ⭐ 45 · 来源：0823
- [pingchesu/hermes-curator-evolver](https://github.com/pingchesu/hermes-curator-evolver) — Evidence-driven skill evolution for Hermes Agent — reports, dry-run proposals, candidate search, and guarded apply ⭐ 39 · 来源：0904
- [FradSer/pi-packages](https://github.com/FradSer/pi-packages) — Frad's Pi Packages English | 简体中文 Native Pi packages for reusable skills, extensions, and workflow commands. Packages @fradser/pi-agent-teams Compact agent delegation and… ⭐ 38 · 来源：0824
- [cpcc/SkillsPlusPlus](https://github.com/cpcc/SkillsPlusPlus) — skills++ 桌面端 Skills 管理工具：管理 Codex、Claude、Cursor 等 AI 工具的 skills，聚合全网 skills，一键安装/卸载/重装 ⭐ 31 · 来源：link-5.txt
- [plannotator/guides](https://github.com/plannotator/guides) — Agent skill: write a Guided Review of a diff and export it as a portable HTML file (plannotator guide export) ⭐ 26 · 来源：0822
- [Capslockb/hermes-live-discord-agent-plugin](https://github.com/Capslockb/hermes-live-discord-agent-plugin) — Hermes Live Discord Agent Plugin — full-duplex Discord voice ↔ Google Gemini Multimodal Live API, with function calling, idle hangup, transcripts, and a 3-min oneshot installer ⭐ 21 · 来源：0904
- [shynloc/Hermes-Agent-QQ-Plugin](https://github.com/shynloc/Hermes-Agent-QQ-Plugin) — 基于OpenClaw的QQ官方插件改装的适配Hermes Agent的QQ聊天插件，可以用QQ和Hermes Agent沟通了哦。 ⭐ 20 · 来源：0904
- [ht426/skillseed](https://github.com/ht426/skillseed) — 一个 skill 工厂 —— 脚手架、触发校验、自我进化。基于 Agent Skills 开放标准。 ⭐ 11 · 来源：link-4.txt
- [lixuvip/codex-agent-orchestration-skill](https://github.com/lixuvip/codex-agent-orchestration-skill) — Codex agent orchestration skill with adaptive thread thinking, callbacks, QA/review gates, fenced automations, project autopilot, and optional agy/Gemini review ⭐ 9 · 来源：0905
- [HYYH-code/agent-skill-store](https://github.com/HYYH-code/agent-skill-store) — Discover, govern, version, and install skills for AI agents ⭐ 8 · 来源：link-5.txt
- [Ron-dali/vibe-coding-rules](https://github.com/Ron-dali/vibe-coding-rules) — Vibe Coding needs Rules. Self-growing code quality pipeline for AI agents. 6 Skills: init→discipline→terminal→self-check→testing→changelog ⭐ 4 · 来源：link-3.txt

<a name="cat-a"></a>

## A. AI Agent 框架与运行时（38 个）

构建 agent 的框架、harness、运行时、多智能体编排引擎、agent 平台

- [langflow-ai/langflow](https://github.com/langflow-ai/langflow) — Langflow is a powerful tool for building and deploying AI-powered agents and workflows ⭐ 154,698 · 来源：agent开发、link-3.txt
- [earendil-works/pi](https://github.com/earendil-works/pi) — AI agent toolkit: unified LLM API, agent loop, TUI, coding agent CLI（原地址 badlogic/pi-mono 已跳转到 earendil-works/pi） ⭐ 104,512 · 来源：0823、0826、0911、agent开发
- [ruvnet/ruflo](https://github.com/ruvnet/ruflo) — 🌊 The original agent harness. Deploy intelligent multi-player swarms, coordinate autonomous workflows, and build conversational AI systems. Features adaptive memory, self-learning intelligence, federation, vector RAG integration, and native Claude Code / Codex / Hermes and many more Integrated ⭐ 72,263 · 来源：0824
- [Mintplex-Labs/anything-llm](https://github.com/Mintplex-Labs/anything-llm) — Stop renting your intelligence. Own it with AnythingLLM. Everything you need for a powerful local-first agent experience ⭐ 65,972 · 来源：0907
- [FoundationAgents/OpenManus](https://github.com/FoundationAgents/OpenManus) — No fortress, purely open ground. OpenManus is Coming ⭐ 58,292 · 来源：link-5.txt
- [agentscope-ai/agentscope](https://github.com/agentscope-ai/agentscope) — Build and run agents you can see, understand and trust ⭐ 31,506 · 来源：agent开发
- [openai/openai-agents-python](https://github.com/openai/openai-agents-python) — A lightweight, powerful framework for multi-agent workflows ⭐ 29,397 · 来源：loop
- [activepieces/activepieces](https://github.com/activepieces/activepieces) — AI Agents & MCPs & AI Workflow Automation • (~400 MCP servers for AI agents) • AI Automation / AI Agent with MCPs • AI Workflows & AI Agents • MCPs for AI Agents ⭐ 24,418 · 来源：agent开发
- [google/adk-python](https://github.com/google/adk-python) — An open-source, code-first Python toolkit for building, evaluating, and deploying sophisticated AI agents with flexibility and control ⭐ 21,520 · 来源：0909
- [pydantic/pydantic-ai](https://github.com/pydantic/pydantic-ai) — How Python does AI. Agents, realtime voice, image generation, embeddings. Every model, every interface, typed end to end ⭐ 19,895 · 来源：loop
- [yc-software/qm](https://github.com/yc-software/qm) — Multiplayer agent harness for work ⭐ 14,875 · 来源：agent开发
- [omnigent-ai/omnigent](https://github.com/omnigent-ai/omnigent) — Omnigent is an open-source AI agent framework and meta-harness: orchestrate Claude Code, Codex, Cursor, Pi, and custom agents — swap harnesses without rewriting, enforce policies and sandboxing, and collaborate in real time from any device ⭐ 9,897 · 来源：0830
- [HKUDS/AutoAgent](https://github.com/HKUDS/AutoAgent) — "AutoAgent: Fully-Automated and Zero-Code LLM Agent Framework" ⭐ 9,786 · 来源：0907、agent开发
- [cloudflare/computer](https://github.com/cloudflare/computer) — Give your agent a computer 👾 ⭐ 9,170 · 来源：autotest
- [HKUDS/ClawTeam](https://github.com/HKUDS/ClawTeam) — "ClawTeam: Agent Swarm Intelligence" (One Command → Full Automation) ⭐ 5,536 · 来源：agent开发
- [apache/maka](https://github.com/apache/maka) — Apache Maka (Incubating) is a high-performance agent workspace that keeps a complete record of everything it did ⭐ 5,311 · 来源：0820、0905
- [jjyaoao/HelloAgents](https://github.com/jjyaoao/HelloAgents) — A agent framework based on the tutorial hello-agents ⭐ 2,985 · 来源：0909、0911、0913
- [ApodexAI/FrontierAgent](https://github.com/ApodexAI/FrontierAgent) — 🧩 FrontierAgent, our agent framework, open-sourced alongside it — native command-line TUI, ReAct and Agent Team modes, one command on macOS and Linux, no preinstall, no hard Docker dependency ⭐ 2,762 · 来源：0826
- [langchain-ai/open-agent-platform](https://github.com/langchain-ai/open-agent-platform) — An open-source, no-code agent building platform ⭐ 1,901 · 来源：0907
- [injaneity/pi-computer-use](https://github.com/injaneity/pi-computer-use) — Let Pi control your apps on MacOS & Windows ⭐ 1,892 · 来源：link-4.txt
- [AMAP-ML/LongHorizon-Harness](https://github.com/AMAP-ML/LongHorizon-Harness) — The long-horizon computer-use harness. Run AI agents across desktop apps and the CLI for extended periods while preserving task state and making reliable progress on complex workflows. Features fresh-context execution, durable verified state, independent auditing, recoverable progress, and native Claude Code / Codex / OpenClaw integration ⭐ 1,501 · 来源：0823、0824
- [agentconnect-md/agentconnect](https://github.com/agentconnect-md/agentconnect) — The open-source, multi-agent alternative to Claude Tag. @ any agent, wherever work happens, they work alongside your team, learning as they go ⭐ 1,361 · 来源：0827
- [deerwork-ai/deer-workflow](https://github.com/deerwork-ai/deer-workflow) — An open-source graph engineering runtime that keeps orchestration in TypeScript and delegates semantic work to replaceable Agent runtimes ⭐ 536 · 来源：agent开发
- [microsoft/Orchard](https://github.com/microsoft/Orchard) — Orchard: An Open-Source Agentic Modeling Framework ⭐ 511 · 来源：agent开发
- [Darwin-Agent/HarnessX](https://github.com/Darwin-Agent/HarnessX) — HarnessX is a harness foundry: forge any number of agent harnesses from reusable processors and bundles, pair each with any model, and evolve them through training ⭐ 461 · 来源：harness
- [lemma-work/lemma-platform](https://github.com/lemma-work/lemma-platform) — The open-source workspace where humans and AI agents work as one team ⭐ 392 · 来源：link-4.txt
- [yan5xu/codexloom](https://github.com/yan5xu/codexloom) — Turn Codex threads into an organization of long-lived domain agents ⭐ 384 · 来源：agent开发
- [ReflexioAI/reflexio](https://github.com/ReflexioAI/reflexio) — Make your agents improve themselves. Reflexio is an AI agent self-improvement harness that enables your AI agents to continuously learn from real user interactions ⭐ 369 · 来源：0907
- [AgentSwarms-fyi/agentswarms](https://github.com/AgentSwarms-fyi/agentswarms) — Unified Agentic AI and Data Platform ⭐ 238 · 来源：agent开发
- [nihalashetty/Forge](https://github.com/nihalashetty/Forge) — Forge is an open-source, self-hosted alternative for visually building and shipping AI agents and workflows without giving up control of your infrastructure ⭐ 159 · 来源：agent开发
- [krishagarwal314/autodev-studio](https://github.com/krishagarwal314/autodev-studio) — Terminal-first, knowledge-grounded multi-agent software delivery pipeline: scope requirements, implement changes, run tests, and gate pull requests with deterministic QA and ensemble code review ⭐ 144 · 来源：harness
- [joe960913/Jixu](https://github.com/joe960913/Jixu) — Durable single-Agent Harness for TypeScript: recoverable Threads, context continuity, explicit side effects, and a native TUI ⭐ 100 · 来源：0823
- [orbi-build/orbi](https://github.com/orbi-build/orbi) — Orbi — the factory that builds and operates AI software factories. GitHub Issues in, releases and runnable system out ⭐ 96 · 来源：0907
- [huanyingtianhe/agents-chat](https://github.com/huanyingtianhe/agents-chat) — An open-source AI agent platform for orchestrating, and collaborating with multiple AI agents. Connect seamlessly to GitHub Copilot CLI, Claude Code, Codex, and any ACP-compatible agent ⭐ 25 · 来源：0819
- [freezetheflame/NanoHarness](https://github.com/freezetheflame/NanoHarness) — A minimal, composable AI agent harness framework in Python ⭐ 17 · 来源：link-5.txt
- [othorizon/easy-agent-team](https://github.com/othorizon/easy-agent-team) — easy-agent-team 面向团队的 AI 能力集中管理与分发平台 。 由少数「能力建设者」开发 Skill、配置 MCP、维护环境与基础设施，平台在 权限管控 下将这些能力分发给团队的其他成员——包括不具备技术背景的同事。平台同时提供「人机求助」机制：AI 无法自行解决的问题可以转交给团队中掌握相关信息的人，所得答复沉淀为可复用的… ⭐ 10 · 来源：0905
- [j20cc/agent-learn](https://github.com/j20cc/agent-learn) — Go Agent — AI 编程助手 基于 Go + Gin + SSE 的 AI Agent 服务器。使用 OpenAI Responses API，支持工具调用、子 Agent、队友协作、后台任务等完整 Agent 能力。 从 s_full.py 完整移植。 快速开始 # 1. 进入目录 cd go-agent # 2. 配置环境变量… ⭐ 6 · 来源：0809、0816
- [daizw/agent-harness](https://github.com/daizw/agent-harness) — make your agents 10x more powerful ⭐ 0 · 来源：0907

<a name="cat-l"></a>

## L. 开发工具与 DevOps / 基础设施（36 个）

版本管理、终端、git 工具、系统工具、K8s、监控、自托管基础设施、运维脚本

- [cypress-io/cypress](https://github.com/cypress-io/cypress) — Fast, easy and reliable testing for anything that runs in a browser ⭐ 51,009 · 来源：autotest、db工具
- [getsentry/sentry](https://github.com/getsentry/sentry) — Developer-first error tracking and performance monitoring ⭐ 44,773 · 来源：0905
- [jdx/mise](https://github.com/jdx/mise) — dev tools, env vars, task runner ⭐ 33,850 · 来源：0821、tool
- [goauthentik/authentik](https://github.com/goauthentik/authentik) — The authentication glue you need ⭐ 25,454 · 来源：tool
- [pranshuparmar/witr](https://github.com/pranshuparmar/witr) — Why is this running? Trace any process, port, container, or file back to what started it - CLI + TUI ⭐ 22,243 · 来源：0813、0816
- [grokability/snipe-it](https://github.com/grokability/snipe-it) — A free open source IT asset/license management system ⭐ 14,932 · 来源：0816、0817
- [satnaing/shadcn-admin](https://github.com/satnaing/shadcn-admin) — Admin Dashboard UI built with Shadcn and Vite ⭐ 14,190 · 来源：tool
- [sohutv/cachecloud](https://github.com/sohutv/cachecloud) — 搜狐视频(sohu tv)Redis私有云平台 ：支持Redis多种架构(Standalone、Sentinel、Cluster)高效管理、有效降低大规模redis运维成本，提升资源管控能力和利用率。平台提供快速搭建/迁移，运维管理，弹性伸缩，统计监控，客户端整合接入等功能。(CacheCloud is a Redis cloud management platform. It supports Standalone, Sentinel, and Cluster architectures for Redis, effectively reducing large-scale Redis operation and maintenance costs, and improving resource management and utilization. The platform provides rapid construction/migration, operation and maintenance management, elastic scaling, statistical monitoring, client integration and access and other functions) ⭐ 9,033 · 来源：link-3.txt
- [max-sixty/worktrunk](https://github.com/max-sixty/worktrunk) — Worktrunk is a CLI for Git worktree management, designed for parallel AI agent workflows ⭐ 7,391 · 来源：0913
- [shiaho777/web-to-app](https://github.com/shiaho777/web-to-app) — The most full featured web-to-app toolkit on Android, a complete APK workshop that runs entirely on your phone ⭐ 6,359 · 来源：link-5.txt
- [pixlcore/xyops](https://github.com/pixlcore/xyops) — The next generation of Cronicle: open-source job scheduling, visual workflows, server monitoring, alerting, and incident response ⭐ 6,189 · 来源：0824
- [DetachHead/rebased](https://github.com/DetachHead/rebased) — A git client based on the IntelliJ platform ⭐ 5,563 · 来源：0816
- [NotHarshhaa/DevOps-Projects](https://github.com/NotHarshhaa/DevOps-Projects) — 🚀 Real-world DevOps projects for aspiring engineers — Beginner to Advanced. Covers AWS, Kubernetes, Docker, CI/CD, Terraform, Jenkins, and more. Hands-on learning with step-by-step guides ⭐ 5,173 · 来源：link-5.txt
- [owu/wsl-dashboard](https://github.com/owu/wsl-dashboard) — A GUI manager for WSL featuring a modern UI — a lightweight, low‑memory, high‑performance dashboard to manage WSL instances. Install, list, start, stop, unregister, and configure your WSL distros​ from one place ⭐ 3,752 · 来源：0824
- [AdventDevInc/kudu](https://github.com/AdventDevInc/kudu) — Free Windows, Mac and Linux cleaner, scanner, and more ⭐ 3,427 · 来源：link-5.txt
- [nuver-labs/vps-audit](https://github.com/nuver-labs/vps-audit) — lightweight, dependency-free bash script for security, performance auditing and infrastructure monitoring of Linux servers ⭐ 3,108 · 来源：0913
- [subtrace/subtrace](https://github.com/subtrace/subtrace) — Network inspector for your backend ⭐ 3,094 · 来源：0816、抓包
- [redtrillix/SpaceSniffer](https://github.com/redtrillix/SpaceSniffer) — SpaceSniffer is a freeware disk space analyzer for Windows that make use of the Treemap concept to view the current disk usage ⭐ 2,038 · 来源：0816
- [veops/oneterm](https://github.com/veops/oneterm) — Provide secure access and control over all infrastructure ⭐ 1,741 · 来源：article
- [feigeCode/navop](https://github.com/feigeCode/navop) — A native, all-in-one workspace for databases, SSH, SFTP, terminals, remote desktop, monitoring, and AI ⭐ 1,337 · 来源：0824
- [nklmilojevic/sofka](https://github.com/nklmilojevic/sofka) — A Kubernetes TUI, reimagined in Rust - built on kube-rs and ratatui, async-first from the ground up ⭐ 1,104 · 来源：0905
- [LiaoGuoYin/lixian.online](https://github.com/LiaoGuoYin/lixian.online) — A one-stop tool to grab installer packages for VSCode extensions, Chrome/Edge add-ons, Docker images, and Microsoft Store apps — download once, install anywhere, even offline ⭐ 578 · 来源：link-3.txt
- [Studio-Saelix/sencho](https://github.com/Studio-Saelix/sencho) — Self-hosted Docker Compose management platform. For single or multi-host compose-first workflow ⭐ 456 · 来源：link-4.txt
- [biplobsd/running_services_monitor](https://github.com/biplobsd/running_services_monitor) — Monitor running services on your Android device. With a clean and intuitive interface, you can easily view system and user apps, check their status efficiently ⭐ 420 · 来源：0816
- [LoD-Dawn/Fast-Vben-Admin](https://github.com/LoD-Dawn/Fast-Vben-Admin) — Fast Vben Admin 是一个面向中后台和多租户 SaaS 场景的模块化全栈管理平台。项目采用“模块化单体 + 构建期 Edition 组合”：FastAPI 提供 API、事务与安全边界，Vue Vben Admin 的 web-antd 应用负责管理界面，Platform 提供认证、租户、RBAC、系统管理和基础设施能力，Items、ERP 等业务模块通过统一契约按 Edition 组合交付。 ⭐ 239 · 来源：link-5.txt
- [ChmaraX/herdr-nvim](https://github.com/ChmaraX/herdr-nvim) — Neovim, fully integrated into your herdr workspace ⭐ 175 · 来源：0824
- [hhht110/UpdateLock](https://github.com/hhht110/UpdateLock) — Win7/10/11 原生单文件 Windows 自动更新关闭与恢复工具 ⭐ 148 · 来源：0820
- [livid/exe](https://github.com/livid/exe) — A personal VM cloud in one Go binary: persistent Linux VMs on macOS, Linux and Windows, AI coding agents working inside them, any port published to HTTPS through Cloudflare Tunnel. Driven from a Mac OS 9 desktop in the browser, over SSH, or by an agent ⭐ 87 · 来源：0816
- [shriram-ethiraj/grayslate](https://github.com/shriram-ethiraj/grayslate) — A lightweight, cross-platform developer scratchpad for working with JSON, CSV, Markdown, logs, code snippets, and large text files. Built with Rust, Tauri, and Svelte 5 ⭐ 60 · 来源：link-5.txt
- [JonasBaeumer/herdr-file-annotator](https://github.com/JonasBaeumer/herdr-file-annotator) — A plugin for herdr to maximize agentic development without losing touch with the actual codebase ⭐ 58 · 来源：0826
- [gocronx/kubevision](https://github.com/gocronx/kubevision) — An AI-native Kubernetes dashboard ⭐ 45 · 来源：0816
- [keplerTR/LocalAI-Advisor](https://github.com/keplerTR/LocalAI-Advisor) — Windows Hardware Diagnostic & Local AI Model Advisor. Fast in-process hardware detection & intelligent local LLM recommender for Ollama ⭐ 35 · 来源：0904
- [tsonglew/git-repo-chronicle](https://github.com/tsonglew/git-repo-chronicle) — Compile git commits into a "Chronicle" ⭐ 28 · 来源：0809、skills
- [ascenx/vscode-git-log](https://github.com/ascenx/vscode-git-log) — A visual Git log, commit graph, history browser, and repository operations extension for Visual Studio Code ⭐ 23 · 来源：0819
- [joeseesun/herdr-guide](https://github.com/joeseesun/herdr-guide) — Herdr 软件全面调研、使用价值分析与上手教程 | An independent practical guide to Herdr ⭐ 23 · 来源：0810
- [Bavoch/hello-gitty](https://github.com/Bavoch/hello-gitty) — 为单人 AI 开发者定制的 git 管理工具 ⭐ 12 · 来源：0819

<a name="cat-e"></a>

## E. Agent 记忆、上下文与知识管理（30 个）

长期记忆、上下文数据库、RAG、代码知识图谱、第二大脑、笔记与 wiki

- [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify) — Turn any codebase, with its docs, SQL schemas, configs, and PDFs, into a queryable knowledge graph. A /graphify skill for Claude Code, Cursor, Codex, and Gemini CLI: local deterministic AST parsing, every edge explained, no vector store ⭐ 116,312 · 来源：link-5.txt
- [Lum1104/Understand-Anything](https://github.com/Lum1104/Understand-Anything) — Graphs that teach > graphs that impress. Turn any code into an interactive knowledge graph you can explore, search, and ask questions about. Works with Claude Code, Codex, Cursor, Copilot, Gemini CLI, and more ⭐ 82,385 · 来源：link-2.txt
- [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG) — [EMNLP2025] LightRAG: Simple and Fast Retrieval-Augmented Generation ⭐ 39,601 · 来源：link-5.txt
- [volcengine/OpenViking](https://github.com/volcengine/OpenViking) — Self-evolving Context Database for AI Agents. Unify Agent Memory, Knowledge RAG and Skills ⭐ 36,905 · 来源：link-5.txt
- [tirth8205/code-review-graph](https://github.com/tirth8205/code-review-graph) — Local-first code intelligence graph for MCP and CLI. Builds a persistent map of your codebase so AI coding tools read only what matters, with benchmarked context reductions on reviews and large-repo workflows ⭐ 31,364 · 来源：0911
- [vectorize-io/hindsight](https://github.com/vectorize-io/hindsight) — Hindsight: Agent Memory That Learns ⭐ 23,542 · 来源：0905
- [mksglu/context-mode](https://github.com/mksglu/context-mode) — Context window optimization for AI coding agents. Sandboxes tool output (98% reduction), persists session memory, and enforces routing across 17 platforms via MCP + hooks ⭐ 22,476 · 来源：0913
- [nashsu/llm_wiki](https://github.com/nashsu/llm_wiki) — LLM Wiki is a cross-platform desktop application that turns your documents into an organized, interlinked knowledge base — automatically. Instead of traditional RAG (retrieve-and-answer from scratch every time), the LLM incrementally builds and maintains a persistent wiki from your sources。 ⭐ 19,213 · 来源：0830
- [langchain-ai/openwiki](https://github.com/langchain-ai/openwiki) — OpenWiki is a CLI that writes and maintains agent documentation for your codebase ⭐ 16,441 · 来源：article、link-3.txt
- [NanoNets/Graft](https://github.com/NanoNets/Graft) — Turbocharge Claude Code, Cursor, Codex, Gemini & every coding agent: faster, cheaper, with contextual understanding specific to your codebase ⭐ 7,336 · 来源：agent开发
- [akitaonrails/ai-memory](https://github.com/akitaonrails/ai-memory) — Solution for long term memory for agent coding CLIs and to facilitate handoff between different agent vendors ⭐ 6,650 · 来源：0819、0823
- [Marker-Inc-Korea/AutoRAG](https://github.com/Marker-Inc-Korea/AutoRAG) — AutoRAG: Now your agent can find anything in your computer. It gets smarter if you are using it frequently ⭐ 5,069 · 来源：link-4.txt
- [agenticnotetaking/arscontexta](https://github.com/agenticnotetaking/arscontexta) — Claude Code plugin that generates individualized knowledge systems from conversation. You describe how you think and work, have a conversation and get a complete second brain as markdown files you own ⭐ 3,489 · 来源：link-4.txt
- [jordan-gibbs/hyperresearch](https://github.com/jordan-gibbs/hyperresearch) — Agent-driven research knowledge base. Agents collect, search, and synthesize web research into a persistent, searchable wiki ⭐ 3,124 · 来源：0909
- [Rion-Wu-tech/wechat-intelligence-hub](https://github.com/Rion-Wu-tech/wechat-intelligence-hub) — Local-first WeChat intelligence system with a read-only CLI, Codex skills, searchable chat history, daily briefings, follow-ups and opportunity tracking ⭐ 2,125 · 来源：0907
- [MemTensor/memmy-agent](https://github.com/MemTensor/memmy-agent) — 🍙 A personal AI agent & local memory hub for all AI agents, gives every AI one shared, fully controlled memory and persistent context — all AI remember the same you. Now supports Claude Code, Codex, OpenClaw and Hermes Agent etc ⭐ 1,896 · 来源：0909、agent开发
- [aws/context-ontology-accelerator](https://github.com/aws/context-ontology-accelerator) — An open-source, ontology-based semantic context accelerator that enables AI agents to make more accurate, consistent, and explainable decisions ⭐ 740 · 来源：方法论
- [zosmaai/pi-llm-wiki](https://github.com/zosmaai/pi-llm-wiki) — Self-maintaining, Obsidian-compatible knowledge base for pi — turn raw sources into an interlinked wiki that compounds. Native Open Knowledge Format (OKF) v0.2 ⭐ 575 · 来源：db工具
- [JordyZomer/lemmalog](https://github.com/JordyZomer/lemmalog) — A Datalog engine for LLM agent memory: stratified rules, provenance-tracked facts, incremental derivation, and an MCP server that lets your harness use it as a shared brain ⭐ 306 · 来源：0907
- [Signet-AI/signetai](https://github.com/Signet-AI/signetai) — Sync and store memories, shared identity files (AGENTS.md, CLAUDE.md), session transcripts, institutional knowledge, and secrets between all of your favorite harnesses and models ⭐ 279 · 来源：0904
- [zqiren/Orbital](https://github.com/zqiren/Orbital) — Context is yours. Agents are replaceable. Orbital — a project agent that turns your context into assets ⭐ 261 · 来源：0817
- [nikolai-vysotskyi/trace-mcp](https://github.com/nikolai-vysotskyi/trace-mcp) — Framework-aware code intelligence MCP server for Claude Code and Codex — 70.5% fewer input tokens to review a pull request, median over 60 merged PRs in repos we don't own, comprehension at parity. 81 languages, 87 frameworks. Your code and index never leave the machine; an anonymous usage ping is on by default and opt-out ⭐ 175 · 来源：0907
- [Patdolitse/engram](https://github.com/Patdolitse/engram) — Local-first AI memory you can see, edit, and override — portable across Claude Code, Codex, Cursor, Windsurf, and other MCP coding tools ⭐ 160 · 来源：link-1.txt、link-2.txt
- [marikagura/kimi-core](https://github.com/marikagura/kimi-core) — 个人用的 agent memory OS——记忆系统 + self-drive 自主情感，内置对抗式自审 harness。内在过程需要外在标准。 ⭐ 89 · 来源：0907
- [melandlabs/opencontext](https://github.com/melandlabs/opencontext) — A temporal context graph, a memory API, retrieval primitives, and a multiple-platform integration mesh — designed to be embedded into any host process ⭐ 77 · 来源：0816
- [Ychangqing/IGraph](https://github.com/Ychangqing/IGraph) — IGraph 把代码仓库解析为「符号节点 + 调用关系边」的知识图谱，叠加 LLM 语义摘要与向量索引，通过双通道（Dense + FTS5）RRF 融合检索对外服务。支持挂载 PRD / DB Schema 等多模态资源建立跨模态关联，内置 MCP Server 可直接接入 Cursor / Claude Code 等 AI 助手。 ⭐ 52 · 来源：agent开发
- [mate-matt/rag-memory-lab](https://github.com/mate-matt/rag-memory-lab) — RAG Memory Lab 一个从零学习本地 RAG 检索核心的实战项目。它不调用外部生成服务、不需要 API Key；你可以逐节运行 Notebook，观察 Markdown 如何被切分、索引、召回、重排，并最终变成可追溯的证据卡。 配套阅读 X 长文：用 Notebook 实战课进行像素级 RAG 拆解 博客文章：用 Noteboo… ⭐ 33 · 来源：link-4.txt
- [danielwanwx/am-memory](https://github.com/danielwanwx/am-memory) — Persistent memory for Claude Code — SQLite-backed knowledge store with BM25+Vector search and MCP integration ⭐ 14 · 来源：0830
- [hlwhut225/gefei-knowledge-base](https://github.com/hlwhut225/gefei-knowledge-base) — 哥飞会员知识库 ⭐ 9 · 来源：0826
- [cetus-tech/openkb](https://github.com/cetus-tech/openkb) — Versioned durable project knowledge for AI agents via MCP ⭐ 7 · 来源：agent开发

<a name="cat-b"></a>

## B. Coding Agent 客户端与工作台（25 个）

Claude Code / Codex / Cursor 等编码 agent 的 CLI、桌面端、IDE、并行 agent 工作台

- [anthropics/claude-code](https://github.com/anthropics/claude-code) — Claude Code is an agentic coding tool that lives in your terminal, understands your codebase, and helps you code faster by executing routine tasks, explaining complex code, and handling git workflows - all through natural language commands ⭐ 144,889 · 来源：0810
- [openai/codex](https://github.com/openai/codex) — Lightweight coding agent that runs in your terminal ⭐ 123,717 · 来源：0911、loop
- [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) — 🙌 OpenHands: AI-Driven Development（原地址 All-Hands-AI/OpenHands 已跳转到 OpenHands/OpenHands） ⭐ 87,718 · 来源：0827、loop
- [cline/cline](https://github.com/cline/cline) — Autonomous coding agent as an SDK, IDE extension, or CLI assistant ⭐ 67,909 · 来源：db工具
- [stablyai/orca](https://github.com/stablyai/orca) — Orca is the ADE for working with a fleet of parallel agents. Run any coding agent with your own subscription. Available on desktop, mobile and remote runtime ⭐ 67,468 · 来源：0905、0911
- [Yeachan-Heo/oh-my-codex](https://github.com/Yeachan-Heo/oh-my-codex) — OmX - Oh My codeX: Your codex is not alone. Add hooks, agent teams, HUDs, and so much more ⭐ 33,119 · 来源：link-5.txt
- [siteboon/claudecodeui](https://github.com/siteboon/claudecodeui) — Use Claude Code, OpenCode, Cursor CLI, and Codex on mobile and web with CloudCLI (aka Claude Code UI). CloudCLI is a free open source webui/GUI that helps you manage your Claude Code session and projects remotely ⭐ 13,668 · 来源：0830
- [Devin-AXIS/iPolloWork](https://github.com/Devin-AXIS/iPolloWork) — Enterprise-grade, local-first Agent Workbench for people and agent teams. A unified multi-engine workspace for Codex Harness, DeepSeek Harness, and OpenCode, with unified plugins and Skills, multi-agent projects and tasks, and editable code, documents, presentations, design, and video ⭐ 5,928 · 来源：0817
- [DevAgentForge/Open-Claude-Cowork](https://github.com/DevAgentForge/Open-Claude-Cowork) — OpenSource Claude Cowork. A desktop AI assistant that helps you with programming, file management, and any task you can describe ⭐ 3,397 · 来源：link-3.txt
- [Yeachan-Heo/gajae-code](https://github.com/Yeachan-Heo/gajae-code) — Gajae Code MVP ⭐ 2,781 · 来源：0824
- [BytePioneer-AI/codex-host](https://github.com/BytePioneer-AI/codex-host) — Run Pi and Claude Code directly in Codex Desktop. 在 Codex Desktop 中直接运行 Pi 和 Claude Code。 ⭐ 2,254 · 来源：0830、0905
- [happier-dev/happier](https://github.com/happier-dev/happier) — Web, Desktop & Mobile client for Codex, Claude Code, OpenCode, Kimi, Augment Code, Qwen, fully end-to-end encrypted ⭐ 1,657 · 来源：0830
- [donvito/codex-astra-luna-orchestrator](https://github.com/donvito/codex-astra-luna-orchestrator) — Use Astra as orchestrator and Luna for subagents in Codex ⭐ 1,149 · 来源：0907
- [LodyAI/Lody](https://github.com/LodyAI/Lody) — Share coding agents with your team on phone and desktop ⭐ 1,043 · 来源：0826
- [AQBot-Desktop/AQBot](https://github.com/AQBot-Desktop/AQBot) — ☁️ 轻量级高性能跨平台AI对话 + AI Agent + AI网关桌面客户端 | Lightweight, high-performance cross-platform AI dialogue + AI Agent + AI gateway desktop client ⭐ 901 · 来源：0817
- [RizRiyz/luvus](https://github.com/RizRiyz/luvus) — Mission control for your AI agents ⭐ 820 · 来源：0824
- [Cjbuilds/Codex-Orchestration](https://github.com/Cjbuilds/Codex-Orchestration) — Bring any model to Codex, assign them any role, use them in /goal or any workflow ⭐ 621 · 来源：link-4.txt
- [juggler-ai/juggler](https://github.com/juggler-ai/juggler) — The Juggler Code Agent ⭐ 592 · 来源：link-5.txt
- [JetBrains/thinkrail](https://github.com/JetBrains/thinkrail) — Vibe code with pi in a lightweight, real IDE - The Vibe You Need ⭐ 459 · 来源：0827
- [langwatch/kanban-code](https://github.com/langwatch/kanban-code) — Kanban Code A beautiful kanban board for managing Claude Code sessions. Native on macOS (SwiftUI liquid glass) and Windows (Tauri). The IDE for 2026. ⭐ 320 · 来源：link-4.txt
- [vicoa-ai/vicoa](https://github.com/vicoa-ai/vicoa) — Vicoa is the agentic IDE for running a team of coding agents from any device. Desktop, mobile, VPS, open-source, self-hostable ⭐ 244 · 来源：0904
- [biheto/DevAgent-Studio](https://github.com/biheto/DevAgent-Studio) — DevAgent Studio is a multi-agent workbench for software project understanding and engineering governance. It helps teams review pull request risks, onboard new developers, understand unfamiliar systems, and continuously identify architecture drift and technical debt.无论是新人接手陌生代码、开发者提交 PR，还是团队进行架构巡检，都可以通过多 Agent 协作生成可追踪、可审核、可沉淀的项目治理结果。 ⭐ 176 · 来源：link-4.txt
- [helloxz/zacp](https://github.com/helloxz/zacp) — 基于 ACP 协议的多 Agent Web 网关，让 CLI Agent 也能在浏览器中使用。 ⭐ 120 · 来源：0810、0816
- [scp3500/pi-manager](https://github.com/scp3500/pi-manager) — Local web console for Pi Coding Agent — models, agents, sessions, usage, OpenVL ⭐ 51 · 来源：link-5.txt
- [helloxz/zlite](https://github.com/helloxz/zlite) — Lightweight CLI AI agent — chat, write code, and manage server software with natural language ⭐ 14 · 来源：0819

<a name="cat-d"></a>

## D. Loop Engineering、Harness 工程与规范驱动开发（20 个）

长程 loop/harness 工程方法、Ralph playbook、spec-driven development、code review 工作流

- [affaan-m/ECC](https://github.com/affaan-m/ECC) — The agent harness performance optimization system. Skills, instincts, memory, security, and research-first development for Claude Code, Codex, Opencode, Cursor and beyond ⭐ 257,343 · 来源：link-3.txt、link-5.txt
- [alibaba/open-code-review](https://github.com/alibaba/open-code-review) — Fast, efficient, battle-tested at Alibaba's scale. Hybrid architecture code review tool: deterministic pipelines + LLM Agent, precise line-level comments, built-in multi-language ruleset (NPE, thread-safety, XSS, SQL injection), OpenAI & Anthropic compatible ⭐ 22,833 · 来源：skills
- [walkinglabs/learn-harness-engineering](https://github.com/walkinglabs/learn-harness-engineering) — Harness engineering beginner tutorial, from 0 to 1 ⭐ 15,147 · 来源：0819、0820、0821、link-3.txt
- [cobusgreyling/loop-engineering](https://github.com/cobusgreyling/loop-engineering) — Practical patterns, starters & CLI tools for loop engineering with AI coding agents. Design systems that prompt and orchestrate agents (inspired by Addy Osmani and Boris Cherny). Includes loop-audit, loop-init, loop-cost ⭐ 11,182 · 来源：0907、link-3.txt、loopengineer.txt
- [diet103/claude-code-infrastructure-showcase](https://github.com/diet103/claude-code-infrastructure-showcase) — Examples of my Claude Code infrastructure with skill auto-activation, hooks, and agents ⭐ 10,018 · 来源：link-5.txt
- [huangruiteng/loopx](https://github.com/huangruiteng/loopx) — Long-horizon agent control plane for durable, governed work across Codex, Claude Code, and other harnesses ⭐ 5,822 · 来源：agent开发、loopengineer.txt、方法论
- [wquguru/harness-books](https://github.com/wquguru/harness-books) — 📚 Two books on harness engineering — the design philosophies behind Claude Code & Codex: constraints, query loops, context governance, multi-agent verification. harness-books.agentway.dev ⭐ 3,091 · 来源：0909
- [Priivacy-ai/spec-kitty](https://github.com/Priivacy-ai/spec-kitty) — Spec-Driven Development for serious software developers. Spec Coding with with Claude, Cursor, Gemini, Codex. Kanban dashboard, git worktrees, auto-merge and more ⭐ 1,623 · 来源：0822
- [NVlabs/SoL-Pi](https://github.com/NVlabs/SoL-Pi) — SoL-Pi: Scaling Auto-Research Loops for Efficient Agent Harnesses ⭐ 1,536 · 来源：0911
- [ClaytonFarr/ralph-playbook](https://github.com/ClaytonFarr/ralph-playbook) — A comprehensive guide to running autonomous AI coding loops using Geoff Huntley's Ralph methodology. View as formatted guide below 👇 ⭐ 1,033 · 来源：loop
- [kunchenguid/backpass](https://github.com/kunchenguid/backpass) — You don't write AGENTS.md. You train it with gradient descent ⭐ 957 · 来源：0824
- [hamelsmu/claude-review-loop](https://github.com/hamelsmu/claude-review-loop) — Claude Code plugin: automated code review loop with Codex ⭐ 725 · 来源：0816
- [Ruhan-Wang/Harness_Handbook](https://github.com/Ruhan-Wang/Harness_Handbook) — Harness Handbook English | 中文 | Русский Turn any codebase into a navigable handbook , then use that handbook to help a code agent find every place a change needs to touch… ⭐ 328 · 来源：harness
- [DEEP-JLU/Awesome-Graph-Engineering](https://github.com/DEEP-JLU/Awesome-Graph-Engineering) — A Survey on Ontology Engineering, Graph Engineering, Loop Engineering, Harness Engineering, Context Engineering and Prompt Engineering ⭐ 323 · 来源：loopengineer.txt
- [sudokar/openspec-plus](https://github.com/sudokar/openspec-plus) — OpenSpec Plus — Agentic skills that enhance OpenSpec's Spec-Driven Development through better discovery, requirements, design decisions, execution planning and execution. Works with Claude Code, OpenCode, Github Copilot and any other AI coding agents ⭐ 185 · 来源：link-5.txt
- [loopgain-ai/loopgain](https://github.com/loopgain-ai/loopgain) — An open-source cost controller for AI agent loops — stops a loop when it's actually converged and rolls back before it degrades, instead of running to a fixed max_iterations cap. Real-time loop-gain (Aβ) bands + best-so-far rollback. Adapters for LangGraph, CrewAI, AutoGen, LangChain, OpenAI Agents, and Claude Agent SDK; raw API for custom stacks ⭐ 124 · 来源：0819、skills
- [ChaoYue0307/awesome-loop-engineering](https://github.com/ChaoYue0307/awesome-loop-engineering) — 🔁 Build reliable recurring AI-agent systems: 1007 resources, 22 operational patterns, 22 loop contracts, 8 runtime starters, an interactive atlas, and a structured dataset ⭐ 57 · 来源：loopengineer.txt
- [dososo/agent-loop-skill](https://github.com/dososo/agent-loop-skill) — A skill for long-running AI agent loops with contracts, traces, restarts, scoring, and cleanup ⭐ 32 · 来源：link-5.txt
- [breath57/how-agent-loop-engineering](https://github.com/breath57/how-agent-loop-engineering) — 深入理解 Loop Engineering 的 8 篇系列文章——从为什么需要到怎么落地，一篇一篇吃透。AI Agent | Loop Engineering | Mermaid | 中文 ⭐ 10 · 来源：loopengineer.txt
- [genkio/slop-review](https://github.com/genkio/slop-review) — Code review done right ⭐ 10 · 来源：link-3.txt

<a name="cat-p"></a>

## P. 生产力、办公与协作（20 个）

Office/文档处理、Markdown 与笔记应用、书签、项目与团队管理、聊天协作

- [makeplane/plane](https://github.com/makeplane/plane) — 🔥🔥🔥 Open-source Jira, Linear, Monday, and ClickUp alternative. Plane is a modern project management platform to manage tasks, sprints, docs, and triage ⭐ 59,299 · 来源：0822
- [frappe/erpnext](https://github.com/frappe/erpnext) — Free and Open Source Enterprise Resource Planning (ERP) ⭐ 39,170 · 来源：db工具
- [block/buzz](https://github.com/block/buzz) — A hive mind communication platform ⭐ 32,622 · 来源：0823
- [iOfficeAI/OfficeCLI](https://github.com/iOfficeAI/OfficeCLI) — OfficeCLI is the first and best Office suite purpose-built for AI agents to read, edit, and automate Word, Excel, and PowerPoint files. Free, open-source, single binary, no Office installation required ⭐ 30,498 · 来源：0911、link-4.txt、tool
- [koreader/koreader](https://github.com/koreader/koreader) — An ebook reader application supporting PDF, DjVu, EPUB, FB2 and many more formats, running on Cervantes, Kindle, Kobo, PocketBook and Android devices ⭐ 29,674 · 来源：tool
- [jordanbaird/Ice](https://github.com/jordanbaird/Ice) — Powerful menu bar manager for macOS ⭐ 29,607 · 来源：0827
- [hcengineering/platform](https://github.com/hcengineering/platform) — Huly — All-in-One Project Management Platform (alternative to Linear, Jira, Slack, Notion, Motion) ⭐ 27,653 · 来源：tool
- [firecrawl/anydoc](https://github.com/firecrawl/anydoc) — Convert Word, PowerPoint, Excel, OpenDocument, RTF, EPUB, CSV, and PDF to clean Markdown. Built in Rust, with Node.js and Python bindings ⭐ 21,298 · 来源：skills、tool
- [usekaneo/kaneo](https://github.com/usekaneo/kaneo) — 🎯 All you need. Nothing you don't. Open source project management that works for you, not against you ⭐ 9,058 · 来源：tool
- [frappe/hrms](https://github.com/frappe/hrms) — Open Source HR and Payroll Software ⭐ 8,783 · 来源：tool
- [ever-co/ever-gauzy](https://github.com/ever-co/ever-gauzy) — Ever® Gauzy™ - Open Business Management Platform (ERP/CRM/HRM/ATS/PM) - https://gauzy.co ⭐ 4,489 · 来源：0913
- [kevin2li/PDF-Guru](https://github.com/kevin2li/PDF-Guru) — PDF Guru Anki是你整个知识世界的“中枢转换器”，与 Anki 的强大记忆引擎无缝融合，能将来自任何地方、任何格式的知识精华，高效、系统、可持续地转化为牢固的长期记忆资产，打造专属自己的个性化Anki知识库，助你高效学习、轻松记忆。 ⭐ 4,219 · 来源：0823
- [nicejade/markdown-online-editor](https://github.com/nicejade/markdown-online-editor) — 📝 基于 Vue2、Vditor，所构建的在线 Markdown 编辑器，支持绘制流程图、甘特图、时序图、任务列表、echarts 图表、五线谱，以及 PPT 预览、视频音频解析、HTML 自动转换为 Markdown 等功能。https://www.niceshare.site ⭐ 3,968 · 来源：link-5.txt
- [yetone/cumora](https://github.com/yetone/cumora) — Where agent teams gather. Cross-platform team chat where AI agents are first-class teammates — with cloud or bring-your-own (Claude Code / Codex) brains ⭐ 3,584 · 来源：0817
- [alexishida/Moji](https://github.com/alexishida/Moji) — Open Markdown files like PDFs. A lightweight, clean desktop app for opening, reading, editing, and exporting Markdown files ⭐ 336 · 来源：db工具
- [zwc456baby/my-tv-webview](https://github.com/zwc456baby/my-tv-webview) — 我的电视 TV直播软件，极简页面支持收看国内电视直播 ⭐ 230 · 来源：0907
- [maoruibin/SlideNote](https://github.com/maoruibin/SlideNote) — Slide notes, always by your side | 侧边笔记，常伴左右 ⭐ 210 · 来源：link-5.txt
- [ricocc/rico-bookmark-manager](https://github.com/ricocc/rico-bookmark-manager) — 一个 Skill 把浏览器书签一键生成多主题的书签导航站，并支持分类、去重、查死链和导回浏览器。 ⭐ 135 · 来源：link-4.txt
- [blue-idea/linkit](https://github.com/blue-idea/linkit) — A smart bookmark & knowledge curation desktop app with AI summarization, semantic search, and cross-device sync. Built with Go + Wails + React ⭐ 47 · 来源：tool
- [runesleo/bookmark-digest](https://github.com/runesleo/bookmark-digest) — Turn X Bookmarks into a receipt-gated agent inbox ⭐ 34 · 来源：0905

<a name="cat-h"></a>

## H. Awesome 清单与资源合集（16 个）

awesome-* 榜单、项目导航、资源合集

- [public-apis/public-apis](https://github.com/public-apis/public-apis) — A collective list of free APIs ⭐ 479,572 · 来源：0817
- [awesome-selfhosted/awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted) — A list of Free Software network services and web applications which can be hosted on your own servers ⭐ 318,894 · 来源：link-5.txt
- [punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers) — A collection of MCP servers ⭐ 94,879 · 来源：0907、0911、_repos.txt
- [1c7/chinese-independent-developer](https://github.com/1c7/chinese-independent-developer) — 👩🏿‍💻👨🏾‍💻👩🏼‍💻👨🏽‍💻👩🏻‍💻中国独立开发者项目列表 -- 分享大家都在做什么 ⭐ 61,411 · 来源：link-4.txt
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — A hand-picked collection of the finest of resources for the most awesome of agents, Claude Code, the undisputed champion of coding companions, from the unstoppable team at Anthropic PBC. A delectable showcase of top tier skills, ambidextrous agents, scintillating status lines, top notch developer tooling, and also we have plugins ⭐ 53,954 · 来源：0907、0911、_repos.txt
- [GitHubDaily/GitHubDaily](https://github.com/GitHubDaily/GitHubDaily) — 坚持分享 GitHub 上高质量、有趣实用的开源技术教程、开发者工具、编程网站、技术资讯。A list cool, interesting projects of GitHub ⭐ 47,880 · 来源：0825
- [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) — A curated collection of 1000+ agent skills from official dev teams and the community, compatible with Claude Code, Codex, Gemini CLI, Cursor, and more ⭐ 34,215 · 来源：0913
- [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) — A collection of 100+ specialized Claude Code subagents covering a wide range of development use cases ⭐ 25,035 · 来源：0907、0909、0911、0913、_repos.txt
- [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) — A curated list of awesome Claude Skills, resources, and tools for customizing Claude AI workflows — particularly Claude Code ⭐ 15,047 · 来源：0907、0911、_repos.txt
- [gregorojstersek/resources-to-become-a-great-engineering-leader](https://github.com/gregorojstersek/resources-to-become-a-great-engineering-leader) — List of books, blogs, newsletters and people! ⭐ 7,556 · 来源：link-4.txt
- [WenyuChiou/awesome-agentic-ai-zh](https://github.com/WenyuChiou/awesome-agentic-ai-zh) — A trilingual (繁中 / English / 简中) learning roadmap for agentic AI: from LLM basics to multi-agent systems, with 240+ curated resources and hands-on examples. 中文 AI agent 學習地圖。 ⭐ 6,916 · 来源：0819、0827、0913、link-3.txt
- [piotrkulpinski/openalternative](https://github.com/piotrkulpinski/openalternative) — Curated list of open source alternatives to proprietary software ⭐ 6,707 · 来源：link-5.txt
- [VoltAgent/awesome-codex-subagents](https://github.com/VoltAgent/awesome-codex-subagents) — A collection of 130+ specialized Codex subagents covering a wide range of development use cases ⭐ 6,165 · 来源：link-5.txt
- [kuchin/awesome-ceo](https://github.com/kuchin/awesome-ceo) — A curated and opinionated list of resources for startup founders and leaders of high-growth companies ⭐ 2,639 · 来源：link-5.txt
- [slavakurilyak/awesome-ai-agents](https://github.com/slavakurilyak/awesome-ai-agents) — Awesome list of 300+ agentic AI resources ⭐ 2,199 · 来源：link-5.txt
- [qualisero/awesome-pi-agent](https://github.com/qualisero/awesome-pi-agent) — Awesome list of add-ons, hooks, tools, skills, and resources for the pi coding agent (pi-mono) ⭐ 1,097 · 来源：0823

<a name="cat-f"></a>

## F. Agent 可观测性、评估与治理（15 个）

会话查看、轨迹与用量分析、telemetry、审计、评测、权限治理、成本控制

- [ifixai-ai/iFixAi](https://github.com/ifixai-ai/iFixAi) — Independent Auditing of AI Agents. Run by human or the agent itself, to answer the most crucial question in the AI Agent Economy. Is the agent doing what is supposed to do? With iFixAi you can have this answer in less than 120 seconds ⭐ 14,191 · 来源：0907
- [kenn-io/agentsview](https://github.com/kenn-io/agentsview) — Local-first session search, analytics, insights, and token use statistics for coding agents, supporting Claude Code, Codex, and more than 20 other agents ⭐ 5,884 · 来源：0830、link-4.txt
- [sipyourdrink-ltd/bernstein](https://github.com/sipyourdrink-ltd/bernstein) — The open‑source AI Agents Governance & Orchestration framework: write the rules declaratively, Bernstein enforces them and produces the verifiable, replayable record. Free, Apache-2.0. https://bernstein.run ⭐ 1,173 · 来源：link-1.txt、link-2.txt
- [Willxup/cpa-usage-keeper](https://github.com/Willxup/cpa-usage-keeper) — Standalone CliProxyAPI usage tracker with SQLite persistence and built-in dashboard ⭐ 1,157 · 来源：0907
- [furkankly/zoetrope](https://github.com/furkankly/zoetrope) — Watch a Claude Code or Codex session as a live flow graph, in your terminal or your browser ⭐ 848 · 来源：0824
- [WEIFENG2333/phistory](https://github.com/WEIFENG2333/phistory) — Phistory automatically archives versioned system prompt snapshots from agent CLIs like Claude Code, Codex, OpenClaw, and Hermes ⭐ 584 · 来源：link-5.txt
- [tommy0103/obelisk](https://github.com/tommy0103/obelisk) — Every past session, subagent, and workflow -- queryable by your agent, browsable by you ⭐ 480 · 来源：0830
- [leoriczhang/teamEvolver](https://github.com/leoriczhang/teamEvolver) — teamEvolver Agent 团队能力进化控制面 把真实 Agent Session 转化为可复用、可验证、可治理的团队 Skill 与团队 Memory。 产品定位 teamEvolver 位于 Agent 运行时之外，负责团队能力的持续进化与治理。它接收真实 Session 和领域资料，提取可追溯 Evidence，生成 Ski… ⭐ 378 · 来源：0824
- [awizemann/harness](https://github.com/awizemann/harness) — AI-driven user testing for iOS Simulator, macOS apps, and web apps. Write a goal in plain language; an LLM agent drives the UI and reports friction. macOS 14+, Swift 6 ⭐ 344 · 来源：db工具
- [icesixgod/codex-trajectory](https://github.com/icesixgod/codex-trajectory) — Privacy-aware trajectory viewer for local Codex task logs ⭐ 246 · 来源：0815、0816
- [tma1-ai/tma1](https://github.com/tma1-ai/tma1) — Local-first observability your agent reads back. TMA1 records every LLM call, then routes what it sees into the agent's next turn via hooks and MCP ⭐ 118 · 来源：0830
- [Makson179/Bello](https://github.com/Makson179/Bello) — Codex is great at writing code, but it can drift during long-horizon tasks. Bello keeps it on track, guards against unsafe actions, and independently reviews the final result for bugs, completeness, and task compliance ⭐ 80 · 来源：0905
- [nujovich/hermes-telemetry](https://github.com/nujovich/hermes-telemetry) — Budget enforcement + observability plugin for Hermes Agent. Stops runaway costs before they happen ⭐ 34 · 来源：0904
- [xingkaixin/codesesh](https://github.com/xingkaixin/codesesh) — One place to see every AI coding session you've ever had ⭐ 11 · 来源：0830
- [CheerChen/session-index-viewer](https://github.com/CheerChen/session-index-viewer) — Local web viewer to browse and resume Claude Code & Codex CLI sessions. macOS, stdlib Python, no deps ⭐ 2 · 来源：0830

<a name="cat-r"></a>

## R. 语言学习与个人成长（14 个）

英语/雅思学习、翻译词典、自学与认知类内容

- [byoungd/up](https://github.com/byoungd/up) — An advanced guide which might benefit you a lot 🎉 . 韩先凯的人生进阶指南 人生进阶指南 离谱的人生 人生进阶 AI学习 AI指南 韩先凯的AI学习指南 英语学习指南/英语学习教程/英语学习/学英语 ⭐ 62,676 · 来源：link-3.txt、link-4.txt、link-5.txt
- [HKUDS/DeepTutor](https://github.com/HKUDS/DeepTutor) — DeepTutor: Lifelong Personalized Tutoring. https://deeptutor.info/ ⭐ 39,464 · 来源：0823、0909、0911、link-4.txt
- [ZuodaoTech/everyone-can-use-english](https://github.com/ZuodaoTech/everyone-can-use-english) — 人人都能用英语 ⭐ 37,479 · 来源：0827、link-5.txt
- [selfteaching/the-craft-of-selfteaching](https://github.com/selfteaching/the-craft-of-selfteaching) — One has no future if one couldn't teach themself ⭐ 16,810 · 来源：0819、0820
- [zyronon/TypeWords](https://github.com/zyronon/TypeWords) — Practice English, one strike, one step forward; 练习英语，一次敲击，一点进步； ⭐ 9,941 · 来源：0904
- [Ceelog/DictionaryByGPT4](https://github.com/Ceelog/DictionaryByGPT4) — 一本 GPT4 生成的单词书📚，超过 8000 个单词分析，涵盖了词义、例句、词根词缀、变形、文化背景、记忆技巧和小故事 ⭐ 6,361 · 来源：link-4.txt
- [knowledgefxg/learning-english](https://github.com/knowledgefxg/learning-english) — 精选优质英语学习资源合集，专注于听说读写等核心技能的提升。包含语法、词汇和媒体资源，助您更好地学习英语。 ⭐ 4,186 · 来源：0911
- [echo-loop/Echo-Loop](https://github.com/echo-loop/Echo-Loop) — Echo Loop 是一款科学、高效的 AI 英语听说训练 App，通过精听、跟读、盲听、复述和间隔复习，自动驱动学习者把每一段音频真正练懂、练熟、练到会说。 ⭐ 3,493 · 来源：0819、0820
- [hefengxian/my-ielts](https://github.com/hefengxian/my-ielts) — 雅思词汇真经、雅思语法、听力 179、阅读 538 同义替换等。Everything during preparing for my IELTS exam ⭐ 3,256 · 来源：link-4.txt
- [llwslc/grammar-club](https://github.com/llwslc/grammar-club) — 《语法俱乐部》- 旋元佑 ⭐ 2,263 · 来源：0913
- [shaogefenhao/a-programmer-s-cognitive-experience](https://github.com/shaogefenhao/a-programmer-s-cognitive-experience) — A e-book about "A programmer's cognitive experience" 《程序员的认知心得》 ⭐ 123 · 来源：0821
- [shaogefenhao/life-game-book](https://github.com/shaogefenhao/life-game-book) — 《人生游戏：自我觉醒、社会规则和执行力》一本关于自我认知（生命和意识起源）和社会游戏规则以及如何自我提升的电子书。 ⭐ 117 · 来源：0821
- [VeejaLiu/duolinting](https://github.com/VeejaLiu/duolinting) — An open-source intensive-listening learning and content-production platform ⭐ 34 · 来源：0815、0816
- [Edwardxlai/ai-timed-challenge](https://github.com/Edwardxlai/ai-timed-challenge) — 名词轮盘 · AI 限时挑战 抽一个 AI 名词，限时学完讲出来，写下自己的理解，再跟标准答案对——差的那块就是今天真正学到的东西。 讲过的词不会再抽到，笔记和标准答案一起存下来，可以导出 Markdown 塞进自己的笔记库。 跑起来 浏览器不允许 file:// 读本地 JSON，所以要起个本地服务： python -m http.se… ⭐ 2 · 来源：0824、net

<a name="cat-i"></a>

## I. 大模型基础：训练、推理与多模态（13 个）

LLM 训练/推理、强化学习、语音合成与识别、OCR、端侧模型、图像视频生成

- [jamiepine/voicebox](https://github.com/jamiepine/voicebox) — The open-source AI voice studio. Clone, dictate, create ⭐ 53,092 · 来源：link-5.txt
- [google-ai-edge/mediapipe](https://github.com/google-ai-edge/mediapipe) — Cross-platform, customizable ML solutions for live and streaming media ⭐ 36,926 · 来源：link-5.txt
- [JustVugg/colibri](https://github.com/JustVugg/colibri) — Run frontier MoE models on hardware you already own — pure C, zero deps, experts streamed from disk. Tiny engine, immense model. 🐦 ⭐ 28,793 · 来源：link-3.txt
- [baidu/Unlimited-OCR](https://github.com/baidu/Unlimited-OCR) — Unlimited OCR Works: Welcome the Era of One-shot Long-horizon Parsing ⭐ 25,559 · 来源：0819
- [Huanshere/VideoLingo](https://github.com/Huanshere/VideoLingo) — Netflix-level subtitle cutting, translation, alignment, and even dubbing - one-click fully automated AI video subtitle team | Netflix级字幕切割、翻译、对齐、甚至加上配音，一键全自动视频搬运AI字幕组 ⭐ 18,429 · 来源：link-5.txt
- [supertone-inc/supertonic](https://github.com/supertone-inc/supertonic) — Lightning-Fast, On-Device, Multilingual TTS — running natively via ONNX ⭐ 13,774 · 来源：0823
- [huggingface/speech-to-speech](https://github.com/huggingface/speech-to-speech) — Build voice agents with open-source models ⭐ 13,177 · 来源：0817
- [cactus-compute/needle](https://github.com/cactus-compute/needle) — 14MB foundation model for tiny devices; phones, wearables, smart home, and robots ⭐ 10,941 · 来源：0816
- [huangserva/ComfyUI_MiniMaxH3_Director](https://github.com/huangserva/ComfyUI_MiniMaxH3_Director) — ComfyUI MiniMax H3 Director workflow ⭐ 991 · 来源：方法论
- [perplexityai/pplx-garden](https://github.com/perplexityai/pplx-garden) — Perplexity open source garden for inference technology ⭐ 910 · 来源：0907
- [Blue-B/WhisperSubTranslate](https://github.com/Blue-B/WhisperSubTranslate) — A free, local desktop app to extract subtitles (SRT) from video and translate them into any language — unlimited use, no signup, no cloud ⭐ 691 · 来源：0821
- [Danau5tin/ai-trains-ai](https://github.com/Danau5tin/ai-trains-ai) — RL-training an AI agent to RL-train AI agents ⭐ 238 · 来源：link-5.txt
- [memovai/mimimodel](https://github.com/memovai/mimimodel) — MimiModel: Agentic LLM on a $5 chip. 100% on-device. Private by design ⭐ 90 · 来源：0823

<a name="cat-m"></a>

## M. 网络、代理与自托管服务（12 个）

反向代理、LLM API 网关、流量看板、网络扫描与拓扑、订阅转换

- [diegosouzapw/OmniRoute](https://github.com/diegosouzapw/OmniRoute) — Never stop coding. Free MIT AI gateway: one endpoint, 352 providers (150+ free), 1200+ models Kimi, Claude, GPT, Gemini, GLM, DeepSeek, MiniMax. Works with Claude Code, Codex, Cursor, OpenCode, Cline & Copilot. Quota-aware auto-fallback, RTK+Caveman compression saves 15-95% tokens, MCP/A2A, Desktop/PWA. Built by 550+ contributors ⭐ 65,456 · 来源：link-3.txt、link-5.txt
- [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) — Wrap Antigravity, ChatGPT Codex, Claude Code, Grok Build as an OpenAI/Gemini/Claude/Codex compatible API service, allowing you to enjoy the free Gemini 3.1 Pro, GPT 5.6 Series, Grok 4.5, Claude model through API ⭐ 51,604 · 来源：link-4.txt
- [aceberg/WatchYourLAN](https://github.com/aceberg/WatchYourLAN) — Lightweight network IP scanner written in Go. With notifications, history, export to Grafana ⭐ 7,624 · 来源：net
- [scanopy/scanopy](https://github.com/scanopy/scanopy) — Network diagrams that update themselves ⭐ 5,722 · 来源：0826
- [yusing/godoxy](https://github.com/yusing/godoxy) — High-performance reverse proxy and container orchestrator for self-hosters ⭐ 4,152 · 来源：net
- [foru17/neko-master](https://github.com/foru17/neko-master) — A modern and elegant dashboard for network traffic visualization and analysis ⭐ 3,943 · 来源：link-4.txt
- [bestruirui/octopus](https://github.com/bestruirui/octopus) — One Hub All LLMs For You | 为个人打造的 LLM API 聚合网关 ⭐ 2,623 · 来源：0826
- [asdlokj1qpi233/subconverter](https://github.com/asdlokj1qpi233/subconverter) — About Utility to convert between various subscription format.Support anytls、mieru、hy2、hy and vless for singbox and clash meta.original git: https://github.com/asdlokj1qpi23/subconverter ⭐ 1,802 · 来源：0909
- [ShadowArcanist/netviz](https://github.com/ShadowArcanist/netviz) — A browser-based app for designing network architectures visually ⭐ 977 · 来源：0817、0827
- [www222fff/free-router](https://github.com/www222fff/free-router) — Free Router English | 中文 Local OpenAI-compatible gateway. Point any client at ` http://127.0.0.1:8787/v1` and use `free-best`. It ranks currently free models across **any… ⭐ 420 · 来源：0909
- [aipayim/codex-proxy](https://github.com/aipayim/codex-proxy) — API proxy tool ⭐ 176 · 来源：0827
- [Niklaus88/Clash-Config](https://github.com/Niklaus88/Clash-Config) — Clash 系代理配置 （解决手机端/桌面端 DNS/WebRTC 泄露问题） ⭐ 37 · 来源：0913

<a name="cat-o"></a>

## O. 设计与可视化（11 个）

设计系统、白板与思维导图、流程图/架构图工具、动画与可视化引擎

- [3b1b/manim](https://github.com/3b1b/manim) — Animation engine for explanatory math videos ⭐ 93,820 · 来源：0904
- [tt-a1i/archify](https://github.com/tt-a1i/archify) — Agent skill for beautiful, verifiable architecture, workflow, sequence, data-flow, and lifecycle diagrams—self-contained HTML with motion and crisp export ⭐ 60,102 · 来源：link-3.txt
- [penpot/penpot](https://github.com/penpot/penpot) — Penpot: The open-source design platform for Product teams that need scalable collaboration ⭐ 59,932 · 来源：0830
- [cathrynlavery/diagram-design](https://github.com/cathrynlavery/diagram-design) — 38 editorial diagram types for Claude Code, Codex, and Pi. Self-contained HTML + SVG. No shadows. No Mermaid slop ⭐ 39,002 · 来源：0909
- [plait-board/drawnix](https://github.com/plait-board/drawnix) — 开源白板工具（SaaS），一体化白板，包含思维导图、流程图、自由画等。All in one open-source whiteboard tool with mind, flowchart, freehand and etc ⭐ 14,708 · 来源：link-5.txt
- [didi/LogicFlow](https://github.com/didi/LogicFlow) — A flow chart editing framework focus on business customization. 专注于业务自定义的流程图编辑框架，支持实现脑图、ER图、UML、工作流等各种图编辑场景。 ⭐ 11,690 · 来源：tool
- [Agents365-ai/drawio-skill](https://github.com/Agents365-ai/drawio-skill) — Agent skill that turns natural language, code, Terraform/K8s, SQL, OpenAPI, AsyncAPI and Protobuf sources into editable, tested draw.io architecture diagrams: incremental sync, multi-view projection, drift diff, CI architecture tests, whiteboard derasterize, interactive HTML/PPTX/Mermaid exports ⭐ 9,286 · 来源：0830、0905
- [braedonsaunders/codeflow](https://github.com/braedonsaunders/codeflow) — Paste any GitHub URL → interactive architecture map. See how files connect, find what breaks if you change something. No install, no accounts — runs entirely in your browser ⭐ 5,196 · 来源：link-4.txt
- [Trystan-SA/claude-design-system-prompt](https://github.com/Trystan-SA/claude-design-system-prompt) — Reverse-engineered system prompt and skill library that turns an LLM into an opinionated, accessibility-aware, AI-slop-resistant design collaborator ⭐ 1,944 · 来源：link-3.txt
- [yizhiyanhua-ai/fireworks-open-eli5](https://github.com/yizhiyanhua-ai/fireworks-open-eli5) — Evidence-aware interactive visual explainers for Codex and Claude Code ⭐ 187 · 来源：0824
- [qiuyiwu1989-star/opendesign](https://github.com/qiuyiwu1989-star/opendesign) — 1,486 real web design systems, extracted into machine-readable specs. An MCP server for AI coding agents — so what they build has taste, not just working code ⭐ 70 · 来源：harness

<a name="cat-k"></a>

## K. 数据、数据库与数据分析（10 个）

数据库客户端/IDE、text-to-SQL 与 GenBI、数据加载与数据工程

- [OtterMind/Chat2DB](https://github.com/OtterMind/Chat2DB) — Chat2DB is a free, cross-platform, local-first database client and SQL workspace for developers, DBAs, analysts, and data teams. Connect to 40+ databases, manage data, edit and run SQL, and use your own AI model to generate, explain, and optimize queries. Available on desktop, web, Docker, and CLI, with MCP support ⭐ 28,115 · 来源：db工具、link-5.txt
- [matomo-org/matomo](https://github.com/matomo-org/matomo) — Empowering People Ethically 🚀 — Matomo is hiring! Join us → https://matomo.org/jobs Matomo is the leading open-source alternative to Google Analytics, giving you complete control and built-in privacy. Easily collect, visualise, and analyse data from websites & apps. Star us on GitHub ⭐️ – Pull Requests welcome! ⭐ 21,861 · 来源：0827、0830
- [t8y2/dbx](https://github.com/t8y2/dbx) — 20 MB lightweight cross-platform database client for 90+ databases, including MySQL, PostgreSQL, SQLite, Redis, MongoDB, DuckDB, SQL Server, and Dameng. Built-in AI, MCP Server, CLI, desktop and Docker. | 轻量级跨平台数据库管理工具，支持 MySQL、PostgreSQL、SQLite、Redis、MongoDB、达梦等 90+ 数据库，提供桌面端、Docker、CLI、内置 AI 助手和 MCP Server。 ⭐ 19,296 · 来源：0819、tool
- [Canner/WrenAI](https://github.com/Canner/WrenAI) — GenBI (Generative BI) for AI agents, an open-source, governed text-to-SQL through an open context layer that turns natural-language questions into trusted dashboards, charts, and SQL across 20+ data sources, such as BigQuery, Snowflake, PostgreSQL, ClickHouse, Amazon Redshift, Databricks and more ⭐ 17,595 · 来源：db工具
- [dlt-hub/dlt](https://github.com/dlt-hub/dlt) — data load tool (dlt) is an open source Python library that makes data loading easy 🛠️ ⭐ 5,845 · 来源：tool
- [clidey/whodb](https://github.com/clidey/whodb) — Where data access meets operational intelligence ⭐ 5,024 · 来源：db工具
- [libredb/libredb-studio](https://github.com/libredb/libredb-studio) — One browser tab for PostgreSQL, MySQL, Oracle, SQL Server, MongoDB, Redis, SQLite, Couchbase, ClickHouse, Druid, DuckDB, Turso and more. An open-source SQL IDE with SSO, audit trail and AI-assisted queries MIT licensed, with nothing held back behind an enterprise wall ⭐ 674 · 来源：0816
- [peisp/catdb](https://github.com/peisp/catdb) — Cross-platform database client based on Wails3 (Go + Vue 3 + WebView). 基于 Wails v3（Go + Vue 3 + WebView）的跨平台数据库客户端。 ⭐ 32 · 来源：db工具
- [lanerchenbuna/QueryForge](https://github.com/lanerchenbuna/QueryForge) — Governed AI analytics from natural language to auditable SQL, powered by a mandatory semantic layer, policy enforcement, and a visual Studio. 从自然语言到可审计 SQL 的受治理 AI 数据分析平台，内置强制语义层、SQL 策略治理与可视化工作台。 ⭐ 13 · 来源：link-5.txt
- [NemoAlex/SQLTunnel](https://github.com/NemoAlex/SQLTunnel) — SQLTunnel A controlled database gateway for agents, automation platforms, and internal applications English | 中文 | 日本語 | 한국어 | Français | Deutsch SQLTunnel lets Codex, Cl… ⭐ 3 · 来源：db工具

<a name="cat-n"></a>

## N. 安全、抓包与逆向（10 个）

渗透测试、抓包分析、Wi-Fi/网络攻击、逆向工程、安全能力库

- [usestrix/strix](https://github.com/usestrix/strix) — Open-source AI penetration testing tool to find and fix your app’s vulnerabilities ⭐ 62,157 · 来源：0826
- [zhaoxuya520/reverse-skill](https://github.com/zhaoxuya520/reverse-skill) — Reverse Engineering / Authorized Penetration Testing / Security Research Skill Router Pack AI-powered routing + On-demand toolchain bootstrapping + Self-evolving knowledge base Supports Claude Code, Kiro, Cursor, Cline, and other AI coding clients 逆向/渗透/安全技能路由包 - AI 自动路由 + 按需自举工具链 + 自动进化经验库 | 支持 Claude Code / Kiro / Cursor / Cline 等代码 AI 客户端 ⭐ 35,721 · 来源：抓包
- [mukul975/Anthropic-Cybersecurity-Skills](https://github.com/mukul975/Anthropic-Cybersecurity-Skills) — 817 structured cybersecurity skills for AI agents · Mapped to 6 frameworks: MITRE ATT&CK, NIST CSF 2.0, MITRE ATLAS, D3FEND, NIST AI RMF & MITRE F3 (Fight Fraud) · agentskills.io standard · Works with Claude Code, GitHub Copilot, Codex CLI, Cursor, Gemini CLI & 20+ platforms · 29 security domains · Apache 2.0 ⭐ 32,699 · 来源：0820
- [bettercap/bettercap](https://github.com/bettercap/bettercap) — The Swiss Army knife for 802.11, BLE, HID, CAN-bus, IPv4 and IPv6 networks reconnaissance and MITM attacks ⭐ 19,971 · 来源：tool
- [emanuele-f/PCAPdroid](https://github.com/emanuele-f/PCAPdroid) — No-root network monitor, firewall and PCAP dumper for Android ⭐ 4,712 · 来源：0820、抓包
- [V33RU/awesome-connected-things-sec](https://github.com/V33RU/awesome-connected-things-sec) — A Curated list of Security Resources for all connected things ⭐ 3,536 · 来源：0816
- [wzhudev/reverse-linear-sync-engine](https://github.com/wzhudev/reverse-linear-sync-engine) — A reverse engineering of Linear's sync engine ⭐ 2,809 · 来源：0907
- [sinyu1012/proxypin-mcp-workbench](https://github.com/sinyu1012/proxypin-mcp-workbench) — 基于 ProxyPin 与 MCP 的自动化抓包、API 分析、Mock 场景和数据归档工作台，支持 移动端/Mac端 抓包并通过 MCP 让 AI 自动分析获取内容 / Mock 一系列接口。 ⭐ 48 · 来源：0823
- [xianyu110/grok-bot-0.18-reconstructed](https://github.com/xianyu110/grok-bot-0.18-reconstructed) — Unofficial source-oriented reconstruction and extension of Grok Bot 0.18.0 for macOS ⭐ 39 · 来源：0824
- [overspace-labs/amber](https://github.com/overspace-labs/amber) — Read and write Burp Suite Proxy HTTP history, byte for byte ⭐ 28 · 来源：0824、0826

<a name="cat-q"></a>

## Q. 硬件、IoT 与机器人（10 个）

物联网平台与设备、智能家居、ESP32、无人机、工业监测、CAD

- [home-assistant/core](https://github.com/home-assistant/core) — :house_with_garden: Open source home automation that puts local control and privacy first ⭐ 90,400 · 来源：link-4.txt
- [Koenkk/zigbee2mqtt](https://github.com/Koenkk/zigbee2mqtt) — Zigbee 🐝 to MQTT bridge 🌉, get rid of your proprietary Zigbee bridges 🔨 ⭐ 15,631 · 来源：0817
- [jetlinks/jetlinks-community](https://github.com/jetlinks/jetlinks-community) — JetLinks 基于Java,Spring Boot ,WebFlux,Netty,Vert.x,Reactor等开发, 是一个全响应式的企业级物联网平台。支持统一物模型管理,多种设备,多种厂家,统一管理。统一设备连接管理,多协议适配(TCP,MQTT,UDP,CoAP,HTTP等),屏蔽网络编程复杂性,灵活接入不同厂家不同协议等设备。实时数据处理,设备告警,消息通知,数据转发。地理位置,数据可视化等。能帮助你快速建立物联网相关业务系统。 ⭐ 6,626 · 来源：0810、link-5.txt
- [slvDev/esp32-ai](https://github.com/slvDev/esp32-ai) — Running a 28.9M parameter LLM on a microcontroller 𝕏 slvDev · LinkedIn This is a 28.9 million parameter language model that generates text on an ESP32-S3 microcontroller.… ⭐ 4,337 · 来源：link-5.txt
- [kritishmohapatra/100_Days_100_IoT_Projects](https://github.com/kritishmohapatra/100_Days_100_IoT_Projects) — A 100-day challenge exploring IoT and embedded systems using ESP32, ESP8266, and Raspberry Pi Pico with MicroPython. Each day covers a new sensor or module with complete code, circuit diagram, and explanation ⭐ 1,197 · 来源：0823、link-5.txt
- [DroneBridge/ESP32](https://github.com/DroneBridge/ESP32) — DroneBridge for ESP32. A secure & transparent telemetry link with support for WiFi and ESP-NOW. Supporting MAVLink, MSP, LTM or any other protocol ⭐ 1,092 · 来源：link-5.txt
- [40rbidd3n/Hydro0x01](https://github.com/40rbidd3n/Hydro0x01) — Secure, production-grade IoT hydroponic automation system with ESP32, MQTT telemetry, and real-time dashboard ⭐ 539 · 来源：link-5.txt
- [bluegrassiot/mqttprobe](https://github.com/bluegrassiot/mqttprobe) — The MQTT diagnostic tool built for IIoT. Connect to any MQTT broker, browse live topic trees, inspect payloads, and chart JSON telemetry in real time. Native Sparkplug B decode and EoN node emulation built in. Open source, no cloud required ⭐ 37 · 来源：link-5.txt
- [bsidio/unifi-protect-airquality-dashboard](https://github.com/bsidio/unifi-protect-airquality-dashboard) — Realtime dashboard for the UniFi UP-AirQuality sensor, with history in ClickHouse. Reads air quality the public Protect API cannot expose ⭐ 6 · 来源：agent开发
- [Itachi7011/Machinexis](https://github.com/Itachi7011/Machinexis) — Multi-tenant predictive maintenance platform for managing industrial equipment, sensors, alerts, maintenance workflows, and real-time operational insights ⭐ 5 · 来源：0816

<a name="cat-s"></a>

## S. 垂直行业应用（10 个）

金融交易、电商、招聘求职、新闻资讯、GIS、行业解决方案

- [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading) — "Vibe-Trading: Your Personal Trading Agent" ⭐ 33,336 · 来源：0911
- [zgwl/chinese-buy-us-stock-guide](https://github.com/zgwl/chinese-buy-us-stock-guide) — 美股指南 ⭐ 7,125 · 来源：link-4.txt
- [codeman008/Financial_freedom](https://github.com/codeman008/Financial_freedom) — Technical guide to making money and investing（最全赚钱投资指南） ⭐ 3,762 · 来源：skills
- [powerycy/goutoujunshi](https://github.com/powerycy/goutoujunshi) — 一个先接住情绪、再分析关系并给出可执行策略的 Codex 恋爱军师，内置心理、法律、社会、人文、哲学、婚姻家庭与性学知识库，支持多元关系。 ⭐ 2,915 · 来源：0905
- [can4hou6joeng4/boss-agent-cli](https://github.com/can4hou6joeng4/boss-agent-cli) — 🤖 Local-assist BOSS Zhipin CLI for AI agents — search, welfare filtering, shortlist, JSON-envelope output; low-risk & compliant by default ⭐ 1,950 · 来源：0904
- [andrew-shwetzer/career-ops-plugin](https://github.com/andrew-shwetzer/career-ops-plugin) — Claude Cowork plugin for job seekers. 9 AI skills: evaluate job postings, generate ATS-optimized resumes, scan company career portals, track applications, draft outreach. Works in any industry ⭐ 491 · 来源：link-4.txt
- [AwaisShah75/Real-Time-Person-Elderly-Fall-Detection-System](https://github.com/AwaisShah75/Real-Time-Person-Elderly-Fall-Detection-System) — 🛡️ Real-Time Person & Elderly Fall Detection System Advanced Occlusion Resilience, Kinematic State Machine & Edge AI Architecture An enterprise-grade, edge-compatible AI… ⭐ 242 · 来源：0816、0817
- [slothsheepking/jobclaw](https://github.com/slothsheepking/jobclaw) — 🦞 AI-powered job hunting agent — scrapes Boss直聘/LinkedIn, matches your profile, auto-applies. Built with OpenClaw ⭐ 226 · 来源：tool
- [Eleven617/mall-ai-after-sales-platform](https://github.com/Eleven617/mall-ai-after-sales-platform) — Trusted e-commerce AI after-sales platform — RAG, controlled tools, human confirmation, Java-authoritative writes, and observable workflows ⭐ 114 · 来源：0911
- [jinit-00/PulseNews-Live-AI-News-Streaming-Platform](https://github.com/jinit-00/PulseNews-Live-AI-News-Streaming-Platform) — A real-time distributed AI news ingestion and analytics platform built using Kafka, FastAPI, RAG-based AI search, and live WebSocket streaming. Pulse News processes live news from multiple sources, enables AI-powered chat over news data, and provides real-time analytics using Elasticsearch and Kibana ⭐ 16 · 来源：0819

<a name="cat-t"></a>

## T. 浏览器自动化与数据采集（10 个）

浏览器 agent、网页抓取、真实浏览器控制、爬虫工具

- [browser-use/browser-use](https://github.com/browser-use/browser-use) — Agents that use the browser ⭐ 114,423 · 来源：0827、0830
- [unclecode/crawl4ai](https://github.com/unclecode/crawl4ai) — 🚀🤖 Crawl4AI: Open-source LLM Friendly Web Crawler & Scraper. Don't be shy, join here: https://discord.gg/jP8KfhDhyN ⭐ 82,886 · 来源：link-3.txt
- [Panniantong/Agent-Reach](https://github.com/Panniantong/Agent-Reach) — Give your AI agent eyes to see the entire internet. Read & search Twitter, Reddit, YouTube, GitHub, Bilibili, XiaoHongShu — one CLI, zero API fees ⭐ 80,053 · 来源：0821
- [browser-use/web-ui](https://github.com/browser-use/web-ui) — 🖥️ Run AI Agent in your browser ⭐ 16,326 · 来源：0827
- [nanobrowser/nanobrowser](https://github.com/nanobrowser/nanobrowser) — Open-Source Chrome extension for AI-powered web automation. Run multi-agent workflows using your own LLM API key. Alternative to OpenAI Operator ⭐ 13,776 · 来源：0822、0827
- [hangwin/mcp-chrome](https://github.com/hangwin/mcp-chrome) — Chrome MCP Server is a Chrome extension-based Model Context Protocol (MCP) server that exposes your Chrome browser functionality to AI assistants like Claude, enabling complex browser automation, content analysis, and semantic search ⭐ 12,405 · 来源：link-3.txt
- [browser-act/skills](https://github.com/browser-act/skills) — Browser automation CLI built for AI agents. Break through anti-bot walls, hand off to humans across platforms when stuck. Parallel multi-task execution, independent multi-session operation, isolated multi-account browsing ⭐ 5,908 · 来源：tool
- [vibheksoni/stealth-browser-mcp](https://github.com/vibheksoni/stealth-browser-mcp) — The only browser automation that bypasses anti-bot systems. AI writes network hooks, clones UIs pixel-perfect via simple chat ⭐ 2,055 · 来源：0827
- [Tencent/BrowserSkill](https://github.com/Tencent/BrowserSkill) — Let AI agents use your real, logged-in browser without interrupting your work. CLI + extension for browser automation across any shell-capable AI agent ⭐ 1,962 · 来源：link-3.txt
- [hrithikkoduri/WebRover](https://github.com/hrithikkoduri/WebRover) — WebRover is an autonomous AI agent designed to interpret user input and execute actions by interacting with web elements to accomplish tasks or answer questions. It leverages advanced language models and web automation tools to navigate the web, gather information, and provide structured responses based on the user's needs ⭐ 1,023 · 来源：tool

<a name="cat-j"></a>

## J. 提示词工程与 AI 应用（9 个）

提示词管理/优化、通用 AI 聊天与助手应用、AI 应用示例集

- [f/prompts.chat](https://github.com/f/prompts.chat) — f.k.a. Awesome ChatGPT Prompts. Share, discover, and collect prompts from the community. Free and open source — self-host for your organization with complete privacy ⭐ 170,191 · 来源：0905
- [open-webui/open-webui](https://github.com/open-webui/open-webui) — User-friendly AI Interface (Supports Ollama, OpenAI API, ...) ⭐ 151,824 · 来源：0904
- [x1xhlol/system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools) — FULL Augment Code, Claude Code, Cluely, CodeBuddy, Comet, Cursor, Devin AI, Junie, Kiro, Leap.new, Lovable, Manus, NotionAI, Orchids.app, Perplexity, Poke, Qoder, Replit, Same.dev, Trae, Traycer AI, VSCode Agent, Warp.dev, Windsurf, Xcode, Z.ai Code, Dia & v0. (And other Open Sourced) System Prompts, Internal Tools & AI Models ⭐ 143,582 · 来源：agent开发
- [Shubhamsaboo/awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps) — 100+ AI Agents, Agent Skills and RAG Apps - Free and Open Source ⭐ 137,800 · 来源：0911、link-3.txt、link-4.txt
- [linshenkx/prompt-optimizer](https://github.com/linshenkx/prompt-optimizer) — An AI prompt optimizer for writing better prompts and getting better AI results ⭐ 34,591 · 来源：0905
- [andrewyng/openworker](https://github.com/andrewyng/openworker) — OpenWorker openworker.com · Download · Issues Beta - OpenWorker is in open beta: fully usable, updates itself, and we're actively polishing rough edges. Issues welcome. A… ⭐ 17,617 · 来源：agent开发
- [rikkahub/rikkahub](https://github.com/rikkahub/rikkahub) — RikkaHub is an Android APP that supports for multiple LLM providers ⭐ 7,587 · 来源：0826
- [skalesapp/skales](https://github.com/skalesapp/skales) — Personal AI desktop agent for Windows, macOS, Linux, Android & iOS. Set a goal, it works on its own. Teams (pair two desktops, agents + humans), Agent2Agent, Workflows, Codework, multi-agent orgs, desktop + browser automation. 15+ AI providers, BYOK. No Docker, no terminal. Agent Skills (SKILL.md). Migration importer. Recurring autonomous tasks ⭐ 1,870 · 来源：0913
- [yarin-zhang/AI-Gist](https://github.com/yarin-zhang/AI-Gist) — ✨ AI Gist 是一款隐私优先的 AI 提示词管理工具，致力于让个人收藏的 AI 提示词能够发挥最大价值。支持变量替换、Jinja 模板、AI 生成与调优、历史版本记录、云端备份等核心功能。 ⭐ 875 · 来源：0822

<a name="cat-broken"></a>

## 失效链接（2 个）

- `aaa/docs` — 链接已失效（404），仓库已不存在或转为私有
- `giswqs/intro-gispr` — 链接已失效（404），仓库已不存在或转为私有

