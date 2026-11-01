---
author: "看山"
handle: "@distance9909"
source: "https://x.com/distance9909/status/2114357895248990443"
date: 2026-10-25T14:06:19.000Z
type: article
---
# 一句话让 AI 帮你测完整个 App：TesterArmy e2e 正在把自动化测试变成“自然语言”

*By 看山 (@distance9909)*

![Banner](<一句话让%20AI%20帮你测完整个%20App：TesterArmy%20e2e%20正在把自动化测试变成“自然语言”.assets/image-01.jpg>)

最近看到一个很有意思的开源项目：

**TesterArmy e2e。**

项目地址：

[GitHub - tester-army/e2e: Next generation e2e testing framework for web and mobile apps. · GitHub](https://github.com/tester-army/e2e?utm_source=chatgpt.com)

它做的事情很直接：

**你不用把每一步点击、输入、跳转都手写成脚本，而是直接用自然语言描述目标，让 AI Agent 自己操作网页或手机 App 完成测试。**

比如你告诉它：

登录测试账号，升级到 Pro 套餐，并确认账单页面显示了按比例计费金额。

传统自动化测试里，这通常要写一堆 selector、click、fill、wait。

而在 e2e 里，可以直接写成：

await agent.act('upgrade the workspace to the Pro plan');  
await agent.assert('the invoice preview shows a prorated amount');

然后再配合传统的精确断言：

await expect(screen.getByRole('status')).toContainText('Pro');

这就是它最核心的设计：

**AI 负责“理解和操作”，传统测试框架负责“精确验证”。**

## 为什么它比普通 AI 测试更有意思？

很多所谓 AI 测试工具的问题是：

AI 会点页面，

但测试结果不够稳定。

TesterArmy e2e 没有完全放弃传统自动化测试，而是把两套方法混在一起：

**Agent Steps + Deterministic Assertions**

也就是说：

模糊、复杂、难写 selector 的步骤，交给 AI。

必须精确验证的地方，继续用 locator 和 assertion。

这样既保留 AI 的灵活性，也保留传统测试的确定性。

## 通过一次，下一次甚至可以不再调用模型

这个设计我觉得非常聪明。

当一个 Agent 步骤成功执行，并且后面的断言证明它确实做对了以后，e2e 会记录这次操作。

下一次再跑同样的测试时，可以直接**重放之前的动作**。

这样做有两个好处：

第一：

**速度更快。**

第二：

**不用每次都消耗 Token。**

换句话说，AI 第一次负责“探索怎么做”。

之后如果界面没变，就直接按已经验证过的路径重放。

有点像：

**AI 先学会，自动化脚本再接管。**

## 一套测试同时跑 Web、iOS 和 Android

e2e 还有一个很实用的地方：

它把网页和移动端放进了同一套测试 API。

Web 端底层可以使用 Playwright。

移动端则可以覆盖：

**iOS Simulator**

和

**Android Emulator**

所以同一套测试思路，可以覆盖：

Chrome、Firefox、WebKit、iOS、Android。

而不需要完全维护三套不同的测试体系。

## 它还能直接交给 Codex、Claude Code 去写测试

TesterArmy e2e 的思路很适合接入 AI 编程 Agent。

比如：

**Codex、Claude Code、Cursor**

这些工具可以先读取你的项目，

再探索真实界面，

最后自动补测试。

整个流程就可能变成：

你让 AI 写功能，

AI 写完以后，

再让它自己打开 App，

跑一遍，

然后把回归测试也补上。

这已经非常接近未来的软件开发方式：

**写代码和测代码，不再是两个完全割裂的过程。**

## 安装也很简单

初始化可以直接：

npx e2e init

然后选择你要测试的是：

**Web**

还是：

**Mobile**

再配置模型。

整个思路就是把“自动化测试工程”尽量压缩成自然语言驱动的任务。

## 为什么这个项目值得关注？

过去的软件自动化测试，本质上是在教机器：

第一步点哪里，  
第二步输入什么，  
第三步等待什么。

而 Agent 测试正在变成：

**告诉 AI 最终要完成什么。**

这两种思路差别很大。

前者是：

“按照我规定的步骤执行。”

后者是：

“你自己想办法完成目标，然后我验证结果。”

这其实已经不是传统意义上的“脚本自动化”了。

而是在向：

**目标驱动测试**

转变。

如果这种方法继续成熟，未来 QA 工程师最重要的能力，可能不只是会不会写 selector。

而是：

**能不能把业务规则和验收条件描述清楚。**

一句话总结：

**以前自动化测试是在教机器怎么点。**