---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2115428696672866729"
date: 2026-10-28T12:00:53.000Z
type: tweet
likes: 60
reposts: 18
replies: 17
bookmarks: 60
views: 5062
---
# meng shao (@shao__meng)

MiniMax Code @MiniMax_Code 团队把他们每天真正在用的 Agent 工作流和 Skills 开源出来了！  
  
MCode Stack 中包含了团队何时拆子 Agent、长命令等多久等、长会话怎么保证不降级、每轮上下文塞什么 token 等等；把这些都写成可安装的 SKILL.md / 清单，并可作为插件进 MiniMax Code 和任意支持 Skills 的 Agents。  
  
开源地址：[github.com/MiniMax-AI/mco…](https://github.com/MiniMax-AI/mcode-stack)  
  
仓库里有什么  
按人打包成 stack（每人一套插件）：  
· stacks/ronny/：最完整，15 个 skill  
· stacks/jiachen/：偏「跳出 Agent 循环、把笔记做成帖/视频」  
· notes/：配套专栏 MCode Engineering Notes（文章讲道理，技能可执行）  
· community/、templates/stack-template/：欢迎外人贡献日常用法  
· 三类物件：Skill（/技能名 或按描述自动加载）、Checklist（逐项过项目）、Workflow（WORKFLOW.md 描述日常怎么用）。Ronny 的 WORKFLOW.md 仍标 Coming soon，先以 skills 为准。  
  
Ronny 那 15 个 Skills  
1. 规划与拆分：subagent-split-check、engineering-small-prs、engineering-change-descriptions；要不要派子 Agent；大改拆成可审小 PR；写清变更与原因  
2. 上下文与工具：tool-budget-audit、context-tax-audit；工具吃掉多少上下文；每轮注入了什么、对 prompt cache 的影响  
3. 长会话与长命令：long-session-check、handoff-doc、long-command-check；摘要/落盘/新开会话；交接文档；长 shell/后台任务十点审计  
4. 评审：independent-review、engineering-code-review 等；干净上下文只看 diff+需求+测试；代码健康评审；写/回评论  
5. 实验与热修复：harness-experiment、trajectory-review、engineering-emergency-changes；一次改一点、看翻盘用例；看轨迹不只看过没过；热修是否真紧急  
  
六条 engineering-* 改编自 Google Engineering Practices，改编部分仍属 CC-BY 3.0（见 stacks/ronny/NOTICE.md）；其余 MIT。  
  
配套观点（Notes 01 为例）  
子 Agent 首先是隔离上下文的工具，不是按「工种/问题类型」开分身；默认主循环 + 按需加载技能，只有量到上下文才是瓶颈再拆；代码评审给独立干净上下文。同系列还有工具过多吃上下文、compaction vs memory、bash 前台阻塞 vs 后台等（06-context-tax 标 upcoming）。

![Image](<meng%20shao%20(@shao__meng).assets/image-01.jpg>)

> **RonnyHe (@Ronny_MiniMax)**
> 
> We just open-sourced MCode Stack: the actual workflows and skills the MiniMax Code team uses every day to build MiniMax Code.  
>   
> https://github.com/MiniMax-AI/mcode-stack…