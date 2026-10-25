---
author: "龙海"
handle: "@longhaiqwe123"
source: "https://x.com/longhaiqwe123/status/2114031712430948410"
date: 2026-10-24T16:30:11.000Z
type: article
likes: 230
reposts: 42
replies: 44
bookmarks: 333
views: 20131
---
# 多 Agent 协作终极解：实测 YC 开源新作 Agent Relay，让 Claude Code 和 Codex 直接对话

*By 龙海 (@longhaiqwe123)*

![Banner](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-01.jpg>)

这周，美国创业孵化器 Y Combinator 2026 年秋季营（YC Fall 2026）的创业公司 Agent Workforce 发布了开源项目 [Agent Relay](https://agentrelay.com/)。公司的两位联合创始人分别是 CEO Will Washburn 和 CTO Khaliq Gant，其中 Khaliq Gant 毕业于康奈尔大学与哥伦比亚大学，此前曾任 YC W23 开源项目 Nango 的创始工程师和 YC W19 项目 Overview 的主程，之前一直做开发者工具。这个项目要解决一个具体问题：现在不少程序员写代码时会同时开几个 AI Agent（如 Claude Code、Codex、Cursor），但这些工具各自运行在独立的终端窗口里互不相通，人只能在中间把一边的输出手动复制粘贴给另一边。Agent Relay 的做法是在不替换现有工具的前提下，让不同 Agent 直接收发消息。

产品在 X 上发布后，创始人 [@willwashburn](https://x.com/willwashburn/status/2113834761110319374) 在帖子里介绍，它除了能连接本机的不同工具，还能跨电脑让不同团队成员的 Agent 直接同步进度。评论区里当然也有开发者提出实际顾虑，比如 [@garyfung](https://x.com/garyfung/status/2113913069177102665) 提到两个 AI 来回发消息会增加额度消耗，[@NaderKanounji](https://x.com/NaderKanounji/status/2113709325369618912) 和 [@astralwave](https://x.com/astralwave/status/2113753815040266610) 则关心省去人工复制粘贴之后，人在哪一步检查结果，以及怎么控制跨会话的权限风险。

我平时写代码正好遇到这个问题。我习惯同时开两个终端窗口，让 Claude Code 写代码，让 Codex 做检查。因为两边隔离，Codex 查出的问题需要我复制给 Claude Code，等改完再复制回去让 Codex 复查。

于是，我立马开始了实测。为了让没折腾过这类工具的新手也能一看就懂，我拿一个最简单的小例子从头试了一遍：让 Claude Code 写一段小程序，让 Codex 负责检查，中间的提问和修改都让它们自己发消息完成。最后顺利跑通了整个流程，Codex 审查代码两轮，Claude Code 按审视意见修改了代码。全程我只在开头布置了一次任务，接下来就是喝茶看戏。

我把每一步输入的命令和界面截图都放了进来，你只要跟着一步步复制粘贴，就能在自己电脑上把两个 AI Agent 连起来。

# 00｜它是怎么让两个 Agent 直接发消息的

平时使用时，Claude Code 和 Codex 各自开在独立的终端窗口里。接入 Agent Relay 后，原有的工具和操作方式不变，它只负责在两个窗口之间转发文本消息。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-02.jpg>)

它通过一个带时限的临时房间（Relay Connect）建立连接。先由其中一个 Agent 在本地发起创建请求，在浏览器里完成一次设备授权后，生成一段带连接地址的邀请文字。另一个 Agent 读取这段邀请文字，就能加入同一个临时房间。房间默认有效期为 60 分钟，用完可以手动关闭，到期也会自动失效。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-03.jpg>)

连接建好后，一个 Agent 发出消息，Agent Relay 会把消息直接发进另一个正在运行的 Agent 会话里，接收方处理完再通过同一条通道把结果发回来。原本需要人来回复制粘贴的步骤，变成了两个工具直接收发消息。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-04.jpg>)

# 01｜下载安装与连接步骤

这个工具目前支持 macOS 和 Linux 系统，暂不支持 Windows。开始前，电脑上需要准备好能正常使用的 Claude Code、Codex 和 Python 环境，并新建一个空文件夹作为测试目录。

整个准备和连接过程分为四步。创建房间的 Agent 需要注册账号并安装桌面客户端，加入房间的另一方不用注册。

**第一步，注册账号并下载安装桌面端。**

