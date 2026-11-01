---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2114615306908029337"
date: 2026-10-26T06:27:53.000Z
type: tweet
---
# meng shao (@shao__meng)

Claude Code 怎么做 Code Review：把「有没有人读过」换成分层检查与快速回滚  
  
在「Claude 写约九成 Claude Code 代码、变更量暴涨」的前提下，团队没有取消 review，而是把质量问题从「有没有人读过每一行」改写成「这改动哪里可能坏、该由哪一层自动或人工接住」：多数阅读与纠错交给机器，人工留给产品判断与高风险面，并重仓 快速发现 + 易回滚；6–9 月 Claude Code 仓合并 PR 数翻倍以上，需 revert 的占比下降逾 40%。  
  
先重定义：review 在干什么  
传统两件事：发版前抓 bug；让团队共享对代码库的理解。Claude Code 团队主攻前者。  
  
关键改写是指标：不再把「有人读过吗」当充分条件，而问「什么会出问题，哪类检查（自动或人）最该抓住它」。时机也变了，从 PR 后闸门，变成写作过程里的 spellcheck：多数变更在人看之前，已被 Claude 至少审过并改过一轮，日常 bug 先滤掉，人留给更高层问题。  
  
人时是双面账  
审 PR 的时间 + 修坏代码的时间，只减前者不减后者等于成本搬家。目标写得很硬：CLI / 桌面网页 / 移动等 Claude Code 仓里，多数 PR 最多一人在环（可以是作者本人），且 revert / 事故率不升。跟踪：人均审批中位数与 P95、人工评论数、等人时长；revert 与事故率作护栏。  
  
# 变更旅程：六层检查，回滚托底  
无论来自工程师本机还是 Slack 里的 Claude 会话，典型路径是多层叠加，每层抓一部分、漏一部分；极少穿透全部，穿透了靠回滚。  
  
1. PR 前验证（大头在这里）  
要求「可跑的结果」而不是口头 done：跑测、构建、截图对设计；再用新上下文 subagent 审 diff。开 PR 前常跑内部 skills：找 bug、缩 diff、端到端、UI 对设计规范。Boris Cherny 估计：给 Claude 配上 verification，产出质量大约 2–3 倍；形态因域而异（bash、测试套件、浏览器/模拟器）。同一类问题反复被审出，就回修 skill / lint / 文档：  
CLAUDE.md：栈、约定、目标与非目标；每次 review 都会载入，大仓宜短并外链。破约定或文档过时要被标。  
  
REVIEW.md：所有者告诉审阅者本地什么算失败（如文档「说谎」、配置「误路由」）、少 nit、忽略路径、必须文件+行号、顶层摘要含数量并在无问题时写清。  
  
2. 每条 PR 的 AI review  
人看之前先过 Claude：对照全仓与 PR 意图，是否真解决问题、是否合系统、是否打别处；修 bug 时追根因。产品侧：Team/Enterprise 的 Code Review；本地 /code-review。研究预览后，内部「有实质评论」的 PR 约从 16% → 54%；>1000 行多数能检出，小 PR 多数干净。深度：Quick / Standard / Deep（漏一个 bug 很贵时用 Deep）。多 agent 并行找 bug → 投票滤假阳性 → 丢掉纯风格/清理类；评论标 Important / Nit / Pre-existing。线程必须修掉或说明关闭；常有 agent 隔夜代修。  
  
3. 可阻断的安全审查  
安全单独过一轮；code/bug 或 security 的 finding 开着就不能合。可修，或短理由关假阳性；后续 commit 修好可自动关。可对齐开源 security Action 或 /security-review。  
  
4. 阻断式自动化检查  
lint、类型、多平台测、构建等；CLI PR 常在人看前跑几十项。模板强制写「怎么测」「怎么 revert」，Claude 可填，作者须核对。  
  
5. 按风险缩放的人工批准  
· Low：文档、文案、仅内部功能、flag 后的加法改动；自动 review + 安全通过后，可不要求人批  
· Medium：普通产品改动；缓存、auto-updater、system prompts 等敏感区；1 人人工批准  
· High：最高风险代码；另有少量路径始终要两人；2 人人工批准  
  
