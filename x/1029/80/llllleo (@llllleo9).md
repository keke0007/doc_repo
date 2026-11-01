---
author: "llllleo"
handle: "@llllleo9"
source: "https://x.com/llllleo9/status/2114365358572548146"
date: 2026-10-25T14:35:59.000Z
type: tweet
---
# llllleo (@llllleo9)

分享一个用 magpie 小技巧，我就踩坑了：接入多个模型到同一路由组后，一次对话内容，如果会出现tmd左右脑互搏（图1），像是2个人在说话，AI自己判断错了两回。  
  
先别怀疑bug在路由组，去 magpie 的 Agent 页点开 在跑的agent框架（我是Claude Code），看「子代理模型」是否设置为【默认】  
  
新用户研究工具，这里点一下那里点一下很正常，改过 agent 的子 agent 配置，记得改回【默认】，重复第二遍  
  
不然换主模型后，子agent它不跟，还按旧槽位顶替过去。（主agent是Kimi，子agent还是之前的DeepSeek）

![Image](<llllleo%20(@llllleo9).assets/image-01.jpg>)
![Image](<llllleo%20(@llllleo9).assets/image-02.jpg>)