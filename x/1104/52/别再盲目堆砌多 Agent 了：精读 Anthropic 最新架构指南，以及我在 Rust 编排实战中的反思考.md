---
author: "lifcc"
handle: "@mylifcc"
source: "https://x.com/mylifcc/status/2115713700674064579"
date: 2026-10-29T07:53:48.000Z
type: article
---
# 别再盲目堆砌多 Agent 了：精读 Anthropic 最新架构指南，以及我在 Rust 编排实战中的反思考

*By lifcc (@mylifcc)*

![Banner](<别再盲目堆砌多%20Agent%20了：精读%20Anthropic%20最新架构指南，以及我在%20Rust%20编排实战中的反思考.assets/image-01.jpg>)

过去一年里，AI 工程圈几乎达成了一种“政治正确”：只要任务稍微复杂一点，架构图上不画四五个 Sub-agent（规划 Agent、编码 Agent、审查 Agent、测试 Agent）来回连线，似乎都不好意思拿出手。

但就在 10 月 22 日左右，Anthropic Applied AI 团队发布了一篇极具分量的技术博客，并同步开源了迁移套件 **anthropics/single-agent-loop-kit**。他们用多个企业级真实部署的数据，直接给“盲目多 Agent 化”泼了一盆冷水：

**在绝大多数真实生产场景中，「单主代理（Single-Agent Loop）+ 按需加载 Skills」在输出质量、Token 成本和端到端延迟上，全面打赢多 Specialist Sub-agent 架构。**

作为一名正在用 Rust 手搓多 Crate 代码编排系统（涉及 Git Worktree 隔离、SQLite 状态机、并发调度与自愈门禁）的开发者，我读完这篇博客后深感共鸣，但也有一些来自深水区的补充视角。

今天这篇文章，我们先拆解 Anthropic 为什么要“砍掉多 Agent”，再来聊聊：**当单 Agent 循环撞上复杂代码工程的物理极限时，我们真正需要的编排层到底长什么样？**

## 一、 为什么你的 Multi-Agent 架构既贵又蠢？

很多团队在把 Demo 推向生产环境时，都会经历一次“多 Agent 幻灭”。Anthropic 在博客中一针见血地指出了多 Specialist 架构的三大硬伤：

## 1. 致命的“传话游戏”（Telephone Game）

在典型的多 Agent 架构中，主控 Agent 会把任务拆解并 handoff（移交）给下游的专家子代理。  
问题在于：**每一次 Handoff 都是一次有损压缩。**  
当“架构师 Agent”把方案传给“程序员 Agent”时，它传递的往往是一段自然语言摘要。那些在探索代码库时发现的隐式假设、微妙的边界条件、刚刚踩过的报错小坑，全在交接过程中丢失了。下游 Agent 只能在残缺的上下文中盲人摸象，最终导致代码质量断崖式下跌。

## 2. 3 到 10 倍的 Token 税

多 Agent 意味着多份独立的 Context Window。每个 Sub-agent 启动时，都要重新注入一遍项目背景、重新读取相关的代码文件；Agent 之间还要花费大量 Token 进行协议对齐、礼貌寒暄和状态确认。Anthropic 的实测数据显示，多代理架构消耗的 Token 通常是单代理循环的 **3 到 10 倍**。

## 3. 翻倍的端到端延迟

在客户对比测试中，将原本复杂的 Multi-agent Harness 重构为 Single-agent Loop 后，**整体运行速度直接快了约 2 倍**。少了中间层的序列化、反序列化以及多次冷启动推理，主循环的推进效率迎来了质变。

## 二、 Anthropic 的解法：渐进式披露（Progressive Disclosure）

既然单 Agent 这么好，当初大家为什么要拆成多 Agent？

根本原因在于 **Context 爆炸与注意力稀释**。如果我们把所有领域的规则、几十个 MCP 工具的 Schema、几百行代码规范全塞进同一个 Agent 的 System Prompt 里，模型很快就会“变笨”，连基本的指令遵循都做不好。

为了解决这个矛盾，Anthropic 在 Claude Code 的新模型适配以及 **single-agent-loop-kit** 中，祭出了核心杀手锏——**Progressive Disclosure（渐进式披露）**，直接把系统提示词（System Prompt）**砍掉了 80% 以上**。

它是怎么做到的？

1. **极简主循环（Ultra-lean Core Loop）：** 主 Agent 启动时，System Prompt 只保留最核心的身份定义、基础推理原则和极少数高频基础工具（如 Bash、文件读写）。
2. **Skills 按需挂载：** 原本属于各个“专家 Agent”的提示词（比如数据库迁移规范、前端组件编写指南、安全审计清单），被剥离成一个个独立的 **Skill** 模块。主上下文里只保留每个 Skill 的一行简短描述（Metadata）。只有当主 Agent 在推理过程中意识到“我现在需要写数据库迁移了”，它才会主动调用指令把对应的 Skill 展开加载到上下文中。
3. **Tool Search（工具动态检索）：** 不再把上百个工具的 JSON Schema 硬塞进初始请求，而是给模型提供一个搜索接口，用到什么工具再去拉取对应的定义。

