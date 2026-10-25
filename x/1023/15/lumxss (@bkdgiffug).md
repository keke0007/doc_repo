---
author: "lumxss"
handle: "@bkdgiffug"
source: "https://x.com/bkdgiffug/status/2113752576097145116"
date: 2026-10-23T22:01:00.000Z
type: tweet
likes: 8
reposts: 1
replies: 6
bookmarks: 5
views: 1012
---
# lumxss (@bkdgiffug)

想搭本地实时语音 Agent？Hugging Face 开源了这套流水线。  
  
speech-to-speech 把语音检测、识别、LLM 对话和语音合成串起来，各模块都能替换，接口设计接近 OpenAI Realtime。  
  
🎙️ 默认识别可用 Parakeet，合成可用 Qwen3-TTS  
🧠 中间的 LLM 支持本地服务或兼容接口  
🔧 Whisper、ChatTTS 等也可作为可选后端  
🔒 可全本地运行，也能只将转写文本交给云端模型  
全本地部署对内存和显存要求不低，实际延迟与效果取决于硬件、量化和部署方式，并非装好就能商用实时通话。克隆音色前也要确认授权。  
项目采用 Apache-2.0 协议，适合自建语音 Agent、研究和模型替换实验。  
🔗 [github.com/huggingface/sp…](https://github.com/huggingface/speech-to-speech)

![Image](<lumxss%20(@bkdgiffug).assets/image-01.png>)