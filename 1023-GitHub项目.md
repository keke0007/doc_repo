# 1023 · GitHub 项目整理

来源：[原始笔记 1023](<1023>)。从原始笔记直接提取并去重，共 **40 个项目**；保留首次出现顺序的编号。

整理日期：**2026-10-25（北京时间）**。简介根据 GitHub 公开仓库信息整理；无简介或简介过短的项目补读官方 README。Star 数是抓取时的快照，不代表质量。本文不下载、安装或运行这些项目，也不扩展收录 X 帖子中额外出现的仓库。

| 分类 | 数量 |
| --- | ---: |
| 记忆、知识库与笔记 | 6 |
| Agent 平台、协作与复核 | 6 |
| 浏览器、测试与操作录制 | 4 |
| 设计与 Figma | 2 |
| API、隔离与安全工具 | 4 |
| 媒体、办公与游戏 | 3 |
| 传统文化与娱乐类 Skills | 15 |

## 记忆、知识库与笔记

| 编号 | 项目 | 用途简介 | 语言 | 许可证 | Star |
| ---: | --- | --- | --- | --- | ---: |
| 1 | [Tencent-Hunyuan/Hy-Memory](https://github.com/Tencent-Hunyuan/Hy-Memory) | 面向 LLM Agent 的双系统认知记忆引擎与 Python SDK，并提供跨应用记忆产品 MemHub。 | Python | 需查看仓库 LICENSE | 1,268 |
| 3 | [zenghui-li/yuxi](https://github.com/zenghui-li/yuxi) | 结合 LightRAG 知识库与知识图谱的多租户 Agent Harness 平台。 | 未标注 | MIT | 173 |
| 6 | [freestylefly/wesight-obsidian](https://github.com/freestylefly/wesight-obsidian) | 为本地 Agent 运行时提供支持的 WeSight Obsidian 插件。 | TypeScript | 需查看仓库 LICENSE | 461 |
| 14 | [nashsu/llm_wiki](https://github.com/nashsu/llm_wiki) | 跨平台桌面应用，让 LLM 从文档增量构建并维护相互关联的持久知识 Wiki。 | TypeScript | 需查看仓库 LICENSE | 20,953 |
| 16 | [Asymptote-Labs/agent-beacon](https://github.com/Asymptote-Labs/agent-beacon) | 跨 Harness 的 Agent 记忆层，项目关注从使用中持续改进记忆。 | Go | MIT | 2,785 |
| 19 | [quickwit-oss/tantivy](https://github.com/quickwit-oss/tantivy) | 受 Apache Lucene 启发、使用 Rust 编写的全文搜索引擎库。 | Rust | MIT | 16,329 |

## Agent 平台、协作与复核

| 编号 | 项目 | 用途简介 | 语言 | 许可证 | Star |
| ---: | --- | --- | --- | --- | ---: |
| 9 | [AsaSuisei/four-powers](https://github.com/AsaSuisei/four-powers) | 用编排者、执行者、审查者和高级审查者分离职责，约束编码 Agent 的自我评审。 | Shell | MIT | 2 |
| 13 | [langfuse/langfuse](https://github.com/langfuse/langfuse) | 开源 LLM 应用和 Agent 可观测性与评估平台，用于追踪、评估和改进应用。 | TypeScript | 需查看仓库 LICENSE | 36,413 |
| 17 | [Tencent/teamai-cli](https://github.com/Tencent/teamai-cli) | 面向团队 AI 原生协作的命令行项目。 | TypeScript | 需查看仓库 LICENSE | 5,645 |
| 20 | [iflytek/astron-agent](https://github.com/iflytek/astron-agent) | 面向企业、强调商业友好的 Agent 工作流与 SuperAgent 构建平台。 | Java | Apache-2.0 | 8,924 |
| 38 | [tigerless-labs/autoharness](https://github.com/tigerless-labs/autoharness) | 从真实 Claude Code 会话提炼、更新并裁减 Skills 的自学习技能层。 | Python | MIT | 10,727 |
| 40 | [ninetyhe-90/dsh-decision-review](https://github.com/ninetyhe-90/dsh-decision-review) | DeepSeek Harness 独立复核工具：第二模型提供建议，主模型保留决策权。 | JavaScript | MIT | 32 |

## 浏览器、测试与操作录制

| 编号 | 项目 | 用途简介 | 语言 | 许可证 | Star |
| ---: | --- | --- | --- | --- | ---: |
| 2 | [tester-army/e2e](https://github.com/tester-army/e2e) | 面向 Web 和移动应用的端到端测试框架；源笔记重点关注自然语言目标驱动测试与可复跑性。 | TypeScript | Apache-2.0 | 11,862 |
| 7 | [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser) | 面向 AI Agent 的浏览器自动化命令行工具。 | Rust | Apache-2.0 | 44,555 |
| 18 | [microsoft/skill-recorder](https://github.com/microsoft/skill-recorder) | 录制屏幕操作，并利用 GitHub Copilot SDK 将意图与步骤整理成可复用 Skill 或 Automation。 | TypeScript | MIT | 4,810 |
| 21 | [browser-act/skills](https://github.com/browser-act/skills) | 面向 AI Agent 的浏览器自动化 CLI，提供多任务、多会话和人工接手等能力。 | Python | MIT | 6,274 |

## 设计与 Figma

| 编号 | 项目 | 用途简介 | 语言 | 许可证 | Star |
| ---: | --- | --- | --- | --- | ---: |
| 10 | [Duoasa/DSH-to-Figma](https://github.com/Duoasa/DSH-to-Figma) | 为 DeepSeek Harness 接入 Figma 原生画布读取、编辑、组件、变量和素材能力。 | HTML | MIT | 18 |
| 11 | [deepfounder-ai/desigma](https://github.com/deepfounder-ai/desigma) | 基于 Rust 的免费 Figma 替代项目。 | Rust | Apache-2.0 | 575 |

## API、隔离与安全工具

| 编号 | 项目 | 用途简介 | 语言 | 许可证 | Star |
| ---: | --- | --- | --- | --- | ---: |
| 4 | [tashfeenahmed/freellmapi](https://github.com/tashfeenahmed/freellmapi) | 聚合免费 LLM 接口，提供统一入口、路由与故障切换；项目定位为个人实验用途。 | TypeScript | MIT | 41,139 |
| 5 | [arxhr007/Aliens_eye](https://github.com/arxhr007/Aliens_eye) | AI 辅助 OSINT 工具，用于发现和映射公开网络上的社交媒体身份。 | Python | MIT | 4,531 |
| 8 | [opentrap/opentrap](https://github.com/opentrap/opentrap) | 开源拦截代理和 Web 应用安全测试套件，包含重放、攻击测试和扫描工具。 | Java | AGPL-3.0 | 416 |
| 12 | [anthropics/sandbox-runtime](https://github.com/anthropics/sandbox-runtime) | 在操作系统层为进程施加文件系统与网络限制的轻量级隔离工具，无需容器。 | TypeScript | Apache-2.0 | 6,539 |

## 媒体、办公与游戏

| 编号 | 项目 | 用途简介 | 语言 | 许可证 | Star |
| ---: | --- | --- | --- | --- | ---: |
| 15 | [kajisho5/ffmpeg-skill](https://github.com/kajisho5/ffmpeg-skill) | 面向编码 Agent 的本地 FFmpeg 视频编辑工具与 MCP 服务，支持裁剪、字幕和导出。 | Python | MIT | 2,658 |
| 22 | [peach-mitao/peach](https://github.com/peach-mitao/peach) | 单用户、本地优先的个人媒体系统「Peach 蜜桃」。 | Python | AGPL-3.0 | 11,276 |
| 39 | [jhaizhou-ops/d2r-mobile](https://github.com/jhaizhou-ops/d2r-mobile) | 《暗黑破坏神 II：狱火重生》的开源兼容层；需要用户自行提供正版游戏文件。 | C | MIT | 4,578 |

## 传统文化与娱乐类 Skills

以下按文化研究、娱乐和交互设计素材收录；项目自述的占卜或预测能力不作为科学结论。

| 编号 | 项目 | 用途简介 | 语言 | 许可证 | Star |
| ---: | --- | --- | --- | --- | ---: |
| 23 | [Renhuai123/ziwei-doushu](https://github.com/Renhuai123/ziwei-doushu) | 紫微斗数排盘引擎，包含排盘算法、四化、格局知识库与古籍数据。 | TypeScript | MIT | 7,319 |
| 24 | [FANzR-arch/Numerologist_skills](https://github.com/FANzR-arch/Numerologist_skills) | 面向奇门遁甲与紫微斗数的 AI Skills 工程框架，强调固定步骤与降低模型幻觉。 | Python | 需查看仓库 LICENSE | 1,498 |
| 25 | [Brhiza/mingyu](https://github.com/Brhiza/mingyu) | 多种传统术数、占星和塔罗工具集合，提供结构化数据、API、MCP 与 Skill。 | TypeScript | AGPL-3.0 | 514 |
| 26 | [jinchenma94/bazi-skill](https://github.com/jinchenma94/bazi-skill) | 用于四柱八字命理分析的 Skill。 | Python | MIT | 3,602 |
| 27 | [gaoxin492/bazi-skill](https://github.com/gaoxin492/bazi-skill) | 计算八字并围绕相关问题提供回答的 Agent Skill。 | Python | 需查看仓库 LICENSE | 30 |
| 28 | [Horace-Maxwell/horosa-skill](https://github.com/Horace-Maxwell/horosa-skill) | 本地优先的术数与占星计算工具集合，提供 MCP 与 CLI。 | Python | AGPL-3.0 | 477 |
| 29 | [Ming-H/yinyuan-skills](https://github.com/Ming-H/yinyuan-skills) | Claude Code 姻缘测算技能，包含八字合婚、生肖配对、求签等交互模式。 | 未标注 | 需查看仓库 LICENSE | 475 |
| 30 | [xr843/Master-skill](https://github.com/xr843/Master-skill) | 基于 FoJin 的佛教 AI 人格框架，强调来源依据、边界与一致性验证。 | Python | 需查看仓库 LICENSE | 472 |
| 31 | [voidforall/fengshui.skill](https://github.com/voidforall/fengshui.skill) | 传统风水顾问 Claude Code Skill，涉及玄空飞星、八宅与择日。 | 未标注 | 需查看仓库 LICENSE | 353 |
| 32 | [zhaoolee/cyber-fortune-telling](https://github.com/zhaoolee/cyber-fortune-telling) | 基于 LLM 的赛博算命与风水兴趣项目。 | JavaScript | 需查看仓库 LICENSE | 92 |
| 33 | [daman-ovo-0404/tarot-skill](https://github.com/daman-ovo-0404/tarot-skill) | AI 塔罗 Agent Skill，包含牌义、牌阵、牌间关系与抽牌脚本。 | Python | 需查看仓库 LICENSE | 276 |
| 34 | [hhszzzz/taibu](https://github.com/hhszzzz/taibu) | 涵盖多种术数与娱乐测算的 AI 工具，支持 MCP 服务和 Skills。 | TypeScript | 需查看仓库 LICENSE | 679 |
| 35 | [muyen/meihua-yishu](https://github.com/muyen/meihua-yishu) | 适用于多种 AI/LLM 的梅花易数占卜技能。 | Python | 需查看仓库 LICENSE | 209 |
| 36 | [qiyan233/meihua-divination](https://github.com/qiyan233/meihua-divination) | 面向 OpenClaw 的结构化梅花易数风格占卜 Skill。 | Python | 需查看仓库 LICENSE | 5 |
| 37 | [banderzhm/ZhouYiLab](https://github.com/banderzhm/ZhouYiLab) | 传统术数计算引擎，项目定位包括算法训练、文化研究和教育。 | C++ | MIT | 142 |

## 原始版本链接

原笔记第 18 个链接指向 [microsoft/skill-recorder v0.7.0 发布页](https://github.com/microsoft/skill-recorder/releases/tag/v0.7.0)，上表归入对应仓库。保留这个版本入口，不将其解释为当前最新版。

## 记录说明

- 40 个仓库均成功读取公开信息；原始笔记不修改，现有 GitHub 汇总文档不覆盖。
- 中文简介是用途摘要，不是安装教程或安全审计；授权要求以项目 LICENSE 和服务条款为准。
- 与安全测试、公开身份调查相关的项目，仅应在合法且获授权的范围内使用。