**本质上，Anthropic 把“空间上的多 Agent 拆分”，变成了“时间轴上的单 Agent 动态加载”。** 同一个大脑、同一份完整上下文，在不同阶段给自己换上不同的“技能插件”，彻底消灭了传话损耗。

## 三、 反思考：单 Agent 循环就是终局了吗？

顺着 Anthropic 的逻辑往下想：既然 **Single-Agent + Skills** 这么强，那我们是不是只需要一个 **while(true)** 循环加上一堆 Markdown 技能包，就能搞定一切软件工程了？

**在单线任务、中小规模仓库里，是的。但在复杂的重度代码工程里，绝对不够。**

我最近一直在用 Rust 开发一套针对多 Crate 大型项目的编码编排系统。在这个过程中我深刻体会到：**Anthropic 淘汰的是“按角色过家家式的伪多 Agent”，但在面对物理资源冲突、上下文污染和长周期任务时，你依然需要一层硬核的工程编排层（Orchestration Layer）。**

以下是我结合实战总结的“何时该合、何时必须拆”的分水岭：

## 1. 认知逻辑要“合”，但物理环境必须“拆”（Git Worktree 并发）

如果你让一个 Agent 扮演架构师、另一个扮演程序员去改同一个模块，这叫内耗，应该合并为单 Agent + Skills。  
但如果你面对的是一个包含十几个 Crate 的 Rust 工作区，需要同时推进 3 个独立特性的开发，或者同时修复 4 个不同模块的编译错误呢？

- **单 Agent 串行太慢：** 等一个 Loop 跑完 **cargo check** 和测试，黄花菜都凉了。
- **同目录并行会打架：** 两个循环改同一个工作区，文件锁和 Git 状态立刻乱成一团。

这时候，编排层的核心价值不是去拆解“角色”，而是利用 **Git Worktree** 为每个独立的单 Agent 循环开辟物理隔离的沙盒环境。每个 Worktree 里跑的依然是 Anthropic 推荐的 **Single-Agent + Skills**，但在宏观层面，它们在各自的轨道上并行狂奔，互不污染。

## 2. Progressive Disclosure 救不了“报错日志污染”

砍掉 80% 的初始 System Prompt 确实让起跑线变得极其干净。但在真实的 Rust 多 Crate 编译中，一次失败的 **cargo test** 或复杂的生命周期报错，吐出来的编译日志动辄几千行。  
当单 Agent 在里面连续挣扎 3 到 4 轮之后，哪怕初始 Prompt 再精简，当前的 Context Window 也已经被失败的尝试和冗长的堆栈彻底污染了（Context Pollution）。此时模型的智商会急剧下降，陷入死循环。

这就是为什么我们在单 Agent 循环之外，必须构建 **Quality Gates（质量门禁）与 Self-healing（自愈状态机）**：

- 编译、Clippy 检查、单元测试不应该完全依赖模型“自觉”去跑，而是由外层 Rust 编排器作为硬性门禁强制执行。
- 当检测到某个 Loop 连续失败或 Token 消耗越界时，编排器会直接杀掉当前被污染的上下文，从 **SQLite** 状态库中提取已确认的进度和提炼后的精简报错，重启一个干干净净的 Single-Agent Loop。

## 3. 状态与可观测性不能只留在 Prompt 里

单 Agent 循环非常擅长处理“当下这一步怎么写”，但它不擅长跨会话的全局状态管理、预算熔断和审计追踪。  
在我的实践中，把任务的依赖图（DAG）、每个子任务的 Token 账单、失败重试次数、以及 Worktree 的合并状态全部沉淀在 **SQLite** 中，由确定性的 Rust 代码来接管调度、重试与安全边界，才能让不可控的大模型循环变成可靠的工程流水线。

## 四、 写在最后：2026 年的 Agent 架构最佳实践

结合 Anthropic 的最新博客与我们的一线工程经验，当下构建复杂 AI 编码系统的最佳范式其实已经非常清晰了：

1. **砍掉角色扮演式的 Sub-agents：** 别再搞“PM Agent -> Dev Agent -> QA Agent”的传话链条了，那只会白白烧掉你 5 倍的 Token 并把上下文搞丢。
2. **在执行单元内部，拥抱 Single-Agent + Skills：** 借鉴 **anthropics/single-agent-loop-kit**，把初始 System Prompt 瘦身到极致，用 Progressive Disclosure 动态加载领域规范与工具。
3. **在执行单元外部，用确定性代码做重度编排：** 把并发隔离（Worktree）、状态持久化（SQLite）、质量门禁（Quality Gates）和上下文污染重置（Self-healing）交给确定性的系统语言（如 Rust）去管控。

**“让单 Agent 保持完整的大脑，让外层编排器提供坚固的物理沙盒。”** 这或许才是通往高可靠自主软件工程的真正路径。