---
author: "Wei"
handle: "@wei_wang"
source: "https://x.com/wei_wang/status/2109119549886935291"
date: 2026-10-10T08:17:45.000Z
type: tweet
---
# Wei (@wei_wang)

我手边也有一台 M4 Mac mini，看到这个组合有点心动。  
  
16GB 的 M4 Mac mini 用 USB-C 外接一张 RTX 3090，跑 Qwen3.8-Flash-Next：  
  
1. 大部分活跃权重放在 3090 上，其余交给 Strata，用 12GB 的 Mac 内存和 256GB 的内置存储  
2. 2 bit 量化，200K KV cache  
3. decode 约 80 tok/s，整套 $2,800  
  
外接显卡走的是 tinygrad 的方案。2 bit 的质量要打个问号，但 $2,800 能跑 Flash-Next，门槛又低了一截。

> **0xSero (@0xSero)**
> 
> Qwen3.8-next-flash on a 16 GB M4 Mac Mini + RTX 3090!  
>   
> Tinygrad's chestnut to connect the GPU via usbc to the mac mini  
>   
> most active weights stays on 3090, the rest is using strata's framework with 12gb of mac ram and 256gb mac storage  
>   
> 2,800$  
> 200K kv cache  
> 80~ tok/s decode  
> 2 bit
> 
> ![Image](<Wei%20(@wei_wang).assets/image-01.png>)