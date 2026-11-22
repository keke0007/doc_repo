---
author: "lifcc"
handle: "@mylifcc"
source: "https://x.com/mylifcc/status/2123582347023159572"
date: 2026-11-20T01:01:00.000Z
type: thread
likes: 169
reposts: 30
replies: 17
bookmarks: 239
views: 9995
---
# lifcc (@mylifcc)

为什么同样是用 Claude Code 或 Cursor 写代码，有人觉得它是能顶半个团队的神器，有人却觉得它写超过三个文件就开始胡编乱造、越改越乱？  
  
这周在 r/ClaudeCode 收获 320 多赞、近两百条评论的一篇实战贴里，一位每天高强度用 AI 编程 6 小时、累计超过 4,500 小时的资深后端工程师 Vaibhav，毫无保留地开源了自己打磨两年的全套工作流仓库 my-ai-coding-workflow。他总结出的核心秘诀只有一句话：永远不要在没有状态门禁的情况下，允许 Agent 直接动手改业务代码。  
  
他把整个 .claude/ 目录设计成了一台严格的三阶段状态机门禁：  
第一道门叫 Plan（架构对齐）：强制 Agent 先只读扫描相关模块，输出带明确文件边界、接口签名与不变式约束的方案文档，此时严禁调用任何写文件工具；  
第二道门叫 Critique（对抗挑刺）：拉起一个独立的审稿子上下文，专门以“冷酷的资深架构师”视角对刚才的 Plan 做攻击性挑刺，找出隐式耦合、边界遗漏和过度设计；  
第三道门叫 Readiness（就绪放行）：只有当方案通过挑刺、明确了验证单测命令后，才解锁代码写入权限。靠这三道看似“慢半拍”的门禁，他把复杂功能的返工率砍掉了 80% 以上。

![Image](<lifcc%20(@mylifcc).assets/image-01.jpg>)

---

4500+ 小时 AI 编程三阶段门禁工作流开源仓库与 r/ClaudeCode 原帖：  
[github.com/vaibhav-space/…](https://github.com/vaibhav-space/my-ai-coding-workflow)  
[reddit.com/r/ClaudeCode/c…](https://reddit.com/r/ClaudeCode/comments/1xva9p0/my_ai_coding_workflow_after_4500_hours/)

> ![Link card preview](<lifcc%20(@mylifcc).assets/image-02.jpg>)
> 
> 🔗 [**GitHub - vaibhav-space/my-ai-coding-workflow: Production-grade AI coding instructions, curated...**](https://t.co/NFOL16W981)
> 
> _From github.com_