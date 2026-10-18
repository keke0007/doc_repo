---
author: "YT"
handle: "@huangcheng943"
source: "https://x.com/huangcheng943/status/2111359790291279995"
date: 2026-10-17T04:31:37.000Z
type: tweet
---
# YT (@huangcheng943)

怎样判断一份 AI Skill 写得真有用，而不是只有一段好听的提示词？obra（Jesse Vincent）维护的 Superpowers「writing-skills」有独立 SKILL.md，把写 Skill 当作测试驱动的流程文档：先设计会暴露问题的场景，观察未加载 Skill 时代理如何失败；再写最小规则，复测并收紧漏洞。适合团队把重复的代码审查、排错或交付步骤沉淀为可验证的 Skill。  
  
安装：Codex App 侧边栏 Plugins → Coding → Superpowers → 安装；Codex CLI 输入 /plugins，搜索 Superpowers，选择 Install Plugin。新开会话或重启以重载；在可用 Skill 列表找到 writing-skills，打开 SKILL.md，核对名称和「先基线、后写规则、再复测」流程。第三方 Skill 安装前先查维护者、SKILL.md 与权限。  
  
可复制调用模板：“请使用 writing-skills，为公开示例仓库的 <重复工作> 设计一个 Skill。先列触发条件和失败场景，记录未加载时的基线，再写最小 SKILL.md，复测并报告差异；未验证的效果请标明。”  
  
练习输入：让代理每次改完公开演示项目的函数后，都附测试命令、退出码和结果。期望输出：一份含 name、description、步骤和验证记录的 SKILL.md，以及基线与复测对照；这只是练习目标，并非保证代理一定遵守。  
  
该 Skill 的文本与核心方法按 MIT 许可免费获取和使用；AI 客户端、模型调用以及可选的云测试或商用支持可能收费，按各服务套餐和用量计算。它可能读取仓库、运行命令和调用测试代理；只在授权的练习仓库执行，先检查命令及权限，不放入令牌、私有代码或客户数据。  
  
#免费AISkill #Skill开发 #软件工程  
  
官方来源  
[raw.githubusercontent.com/obra/superpowe…](https://raw.githubusercontent.com/obra/superpowers/main/skills/writing-skills/SKILL.md)

> **YT (@huangcheng943)**
> 
> 写功能前先写一个会失败的测试，究竟怎么用 AI 带着做？obra（Jesse Vincent）维护的具体 Skill「test-driven-development」有独立 SKILL.md，要求按红→绿→重构推进：先写只验证一项行为的测试，亲眼看到它因缺少功能而失败；再写最少代码让它通过；重构后重新跑测试。适合学习小功能、修 bug x.com/huangcheng943/…