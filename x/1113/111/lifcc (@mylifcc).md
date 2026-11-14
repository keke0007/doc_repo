---
author: "lifcc"
handle: "@mylifcc"
source: "https://x.com/mylifcc/status/2119447519155589511"
date: 2026-11-08T15:10:40.000Z
type: thread
likes: 87
reposts: 5
replies: 6
bookmarks: 72
views: 7768
---
# lifcc (@mylifcc)

为什么很多人觉得“手机跑本地小模型”是个玩具？因为大家都错了方向——非要拿 2B 的端侧模型去跟云端满血大模型比写代码、写作文。。。。  
  
Reddit r/LocalLLaMA 今天冲上热榜的开源项目 Observer，直接展示了端侧小模型最正确的打开方式： 把你抽屉里吃灰的旧手机，变成 100% 本地运行、零 API 成本的 24 小时视觉监控 Agent（iOS / Android 通吃）。 👇

![Image](<lifcc%20(@mylifcc).assets/image-01.jpg>)

---

先算一笔云端 API 算不过来的账：  
假设你想让多模态大模型帮你盯一个画面（比如看家、盯 3D 打印、或者盯长耗时任务屏幕），每 5 秒抽一帧图，一天就是 17,280 张图片请求。 用云端大模型？钱包一天就空了，还得把私人摄像头或工作屏幕全传给第三方服务器🤡。 用本地 2B/3B 视觉小模型？Token 成本归零，断网也能跑，一天电费不到几分钱。

---

Observer 的工程设计非常克制，核心就是一个微Agent循环：Sensors -> Model -> Code：  
1️⃣ 输入层：接管手机摄像头、麦克风或桌面屏幕特定区域截图；  
2️⃣ 推理层：基于 Rust + llama.cpp / WebGPU，直接跑本地轻量级多模态模型（GGUF + mmproj），也可连局域网 Ollama；  
3️⃣ 动作层：不用写一行计算机视觉代码，直接用自然语言设条件——一旦命中，自动发 Telegram、WhatsApp、短信，或者直接给你打个电话。

---

几个实测极其好用的极客场景：  
• 物理外挂盯任务：跑模型微调、大工程编译、渲染或游戏排队，不想写复杂的 Hook 脚本，直接拿手机或桌面端框住进度条，跑完/报错立刻弹手机通知；  
• 零配置智能安防：旧手机架在窗台或门口，写一句“有人动我的车 / 快递放门口时发带图通知”，秒杀一堆收费云存储摄像头。  
  
端侧小模型真正的杀手级应用，不是做缩水版 ChatGPT，而是做不知疲倦的“高频传感器”。  
  
开源仓库与原帖地址：  
• GitHub 仓库（Rust + Tauri / 支持 iOS & Android & 桌面端）：[github.com/Roy3838/Observ…](https://github.com/Roy3838/Observer)• Reddit 演示原帖：[reddit.com/r/LocalLLaMA/c…](https://reddit.com/r/LocalLLaMA/comments/1xqd058/use_your_phone_to_monitor_anything_with_local/)

> ![Link card preview](<lifcc%20(@mylifcc).assets/image-02.jpg>)
> 
> 🔗 [**GitHub - Roy3838/Observer: The agent that monitors anything for you. Open-source, running complet...**](https://t.co/6qPOWmlNyY)
> 
> _From github.com_