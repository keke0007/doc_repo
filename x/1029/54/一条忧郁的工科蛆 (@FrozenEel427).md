---
author: "一条忧郁的工科蛆"
handle: "@FrozenEel427"
source: "https://x.com/FrozenEel427/status/2114499749319729557"
date: 2026-10-25T23:30:00.000Z
type: tweet
---
# 一条忧郁的工科蛆 (@FrozenEel427)

一块几十块的 ESP32，现在能直接接上 Meta 的 AI Agent 了。  
Meta 把 Muse Gadgets 的 SDK 和固件开源了，Apache 2.0 协议。  
昨天聊的是“大脑塞进本地板子”，这次正好反过来：板子开源，大脑在 Meta 云端。  
  
先看它能干什么。据 Hackster 和 Open Source For You 的报道：  
- 有两个 SDK。ESP32 版管屏幕、麦克风、喇叭、按键、传感器；Linux 版给树莓派这类电脑用，还能执行自定义命令、对接 Home Assistant。  
- 官方示例有三个：Waveshare 的 1.75 寸圆形 AMOLED 触屏 ESP32-S3 板；Seeed 的 reTerminal E1002 彩色墨水屏，用来显示每日简报、提醒、购物清单；还有树莓派 5 接 Home Assistant。  
- 不想自己焊的，Meta 还做了一个 Muse Home Link，基于 ESP32-C5，支持双频 Wi-Fi 6、蓝牙 LE 和 Zigbee/Thread。美国的活跃订阅用户可以免费领，送完为止。  
- 自己开发需要一个 SDK token，再用 Muse 手机 App 配对。  
  
拿它跟 ESP32 小智比，会更清楚。  
  
小智的思路是：板子只负责听和说，真正的大脑在云端服务器上。固件开源，社区里也有自建服务端的项目，所以你可以选择接官方服务，也可以把整条链路搬到自己机器上，换成自己想用的模型。  
  
Muse Gadgets 也是固件开源、大脑在云端，但这个大脑只能是 Meta 的 Muse。你要先有 token、要用他们的 App 配对，能做什么、怎么收费、哪天停服，都由 Meta 决定。  
  
所以“算不算真开源”，我个人觉得要分两层看：  
- 代码层面，Apache 2.0 是很宽松的协议，硬件接口、外设驱动这些你都能随便改，这部分是真开源。  
- 能力层面，最核心的那个 Agent 不在你手里。离开 Meta 的账号和服务，这块板子能做的事我猜会很有限。  
  
这不一定是坏事。大厂 SDK 的好处很明显：Agent 能力强，生态现成，几乎不用自己调模型。代价是隐私和长期可控性。全自己搭正好相反，自由度最高，但从语音识别到模型到服务端都得自己扛。  
  
再结合昨天那条 4GB 本地模型，现在桌面 AI 小助手大概有三条路：大厂云端、开源云端可自建、完全本地离线。  
  
做桌面 AI 小助手，你更想用大厂 SDK，还是全自己搭？  
  
Hackster 报道：[hackster.io/news/meta-brin…](https://hackster.io/news/meta-brings-its-muse-ai-agent-to-esp32-and-raspberry-pi-f6694ff1a306)  
Open Source For You 报道：[opensourceforu.com/2026/10/meta-o…](https://opensourceforu.com/2026/10/meta-opens-muse-gadgets-for-diy-hardware/)  
小智 ESP32 项目：[github.com/78/xiaozhi-esp…](https://github.com/78/xiaozhi-esp32)  
  
#ESP32 #开源硬件 #AIAgent #树莓派

> ![Link card preview](<一条忧郁的工科蛆%20(@FrozenEel427).assets/image-01.jpg>)
> 
> 🔗 [**Meta Brings Its Muse AI Agent to ESP32 and Raspberry Pi**](https://t.co/B3ikZsQ6N4)
> 
> _From hackster.io_