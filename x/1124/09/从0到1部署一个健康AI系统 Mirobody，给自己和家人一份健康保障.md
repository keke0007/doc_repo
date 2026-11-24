---
author: "想风"
handle: "@xaiwind"
source: "https://x.com/xaiwind/status/2124724609358631007"
date: 2026-11-23T04:39:57.000Z
type: article
likes: 82
reposts: 15
replies: 34
bookmarks: 119
views: 10909
---
# 从0到1部署一个健康AI系统 Mirobody，给自己和家人一份健康保障

*By 想风 (@xaiwind)*

![Banner](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-01.jpg>)

大家好，我是想风！

今天来看 Mirobody，一个开源的健康 AI 系统，新手也能学会！

项目地址：[github.com/thetahealth/mirobody](https://github.com/thetahealth/mirobody)

体检报告年年做，做完往抽屉里一塞。

三家医院、五年的单子，同一项指标写法都不一样，单位也不一样，想看个趋势都难。

Mirobody 干的就是这件事——把你手里零散的健康数据收进一份记录，统一好名称和单位，再交给 AI 来回答。整套系统可以跑在你自己的电脑上，数据不出门。

**目录**

1. Mirobody 是什么
2. 为什么值得花时间：最难的不是 AI，是数据
3. 它和 ChatGPT Health、阿福有什么不同
4. 三种上手方式，挑一个
5. 一条命令，先体验标准化
6. Docker 部署：100% 跑在自己电脑上
7. 上传化验单，看一条趋势
8. 给家人也建一份
9. 接入 Claude Code（可选）
10. 常见坑
11. 参与开源：Star、Issue、PR
12. 最后

---

# 一、Mirobody 是什么

简单说，Mirobody 是 Theta Health 开源的健康数据引擎，协议是 Apache-2.0，可以免费用、随便改。

它的口号是「Mirror your body in data」——用数据给身体照一面镜子。

几个名字先理清，免得搞混：

- **Mirobody**：开源项目本身，代码在 GitHub
- **国内网页版** [chat.mirobody.cn](https://chat.mirobody.cn/demo) 和微信小程序「Mirobody」：都基于这个开源引擎
- **Theta**：海外版应用，[thetahealth.ai](https://www.thetahealth.ai/)，有网页版和 App

整套系统分三步：**收集 · 标准化 · 智能体**。

**收集**：体检报告（PDF、手机照片、表格等 23 种文件）、Apple Health、Garmin、Oura、Whoop、基因文件、一句话健康日记，汇进同一份记录。

**标准化**：同一项检查的不同写法，认成同一项。LDL-C、低密度脂蛋白胆固醇、LDL Cholesterol，是同一个东西；mg/dL 和 mmol/L，统一换算。

**智能体**：AI 不凭记忆瞎答，而是调工具去查这张整理好的表，每个数字都标明出自哪份文件。

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-02.jpg>)

这里有个关键点——**编码和单位不是模型生成的**。

名称对的是国际通用的 LOINC 编码，单位用的是 UCUM 标准，都来自公开的词表和规则。同样的输入，在任何一台机器上，结果都一样。

最新 README（1.6.2 版）里的几个数：

- 常规体检会打印的项目，**347/347** 全部覆盖，含中、英、日、俄、爱沙尼亚、土耳其文
- 从 LOINC 2.83 切出 **63,416 个编码**，指向它们的名称共 **1,023,273 条**
- 每个数字仓库里都附了测试脚本，可以自己跑一遍复现

Mirobody 不做诊断、不开处方，回答不构成医疗建议。这句 README 里写得很清楚。

---

# 二、为什么值得花时间：最难的不是 AI，是数据

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-03.jpg>)

陈天桥今年 8 月在 LinkedIn 介绍 Mirobody，第一句话是：

"For years, the hardest problem in health tech wasn't AI — it was data."

多年来，健康科技最难的问题不是 AI，而是数据。

原帖：[LinkedIn](https://www.linkedin.com/posts/tianqiao-chen_mirobody-mirror-your-body-in-data-activity-7492019055428739072-esIG)

几个全国性调查的数字：

- 高血压患者，知道自己患病的只有 **46.9%**（Circulation 2018）
- 糖尿病 **36.7%**（JAMA 2021）
- 慢性肾病 **10.0%**（JAMA Internal Medicine 2023）

药是现成的，体检年年都量。

缺的是有人把数据连起来看。

两个公开报道的故事，都不是 Mirobody 的案例，但说明了同一件事。

**Alex 的故事**：4 岁起慢性疼痛，三年看了 17 位医生。母亲把症状和 MRI 记录一行一行输进 ChatGPT，得到一个此前没人提过的方向「脊髓栓系」，神经外科医生看过影像后确认，做了手术。

数据够清晰、够完整，AI 的帮助可以大得惊人。

同一篇报道里医生也提醒：ChatGPT 会编造内容，确认和手术的是医生。来源：[Today](https://www.today.com/health/mom-chatgpt-diagnosis-pain-rcna101843)

**Sid Sijbrandij 的故事**：GitLab 联合创始人，2022 年底查出骨肉瘤，标准治疗后复发，也没有合适的临床试验。他把能做的检测都做了，数据超过 30 TB，全部公开；测序找到一个靶点，对应的放射疗法早就存在。治疗后一年多没有疾病证据，他本人不说自己被治愈。

他也用 ChatGPT 整理和解读自己的数据，和医生团队一起找方向。

每个人都是自己健康数据的第一责任人。

他也说这条路非常昂贵，照现在的样子没法推广——这正是 Mirobody 开源想解决的。来源：[本人撰文（The Cancer Letter，2026 年 8 月）](https://cancerletter.com/guest-editorial/20260814_2/)、[OpenAI Forum（2026 年 3 月）](https://forum.openai.com/public/events/virtual-from-terminal-to-turnaround-how-gitlabs-co-founder-leveraged-chatgpt-in-his-cancer-fight-k3m7gks1bt)

**标准化到底值多少分？**

官方做过一组对比，相关论文即将发表，数字以论文为准。同一批题、同一个模型，只把杂乱导出换成标准化后的表，三个公开的可穿戴数据集，共 1,146 题：

- 端侧 2B 小模型（MiniCPM5-2B，Mirobody 默认的本地模型）：PHIA 从 44% 到 84%，LifeSnaps 从 62% 到 82%，AI4FoodDB 从 79% 到 82%（最后一组不显著）
- 托管强模型（DeepSeek V4.1 Flash）：读杂乱导出本来就有 94%–99%，标准化后 98%–100%

**模型越小，标准化帮得越多。**

两点要如实交代：杂乱导出是按规则从数据集的真实数值模拟出来的，不是设备的原样文件；小模型读杂乱导出时，常常推演到输出上限还没给出答案，这也是差距的一部分。

---

# 三、它和 ChatGPT Health、阿福有什么不同

ChatGPT Health（OpenAI，今年 1 月）、Claude（今年 1 月接入健康记录）、Copilot Health（微软，今年 3 月），国内的蚂蚁阿福，和 Mirobody 要解决的是同一个需求：**让 AI 基于你自己的健康数据回答。**

不同在四个地方：

- **数据在哪**：它们在厂商的云端；Mirobody 在你自己的机器上
- **能接什么数据**：它们由厂商接入、分地区开放，接入医院病历目前主要在美国；Mirobody 读你手里的文件，23 种，不限地区
- **指标怎么处理**：它们没有公开；Mirobody 对到标准编码，查表回答，每个数字标明来源
- **能不能改**：它们闭源；Mirobody 开源，模型、提示词、工具都能换

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-04.jpg>)

这些应用体验做得很好，注册就能用。

两者不冲突——它们是给所有人用的健康助手，Mirobody 是给想自己掌握数据的人用的开源底座。

为什么要本地、要开源，三个理由：

**风险**：健康数据是一个人最私密的数据，不该变成针对你的广告，或者别人给你定价的依据。

**掌控**：自己的数据、自己的机器、自己的模型，你说了算。

**开源**：做得起，一台自己的电脑就能开始；看得见，规则、词表、测试全部公开；补得上，一种新写法就是词表里的一行。

---

# 四、三种上手方式，挑一个

从轻到重：

**① 什么都不装**：国内直接打开 [chat.mirobody.cn/demo](https://chat.mirobody.cn/demo) 看演示，或者微信搜「Mirobody」小程序；海外用 Theta 网页版或 App。注册账号就能用。

**② 一条命令**：只体验「标准化」这一步，不需要模型、不需要 Key，下一节讲。

**③ Docker 本地部署**：应用、数据、模型全在自己电脑上，第六节讲。

还有两条路，这篇不展开：GitHub Codespaces 在浏览器里跑完整应用（要自己的模型 Key，Codespaces 有额度和计费）；不用 Docker、直接装在 Debian/Ubuntu 上。README 里都有步骤。

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-05.jpg>)

---

# 五、一条命令，先体验标准化

先装 uv，一个 Python 工具管理器，[安装说明](https://docs.astral.sh/uv/getting-started/installation/)。

macOS / Linux：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh   # 安装 uv
```

装好后，把这条命令粘贴到终端：

```bash
uvx --python 3.12 mirobody resolve "LDL cholesterol" 血红蛋白 ヘモグロビン "Total Kolesterol" 血脂
# 一次丢进去 5 个写法：英文、中文、日文、土耳其文，和一个类别词
```

不用克隆仓库，不用 Docker，不用 Key。首次运行要联网下载依赖，之后离线就能用。

看结果：

- 血红蛋白 和日文 ヘモグロビン，得到同一个编码 718-7
- 血脂 **保持未解析**——它是一个类别，不是一项具体检查

认不出的就老老实实说认不出，不硬猜。

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-06.jpg>)

---

# 六、Docker 部署：100% 跑在自己电脑上

**准备：**

- 装好 Docker（带 Compose），Mac 用 Docker Desktop 就行
- 内存建议 16 GB，**给 Docker 分至少 8 GB**
- 不需要显卡
- Windows 要在 **WSL 2** 终端里执行，不能直接在 PowerShell 里跑

**两条命令：**

```bash
git clone --depth 1 https://github.com/thetahealth/mirobody.git && cd mirobody   # 拉代码
COMPOSE_PROFILES=local-cpu ./deploy.sh   # 用 CPU 跑本地模型，一键启动 docker需先启动
```

第一次启动要下载容器镜像和模型权重，时间看网速。

国内网络拉不到镜像时，[deploy.sh](https://deploy.sh) 会提示，并自动改用镜像站；想用自己的镜像站，在 .env 里改 DOCKER_MIRROR。

下载完成后，文档识别、标准化、回答，全在本机完成。

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-07.jpg>)

**登录：**

打开终端打印的链接，本地模型已经自动选好。

用示例账号登录：在「邮箱验证码」标签填 you@mirobody.ai，验证码 111111。

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-08.jpg>)

**先问一句：**

示例账号里预置了合成数据，直接问：

```plaintext
我的胆固醇有什么变化？
```

AI 会一步步显示它在查什么，最后画出趋势，点开数字下面的虚线，能看到读数和它出自哪个文件。

**注意：CPU 跑本地模型，一次回答可能要几分钟。**这不是卡死，是在算。

| 情况 | 容器内（纯 CPU） | 本地原生（Metal） |
| --- | --- | --- |
| 首问 | 102 秒 | 14 秒 |
| 每条回复 | 2～3 分钟 | 4～5 秒 |
| 生成速度 | 5.3 tok/s | 61～77 tok/s |

实测补充：我的电脑配置Mac pro 48G， 首次部署大概花了16分钟！ 容器部署无法使用GPU 卡段，转成本地原生速度才能接受！

**不想等？换云端模型。**

不加 COMPOSE_PROFILES=local-cpu，直接 ./deploy.sh，在设置页填一个模型 Key（OpenRouter、OpenAI、Gemini、Anthropic、DeepSeek、DashScope 都行）。

代价要讲清楚：**应用和记录还在你电脑上，但要处理的文档和提问会发给你选的模型服务商。**100% 本地时，没有数据离开本机。

---

# 七、上传化验单，看一条趋势

仓库里准备了几份**合成**化验单，在 [demo/upload](https://github.com/thetahealth/mirobody/tree/main/demo/upload)：

- you*lab*report*zh*2023-03.pdf：中文 PDF
- you*checkup*photo*zh*2024-04.jpg：中文手机照片
- you*lab*results*en*2025-03.xlsx：英文表格

三种写法、两种单位，故意弄得乱七八糟。

在「数据」页上传，等它识别完，再问一次胆固醇。

三份不同格式、不同语言的单子，会连成一条趋势。

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-09.jpg>)

**想用自己的体检报告？**

可以，但先想清楚隐私：

- 用 100% 本地模式，数据不出本机
- 用云端模型 Key，报告内容会发给模型服务商
- 截图发文章时，姓名、身份证号、医院名、手机号都要打码

**记健康日记：**

在对话框里写一句：

```plaintext
昨晚开始头疼，血压150/95，没发烧
```

它会拆成一条症状、两条血压读数，「没发烧」不会被记成发烧。

---

# 八、给家人也建一份

开源版自带「关爱圈」。

**加入关爱圈本身，不会共享任何健康数据。**

家人各自保管自己的档案，给谁看、给谁改，本人决定。

不会自己登录的长辈，可以先由你代管一份；等他自己接管了，再重新决定保留你哪种权限。

示例数据里就有：mom@mirobody.ai 以只读方式共享给了 you@mirobody.ai，登录后可以切到妈妈的档案问一次。仓库里也有给妈妈准备的合成单子 mom*physical*2026-06.jpg、mom*clinic*visit_2026-07.xlsx。

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-10.jpg>)

今天没存下来的数据，明天再强的 AI 也补不回来。

---

# 九、接入 Claude Code（可选）

MCP 是一种让 AI 助手调用外部工具的协议，相当于给 AI 装插件。

**只用标准化引擎，不需要启动应用：**

```bash
claude mcp add mirobody -- uvx --python 3.12 mirobody mcp   # 把引擎作为工具接进 Claude Code
```

之后你贴给 Claude 的指标名称、读数、单位，它都能在本机标准化，不需要 Key。

**查你自己的完整记录：**

在运行中的应用里打开「设置 → MCP 链接」（默认 10 天有效），按 [文档](https://docs.mirobody.ai/zh/tools/mcp-integration) 配进客户端。

官方支持 Claude Code、Codex、Cursor、Gemini CLI、Claude Desktop。

示例codex 接通

```plaintext
codex mcp add mirobody --url http://localhost:18060/mcp/<secret>
```

这是我的示例 实际要意你的为准 设置里 生成链接后复制

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-11.jpg>)

执行后需要网页授权，再用codex 来问

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-12.jpg>)

---

# 十、常见坑

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-13.jpg>)

**坑一：Windows 直接跑** [**deploy.sh**](https://deploy.sh) **跑不起来。**

要先装 WSL 2，在 WSL 终端里执行。

**坑二：Windows PowerShell 里加 MCP 报错 unknown option '--python'。**

PowerShell 把 claude 当成它自己的脚本，把 -- 吞掉了。改用 claude.cmd：

```powershell
claude.cmd mcp add mirobody -- uvx --python 3.12 mirobody mcp   # PowerShell 专用写法
```

CMD 和 Git Bash 用原命令就行。

**坑三：Docker 内存给少了。**

Docker Desktop 默认分的内存可能不够，到设置里调到 8 GB 以上。

**坑四：本地模式回答慢，以为卡死了。**

没显卡，一次回答几分钟是正常的。有 GPU 的看 [本地模型配置](https://github.com/thetahealth/mirobody/blob/main/docs/local-models.zh-CN.md)，Mac、NVIDIA、Windows 都有说明。

**坑五：血脂 解析不出来，以为是 bug。**

不是 bug。血脂是一类检查的统称，不是一项具体检查，引擎故意不猜。

**坑六：用 Codespaces 忘了关。**

Codespaces 按时长计费，体验完记得停掉。

---

# 十一、参与开源：Star、Issue、PR

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-14.jpg>)

项目地址：[github.com/thetahealth/mirobody](https://github.com/thetahealth/mirobody)

觉得有用，**先点一颗 Star**，让更多人看到。

最有价值的贡献，是找出解析器答错的术语：

```bash
uvx --python 3.12 mirobody resolve "你单子上的写法"   # 结果错了或为空，就是一个 issue
```

结果不对，[提交 issue](https://github.com/thetahealth/mirobody/issues/new?template=wrong-term.yml)；会写代码的，加一行词表、加一个测试用例，直接提 PR。

俄文、爱沙尼亚文、土耳其文的写法，就是这样由外部贡献者补进来的（[#88](https://github.com/thetahealth/mirobody/pull/88)、[#140](https://github.com/thetahealth/mirobody/pull/140)）。

中文体检单的写法千奇百怪，各家医院各有各的叫法——这块正需要中文用户来补。

---

# 最后

![Image](<从0到1部署一个健康AI系统%20Mirobody，给自己和家人一份健康保障.assets/image-15.jpg>)

Mirobody 本地考验实操能力 ，和你的机器、软件环境、CPU、GPU 配置有关。手动本地部署，可以边手动 边让agent 协助，比如我就是一直deepseek 资讯部署遇到的问题！首次启动问题，可能会遇到卡顿，资料无法读取等问题，就是没有配置好 让agent 帮你就好了！

Mirobody 做的事，就是把散落在抽屉、App、手环里的健康数据，变成一份 AI 能读懂、你能核对的记录。

从一次体检、一个手环、一句症状记录开始存。

素材和演示：

- 中文截图和动图：[docs/images](https://github.com/thetahealth/mirobody/blob/main/docs/images/README.zh-CN.md)
- 合成化验单：[demo/upload](https://github.com/thetahealth/mirobody/tree/main/demo/upload)
- 中文在线演示：[chat.mirobody.cn/demo](https://chat.mirobody.cn/demo)

本文不构成医疗建议，身体不适请就医。