---
author: "Xiao Tan"
handle: "@tvytlx"
source: "https://x.com/tvytlx/status/2119453240928670051"
date: 2026-11-08T15:33:24.000Z
type: tweet
likes: 9
reposts: 5
replies: 15
bookmarks: 4
views: 942
---
# Xiao Tan (@tvytlx)

最近在做一个小工具，里头需要一个本地语音转文字的模型，所以我正好从内存、参数量、中文转录质量等等大概进行了一个调研。  
  
我对比了 Whisper tiny 和 Ming-3.0-asr-tiny。Ming 的效果比我想象中好很多。  
  
我用的是社区提供的 MLX 4-bit 版本，在一台 M4 Pro Mac 上测的。整个推理过程禁止联网，录音都在本机完成处理。  
  
我发现 MLX 确实比 CoreML 启动速度更快。其实启动 Whisper 的时候它很慢的，剪辑的时候甚至剪掉了几秒。但是 MLX 的 Ming-3.0-asr-tiny 启动速度非常快。  
  
然后关于转录质量，除了视频里这个简单的音频外，我还测了一段复杂的音频。  
  
例如，一段包含桥梁净空高度和通车时间的录音，参考文本使用“15 米”“2011 年 8 月”等数字写法，Ming 输出了“十五米”“二零一一年八月”，还是很标准的，蚂蚁百灵的这个新一代的音频模型还是很不错的。  
  
Ming 官方还提供热词和对话上下文输入能力。所以当你需要处理产品名、专业术语或者特定业务录音的时候，这些接口也都很值得去调试一下。  
  
这些开源模型，你可以自己在本地跑，都是免费的，所以如果你转录量很大的话，真的可以让 Agent 给你搭一个。

![🎥 Video](<Xiao%20Tan%20(@tvytlx).assets/image-01.jpg>)