---
author: "YT"
handle: "@huangcheng943"
source: "https://x.com/huangcheng943/status/2114337109733671235"
date: 2026-10-25T08:07:00.000Z
type: tweet
---
# YT (@huangcheng943)

收到一串代码审查建议，先别让 AI 一口气全改。上一帖讲如何发起审查，这次讲怎样处理反馈。  
  
Superpowers 的 receiving-code-review Skill，要求先读完反馈、理解要求，再对照当前代码核验，逐条修改并测试。适合 PR 评审、同事建议和另一个 AI 给出的修改清单。  
  
安装：Codex App → Plugins → Coding → 安装 Superpowers；重新打开任务，再调用 $receiving-code-review。未识别时检查插件是否启用并重新加载。第三方 Skill 安装前请读 SKILL.md，检查权限范围。  
  
可复制模板：  
“使用 $receiving-code-review 处理以下反馈。先复述每项技术要求，对照代码、兼容目标和测试核验；不清楚的先澄清，建议不适用时说明证据。每次只改一项并运行相关测试。”  
  
示例：审查意见是‘删除旧接口，改用新 API’，项目却仍支持旧系统。合理输出应先检查目标版本与新 API 可用范围，再决定保留兼容实现或替换；不能只因为审查者语气确定就照改。此为示意案例，不是实测结果。  
  
验证是否生效：看它是否先核验建议与项目条件，是否逐项给出修改及测试证据；无法验证时应明确缺少什么信息。Skill 是工作流程，不能保证审查意见或修复自动正确。  
  
MIT 开源免费；模型调用、云端运行可能另计费。提供反馈时只带必要代码与错误信息，去掉密钥和私人数据，限制命令执行范围。  
  
#实用Skill #代码审查 #AI编程  
  
官方来源  
[github.com/obra/superpowe…](https://github.com/obra/superpowers)  
[github.com/obra/superpowe…](https://github.com/obra/superpowers/blob/main/skills/receiving-code-review/SKILL.md)

> **YT (@huangcheng943)**
> 
> 让 AI 检查代码，只丢一句“帮我看看”，常常换来几条泛泛建议。免费开源的 requesting-code-review Skill，提供了一套更具体的审查准备流程。  
>   
> 它适合功能完成、复杂 bug 修复或合并前：把需求、实现摘要和 Git 起止提交交给审查者，按严重程度处理反馈。  
>   
> 从零安装：Codex App → Plugins → Coding → x.com/huangcheng943/…