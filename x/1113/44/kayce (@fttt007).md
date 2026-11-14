---
author: "kayce"
handle: "@fttt007"
source: "https://x.com/fttt007/status/2120358281558319110"
date: 2026-11-11T03:29:43.000Z
type: tweet
likes: 56
reposts: 6
replies: 7
bookmarks: 44
views: 4684
---
# kayce (@fttt007)

今天继续学习了 Pi Durable（pi 项目）的 Tools 和 Extension 机制，发现它的设计思路很有意思。  
  
几个值得记录的知识点：  
  
1. Tool 不只是一个普通函数  
  
通过 defineTool() 定义工具，包括 name、description、parameters 和 execute()。其中参数 Schema 不仅告诉模型如何调用工具，也为 TypeScript 提供类型约束。  
  
2. Extension 是能力的组织与分发单元  
  
Tool 通过 defineExtension() 组织，再通过 registry.install() 安装到 Registry。  
  
整个过程可以理解为：  
  
Tool → Extension → Registry → Conversation  
  
Registry 负责管理已安装的 Extension，Conversation 则决定当前对话使用哪些 Extension。不同 Conversation 可以拥有不同的工具集合。  
  
3. 每次 Tool Call 都是一个独立的 Durable Task  
  
这是 Pi Durable 与普通 Agent Framework 很大的区别。  
  
工具调用不再只是执行一个函数，而是进入持久化任务体系。工具执行过程中的输出可以通过 api.output() 提交和保存。  
  
同时，replay: "safe" 可以声明工具是否允许在崩溃恢复时重新执行。这里必须考虑副作用，例如转账、发邮件等操作不能随意重放。  
  
4. Tool 还能影响 Agent 的执行流程  
  
通过 usage 记录工具消耗，通过 control.terminate 控制当前 Run 是否结束。  
  
5. Extension 支持热更新  
  
这是今天最让我感兴趣的部分。  
  
相同名称的 Extension 可以通过 registry.install() 原位替换，不需要重启 Node.js 进程。  
  
已经开始执行的 Tool Call 保留旧实现，后续新的 Tool Call、Request 和 Task Phase 使用新实现。  
  
如果进程重启，则需要重新安装 Extension，才能恢复相应的待执行任务。  
  
今天最大的收获：  
  
Pi Durable 把 Agent 的能力组织、工具调用、任务持久化、执行恢复和热更新放进了一套统一的架构中。  
  
它不只是让 Agent 能够调用工具，而是在尝试解决一个更深层的问题：  
  
如何让一个 Agent 在长时间运行、进程崩溃、工具更新之后，依然能够可靠地继续工作。  
  
#Pi #Agent #AIAgent #DurableExecution #TypeScript