另有分类器可在「档位未要求」时仍加一人；所有者标为敏感的路径、CI workflow 等常强制人批。受信名单（多为本仓团队）才有放宽路径，其余需第二人。Agent 批准永不顶替人；低风险可用自动审查清关，但需要人的坑位机器填不了。Agent 开的 PR，操作者算一名必需审阅者。批准只对当前那一 commit有效：不可信的新 commit（agent 再推、手解冲突等）会撤批，须重批。  
  
6. Merge queue、feature flag、分阶段放量  
Queue 在「带上该变更的 main」上再跑通，避免两个各自绿的 PR 合出坏 main。新功能通常先 flag、先员工；公开放量先 latest，stable 滞后。  
  
回滚层：快发现、快撤  
坏合入难免。Flag 可关不必重部署；周轮值 on-call；事故复盘问「哪一层该抓住」。发现仍靠作者、on-call 与 revert/事故看板。尚无按这些指标自动收紧审批的 circuit breaker，但在更多「零人工合入」前工程侧希望有。优先提高发现与纠正速度，而不是靠整体减速（减速降错多半因为做得更少）。  
  
让审阅者变好：事故当 eval，强制收敛  
· 事故 PR + fix-forward/revert 做成评测集，懂代码的人核实 bug 位置；审阅系统每日打分；改 prompt/工具/模型就重打全集。  
· 用人对评论的处置（修/拒/澄清故意/无所谓）管噪声；也对已知干净 PR 跑审，看会不会诱导有害改动。优先级写死：先真 bug，再减轮次。  
· Agent 作者处理 nit 的成本远高于人一眼关掉，会再 commit、再审一轮。  
· 曾见 CLI 上作者 agent 与审阅 agent 隔夜循环 50+ 轮；git blame 显示再报问题约 59% 是第一轮就已有的旧码。对策：约三轮后要求重评方案；后续只审新 diff；按 bug 类一批修；勿重复报；夜间扫历史代码。收敛后内部平均审阅轮次下降。  
  
人仍必须 in loop 的事  
该不该建、品味与范围；计费 / auto-updater / CI 等危险面；会定约定的架构；核验「证明」（测输出、命令、截图、审阅意见），说不清检查会漏什么，就别只求 👍；对每条合入负责的是人，不论谁敲的字。  
  
共享理解怎么补  
机器读了大部分 diff 后，无人脑内装下整仓，也不追求。要学某切片，就让 Claude 写 explainer（可再做成可视化小工件），比扫一行文案 diff 有用得多；默认公开频道协作，边做边看见别人在建什么。  
  
You land it, you own it  
中小变更出问题，规范是作者通常自修。靠：变更小、可回滚、爆炸半径定门槛、遥测与内部狗粮快暴露、事故看系统不甩锅；开放频道让非所有者也能反馈。Agent 让「开写」变便宜，频道里有人指出歧路就早关 PR / 改道，不把沉没成本当理由拖到终审才挡。  
  
可迁移落地的经验  
· PR 前：短 CLAUDE.md 检查清单 + hooks；本地 /code-review、/security-review 或新上下文 subagent；REVIEW.md 收重复意见。  
· PR 上：CI；模板含 Testing / Risk / How to revert；先一仓试 AI 审并调几周 REVIEW.md；安全审查；线程须关闭；按风险选 Quick/Standard/Deep；限轮次，之后只审新 diff。  
· 批准：按变更写风险档，先严后松；受信才放宽；bot 不可顶替人批；新 commit 撤批；CODEOWNERS 护 auth、billing、CI、发布与安装器。  
· 合并后：queue、flag、内测、分阶段、一键 revert；事故/revert 进 eval；盯人时，以 revert/事故为护栏。  
· 方向：产品对外仍是「贴发现、人做最终批准」；自家仓 owners 已在让部分低风险在检查绿后自动合，边界继续外移，漏检就补 CLAUDE.md / skill / 新检查。  
· 目标未变：降每人次人时，revert 不差于现状。

![Image](<meng%20shao%20(@shao__meng)%20(2).assets/image-01.jpg>)

> **Addy Osmani (@addyosmani)**
> 
> ![Article cover](<meng%20shao%20(@shao__meng)%20(2).assets/image-02.jpg>)
> 
> 📝 **How the Claude Code team does code review**
> 
> We previously mentioned that Claude writes about 90 percent of the code for Claude Code. So how do we do code review at this scale? Who reads code anyway? What happens if code goes wrong?
> This post...