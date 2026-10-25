---
author: "卢卡lukasen"
handle: "@lukasen_xyz"
source: "https://x.com/lukasen_xyz/status/2113532144287150367"
date: 2026-10-23T07:25:05.000Z
type: tweet
likes: 36
reposts: 6
replies: 5
bookmarks: 50
views: 3249
---
# 卢卡lukasen (@lukasen_xyz)

改复杂流程，很多人第一反应是：直接让 Agent 动手改代码  
  
poteto 的做法反过来  
她先让 Agent 画一张状态图  
再拿自己用 pstack 做的形式化验证 skill，对着规格找 bug  
说是既能看清代码，也能看清用户流程本该怎么走  
  
她现在还在做配套 skill：让 Agent 把图画好看，再跟规格对一遍  
还没正式放出来  
  
形式化验证门槛高，普通业务未必上得起 TLA+  
但前面那一步，状态图，其实谁都能先做  
  
很多 bug 不是逻辑写错  
是隐含规则没摊开：这个按钮在什么状态下能点，哪个状态之间根本不该跳  
  
下次改订单、审批、登录这类多状态流程，先别让它改代码，丢这句：  
“先别改代码。把这个功能的用户流程画成 mermaid stateDiagram，列出每个状态的进入条件，再列出所有不该出现的状态转移，逐条到代码里找有没有可能触发”