---
author: "it_works"
handle: "@it_works_123"
source: "https://x.com/it_works_123/status/2120164911485714668"
date: 2026-11-10T14:41:20.000Z
type: tweet
---
# it_works (@it_works_123)

写 Pi Bot的时候，发现很多人对 Agent Loop 有误解。  
  
以为 Agent 就是一个 while(true)，不停调用大模型。  
  
实际上，每轮模型会根据上下文决定：直接回答，还是调用工具。  
  
如果调用工具，程序执行后把结果送回模型，再进入下一轮。  
  
听起来很简单，直到你开始考虑：  
  
工具失败怎么办？模型不返回怎么办？用户中途插话怎么办？任务什么时候结束？  
  
普通 while 循环只需要判断条件。  
  
Agent Loop 还得判断模型是不是在认真干活。  
  
这下知道为什么 Agent 需要 Harness 了吧  
  
模型负责决定下一步做什么，Harness 负责防止它把下一步变成下辈子