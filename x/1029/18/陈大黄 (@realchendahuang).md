---
author: "陈大黄"
handle: "@realchendahuang"
source: "https://x.com/realchendahuang/status/2115127451701698568"
date: 2026-10-27T17:04:16.000Z
type: tweet
likes: 802
reposts: 122
replies: 61
bookmarks: 1312
views: 71582
---
# 陈大黄 (@realchendahuang)

最近研究 Ego Lite，顺便把市面上主流的 AI 浏览器自动化方案完整梳理了一遍，市面上大致形成了八条主流路线。  
  
①传统自动化派：Playwright、Puppeteer 和 Selenium。  
这是最成熟的一派。  
  
- 优势是控制精准、确定性高、可测试、可复现，特别适合固定业务流程和高频批量操作。  
- 短板是页面一旦改版，维护选择器的工程成本很高；遇到完全未知的页面或者动态流程，必须引入 AI 才能看懂页面。  
- 只要业务路径是确定的，传统脚本单次执行只有毫秒级，零 Token 消耗，永远比让大模型每走一步都思考一次更快、更便宜。  
  
②CDP 配合 MCP 或 CLI ：代表方案包括 Chrome DevTools MCP、Playwright MCP 与 CLI、agent-browser 以及 Browser Harness。  
  
- 核心逻辑是借力打力，把现成浏览器的调试与控制能力包装成 Agent 能调用的标准接口。  
- DevTools MCP 偏向网络、控制台和性能调试；  
- Playwright MCP 依靠无障碍树快照来操作页面；  
- agent-browser 采用 Rust CLI 提供轻量级控制；  
- Browser Harness 则专注控制本机真实、已登录的 Chrome。  
  
这条路线生态成熟、开箱即用，不需要自己维护底层浏览器内核。但 CDP 本身只是个底层通信协议，并不等于开箱即用的完整 Agent 浏览器产品，每一步操作若塞进过多数据，很容易把上下文撑爆。  
  
③浏览器插件：代表方案包括 OpenCLI、Kimi Browser Extension、ChatGPT Browser Extension、Claude in Chrome 以及 Nanobrowser。  
  
这条路线的设计非常聪明：既然用户在日常 Chrome 里早就登录好了各种网站，为什么非要在隔离沙箱里重新启动一个浏览器？直接通过插件配合本地 Bridge 服务，把现有浏览器的控制权借给 Agent，就能直接复用所有 Cookie 和认证会话。  
  
其中 OpenCLI 尤其值得留意。它不仅能让 Agent 点击网页，还能把高频网站封装成可重复调用的本地 CLI 命令，优先借助页面已有的网络响应提取结构化数据。这意味着 Agent 可以从每次重新看页面，升级为直接调用沉淀好的网站能力。  
  
这派的优势是门槛极低、复用登录态极方便；短板则是扩展权限要求高，受宿主浏览器的稳定性和兼容性制约，做高并发或者长期无人值守任务需要专门设计防故障机制。  
  
④原生 Agent 浏览器派。最典型的代表是 Ego Lite 和 BrowserOS neo。  
  
这派不再满足于给 Chrome 套插件，而是直接基于定制的 Chromium 内核构建专门面向 Agent 的运行时。  
  
比如 Ego Lite 核心做了几项深度优化：  
在浏览器内部直接内嵌 Node.js，让 Agent 能批量执行 JS 逻辑；  
提供浏览器原生的 Snapshot，大幅降低页面理解成本；  
开辟独立 Space，支持 Agent 和人类用户并行操作；  
复用真实登录态，遇到复杂验证码允许随时人机接管；  
通过 Skill 机制无缝对接不同的 Coding Agent。  
  
在开源策略上两者选了不同路径：  
Ego Lite 的 Harness 和 Skill 采用开源，但浏览器核心仍然闭源；而 BrowserOS neo 走了完全开源的定制构建路线。  
  
两者斗殴证明了未来的浏览器可能需要从产品底层原生适配智能体，而不是事后打补丁。  
  
⑤AI 自主操作派。  
代表包括 Browser Use、Stagehand、Skyvern 和 Jev Ultrafast。  
  
Browser Use 强调端到端的全自主探索，适应能力强，但在复杂长链路任务中，Token 开销大且结果存在波动；  
  
Stagehand 走的是混合路线：已知的固定步骤用代码写死，未知的动态节点交给 AI 决策，同时支持结构化数据提取；  
  
Skyvern 更偏企业级复杂业务流，通过视觉模型与工作流引擎驱动操作；  
  
Jev Ultrafast 则走向另一种极致：把动作空间和交互元素极度压缩，以此压低推理延迟和 Token 成本，不过目前覆盖的网页复杂度还相对有限。  
  
自主探索的最大优势是灵活，但在实际工程落地中，确定性脚本加上 AI 动态兜底的混合方案，往往可靠性更高。  
  
⑥云端浏览器派。代表方案有 Browserbase、Steel Browser 和 Cloudflare Browser Run。  
  
当任务从单机走向云端服务器批量运行，核心矛盾就不再是单纯点击页面，而是浏览器实例的并发调度、资源隔离、会话持久化、代理轮换、防封禁与故障恢复。  
  
Browserbase 提供全托管的云端基础设施；Steel Browser 支持自托管部署并提供管理 API；Cloudflare Browser Run 则让浏览器直接跑在其全球边缘网络上，兼容 CDP、Playwright、Puppeteer 和 Stagehand 等多种接入方式。  
  
这派的核心价值是替开发者兜住底层运维复杂度，代价是云端账单开销以及多租户安全隔离的复杂性。  
  
⑦专业数据采集派。代表是 Crawl4AI 和 Firecrawl。  
  
如果核心目标只是大批量扒取网页并做结构化清洗，没必要动用通用 Agent 去模拟鼠标点击。  
  
这类工具专门针对大规模抓取、动态渲染、噪声剔除以及 Markdown 和 JSON 转换做过深度优化，吞吐量和成本表现远优于通用浏览器方案。但遇到强交互或复杂登录拦截的网站，通用浏览器控制依然不可替代。  
  
⑧正在演进中的底层标准：WebMCP。  
这是 Google 和微软等正在推动的浏览器原生 Agent 接口标准。让网站自身通过标准化接口把可执行能力直接暴露给 Agent，免去模型在前端猜按钮位置、算坐标的黑盒困境。  
  
目前 WebMCP 仍处于早期试验阶段，距离全网普及还有很长的路要走，但它代表了让网站主动适配 Agent 的长期方向。  
  
综合这些方案，在实际工程中怎么选？  
  
- 如果是稳定的端到端测试与固定自动化任务，选 Playwright；  
- 如果需要接管本地已登录的日常 Chrome，选 OpenCLI、Browser Harness 或 Kimi 扩展；  
- 如果希望 Agent 和真人能在同一浏览器里并行协作，选 Ego Lite 或 BrowserOS neo；  
- 如果需要自由探索陌生站点，选 Browser Use；  
- 如果开发可维护的混合型自动化工具，选 Stagehand 配合 Playwright；  
- 如果是企业内部的复杂长链路业务，选 Skyvern；  
- 如果是服务器端高并发多实例任务，选 Browserbase、Steel 或 Cloudflare Browser Run；  
- 如果是海量公开网页抓取与知识库清洗，选 Crawl4AI 或 Firecrawl。