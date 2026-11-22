---
author: "yibie"
handle: "@yibie"
source: "https://x.com/yibie/status/2121790251735912846"
date: 2026-11-15T02:19:51.000Z
type: thread
likes: 11
reposts: 1
replies: 9
bookmarks: 10
views: 794
---
# yibie (@yibie)

用 80 行代码让 agent 记住约束  
  
在项目根目录放一个 Markdown 文件，硬性限制在 80 行以内，让 agent 每次开新会话先读它。它解决的是长任务里最烦的事：开头交代过的禁令和环境约束，经过一次 compaction（上下文自动压缩）或换个会话就被忘掉，同一个坑反复踩。  
  
文件里只放四样东西：  
1. Active Landmines：绝对不能踩的约束和禁令  
2. Equipment/Context：该参考的设计文档、工具的标题链接  
3. Next Tasks：没做完的待办清单  
4. Current Location 加 Pin Buffer：当前进度，以及干活时临时记下的教训  
  
80 行是硬上限。规则是不追加历史，只把旧进度总结覆写成「当前位置」；一旦放任它变长，agent 就会跳过不读。  
  
干活中发现新陷阱或解法，不要立刻写进长期知识库，只在缓冲区记 1 到 2 行。会话结束时再审计这些记录，只把真正可复用的经验整理出去，写进一个带双链的 Markdown 知识库，按五段记：情境、踩坑、怎么校准、最终应有的状态、下次什么关键词触发回忆。搬走后清空缓冲区，临时试错就不会污染长期记忆。  
  
长期库变多以后，调取时先只返回一行摘要和相关度，agent 觉得必要再读全文，避免五条结果就吃掉几万 token。原文还建议设 7 天周期，长期没被引用的经验降权、归档，也就是主动「遗忘」。  
  
好在整套东西就是纯 Markdown 加一个读取接口（比如 MCP），不依赖某个向量库。作者没有公开具体实现代码，文件里放什么、什么时候清空，照上面这套做就行。  
  
原文链接在评论区  
#小技巧

![Image](<yibie%20(@yibie).assets/image-01.jpg>)

---

原文：

> ![Link card preview](<yibie%20(@yibie).assets/image-02.jpg>)
> 
> 🔗 [**Why do AI agents forget constraints and past lessons? The overall design of a 'brain-inspired...**](https://t.co/H5lGCtzKK7)
> 
> _From note.com_