---
author: "ReignSwords"
handle: "@ReignSwords"
source: "https://x.com/ReignSwords/status/2114652755046564221"
date: 2026-10-26T09:37:59.000Z
type: thread
likes: 32
reposts: 3
replies: 19
bookmarks: 40
views: 11398
---
# ReignSwords (@ReignSwords)

今天 GitHub Trending 第 1 的 Cindy，我刚把它 490MB 的仓库 clone 下来翻了一遍。  
  
它牛在哪，我数了 3 点：  
  
1. 一个任务能拆给不同 harness × 模型组合并行跑。Claude Code 和 Codex 混着用，中途换模型，workspace、记忆、skill 不断线。这才是 agent 该有的样子，不是换壳聊天框。  
  
2. 能直接用我已经付过钱的 Claude Code / Codex 套餐，不用再交一份钱。也能接自己的 API key，或者跑本地模型。这点很实在。  
  
3. 代码里藏的比 README 写的还多：pi 的 manager 已经在了，微信接入、Slack hook、浏览器控制、手机联动全在包里。34 个包，13,007 个 TS 文件，今天 17:02 还在合 PR #7229，是活的。  
  
冷水也得泼两盆：  
  
开源的只是客户端，后端服务不在这个仓库，README 自己写了。喊“完全开源”之前先看清楚。  
  
官方发行版带 TapDB 统计。README 说在 index.tsx 删一行 initTapdb() 就能去掉，我翻代码发现调用早挪到 main-entry.tsx 第 223 行了，文档比代码慢半拍。自己编译能去掉，嫌麻烦就别装官方版。  
  
我站它，不是因为它 star 涨得快，是因为它把“不重复交钱”和“本地跑”这两件事做实了。搞开源/开发者工具的蓝V朋友欢迎互关。  
  
你们觉得这种“客户端开源、后端闭源”的 agent，算真开源还是半开源？👇  
  
🔵 蓝V互关 | Fellow Premium members, let's connect! 欢迎互关，一起交流 AI 与科技 👥

---

Source:

> ![Link card preview](<ReignSwords%20(@ReignSwords).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - makecindy/cindy: Consider it done. The open-source AI agent that works out of the box ·...**](https://t.co/kEkv3S1tUg)
> 
> _From github.com_

---

补充个硬核细节：我 clone 下来数过，17,088 个文件、34 个包、13,007 个 TS 文件，最新提交就是今天 17:02 合的 PR #7229。还有，README 说在 index.tsx 删 initTapdb() 就能去统计——实际调用在 main-entry.tsx 第 223 行，文档比代码慢半拍。

---

490MB，clone 的时候我都愣了一下