打开官网 [agentrelay.com](https://agentrelay.com) ，点击页面中间的「Connect your agents for free」免费注册，不用绑定信用卡。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-05.jpg>)

这里我直接用 Google 账号登录。登录成功后，网页会进入「Connect your computer」引导页，点击第一步里的「Download for Mac」下载安装包。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-06.jpg>)

双击打开下载好的安装包，把 Agent [Relay.app](https://Relay.app) 拖进 Applications 文件夹完成安装。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-07.jpg>)

接着从应用程序里打开 Agent Relay 桌面客户端，客户端启动后会完成初始化。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-08.jpg>)

**第二步，在 Claude Code 中创建连接房间。**

在新建的空文件夹里打开 Claude Code，把下面这段话原样发给它，让它读取官方连接说明并创建一个 60 分钟的临时房间，任务内容是编写和审查一个把文章标题转为网址路径的 Python 函数：

```text
Read https://agentrelay.com/connect and create a Relay Connect for a 60-minute session about writing and reviewing a Python slugify function. Give me the share_text to forward.
```

Claude Code 读取 [agentrelay.com/connect](https://agentrelay.com/connect) 的页面说明后，会先在界面中弹出一个设置询问：提示 ~/.claude/settings.json 里的 crossSessionInbound 当前为 accept（房间里别人的消息会自动作为新一轮对话进来），问要不要改成 hold（每条消息先等你批准）。第一次测试可以按需选择或保持 accept。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-09.jpg>)

接着它会发起设备连接请求，并在浏览器里打开 Authorize a device 页面，填入或核对设备码（Device code）后点击 Continue 完成设备授权。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-10.jpg>)

授权通过后，Claude Code 显示房间已经建好，并给出一段要转发给合作方的邀请文字（share_text）以及房间信息表（包含任务描述、60 分钟时长和 8 人上限）。需要注意，邀请文字和设备码带有连接权限，发截图前要打码，不要外传。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-11.jpg>)

**第三步，让 Codex 加入房间。**

打开 Codex，把刚才 Claude Code 生成的那段 Go to [agentrelay.com/connect](https://agentrelay.com/connect) and join ... 邀请文字原样粘贴给它。Codex 执行后会显示 Joined successfully as guest-648c97d630c4，代表已经以访客身份加入房间（如果 Codex 提示连不上网，可按第四节的命令开启网络权限）。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-12.jpg>)

**第四步，确认双方连接成功。**

Codex 加入后，切回 Claude Code 窗口，可以看到它已经收到一条来自 guest-648c97d630c4 的跨会话消息，显示合作方已加入房间并准备就绪，询问接下来如何开始。到这里，两个 Agent 之间的消息通道就建好了。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-13.jpg>)

# 02｜实测：Claude Code 写代码，Codex 负责审查

这次测试的任务是写一个把文章标题转成 URL 路径字符串的 slugify 函数和对应的测试文件。选这个任务是因为代码短，但在中文字符、连续标点、Unicode 组合字符等边界处理上容易出现不同做法，方便观察两个 Agent 能不能通过收发消息把问题改完。

连好之后，我先给 Claude Code 发了指令：

```text
在当前目录写 slugify.py 和 test_slugify.py。写完后通过 Relay 告诉 Codex 去审，Codex 有问题会直接问你，你直接回答并按它的意见改，不用经过我。
```

接着切到 Codex 窗口，给它发了审查要求：

```text
你是审查方。收到 Claude Code 的消息后读 slugify.py，有不清楚的地方通过 Relay 直接问它，审完把修改意见发回去。
```

发完这两段话，我就没有再操作。

Claude Code 写完初版代码和测试文件后，用 connect send 命令给 Codex（guest-648c97d630c4）发了一条约 9.6 KB 的消息，里面包含运行环境、预期行为以及两个文件的完整代码。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-14.jpg>)

Codex 收到完整代码和测试后，开始在本地跑用例检查边界情况，复现出了一个删除撇号后导致字母与组合符号重新相邻、再次调用结果改变的 Unicode 边界问题，随后把最小复现用例、三处规格澄清问题和 9 类补充测试建议通过 Relay 发回给 Claude Code。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-15.jpg>)

Claude Code 收到 Codex 发来的初轮审查意见后，直接修改了 [slugify.py](https://slugify.py) 修补规范化缺口、澄清了三处规格，并把对方建议的 9 类测试用例补进 test_slugify.py。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-16.jpg>)

针对第二轮沟通中 Codex 追问的 4 个细节（包括 separator 报错行为、硬截断返回空字符串等），Claude Code 逐条作了回复，并在 test_slugify.py 中补上第 40 个测试用例，在本地跑通 python3 -B -m unittest test_slugify 的全部 40 个测试后把结果发回给 Codex。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-17.jpg>)

