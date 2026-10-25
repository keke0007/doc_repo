---
author: "旺旺"
handle: "@TiagerBao"
source: "https://x.com/TiagerBao/status/2113759355879436760"
date: 2026-10-23T22:27:56.000Z
type: tweet
likes: 3
replies: 2
bookmarks: 2
views: 251
---
# 旺旺 (@TiagerBao)

Claude Code 跑完任务就不管了，留下七八个 Worktree，最后还得我自己一个个查、一个个删。  
  
找到一个专门干这事的 Skill：worktree-cleaner。  
  
它会检查每个 Worktree：  
  
分支是否合并、PR 是否关闭  
有没有未提交的代码或未推送的 Commit  
有没有其他 Agent 正在使用  
哪些目录已经失效  
  
然后标记为 SAFE、REVIEW、AT_RISK、IN_USE 等状态。  
  
重点是：不会上来就删。  
  
先列出检查结果，用户确认后才清理。删除前重新检查，删除后验证结果。  
  
有未提交代码的不乱动，正在使用的不碰，本地分支也不会顺手删掉。  
  
多 Agent 并行开发经常留下 Worktree，这个 Skill 正好解决收尾问题。  
  
GitHub：[github.com/davila7/claude…](https://github.com/davila7/claude-code-templates/blob/main/cli-tool/components/skills/git/worktree-cleaner/SKILL.md)  
  
#ClaudeCode #Agent #Git #Worktree #Skills #OpenSource

> ![Link card preview](<旺旺%20(@TiagerBao).assets/image-01.jpg>)
> 
> 🔗 [**claude-code-templates/cli-tool/components/skills/git/worktree-cleaner/SKILL.md at main · davila7/...**](https://t.co/zTaLLMN6uC)
> 
> _From github.com_