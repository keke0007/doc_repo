---
author: "meow"
handle: "@meow2149"
source: "https://x.com/meow2149/status/2126139626985521608"
date: 2026-11-27T02:22:43.000Z
type: thread
likes: 159
reposts: 21
replies: 10
bookmarks: 207
views: 12194
---
# meow (@meow2149)

Coding Agent 如何组队？从 Agent Teams 到 Swarm 的七种协作方案  
  
最近研究了几种 Coding Agent 的多 Agent 协作方案，发现不同产品在团队编排上的设计各有巧思。  
  
Claude Code 有 Agent Teams，Antigravity 有 Swarm 和 Teamwork，PandaCode、Orca、Paseo 等工具也在不同 Coding Agent 之上构建了自己的编排层。  
  
同时开几个 Agent 并不难，难的是让它们围绕同一个目标工作。  
  
比如前端 Agent 需要知道后端 Agent 定义的接口，测试 Agent 需要等待功能实现，Review Agent 发现问题后，还需要把任务交回对应的开发 Agent。  
  
这些任务怎么分配，成员之间怎么通信，如何避免代码冲突，失败后怎么恢复，最终又由谁来验收？  
  
结合官方文档和公开源码，整理了目前很有代表性的七种协作方案。  
  
1. Claude Code · Agent Teams  
  
Claude Code 的 Agent Teams 由四个核心组件组成：Team Lead、Teammates、Task List 和 Mailbox。  
  
Lead 负责创建团队、协调任务和汇总结果，每个 Teammate 都是一个拥有独立上下文的 Claude Code 实例，成员之间可以直接通信。  
  
具备 Task 工具的成员既可以接受 Lead 分配的任务，也可以从共享任务列表中主动领取未分配且依赖已解除的任务。任务认领通过文件锁避免竞争。  
  
Claude Code 还提供了 TeammateIdle、TaskCreated、TaskCompleted 等 Hooks，可在成员进入空闲状态、任务创建或完成时执行额外检查。  
  
有意思的是，现在 Claude Code 中显式命名的 Subagent 也可以互相发送消息。  
  
因此，能否直接通信已不能作为区分 Subagent 和 Agent Teams 的唯一依据，更需要关注它们的会话组织方式、团队级任务状态以及成员的自主协调能力。  
  
目前 Agent Teams 仍属于实验性功能，默认关闭。  
  
2. Antigravity · Swarm  
  
Google Antigravity 的 /swarm 主要围绕一块共享消息板（Message Board）展开协作。  
  
当前会话作为 Coordinator，负责拆解任务、创建具名 Worker，并明确每个成员的工作范围和完成标准。  
  
成员之间通过 @name、@group 或 @all 在共享消息板上交流，暂时没有任务的成员可以进入等待状态，被其他成员提及时再唤醒。  
  
Swarm 还引入了两个专门的辅助角色：  
  
▪ Verifier：检查代码变更、运行测试，并将发现的问题反馈给对应的 Worker。  
▪ Monitor：当任务尚未完成、团队却陷入沉默时，提醒相关成员继续推进。  
  
Worker 默认复用当前会话的模型和工作区，也可以指定其他可用模型，或为可能产生编辑冲突的任务启用独立工作副本。  
  
桌面端还提供 Graph 视图，可以查看成员之间的通信关系和消息流转过程。  
  
这套机制适合按模块、文件或调研方向拆解的并行任务，目前仍处于预览阶段。  
  
3. Antigravity · Teamwork  
  
Antigravity 还有一个 /teamwork，面向更复杂、执行周期更长的项目。  
  
它会先和用户确认项目需求、约束及验收标准，确认后再进入自主执行阶段。  
  
Sentinel 负责项目入口、整体协调和最终汇报，Project Orchestrator 则负责拆解里程碑，并组织不同角色推进项目。  
  
除了负责调研和开发的 Explorers、Workers，还引入了多种审查与验收角色：  
  
▪ Critic：审查代码质量和逻辑正确性。  
▪ Challenger：寻找边界条件和潜在问题。  
▪ Auditor：检查实现完整性和测试证据。  
▪ Success Auditor：负责项目的最终验收。  
  
团队通过结构化的需求、计划和进度文件进行信息传递与上下文交接，同时配合文件所有权分配机制，以及不同严格程度的验证模式。  
  
Swarm 更侧重单次任务中的并行协作与实时通信，Teamwork 则更关注长期项目的阶段性交付和独立验收。  
  
目前 /teamwork 面向付费套餐用户开放。  
  
4. PandaCode · Agent Teams  
  
前面介绍的 Claude Code 和 Antigravity 都是在自身的 Agent 运行体系内组队。  
  