Codex 收到修改版和完整测试后，跑完了 39 项完整测试、11 组独立回归和 20,000 组组合检查，确认原 Unicode 问题已修复且无阻塞问题，将最终通过结论通过 Relay 发回给 Claude Code，双方沟通完成。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-18.jpg>)

用完之后，只需要按 Claude Code 界面底部的提示，对它说一句“关闭房间”，它就会执行 connect end 关掉当前房间并删除本地的临时目录。

# 03｜谁适合现在用，以及对我的协同思路启发

实际用下来，因为两个 Agent 会在后台来回发消息，如果任务比较长，多轮对话会用掉更多模型额度，使用时需要留意自己的 Token 余量。如果你平时只用一个 AI 工具，或者还没把 Claude Code、Codex 用熟，现在不用装这个工具。但如果你经常同时开几个工具写代码，或者要和同事各自用 AI 核对同一个项目的进度，它能省掉来回复制粘贴的操作。

跑通这次测试后，我顺着这个方式想到了接下来可以试的一条产品开发流程：先让 Grok 做前期调研，把结果传给 Claude Code 出架构设计并写前端界面，同时让 Codex 写后端代码并负责代码审查；等代码定稿后，再让 Claude Code 生成产品宣传视频，最后交给 Antigravity 写产品介绍文章。把不同工具串在同一条消息通道上，发挥各家模型的长处，各自处理更适合模型能力的环节，同时把我订阅的所有模型都用起来。

![Image](<多%20Agent%20协作终极解：实测%20YC%20开源新作%20Agent%20Relay，让%20Claude%20Code%20和%20Codex%20直接对话.assets/image-19.jpg>)

# 04｜进阶配置与补充说明

如果在连接时遇到权限、网络报错，或者想看具体的参数限制，可以参考下面这些来自官方文档的说明。

**消息人工审批与 Codex 联网设置。** 默认情况下，对方发来的消息会直接发进 Claude Code 触发下一轮执行（即 accept 模式）。如果想让每条跨会话消息都先由人工确认后再放行，可以在 ~/.claude/settings.json 里加上 "crossSessionInbound": "hold"，Codex 不用配这一项。另外，如果 Codex 提示连不上网，退出后用下面这条命令重新启动即可：

```bash
codex -c 'sandbox_workspace_write.network_access=true'
```

**会话限制与常见报错码。** 每个临时会话默认时长为 60 分钟，创建时可以改。单个会话最多可接入 8 个 Agent，单条消息最长 16000 个字符，在有效期内拿到邀请文字就能加入。如果连接时报错，connect_expired 或 connect_ended 表示会话已过期或已关闭，重新创建一个即可；connect_already_joined 表示当前 Agent 已经在另一个会话里，需要先退出原来的会话；connect_full 表示会话人数已满。

**密钥权限与命令行工具版本。** 工作区密钥以 rk_live_ 开头，权限最高，不要放进公开链接、聊天记录或截图中。如果只想让别人看过程而不操作，可以用 ot_live_ 开头的只读观察者链接，这种链接会过期，也可以随时撤销。除了让 Agent 自动安装，也可以用 npm install -g agent-relay 手动安装命令行工具。需要注意，11.3.0 及更早的版本在运行 node up 和 node status 时会在终端打印工作区密钥，建议运行 agent-relay --version 查看版本并升到 11.3.1 以上。

**开源仓库与更多功能。** 这篇只测了实时发消息的功能，官方文档里还有团队共享会话历史、多步流程以及接入 GitHub 和 Slack 等功能，代码已在 [GitHub 开源](https://github.com/agentworkforce/relay)，采用 Apache-2.0 协议。

---

整个实测过程，我感觉非常顺畅，产品设计也比较优雅。但也引发了我另外 2 个问题的思考：

1. Agent Relay 内部实现机制是怎么样的？
2. 相比其他类似方案，Agent Relay有什么优势？

也许后续我会继续深入研究，并继续分享给大家。

---

龙海｜前大厂程序员 - 项目经理 - 产品经理，目前 OPC 实践（失业）中｜立志成为 X 上最会找痛点的男人｜做了几款小产品｜记得关注我 [x.com/longhaiqwe123](https://x.com/longhaiqwe123) ｜ 联系方式见主页