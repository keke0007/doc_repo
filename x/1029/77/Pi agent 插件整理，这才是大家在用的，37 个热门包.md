---
author: "苏茶"
handle: "@Suercha007"
source: "https://x.com/Suercha007/status/2114296967685963935"
date: 2026-10-25T10:04:13.000Z
type: article
---
# Pi agent 插件整理，这才是大家在用的，37 个热门包

*By 苏茶 (@Suercha007)*

![Banner](<Pi%20agent%20插件整理，这才是大家在用的，37%20个热门包.assets/image-01.jpg>)

Pi 官方目录目前约有 **5,348 个包**。下面这份清单筛掉了新版本已经内置的基础 MCP、主题和 statusline 类重复项。

我的常驻优先项是：**pi-web-access、pi-subagents、**[**@juicesharp/rpiv-todo**](https://x.com/@juicesharp/rpiv-todo)**、pi-lens、**[**@gotgenes/pi-permission-system**](https://x.com/@gotgenes/pi-permission-system)**、context-mode**。其余按工作流选择。

**核心底座**

1. **pi-web-access｜801.8K/月**：提供搜索、网页抓取，以及处理 PDF、YouTube 和 GitHub 的能力，是 Pi 的联网入口。[GitHub](https://github.com/nicobailon/pi-web-access)
2. **pi-subagents｜558.3K/月**：支持委派子代理，覆盖 single、chain、parallel、async、forked-context 五种模式。[GitHub](https://github.com/nicobailon/pi-subagents)
3. [**@juicesharp/rpiv-ask-user-question**](https://x.com/@juicesharp/rpiv-ask-user-question)**｜231.4K/月**：当模型会靠猜时，改为弹出结构化问题让你选择，减少需求理解错误。[GitHub](https://github.com/juicesharp/rpiv-mono)
4. [**@juicesharp/rpiv-todo**](https://x.com/@juicesharp/rpiv-todo)**｜151.5K/月**：为模型提供实时 TODO 浮层，即使发生 /reload 或上下文压缩也不会丢失。[GitHub](https://github.com/juicesharp/rpiv-mono)
5. **pi-lens｜88.8K/月**：为 Pi 加上 LSP、诊断、AST 搜索和结构化代码分析能力。[GitHub](https://github.com/apmantza/pi-lens)
6. [**@gotgenes/pi-permission-system**](https://x.com/@gotgenes/pi-permission-system)**｜60K/月**：为工具调用增加权限控制，适合不希望 Agent 随意执行命令的场景。[GitHub](https://github.com/gotgenes/pi-packages)

**多代理与工作流**

1. [**@plannotator/pi-extension**](https://x.com/@plannotator/pi-extension)**｜107.1K/月**：交互式计划审阅、消息标注和代码/PR Review，适合先讨论方案再执行。[GitHub](https://github.com/backnotprop/plannotator)
2. [**@dietrichgebert/ponytail**](https://x.com/@dietrichgebert/ponytail)**｜95.9K/月**：给 Agent 套上“懒资深工程师”模式，优先少写代码、少过度设计。[GitHub](https://github.com/DietrichGebert/ponytail)
3. **bigpowers｜89.2K/月**：打包 73 个软件工程 Skills，强调规范化的规划、实现和验证流程。[GitHub](https://github.com/danielvm-git/bigpowers)
4. **pi-goal-x｜88.9K/月**：用 /goal 管理长期目标、持久进度和独立完成审计，适合跨多轮推进任务。[GitHub](https://github.com/tmonk/pi-goal-x)
5. **pi-advisor-flow｜70.2K/月**：将执行者与 Advisor 分离，让强模型负责指导和审查，执行模型负责动手。[GitHub](https://github.com/philipbrembeck/pi-advisor)
6. **pi-harness-runtime｜53.1K/月**：提供 Codex 风格用量状态和可恢复的自主编码循环；/harness 仍是 Beta。[GitHub](https://github.com/ManotLuijiu/pi-harness-runtime)
7. **pi-background-tasks｜44.9K/月**：启动可持久化的后台 Shell 任务和只读子代理，长任务不必一直占着前台会话。[GitHub](https://github.com/ismailsaleekh/pi-background-tasks)
8. **pi-btw｜38.5K/月**：用 /btw 开一个并行侧聊会话，适合主任务之外临时研究或确认细节。[GitHub](https://github.com/dbachelder/pi-btw)
9. **pi-fabric｜31.6K/月**：可编程的 Tool 与 Agent Runtime，用 fabric_exec 编排工具、MCP、子代理和工作流；能力重，配置也重。[GitHub](https://github.com/fabric-runtime/pi-fabric)
10. [**@akagilnc/pi-workflow-roles**](https://x.com/@akagilnc/pi-workflow-roles)**｜30.4K/月**：把 coder、reviewer、judge、notary 等角色串成带门禁的工作流，偏团队治理。[GitHub](https://github.com/Akagilnc/ak-pi-workflow-roles)
11. [**@kontextmind/kxm**](https://x.com/@kontextmind/kxm)**｜29.5K/月**：本地优先的多 Agent 协调层，提供持久消息、Webhook 工作流、任务面板和跨 Pi/Claude Code 协作。[GitHub](https://github.com/kontextmind/kxm)
12. **pi-intercom｜29.3K/月**：让同一台机器上的多个 Pi 会话一对一通信、提问和交接上下文，适合 planner、worker、reviewer 分窗口协作。[GitHub](https://github.com/nicobailon/pi-intercom)

**上下文、检索与项目知识**

1. **billion-context｜1.8M/月**：强力上下文压缩，让长会话和超大仓库在有限窗口内继续推进。[GitHub](https://github.com/ranxianglei/billion-context)
2. **openwiki｜171.9K/月**：用文档 Agent 为代码库生成并持续维护 OpenWiki，适合补项目知识库。[GitHub](https://github.com/langchain-ai/openwiki)
3. **context-mode｜91.4K/月**：把日志、构建输出、网页和 diff 放进沙箱处理，再用 FTS5 知识库按需检索。[GitHub](https://github.com/mksglu/context-mode)
4. **pi-memory｜42K/月**：给 Pi 增加跨会话的语义记忆，覆盖日志、长期记忆和临时记录。[GitHub](https://github.com/jayzeng/pi-memory)
5. [@ff](https://x.com/@ff)**-labs/pi-fff｜31.6K/月**：高速模糊文件与内容搜索，适合大仓库快速定位代码。[GitHub](https://github.com/dmtrKovalenko/fff)
6. **gentle-engram｜27.3K/月**：本地或云端共享的持久记忆，可跨会话、上下文压缩和 MCP Agent 使用。[GitHub](https://github.com/Gentleman-Programming/engram)

**外部工具与可观测性**

1. [**@langfuse/pi-observability-plugin**](https://x.com/@langfuse/pi-observability-plugin)**｜249.3K/月**：把提示词、模型调用、工具执行和 Agent turn 全链路追踪到 Langfuse。[GitHub](https://github.com/langfuse/pi-observability-plugin)
2. [**@postman/postman-plugin**](https://x.com/@postman/postman-plugin)**｜164.7K/月**：把 Postman 的 API 设计、调试和测试 Skill 接入编码 Agent。[GitHub](https://github.com/postmanlabs/postman-plugin)
3. **pi-claude-bridge｜80K/月**：通过 Agent SDK 将 Claude Code 作为 Pi 的模型提供方，并增加 AskClaude 工具。[GitHub](https://github.com/elidickinson/pi-claude-bridge)
4. [**@langchain/langsmith-pi-extension**](https://x.com/@langchain/langsmith-pi-extension)**｜72.4K/月**：把 Pi 会话、模型调用和工具执行接入 LangSmith 追踪与评估。[GitHub](https://github.com/langchain-ai/langsmith-pi-extension)
5. [@companion](https://x.com/@companion)**-ai/feynman｜68.1K/月**：在 Pi 内使用 Feynman 研究 Agent，适合资料调研、技术选型和报告整理。[GitHub](https://github.com/Companion-Inc/feynman)
6. **confluence-cli｜42.1K/月**：读取、搜索、创建和更新 Confluence 页面，支持只读模式和空间白名单。[GitHub](https://github.com/pchuri/confluence-cli)
7. [**@braintrust/pi-extension**](https://x.com/@braintrust/pi-extension)**｜28.4K/月**：自动追踪 Pi session、LLM 调用和 tool execution，并发送到 Braintrust。[GitHub](https://github.com/braintrustdata/braintrust-coding-agent-plugins)
8. **pi-agent-browser-native｜24.3K/月**：把 agent-browser 暴露成 Pi 原生浏览器工具；和 betterwright 二选一即可。[GitHub](https://github.com/fitchmultz/pi-agent-browser-native)
9. **betterwright｜22.5K/月**：带网络策略、凭据填充和截图证明的持久 Playwright 浏览器，更偏安全与审计。[GitHub](https://github.com/BetterWright/betterwright)

**成本、本地化与代码质量**

1. [**@narumitw/pi-usage**](https://x.com/@narumitw/pi-usage)**｜62.4K/月**：查看模型额度、重置时间、API 余额和 Codex Fast 模式状态。[GitHub](https://github.com/narumiruna/pi-extensions)
2. **pi-simplify｜37.9K/月**：检查最近改动，从清晰度、一致性和可维护性角度清理代码坏味道。[GitHub](https://github.com/MattDevy/pi-extensions)
3. **pi-cache-optimizer｜34K/月**：优化稳定提示词和缓存键，提高 Prompt/KV Cache 命中率。[GitHub](https://github.com/jiangge/pi-cache-optimizer)
4. [**@juicesharp/rpiv-i18n**](https://x.com/@juicesharp/rpiv-i18n)**｜30.4K/月**：为 rpiv-* 系列提供中文界面、/languages 切换和统一本地化。[GitHub](https://github.com/juicesharp/rpiv-mono)

**选择建议**

1. 第一次配置 Pi：先装 pi-web-access、pi-subagents、[@juicesharp/rpiv-todo](https://x.com/@juicesharp/rpiv-todo)、pi-lens 和 context-mode。
2. 需要处理长任务：再考虑 pi-goal-x、pi-background-tasks、pi-harness-runtime。
3. 多会话、多 Agent：在 pi-intercom、[@kontextmind/kxm](https://x.com/@kontextmind/kxm)、pi-fabric 中按复杂度选一个，不要全部堆上。
4. 浏览器自动化：pi-agent-browser-native 和 betterwright 功能重叠，按使用习惯二选一。
5. 重视安全：先看 [@gotgenes/pi-permission-system](https://x.com/@gotgenes/pi-permission-system)，但第三方包本身仍应审查源码。

最后提醒一句：**pi-fabric、kxm、workflow-roles 都是重度工作流方案，功能有重叠，先选一个；pi-harness-runtime 的自主循环仍是 Beta，建议先用它的 /usage。**