PandaCode 则可以把 Claude Code、Codex、Pi、OpenCode 等不同 Harness 的执行实例放进同一个团队。  
  
它采用 Architect + Workers 架构，Architect 负责规划、任务分派和结果审查，Worker 拥有独立会话，并通过团队邮箱、共享笔记和进度检查点进行协作。  
  
任务可以声明依赖关系和文件修改范围，调度层会对存在已声明写入冲突的任务进行串行调度。  
  
不过，Worker 使用的是父任务所在的 checkout，并不会自动为每个 Worker 创建独立的 Worktree。文件范围声明也不等同于文件系统级别的隔离。  
  
PandaCode 最让我感兴趣的是它的 Panda Router。  
  
启用自动路由后，系统会在本地分析任务特征，结合模型能力评分、成本、延迟和预估成功率，为 Worker 选择合适的模型。  
  
目前提供三种自动组队策略：  
  
▪ Cheap：在满足最低预估成功率要求的候选模型中，优先降低成本。  
▪ Balanced：综合考虑预估成功率、成本和延迟。  
▪ Best：优先选择预估成功率更高的模型。  
  
它还会区分两种不同的失败情况。  
  
如果是 Harness 本身出现运行故障，会尝试通过其他可用 Harness 运行同一个模型。  
  
如果模型正常完成了任务，但结果未通过 Architect 的测试或审查，则会重新分派任务，由路由器尝试更强的模型。  
  
自动路由对同一项任务最多进行三次尝试，包括首次执行和两次重试。  
  
PandaCode 不仅在做多 Agent 编排，也把模型选择和执行环境的故障恢复放进了同一套协作机制里。  
  
5. Orca · Orchestration  
  
Orca 的多 Agent 编排更接近一套结构化任务调度系统。  
  
其中有五个核心对象：  
  
▪ Run：一次协作的持久化命名空间及协调者收件箱。  
▪ Task：记录任务描述、依赖关系和执行状态。  
▪ Dispatch：某个 Task 的一次具体分派与执行。  
▪ Message：传递问题、进度、心跳和完成报告。  
▪ Decision Gate：在相关决策解决之前阻塞任务推进。  
  
其中 Task 和 Dispatch 的分离特别值得研究。  
  
假设一个 Worker 执行失败，Coordinator 将任务重新分派给另一个 Worker。  
  
此时 Task ID 没有改变，但对应的 Dispatch ID 已经变了。  
  
如果旧 Worker 这时才发回完成报告，Orca 会校验发送者、Task 与 Dispatch 的对应关系，以及该 Dispatch 是否仍然有效，避免过期的执行结果误改任务状态。  
  
这部分在 Orca 的生命周期管理源码中有明确实现。  
  
此外，Orca 还支持 DAG 任务依赖、受监督的 Worker、子 Worktree 和远程执行。  
  
当然，Dispatch 校验只能解决任务状态的一致性问题，并不能撤销旧 Worker 已经产生的文件修改或外部副作用，这仍然需要执行隔离等机制配合。  
  
目前 Orchestration 仍属于实验性功能，官方文档注明其可视化面板及对应开关暂时不可用。  
  
6. Paseo · Ensemble  
  
Paseo 本身已经提供了跨 Provider 的 Agent 启动、Worktree 管理和消息通信能力。  
  
社区插件 Ensemble 则在这些基础能力之上，实现了一套团队协作流程。  
  
它提供三种模式：  
  
▪ Delegation：Orchestrator 拆解并分派任务，成员独立执行，再汇报结果。  
▪ Team：确认需求和计划后建立共享任务板，成员按依赖领取任务、声明文件占用、互相通信，并在开发完成后进行交叉审查。  
▪ Auto：由 Orchestrator 选择 Team 或 Delegation，首次启动协作后锁定当前会话的协作模式。  
  
虽然 Paseo 本身支持 Worktree 管理，但 Ensemble 的团队成员共享 Orchestrator 所在的 checkout，主要通过任务认领和文件占用声明协调并行修改。  
  
我很喜欢 Ensemble 对团队配置的处理方式。  
  
成员、角色、模型和协作模式可以保存在项目的 .agents/teams/ 目录下，通过 Git 进行版本管理。  
  
比如一个项目可以配置专门负责前端、后端、测试和代码审查的 Agent，再为每个角色选择不同的模型和指令。  
  
其他开发者也可以复用这套配置，不需要每次都重新组建团队。  
  
这相当于把 Agent Team 的组织方式也变成了项目工程配置的一部分。  
  
7. OpenBot · Shared Channels  
  
