---
author: "meow"
handle: "@meow2149"
source: "https://x.com/meow2149/status/2125889746081906827"
date: 2026-11-26T09:49:47.000Z
type: thread
likes: 38
reposts: 2
replies: 6
bookmarks: 36
views: 3566
---
# meow (@meow2149)

Jev 这类决策模型，在 Coding Agent 中到底有什么用？  
  
Claude Code 完成一次函数重命名后，改了两个文件，也同步更新了测试代码，正准备结束任务时被一个插件拦了下来。等它真正跑了一遍测试，才发现还有一个和 Windows 换行符有关的 bug。  
  
这是 Claude Code 社区插件 jev-belay 作者记录的一次真实运行。整个过程中，Jev 没有参与任何代码编写，只负责判断 Claude Code 当前到底应不应该结束任务。  
  
这也是我最近研究 Agent Harness 时，觉得决策模型挺有意思的地方。一个 Harness 在管理上下文、工具调用和任务流转时，会遇到大量边界相对明确的决策问题：  
  
下一步该用哪个模型？  
当前任务需要加载哪个 Skill？  
这条命令要不要让用户审批？  
上下文里哪些信息值得保留？  
多 Agent 协作时，任务该交给谁？  
  
TypeSafe 的 Jev、Google 的 Gemini Decision 1，以及 OpenAI 通过 Decisions API 提供的 GPT-6 Luna，都可以用来处理这类问题。Harness 提供当前状态和候选项，决策模型返回选择、概率或评分，再由程序根据结果推进下一步。  
  
我翻了几个开源项目的源码，整理了六个比较有代表性的应用场景。  
  
1. 模型路由：什么时候需要切强模型  
  
Google 已经在 Gemini Notebook 中用了这种方案，通过 Gemini Decision 1 选择合适的模型档位和思考强度。研究项目 JevAgentHarness 则更进一步，把模型路由细化到了 Agent 的每一步执行中。  
  
它可以作为代理，接在 Pi、Codex CLI 和模型服务之间：  
- 任务开始时默认使用强模型  
- 后续根据任务描述、近期操作和工具执行结果，由 Jev 判断下一步该用哪个模型  
- 常规操作交给快模型，复杂操作切回强模型  
- 如果快模型准备结束任务，再由强模型用相同的输入执行一次  
  
项目作者用 Pi 在 Terminal-Bench 2.0 的 74 道题上做了测试，每种配置各跑一次。相比全程使用 Gemini 4 Argon，按需切换到 Gemini 3.5 Flash-Lite 后，平均每道题的模型调用成本从 $0.62 降到了 $0.42，其中已经包含 Jev 自身的调用费用。  
  
2. Skill 路由：当前任务到底需要哪些 Skill  
  
OpenCode 的社区插件 jev-for-all 会把已安装 Skill 的名称、描述和用户请求交给 Jev，由它决定加载哪些 Skill。  
  
这里有个挺实用的设计，即使存在排名第一的 Skill，插件也可以一个都不加载。比如用户只是想改一下项目版本号，哪怕装了几十个 Skill，也没必要为这种简单任务引入额外的上下文。  
  
当候选项较多，或者初次判断的置信度不够时，插件还会进一步读取部分候选 Skill 的正文，再做一次筛选。对装了大量 Skill 的 Agent 来说，这能明显减少无关指令进入上下文。  
  
3. 权限审批：这条命令需要用户确认吗  
  
Hermes Agent 的社区插件 hermes-jev-approvals 把 Jev 接进了原有的智能审批流程，用来审查被 Hermes 标记出来的命令。它会从几个维度做判断：  
- 是否符合用户设定的规则  
- 是否涉及读取敏感凭据  
- 是否可能向外部发送本地数据  
  
插件再根据这些结果，通过代码决定放行、拒绝或请求用户确认。  
  
源码里还有个很稳妥的细节。如果命令过长，导致送给 Jev 审查的内容被截断，即使 Jev 建议放行，插件依然会要求用户确认。模型没看到完整命令，就不能据此授权执行。  
  
4. 上下文管理：哪些历史信息值得保留  
  
Claude Code 的社区插件 fast-jev-compaction 把工具调用记录的压缩拆成了两个独立判断：  
- 是否需要保留这次工具调用本身  
- 是否需要保留这次工具调用的完整输出  
  
比如 Agent 之前搜索过某个文件，返回了几百行结果。后续步骤可能还需要知道它搜过什么，但已经不需要完整的搜索结果了。插件可以根据 Jev 的判断，选择完整保留、保留调用参数并缩短输出，或者移除整条记录。相比简单按时间顺序裁剪历史，这种方式更能结合当前任务来判断信息的保留价值。  
  
5. 任务验证：Agent 真的完成工作了吗  
  
回到开头提到的 jev-belay。它会在 Claude Code 准备结束任务时检查执行记录，看最后一次文件修改之后，有没有成功运行测试、构建或 lint 的记录。如果已经有符合要求的验证记录，直接放行，不需要调用 Jev。  
  
只有当 Agent 改了文件、却没有通过后续检查时，插件才会让 Jev 判断，当前的回复只是在汇报进度，还是已经宣称任务完成。如果任务需要验证而 Agent 准备直接结束，插件就会要求它继续执行检查。  
  
这里的职责划分很清楚。测试有没有运行、结果是否通过，由程序查执行记录；自然语言回复是否意味着任务已经完成，交给决策模型判断。  
  
6. 多 Agent 编排：任务该交给谁，什么时候推进  
  
