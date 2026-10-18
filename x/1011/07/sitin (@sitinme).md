---
author: "sitin"
handle: "@sitinme"
source: "https://x.com/sitinme/status/2107649428874285100"
date: 2026-10-07T01:49:15.000Z
type: thread
---
# sitin (@sitinme)

给 Claude Code 装过记忆插件的人，可能碰到过这种事。  
  
上个月项目从 pip 换成了 uv，在会话里交代过一次，插件也确实记下了，这周让它补个依赖，它张口还是 pip install。  
  
去记忆库里一翻，项目用 pip、项目改用

![Image](<sitin%20(@sitinme).assets/image-01.jpg>)

> **sitin (@sitinme)**
> 
> 让 Agent 真的“记住并学会”  
>   
> Hindsight 是 https://Vectorize.io 开源的 Agent 长期记忆系统，它更想解决的问题是：怎么让一个 Agent 真正拥有长期记忆，而且这些记忆还能随着时间不断更新。  
>   
> 把整个记忆过程拆成了几个部分：  
> retain 负责记住信息，  
> recall 负责把相关证据找回来，  
> reflect
> 
> ![Image](<sitin%20(@sitinme).assets/image-02.jpg>)
> ![Image](<sitin%20(@sitinme).assets/image-03.jpg>)
> ![Image](<sitin%20(@sitinme).assets/image-04.jpg>)

---

三个接口：存、查、想  
Hindsight 的接口很少，核心就三个动词。先起一个服务，官方推荐 Docker：  
  
export OPENAI_API_KEY=sk-xxx  
docker run -it --pull always --name hindsight --restart unless-stopped -p 8888:8888 -p 9999:9999 \  
  -e HINDSIGHT_API_LLM_API_KEY=$OPENAI_API_KEY \  
  -v hindsight-data:/home/hindsight/.pg0 \  
  [ghcr.io/vectorize-io/h…](http://ghcr.io/vectorize-io/hindsight:latest)  
  
8888 是 API，9999 是管理界面。模型不绑 OpenAI，用 HINDSIGHT_API_LLM_PROVIDER 能切到 Anthropic、Gemini、DeepSeek，本地的 Ollama、LM Studio 也行，还能直接借 Claude Pro/Max、ChatGPT Plus/Pro 的订阅，不用另配 API key。  
不想单独起服务，pip install hindsight-all 之后在 Python 进程里内嵌一个也行，Intel 芯片的 Mac 要换成 hindsight-all-slim。  
  
客户端这样接：  
  
from hindsight_client import Hindsight  
  
client = Hindsight(base_url="http://localhost:8888")  
  
# 存  
client.retain(bank_id="my-repo", content="项目依赖管理从 pip 切到了 uv",  
              timestamp="2026-09-10T10:00:00Z")  
  
# 查  
client.recall(bank_id="my-repo", query="这个项目用什么装依赖？")  
  
# 想  
client.reflect(bank_id="my-repo", query="给这个项目加依赖要注意什么？")  
  
bank_id 是记忆库，一个用户、一个 Agent 或者一个项目各用一个，库和库之间严格隔离。  
  
retain 存的时候会调一次大模型，把内容拆成事实、时间、实体和关系，再分进两条通道：world 是关于外部世界的事实，experience 是 Agent 自己做过的事，怎么分看是谁在说话。用户说我买了特斯拉，记成关于用户的 world 事实；Agent 说我修了鉴权 bug，才算 experience。  
  
recall 是四路并行检索：向量语义、BM25 关键词、实体图谱、时间范围。四路结果用 RRF 融合，再过一遍 cross-encoder 重排，最后按 token 预算截断，你问去年春天做了什么，它把去年春天解析成 3 到 5 月去过滤，而不是拿春天这个词硬匹配。  
  
reflect 不是检索，是一个小 Agent。它自己决定调哪些工具找证据，最多跑 10 轮，答案里只准引用真正查到过的记忆 ID。  
  
recall 给原料，reflect 给结论，想自己掌控推理用前者，想让它直接拿主意用后者。

![Image](<sitin%20(@sitinme).assets/image-05.jpg>)

---

多出来的那一层：看法  
  
开头 pip 和 uv 的问题，Hindsight 靠的是 retain 之后在后台自动跑的一步，叫 consolidation（合并），产物叫 observation。我更愿意叫它看法。

![Image](<sitin%20(@sitinme).assets/image-06.jpg>)

---

再往上一层：提前写好的答案  
  
看法之上还有一层 mental model（心智模型）。  
  
用法是你先定一个问题，比如这个用户有哪些偏好，Hindsight 把答案写好存起来，库里有新东西就在后台重写。读它就是一次数据库查询，不检索，也不调模型。  
  
reflect

---

装进 Claude Code：先读 300 条 commit  
  
这部分跟写代码的人关系最大。  
  
Hindsight 给编程 Agent 单独做了一个包，一条命令装：  
  
npx @vectorize-io/hindsight-coding-agents install claude-code   # 只装 Claude Code  
npx @vectorize-io/hindsight-coding-agents install all           #

![Image](<sitin%20(@sitinme).assets/image-07.jpg>)

---

写在最后  
  
Hindsight 跟不少记忆层不一样的地方，在存进去之后那一步。retain 把对话拆成事实，recall 分四路并行去找，reflect 自己翻证据给结论；  
  
后台还会把零散事实合并成会随新证据改口的看法，上面再叠一层提前写好的答案，项目从 pip 换成 uv 这种事，它分得清先后。写代码的人装上 coding