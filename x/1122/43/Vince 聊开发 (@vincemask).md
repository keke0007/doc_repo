---
author: "Vince 聊开发"
handle: "@vincemask"
source: "https://x.com/vincemask/status/2122700325840646213"
date: 2026-11-16T22:31:41.000Z
type: tweet
likes: 61
reposts: 11
replies: 4
bookmarks: 60
views: 7747
---
# Vince 聊开发 (@vincemask)

Claude Code 团队分享了他们如何验证生成的代码。  
  
文章提到目前约 90% 的代码由 Claude 编写和 Review。随着 AI 参与的开发越来越多，如何让 Agent 自己验证结果，成为提升交付效率的关键。  
  
文章里有几个值得借鉴的实践：  
  
1. 优先投入三类测试  
  
E2E 测试：验证真实用户操作流程，让 Agent 实际运行应用，通过浏览器、截图和执行记录定位问题。  
  
Property Testing：通过随机生成大量输入，检查幂等性、数据一致性等必须满足的约束。  
  
Differential Testing：用相同输入对比新旧实现，适合重构、迁移和性能优化，避免引入行为差异。  
  
2. 给 Agent 配置专门的 Verify Skill  
  
在项目的 .claude/skills/ 下建立验证 Skill，定义如何启动应用、执行测试和检查结果。  
  
让 Agent 完成修改后能够自己运行、发现问题并修复，形成完整的反馈闭环。  
  
3. Code Review 规则按需触发  
  
团队维护了 39 项可靠性检查规则，每项都有明确的触发条件。Agent 根据代码 Diff 选择相关规则，通常只需要检查 3～10 项。  
  
这样既能利用历史故障经验，也能避免每次 Review 都执行一大堆无关检查。  
  
4. 自动修复要有边界  
  
检查失败后，Claude 会尝试自行修复。如果连续三轮仍未通过，就交给人工处理。  
  
5. 验证机制本身也需要验证  
  
Review 规则从基准分支读取，避免代码修改自己的检查标准。  
  
新增质量门禁先运行 Observe 模式，确认误报情况后再开启强制拦截。  
  
根据真实线上故障、回滚和 Review 漏检持续改进规则。  
  
定期清理无效测试和冗余规则，防止验证成本持续膨胀。  
  
最值得借鉴的是团队对 Agent 工作方式的调整：  
  
开发者逐渐从逐行检查 AI 代码，转向定义验收标准、建立验证工具，并根据实际执行结果判断代码是否可以交付。

![Image](<Vince%20聊开发%20(@vincemask).assets/image-01.png>)

> **ClaudeDevs (@ClaudeDevs)**
> 
> Claude writes better code when it can verify its work. Checks that used to be expensive, like tests, benchmarks, and end-to-end runs, are now cheap.  
>   
> Here’s how the Claude Code team thinks about verification:  
> https://claude.dev/blog/how-the-claude-code-team-thinks-about-verification/…