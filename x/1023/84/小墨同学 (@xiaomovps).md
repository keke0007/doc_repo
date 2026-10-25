---
author: "小墨同学"
handle: "@xiaomovps"
source: "https://x.com/xiaomovps/status/2112803313272156437"
date: 2026-09-10T09:55:05.000Z
type: tweet
---
# 小墨同学 (@xiaomovps)

分享几个 Pi 进阶使用技巧，这次不聊快捷键，聊点真正能改变工作流的东西🔥  
  
1、让每个 Agent 都在独立的 Git Worktree 里干活  
  
试试 pi-bare-worktrees：[github.com/jordyvanvorsel…](https://github.com/jordyvanvorselen/pi-bare-worktrees)  
  
多个 Pi Session 同时操作一个仓库，最怕的就是改到同一份代码，甚至把 main 分支搞乱  
  
这个插件可以给任务创建独立 Worktree，默认保护 main，后续工具调用也会自动在当前 Worktree 里执行  
  
2、给 Pi 配一套真正的 Subagent 系统  
  
[github.com/tintinweb/pi-s…](https://github.com/tintinweb/pi-subagents)  
  
可以定义不同类型的 Agent，让它们使用各自的模型、System Prompt 和工具权限，还能后台并行执行，中途随时追加指令  
  
比如一个负责读源码，一个负责补测试，一个负责 Review，最后让主 Agent 汇总结果  
  
3、自己定制 Context 压缩策略  
  
Pi 默认会在上下文接近限制时触发 Compaction，但你也可以通过 Extension 改写这套逻辑  
  
官方就有一个示例，使用另一个模型来总结完整上下文。如果主模型比较贵，可以单独用便宜的模型做摘要，再按自己的需求保留关键决策、修改进度和待办事项  
  
4、别把所有项目规则都塞进 AGENTS.md  
  
Pi 的 Skills 支持按需加载完整指令，还可以附带脚本、参考资料和其他文件  
  
常用的编码规范放在项目规则里，数据库迁移、发布流程、代码审查这些专项工作则单独做成 Skill，需要时再调用，避免每次都把所有说明塞进 Context  
  
5、Pi 不一定只能运行在终端里  
  
官方支持 RPC 和 SDK，可以通过 JSON 消息控制 Pi，也可以把它嵌入自己的应用  
  
如果你想给 Pi 做一个 Web UI、接入自动化脚本，或者让其他程序直接调用它，就不必从头实现一套 Agent Runtime  
  
这些其实都是在我的 PI 蓝皮书里面已经写了的，我很久没有去推我的项目了，但是也收获到了不错的支持，有很多朋友都过来感谢我  
  
其实我最近也一直在默默的更新，增加了许多内容，希望以后也会越来越多的人了解和学习 PI

> **小墨同学 (@xiaomovps)**
> 
> 📝 **从零开始，开源一本属于自己的 Pi 蓝皮书**
> 
> 这两个月以来我都在深度的学习和使用Pi。
> 一开始只是因为单纯的好奇，为什么会有人使用一个这么简洁的Agent？直接使用Codex、Claude Code不好吗？抱着这个疑问，我就开始了我的Pi Agent学习之旅。...