---
author: "AIGCLINK"
handle: "@aigclink"
source: "https://x.com/aigclink/status/2091076588309971118"
date: 2026-08-22T08:14:43.000Z
type: thread
likes: 101
reposts: 14
replies: 6
bookmarks: 141
views: 11528
---
# AIGCLINK (@aigclink)

Agent Orchestrator (AO)：管理多coding agent 用的"项目管理系统，也就是面向AI程序员的tapd或tower，我们正从卷 coding agent 本身，过渡到"agent 变多之后怎么管"，AO 卷的就是这一块。  
  
场景面：  
一个 agent 你盯得过来，五个 agent 同时改一个 repo，你面对的是五个终端、五条分支、五个 PR、一堆红色的 CI：工具没变强，你的注意力被切碎了，我们日常会用claude fable做prd，然后用codex做实现，用grok cli做审查，多个切来切去，很容易搞混乱，而且有时候没办法很好的并行工作。  
  
AO 的解法是把 IDE 换成"调度台"：上面一层 orchestrator 负责规划和派活，下面每个 worker 一个任务一个 worktree，中间用一块实时看板把 PR / CI / review 状态映射成"谁在跑、谁卡住、谁能合"。  
  
AO支持 26 个主流code agent，每个 worker 独占一条分支和 worktree，看板从 PR、CI、评审的真实状态推导出来。有意思的地方在于卡片状态是推导出来的，不是人工维护的——这基本承认了一件事：agent 多了以后，真正稀缺的不是算力，是人的注意力该放在哪。  
  
它把结构分成三层，划得很干净：  
  
① Worker——执行单元  
一个任务 + 一个编程 agent + 一个隔离工作区，Git 项目下，每个 worker 独占自己的分支和 worktree（临时活则给一个 AO 托管的无分支目录），任务、对话、终端、改动文件、浏览器预览、PR、CI、评审状态，从头到尾挂在这一个会话上——所以并行的活不会塌成一锅粥。  
  
② 项目编排者——常驻的规划 agent，工作在任务之上那一层  
管的是产品方向、技术策略、优先级、跨仓库的工作顺序。它的项目级对话保留目标、决策、约束和此前的推理，并把这些跟仓库上下文、以及 AO 的实时状态（哪些 worker 在跑、谁负责、PR、CI、评审）合在一起用。计划成型后，它能拆成任务、生成或重定向 worker、把该给的上下文分发下去、跟进度。  
  
分工用一句话写死了："编排者拥有规划与委派；worker 拥有实现、测试、提交与 PR。"  
  
③ 看板——不是手动拖的，是从事实推导的，AO 根据会话、PR、CI、评审的真实状态决定卡片位置：  
  
.Working：正在实现，或等你下一条指令  
.Needs you：阻塞、缺输入、CI 挂了、被要求改动、信号丢失  
.In review：开着的或草稿 PR，等检查或等人看  
.Ready to merge：已批准或可合并，合并后还留在板上直到你归档  
  
这点比"AI 公司"类项目实在得多——它的状态不是模型自称的，是 PR 和 CI 说了算。  
  
对比昨天介绍的项目Wake 读本地会话文件，所以 Amp、Droid 这种把会话存云端的它读不到；而AO 是驱动 agent 干活，所以云端会话的 agent 照样能管。  
#AIagent #多智能体 #开发者工具 #编程agent #开源

![Image](<AIGCLINK%20(@aigclink).assets/image-01.jpg>)

---

github：

> ![Link card preview](<AIGCLINK%20(@aigclink).assets/image-02.jpg>)
> 
> 🔗 [**GitHub - OrchestratorInc/agent-orchestrator: Run and supervise teams of coding agents from planning...**](https://t.co/CFsna5ga4m)
> 
> _From github.com_