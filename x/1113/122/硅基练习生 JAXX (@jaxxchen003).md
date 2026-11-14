---
author: "硅基练习生 JAXX"
handle: "@jaxxchen003"
source: "https://x.com/jaxxchen003/status/2119409655948661178"
date: 2026-11-08T04:20:41.000Z
type: tweet
likes: 120
reposts: 18
replies: 8
bookmarks: 169
views: 12413
---
# 硅基练习生 JAXX (@jaxxchen003)

最近看到 @DIYgod 开源了 Pinky，加上之前关注的 Poki，手机端 Phone-Use Agent 显然正在迎来一轮爆发。  
  
把两个项目拉通对比后，发现两者技术选型与产品思考截然不同：  
  
1/ Pinky（DIYgod）—— 做面向最终交付的安全管家  
  
•跨平台通用：兼顾 iOS + Android，桌面端原生打包（仅十几 MB），通过  brew cask  就能装好跑起来。  
•硬核拦截（Human-in-the-loop）：最亮眼的是它的网关审批。Agent 执行“支付/转账/发消息/删除”动作时会被强行扣住，等你在电脑端确认才放行，从物理层杜绝提示词注入风险。  
•全流程审计：自带动作录屏与状态标记，支持导出 HTML 报告，非常适合需要可信交付的商业自动化流程。  
  
2/ Poki（Joseph Zhou）—— 面向极客的高性能底层桥接  
  
•专注深挖 iOS：免越狱且手机不装任何 App，直接挂载苹果原生无障碍控件树与私有通道，相比纯视觉截图猜坐标，定位精准度更扎实。  
•带执行记忆：内置自研 Agent 具备路径学习能力，跑顺的流程后续可直接复用，甚至免调大模型，省下大笔 Token。  
•门槛更偏开发者：需本地编译环境（Xcode/Rust/Node），更像给资深工程师打造的无头执行引擎。  
  
3/ 我个人认为：  
  
手机端 GUI Agent 的瓶颈从来不只是“能不能点到按键”，而是风控断点接管和复杂页面健壮性。  
Pinky 给出了目前最符合人类直觉的落地方案：让 Agent 随便折腾，但只要动钱动社交，立刻弹窗找人兜底；而 Poki 展现了无需模拟器、深度咬合原生控件树的性能上限。  
  
两个项目都支持标准 MCP，搭配 Claude Code 或 Cursor 就可以直接给 Agent 接入真实手机操作权限。做海外社媒自动化、低频工作流或测试脚本的同学，非常建议拉下来试跑一下。  
  
项目地址：  
•Pinky: [github.com/DIYgod/pinky](https://github.com/DIYgod/pinky)  
•Poki: [github.com/JosssphZhou/po…](https://github.com/JosssphZhou/poki)

![Image](<硅基练习生%20JAXX%20(@jaxxchen003).assets/image-01.jpg>)

> **brucexu.eth (@brucexu_eth)**
> 
> 卧槽，这个直接把 iPhone Use 这个赛道杀死比赛了。试了一个任务太无敌了。跟之前的 iPhone Use 对比了一下，有几个很关键的点：  
>   
> 1. 之前 iphone-use 做不到的，现在能做到了。之前有一个功能，或者是微信小程序的几个表单老是点击不到，但是现在是 OK 的。  
> 2. 使用非常方便，就是下载 APP x.com/DIYgod/status/…