这部分我主要看了 T3 Code、Orca 和 OpenBot。虽然都和多 Agent 有关，但各自处理的决策问题不太一样。  
  
T3 Code（基于语义的事件通知）  
假设在 T3 Code 里同时开了三个 Agent 处理登录相关任务，我希望等其中一个提交 PR，并明确跑完登录回归测试后，再通知负责协调的主线程。社区工具 t3threads 可以监看指定线程，把近期活动和预设条件交给 Jev 判断。当条件成立的概率达到阈值（默认 0.9）时才会通知主线程，如果主线程正忙，就等它空闲后再投递。这样就能按实际进展触发后续工作，而不只是看线程有没有停下来。  
  
Orca（执行配置与任务路由）  
Orca 的社区插件 Fairway 支持提前定义多套执行配置，每套包含 Agent、模型、思考强度和适用场景。简单的回归测试用一套，复杂的并发问题用另一套。开启 Jev 路由后，Fairway 把任务描述和候选配置交给 Jev，由它返回合适的配置 ID，实际参数仍由程序从本地配置读取。目前 Fairway 默认关闭付费 Jev 路由，选中配置也不会直接启动任务，需要显式启用执行再派发。  
  
OpenBot（多 Agent 消息路由）  
OpenBot 的频道里可以同时有前端、后端和测试多个 Agent。当用户没有明确指定收件人、现有路由规则也定不下来时，OpenBot 会通过频道负责人配置的模型，结合成员职责、近期消息和未完成任务，选出应该回应的 Agent。  
  
比如这两句：  
“前端和测试分别检查一下登录功能。”  
“让前端汇总一下测试人员的意见。”  
  
两句话都提到了前端和测试，但前者需要两个 Agent 分别回应，后者只需要前端来协调汇总。OpenBot 目前在这里用的是频道负责人配置的模型，并不是 Jev 的现成集成，但本身很适合接入专用决策模型。  
  
当然，决策模型并不是所有环节都适合直接接入。  
  
JevAgentHarness 论文里记录了一个很典型的失败案例。在一项模拟客服退款任务中，用户还没提供退款方式，Jev 就以 0.98 的置信度选择了退款工具，随后主模型自己填了一个用户从未提供过的退款方式。原本应该向用户询问的信息，被一次高置信度的错误决策跳过了。  
  
在这组客服实验里，把每一次工具选择都交给 Jev，任务成功率反而下降了 19 个百分点。后来通过历史执行记录校准决策阈值，只把部分工具和参数选择交给 Jev，以 Gemini 3.5 Flash 为主模型的实验组，在 500 个任务成败结果完全不变的情况下，把成本降低了 14.2%。  
  
如果以后给自己的 Agent Harness 接入这类模型，我会优先从模型路由、Skill 选择、上下文管理和事件通知这些边界明确的环节开始。先在真实任务里积累一批决策数据，再逐步放开权限，重点观察它对任务成功率、执行步数和整体成本的影响。  
  
毕竟单次模型调用便宜，不代表整个 Agent 跑完任务的成本就一定会降低，最终还是要看完整执行下来的实际开销和效果。

![Image](<meow%20(@meow2149).assets/image-01.jpg>)

---

正文涉及的决策模型应用及源码：  
  
▪︎ JevAgentHarness：Agent 执行过程中的动态模型路由  
[github.com/JevAgentHarnes…](https://github.com/JevAgentHarness/JevAgentHarness)  
  
▪︎ jev-for-all：OpenCode 的 Skill 选择  
[github.com/emirbartu/jev-…](https://github.com/emirbartu/jev-for-all)  
  
▪︎ hermes-jev-approvals：Hermes Agent 的命令权限审批  
[github.com/anpicasso/herm…](https://github.com/anpicasso/hermes-jev-approvals)  
  
▪︎ fast-jev-compaction：Claude Code 的上下文压缩  
[github.com/tamaratran/fas…](https://github.com/tamaratran/fast-jev-compaction)  
  
▪︎ jev-belay：Claude Code 的任务完成判断与验证  
[github.com/valentynkit/je…](https://github.com/valentynkit/jev-belay)

> 🔗 [**GitHub - JevAgentHarness/JevAgentHarness**](https://t.co/kBIdLcoklr)
> 
> _From github.com_

---

多 Agent 编排相关实现及技术资料：  
  
▪︎ T3 Code / t3threads：基于语义条件的线程监控与事件通知  
[github.com/attunehq/t3thr…](https://github.com/attunehq/t3threads)  
  
▪︎ Orca / Fairway：Agent、模型及思考强度的路由选择（目前仅完成离线模拟验证）  
[github.com/spscream/orca-…](https://github.com/spscream/orca-fairway)  
  
▪︎ OpenBot：多 Agent 频道消息路由（潜在接入场景，尚未集成 Jev）  
[github.com/nightly-labs/o…](https://github.com/nightly-labs/openbot/blob/main/docs/architecture/channels-and-messaging.md)  
  
▪︎ JevAgentHarness 论文：决策模型在 Agent Harness 中的应用与实验结果  
[arxiv.org/html/2610.2141…](https://arxiv.org/html/2610.21412v1)  
  
▪︎ Google Gemini Decision 1  
[blog.google/innovation-and…](https://blog.google/innovation-and-ai/technology/developers-tools/gemini-decisions-api/)

> ![Link card preview](<meow%20(@meow2149).assets/image-02.jpg>)
> 
> 🔗 [**GitHub - attunehq/t3threads: Read, search, and start T3 Code threads from a CLI, MCP server, or...**](https://t.co/xcVFzyqGWX)
> 
> _From github.com_