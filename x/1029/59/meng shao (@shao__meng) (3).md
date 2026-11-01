---
author: "meng shao"
handle: "@shao__meng"
source: "https://x.com/shao__meng/status/2114539832555065471"
date: 2026-10-25T15:58:12.000Z
type: tweet
---
# meng shao (@shao__meng)

How I Work with Agents：让 Agent 横向打通构建、发版与生产观测  
  
@thorstenball (@AmpCode) 把「怎么用 Agent」总结成了一套可执行世界观：软件变便宜、变化速率必须进计划、「堆 slop」论已过时、prompt 写 what 不写 how；真正放大收益的是 Agent × 代码库可读性 × 流程 三角，并让 Agent 横向覆盖构建 → 发版 → CI/生产日志/feature flag/监控，不只替换编辑器里的打字。  
  
他怎么看当下  
· 修 bug、改功能、出原型的时间常数变了；feature flag、Grafana 等过去嫌麻烦的事，现在交给 Agent 反而划算。  
· 不能只按「2026 年 10 月模型能做什么」定流程，要计入 token 降价与 frontier 约两周一更——可能处于「六个月后更便宜更快」的通缩期。  
· 「不审每一行就会堆成不可维护 slop」他认为已不成立：会有 slop，但强模型也能帮你清；去年模型常迷糊，现在可丢给很脏的代码修。  
· Prompt 升一层：不再强调文件 @ 提及与复杂排版；例：「把字节码编译器和 VM 提速一倍」「按这些截图做一个类似项目」。Amp Web/macOS/iOS 已去掉 @ mentioning。  
  
三角：Agent · 代码库 · 流程  
只把 Agent 塞进「以前文本编辑器那一格」，再照旧 push/review，GDP 不会涨。要让 Agent 竖向吃透前后端/库/基建之外，再 横向会部署、看 CI、读生产日志、拨 flag、看指标。Amp 自研叙事：约 200 万+ 行 monorepo、几乎全由 AI 做出；上 Orbs/云 Agent 后提交速度明显起来。  
  
个人日常用法  
· 永远用最强模型（点名 Opus 5.5）；便宜/开源模型适合固定工作负载，否则多轮纠错可能更贵。  
· 口述在升温；不再手工管 context（handoff、thread map 等被弱化）。  
· Agent 不写给人读的散文（Slack 人手写）；写机械文案：排障步骤、事故报告参数说明等。  
· Automations/定时唤醒 成刚需：发版后两小时再查错误率；sidebar 用「snooze until next message」保持干净。  
· 不要干等 Agent：异步推进；要改 UI 就一次要 多版变体 再点选，而不是来回迭代。  
· 只审承重代码；边缘（如 CSS）容错更高。口头禅（来自 Quinn）：「给我无可辩驳的证据，证明这能跑。」——E2E、截图、录屏、测试矩阵；有了证明再进 review，心态完全不同。  
· 约 8 周几乎不本地写代码；本机偶发杂务 / Pi 上跑 amp --no-tui 做家庭自动化。  
  
团队：让代码库对 Agent「可读」  
· 约 15 个 AGENTS.md + skills（dev server、gcloud、BigQuery、部署等）：能看见、能启动、能注入数据，才少幻觉。  
· Threads 作共享记忆：Agent 可 read_thread 追到同事某次提交的讨论。  
· All-in Orbs：一线程一临时机、空闲缩零；随便装依赖/FFmpeg/自建 slideshow。需要 macOS 特化时用 Runner，Orb 与本机线程互传文件与证明。  
· 生产侧：canary、日志、专给 Agent 走的生产路径（类比「为测试改代码」）；插件拨 flag、查健康、读 bug 诊断包。  
· 流程偏 AI-native：推 main、少/无 PR（避免 Orb 跑 20 分钟 CI 再远端重跑一遍）、设计师也在 Orb 里出原型与修代码、不用 Figma。  
· Slack 一点开 bug → Agent 拉诊断 → 修 → 发邮件/发积分；事故用 get system health 类插件。  
· 评测 Agent 友好度：改完 AGENTS.md 就 spawn 多个干净 Orb（含对照组）验证是否真走新脚本/插件，再合入。  
  
案例：Orbs OOM  
Slack 接报 → 起 Opus 线程查 top 内存进程与告警 → 查日志/BigQuery/相似 bug → 方案迭代 → feature flag 包起来 → 发版 → 人批准开 flag → 再 spawn Orb 人为吃内存验证 → 作者回传录屏确认。强调：小步发生产 + flag，在有 Agent 时比过去本地点点点便宜得多。

![Image](<meng%20shao%20(@shao__meng)%20(3).assets/image-01.jpg>)

> **Thorsten Ball (@thorstenball)**
> 
> ![Article cover](<meng%20shao%20(@shao__meng)%20(3).assets/image-02.jpg>)
> 
> 📝 **How I work with Agents**
> 
> ... as of October 2026.
> Use the latest and greatest models. Everything else is potentially a waste of time. If a model is wrong 10% of the time, you don't know which 10%, so you end up making multiple...
> 
> Finally summarized what I've been sending to friends in DMs