OpenBot 的团队协作方式更像是把一组长期使用的 Agent 拉进同一个群聊。  
  
每个 Agent 都有独立的身份、指令、模型、会话和工作空间，多个 Agent 可以通过共享频道进行协作。  
  
一个 Channel 可以包含多名 Agent，也可以设置负责请求路由的 Lead，并维护独立的消息和任务记录。  
  
用户可以直接指定某个成员处理请求，也可以让 Lead 根据内容选择合适的成员，还可以让多个成员分别给出自己的结果。  
  
频道中的每个任务都有明确的 Owner，支持委派子任务和转移任务所有权。  
  
OpenBot 通过 SQLite 持久化频道记录、任务及相关执行状态，并支持任务暂停、恢复和重新分配。  
  
这种协作方式并不局限于 Coding Agent。  
  
完全可以组建一支包含开发、代码审查、信息调研和内容运营等不同角色的团队，根据日常需求持续安排任务。  
  
相比 Orca 围绕任务生命周期展开的编排，OpenBot 更关注持久化的 Agent 成员和长期协作关系。  
  
多 Agent 编排需要解决哪些工程问题？  
  
看完这些项目，可以从六个维度来理解它们的设计：  
  
▪ 任务管理：任务拆解、依赖关系与所有权。  
▪ 消息通信：成员间的信息传递、按需唤醒和上下文管理。  
▪ 工作区与冲突管理：Worktree、文件占用和并行修改。  
▪ 执行单元选择与模型路由：如何为不同任务选择合适的 Harness 和模型。  
▪ 失败恢复：异常退出、任务重派和过期执行结果。  
▪ 结果验证：代码审查、测试和验收标准。  
  
不同产品在这些环节各有侧重，几种协作机制之间也并不互斥。  
  
而且 Agent 数量并不是越多越好。  
  
如果任务之间存在强依赖，或者多个 Agent 需要频繁修改同一份代码，额外的通信、调度和验证成本，很可能会抵消并行执行带来的收益。  
  
之前研究 Agent Harness 时，更多关注的是如何让单个 Agent 管理好上下文、调用工具并完成任务。  
  
现在再看这些多 Agent 编排项目，会发现 Harness 需要处理的事情已经延伸到了任务调度、消息通信、资源管理、状态一致性和结果验证。  
  
很多传统分布式系统和软件工程中的经典问题，正在 Agent 工程中重新出现。  
  
这也是我最近觉得多 Agent 编排特别值得研究的地方。

![Image](<meow%20(@meow2149).assets/image-01.jpg>)

---

文中涉及的项目及相关技术资料：  
  
▪ Claude Code · Agent Teams  
[code.claude.com/docs/en/agent-…](https://code.claude.com/docs/en/agent-teams)  
  
▪ Antigravity · Swarm & Teamwork  
[antigravity.google/docs/swarm/](https://antigravity.google/docs/swarm/)  
[antigravity.google/docs/teamwork/](https://antigravity.google/docs/teamwork/)  
  
▪ PandaCode · Agent Teams  
[github.com/aicodeking/pan…](https://github.com/aicodeking/pandacode)  
[github.com/aicodeking/Pan…](https://github.com/aicodeking/PandaOrchestrator)  
  
▪ Orca · Orchestration  
[github.com/stablyai/orca](https://github.com/stablyai/orca)  
[orca.dev/docs/cli/orche…](https://orca.dev/docs/cli/orchestration)  
  
▪ Paseo · Ensemble  
[github.com/getpaseo/paseo](https://github.com/getpaseo/paseo)  
[github.com/DueGin/paseo-e…](https://github.com/DueGin/paseo-ensemble)  
  
▪ OpenBot · Shared Channels  
[github.com/nightly-labs/o…](https://github.com/nightly-labs/openbot)

> ![Link card preview](<meow%20(@meow2149).assets/image-02.jpg>)
> 
> 🔗 [**Orchestrate teams of Claude Code sessions - Claude Code Docs**](https://t.co/3daR7Qpzfr)
> 
> _From code.claude.com_

---

下一篇：多 Agent 协作，真的比一个全栈 Agent 更高效吗？

> **meow (@meow2149)**
> 
> 多 Agent 协作，真的比一个全栈 Agent 更高效吗？  
>   
> 最近在深度使用 OpenBot 的过程中，我一直在思考这个问题。  
>   
> 以前完成一个开发需求，我基本上只需要用 Codex。从梳理产品需求、编写 PRD，到代码开发、通过 Computer Use 调整页面、调用接口测试、Code
> 
> ![Image](<meow%20(@meow2149).assets/image-03.jpg>)