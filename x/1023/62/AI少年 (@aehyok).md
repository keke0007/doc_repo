---
author: "AI少年"
handle: "@aehyok"
source: "https://x.com/aehyok/status/2113083667417608344"
date: 2026-10-20T13:48:52.000Z
type: tweet
---
# AI少年 (@aehyok)

兄弟姐妹们，最近从零开始玩AI 硬件我算是进门了  
全程呼叫你的AI 什么不会就问它就完事了  
  
推荐一下我玩 ESP32 开发板的四个开源项目  
  
1. [github.com/78/xiaozhi-esp…](https://github.com/78/xiaozhi-esp32) ：ESP32 端的小智语音机器人固件⭐ 29.6k  
2. [github.com/xinnan-tech/xi…](https://github.com/xinnan-tech/xiaozhi-esp32-server) ：小智的自建后端服务⭐ 10.5k  
3. [github.com/Liu-zhongxian/…](https://github.com/Liu-zhongxian/zongzhu-bot) ：圆屏 ESP32 上的 Grok 语音终端⭐ 216  
4. [github.com/justhalfbit/xi…](https://github.com/justhalfbit/xiaozhi-hub-server) ：按名字把语音任务分派给不同 AI 的网关⭐ 6  
  
我手里买的就是一块板子 128元的嘉立创的开发板，原价是148元，新用户有20块的优惠卷。  
  
前两个开源项目  
  
1、一个是固件源码，也就是要把源码刷入到ESP32设备中的，使用window 或者mac 下的 workbuddy、codex、claude code、甚至 pi agent 都是没问题的  
  
2、第二个就是针对设备以及固件程序的后端配置源码，同时如果你刚上手可直接先对接 [xiaozhi.me](http://xiaozhi.me) 官网后端，不用自己进行部署，熟悉之后可以再部署深入  
  
前两个开源项目你要是稍微熟悉了，可以上手后面两个  
  
这两个开源项目都能接入自己的 Grok Bot,两个我都进行亲手实操了的  
  
下面的视频就是我使用第四个开源项目自己部署的，这是我一个朋友自己在玩的项目，还可以接入 Claude Code 真心推荐

> **半格 / HalfBit (@justhalfbit)**
> 
> 📝 **让 ESP32 小机器人指挥 Claude Code 和任何 AI｜一行命令接通、不刷固件、喊名字切换**
> 
> ESP32 设备买到手之后，看我这篇就够了。这是全网最简单的接 Claude Code 方法：一行命令装好一个网关，不刷固件，10 分钟，桌上这台 StackChan 就能指挥我本机的 Claude Code 干活。
> 对它说"嗨...