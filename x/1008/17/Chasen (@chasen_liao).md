---
author: "Chasen"
handle: "@chasen_liao"
source: "https://x.com/chasen_liao/status/2106071767454531597"
date: 2026-10-02T17:20:12.000Z
type: tweet
likes: 44
reposts: 4
replies: 8
bookmarks: 35
views: 4451
---
# Chasen (@chasen_liao)

Pi 1.0 终于来了！我最想分享的是 codemode，还有通过它直接调用图片模型的能力  
  
看完文档，感觉 codemode 简直是「上下文的克星」：LLM 可以用比较少的上下文发现工具、了解用法，再通过脚本完成调用  
  
工具越来越多的时候，这个设计就很有意思了  
  
如果把所有工具的描述、参数和返回结构都提前塞给模型，任务还没开始，工具说明已经占掉一块上下文，接进来的 MCP 越多，这部分开销也可能越大  
  
Pi 的 codemode 把这件事拆开了  
  
以默认通过 codemode 暴露的 MCP 工具为例，模型先看到 server 的简短介绍，需要具体能力时，再搜索相关工具，按需读取工具说明和参数类型，不用开场就把整套工具文档读完  
  
拿到用法后，模型写一段 JavaScript，把需要的工具调用组织起来，可以并行查资料，也可以把上一步的结果交给下一步处理  
  
这里还有一个很关键的地方：工具返回的大量数据，可以先留在脚本里处理，筛选、统计、裁剪之后，再把需要的结果交给模型  
  
比如查到一大批记录，任务只需要其中符合条件的几条，就可以先筛出来，再返回这些记录和必要字段，省去让 LLM 阅读整份结果的开销  
  
这样省下来的上下文来自两头：调用前按需加载工具信息，调用后控制进入对话的结果量  
  
发现工具、理解接口、执行调用、处理结果，都能围绕当前任务展开  
  
不过这个节省效果也取决于配置和脚本写法，直接暴露给模型的工具仍会占用工具声明，把完整结果全部打印回来也照样占上下文  
  
Pi 1.0 又进一步缩短了 codemode 的说明、工具描述和 system prompt 中的相关内容  
  
官方给的例子是，默认工具加上 codemode 的 GPT-5.6 请求，prompt 从约 5,300 tokens 降到 3,300，减少约 38%，这是特定配置下的 prompt 开销变化，完整任务能省多少还得看实际调用过程  
  
报错也更适合模型继续处理了，工具名写错会给出相近名称，参数不符合要求会提示预期结构，选了不存在的模型会引导查询可用模型，减少出错后盲猜接口的情况  
  
另一个新增能力是，codemode 现在能直接调用图片模型  
  
负责当前任务的聊天模型，可以在脚本里查询可用的图片模型，调用生成接口，再把图片返回到当前对话里，不需要把聊天模型切换成图片模型  
  
它使用 session 已配置的 provider 凭据，1.0 文档列出的例子是 OpenRouter 的图片模型，生成用量也会计入 session 成本  
  
输入既可以是文字，也可以带图片，用来编辑或作为参考，具体支持什么取决于所选图片模型，生成结果不会自动保存到磁盘，需要留文件还要另行保存  
  
对内容工作流来说，这就多了一条可以编排的路径：查资料、提取关键信息、整理配图要求，再调用图片模型，工具和图片生成可以衔接在同一个任务里  
  
这条工作流是基于接口能力的举例，我还没有做完整实测  
  
codemode 和内置 MCP 在 0.99 就已经加入了，1.0 这次是在继续降低说明开销、改善错误恢复，并新增图片生成调用  
  
我觉得最有意思的就是这种按需发现、按需调用、按需返回的思路，工具可以越来越多，模型每一步只读当前需要的那部分信息  
  
传送门 👉  
  
Pi 1.0 发布说明  
[github.com/earendil-works…](https://github.com/earendil-works/pi/releases/tag/v1.0.0)  
  
codemode 文档  
[github.com/earendil-works…](https://github.com/earendil-works/pi/blob/v1.0.0/packages/coding-agent/docs/codemode.md)  
  
MCP 工具发现与暴露方式  
[github.com/earendil-works…](https://github.com/earendil-works/pi/blob/v1.0.0/packages/coding-agent/docs/mcp.md)  
  
图片模型文档  
[github.com/earendil-works…](https://github.com/earendil-works/pi/blob/v1.0.0/packages/coding-agent/docs/models.md)

> ![Link card preview](<Chasen%20(@chasen_liao).assets/image-01.jpg>)
> 
> 🔗 [**Release v1.0.0 · earendil-works/pi**](https://t.co/9cvUCLq5fI)
> 
> _From github.com_