---
author: "老金"
handle: "@freeman1266"
source: "https://x.com/freeman1266/status/2055560273520697600"
date: 2026-05-16T08:05:34.000Z
type: tweet
---
# 老金 (@freeman1266)

解决了 Claude Code 上下文遗忘的一个思路：  
   
 CLAUDE.md  — 项目规则和约束，每次会话自动加载  
  
 PROGRESS.md  — 任务状态追踪，AI 不能自己宣布完成  
  
 memory/  — 跨会话的决策记录和踩坑日志  
   
关键机制：每个任务结束，AI 必须更新这三个文件。不更新，不算完成。