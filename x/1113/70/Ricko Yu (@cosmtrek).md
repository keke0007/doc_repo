---
author: "Ricko Yu"
handle: "@cosmtrek"
source: "https://x.com/cosmtrek/status/2119952258758774866"
date: 2026-11-10T00:36:19.000Z
type: thread
---
# Ricko Yu (@cosmtrek)

给 Claude Code 写了个委派任务技能，分工很简单：Codex 负责「看清楚」，Claude 负责「想清楚」，实现只在想清楚之后才派出去。

![Image](<Ricko%20Yu%20(@cosmtrek).assets/image-01.jpg>)

---

为什么这么分  
统计下来，主会话约 80% 的开销是缓存读，上下文越长越贵。把翻代码、查日志这些移出主会话，比省输出 token 有用得多。  
  
技能持续优化  
每次派发自动记台账，复核后补上结论和失败原因。定期复盘，把学到的写回技能，具体事例留在复盘记录里，技能正文只写原则。