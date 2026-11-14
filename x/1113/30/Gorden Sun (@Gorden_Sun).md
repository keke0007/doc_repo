---
author: "Gorden Sun"
handle: "@Gorden_Sun"
source: "https://x.com/Gorden_Sun/status/2120702460927045684"
date: 2026-11-12T02:17:22.000Z
type: tweet
likes: 32
reposts: 5
replies: 39
bookmarks: 14
views: 3129
---
# Gorden Sun (@Gorden_Sun)

Edge0：只用2.5GB内存，在电脑和手机本地运行35B MoE模型  
  
Edge0-AI团队开源edge0-infer推理引擎，平时把模型文件存放在固态硬盘里，通过预先训练好的路由模块提前预测下一步要调用的模块，在计算当前内容的同时后台提前把所需数据从硬盘读进内存。  
  
这个方法太巧妙了，因为35B的MoE模型实际激活只有3B，所以提前预测要激活的3B，把对应的模块放到内存即可。  
  
16GB内存的Mac电脑、iPhone17 Pro Max手机、安卓手机以及普通Windows电脑都能流畅运行35B的MoE模型。运行时设备内存占用稳定在2.5GB到3.4GB左右，在苹果M5芯片上每秒能生成约25个token，在手机上每秒也能输出11到13个token。  
  
GitHub：[github.com/Edge0-AI/Edge0](https://github.com/Edge0-AI/Edge0)

![Image](<Gorden%20Sun%20(@Gorden_Sun).assets/image-01.jpg>)