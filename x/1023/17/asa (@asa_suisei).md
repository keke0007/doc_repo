---
author: "asa"
handle: "@asa_suisei"
source: "https://x.com/asa_suisei/status/2113260016648290693"
date: 2026-10-22T13:23:45.000Z
type: tweet
likes: 12
reposts: 2
replies: 14
bookmarks: 12
views: 1427
---
# asa (@asa_suisei)

开源一个multi-agent的工作流，适用于claude code, codex, opencode等agentic coding工具  
  
「四权分立」（Four Powers）：给 coding agent 的权力分立工作流。  
  
我在用 agent 做长周期的scientific computing项目，比如性能优化、数值代码、要跑几小时的实验。过程中反复遇到这几个问题：  
  
① agent 写完代码后自己判断"没问题"，跑了两天实验后发现完全在浪费时间做无意义的实验。  
② 审查意见被悄悄改写成一句"注意事项"，然后照样推进  
③ "测过一次"慢慢变成了"已确立的事实"  
④ 盯着一个问题反复优化，钻牛角尖，忘了项目的总目标  
⑤ 换了会话，上下文就断了；状态文件越写越长，最后变成流水账  
⑥ 出了问题没法追溯：当时给 agent 的指令是什么？审查者反对的又是什么？  
  
对应的设计：  
  
1. 四权分立  
统筹（Orchestrator）、执行（Executor）、审查（Reviewer）、终审（Senior Reviewer）由四个 agent 分别担任。  
执行者没有决策权，审查者不能改文件。  
阻断性的审查意见只有两种方式能推翻：拿出新证据，或者由终审裁决。只靠重新解读不行。  
  
2. project_state.md + project_log.md  
state 只记录"现在"：总目标（北极星）、当前阶段、已确立的事实、待验证的假设、下一步最有价值的行动。过时的内容移到只追加的 log。  
  
state 有硬性上限（250 行 / 30 KB），由 pre-commit hook 检查，发现"日志式写法"就拦截提交。每个会话开始时先问一句："没做完的事，还是最有价值的吗？"  
  
3. agent_log  
每一次委派、回复和裁定都逐字归档，不做摘要，并建立索引，记录 Task ID、返工轮次 R1/R2/R3 和裁定结果。这样有两个用处：  
· 可追溯：几周后如果质疑某个结论，原始的指令和异议都还在  
· 可测量：给四个角色换了模型以后，如果索引里大量出现 R3 和升级，说明这组模型不适合这项工作。  
  
终审的裁决还能告诉你该换哪个角色：  
经常支持审查者，那么问题在执行者。  
经常推翻审查者，那么问题在审查者  
  
当积累到一定程度的时候，你甚至还能做一个统计，比如说分别用codex和claude的model的时候，哪个组合成天在project里面反复做无用功。  
  
（下图解说， codex 的 23 次 Senior 裁决里有 10 次是 UPHOLD（支持 Reviewer），说明主要是 Executor 一再过不了同一个异议，也就是在反复做无用功。claude 的 45 次里有 33 次是 REFRAME（重新界定问题），只有 1 次 UPHOLD。也就是说，claude 的升级大多是在讨论问题本身该怎么定义，不是执行被卡住。）  
  
4. 小工具wait_run.py  
agent 等待长实验时爱用 sleep 循环，浪费工具调用和上下文，有时还会在任务没跑完时就交差。这个工具会一直阻塞，直到进程退出、报错、卡住或进入新阶段才返回。  
  
注意事项  
这个工作流用速度来换可信度：适合重视正确性、要跑长时间实验的项目。如果更看重快速反馈，比如做 UI 之类的就不太合适。  
  
包含 AGENTS.md、3 个 Claude Code subagent 和会话工具，只依赖 Python 标准库，MIT 许可。  
[github.com/AsaSuisei/four…](https://github.com/AsaSuisei/four-powers)

![Image](<asa%20(@asa_suisei).assets/image-01.jpg>)