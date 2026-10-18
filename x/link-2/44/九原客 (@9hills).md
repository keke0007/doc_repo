---
author: "九原客"
handle: "@9hills"
source: "https://x.com/9hills/status/2058119414969745443"
date: 2026-05-23T09:34:41.000Z
type: thread
---
# 九原客 (@9hills)

在使用 Coding Agent 的时候，会话管理有两大流派：  
  
1. 单会话，就是维持一个持续 N 天的长会话（不断 compact），可以辅助 subagent 或者分支会话  
2. 即用即抛型：每个 Issue 都从头开始，尝试用 AGENTS.md + docs/ + Memory 解决记忆问题。  
  
我原来是2，现在是1。在 Pi 中我是这么做的：  
  
1. 每个大功能，开 worktree（有时直接复制目录）。单会话，加提示词（所有任务必须派发子 Agent 完成，不允许在主Agent编辑代码和运行测试）。这个会话持续到大功能结束。  
2. 灵活运用 /tree /fork 等命令，在 session tree 上来回移动，保持主干上下文的高效整洁。  
3. 当功能关闭后，整理单份文档放到代码库中，作为参考资料。

---

这里的大功能粒度很大，可能一个大版本就一个。