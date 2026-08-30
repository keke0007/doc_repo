# nanobot PyCharm 调试指南（execution flow 观察向）

> 目标：在 PyCharm 里跑起 nanobot，并通过打断点观察**一次完整对话闭环**的执行流程。
> 文中所有 `文件:行号` 均基于当前本地代码核对过，改动代码后行号会漂移，以函数名为准。

## 一、环境现状（已验证）

| 项 | 值 |
|---|---|
| 项目根 | `C:\Users\ke\Desktop\zcode\nanobot-main` |
| 虚拟环境 | `.venv`（**editable 安装**，见 `.venv/Lib/site-packages/_editable_impl_nanobot_ai.pth`） |
| PyCharm SDK 名 | `uv (nanobot-main)` |
| 配置文件 | `~/.nanobot/config.json` |
| 会话历史 | `~/.nanobot/sessions/<hash>/<base64 会话key>.jsonl` |
| 日志 | `~/.nanobot/logs/gateway.log` |
| 实际 provider | `OpenAICompatProvider` → `https://api.deepseek.com/v1`，模型 `deepseek-v4-flash` |
| 已启用 channel | 仅 `websocket` |
| gateway | `127.0.0.1:18790` |

editable 安装意味着**改源码立即生效**，不需要重新 `pip install`。

**一个容易误判的点**：`agents.defaults.model` 里写的是 `anthropic/claude-opus-4-5`，但
`modelPresets.primary`（`provider: custom` + `model: deepseek-v4-flash`）把它覆盖了。所以断点里
看到请求打向 deepseek 是正常的，不是 bug。

命令行冒烟验证（确认环境可用，会真实调用一次 LLM）：

```bash
cd C:/Users/ke/Desktop/zcode/nanobot-main
./.venv/Scripts/python.exe -m nanobot agent --classic -m "只回复两个字：收到" -s "cli:debug-smoke"
```

结束时可能出现 `httpcore2 ... GeneratorExit / generator didn't stop after athrow()` 的堆栈，
那是解释器退出阶段异步生成器关闭的噪音，不影响功能。

## 二、最关键的一个坑：别用默认的 `nanobot agent`

默认交互模式启动的是 **bun 写的原生 TUI**：`nanobot/cli/tui_launcher.py:129` 用
`subprocess.Popen` 拉起一个独立进程（源码在项目根的 `tui/`）。Python 断点进不去那个进程。

> 调试一律加 `--classic`（等价于 `--no-tui`）。

派生规则见 `nanobot/cli/agent.py:86`：`native_tui = message is None and not classic`，
即 **给了 `-m` 或加了 `--classic` 就走纯 Python 路径**。

## 三、运行配置

已写好三个共享运行配置，放在 `.idea/runConfigurations/`，重开项目后出现在右上角下拉框：

| 配置名 | 参数 | 用途 |
|---|---|---|
| `agent oneshot (debug)` | `agent --classic --logs --no-markdown -s cli:pycharm -m "你好，先介绍一下你自己"` | **首选**。单进程直调 `process_direct`，一条路走到底 |
| `agent interactive (classic)` | `agent --classic --logs -s cli:pycharm` | 多轮对话，走 MessageBus（和真实 channel 同一条链路） |
| `gateway (webui backend)` | `gateway --foreground --verbose --port 18790` | 配合 WebUI 调试真实 channel |

三个配置的共同要点：

- **Module name** = `nanobot`（不是 script path），等价于 `python -m nanobot`
- Working directory = 项目根
- 环境变量 `PYTHONUNBUFFERED=1`、`PYTHONIOENCODING=utf-8`（Windows 中文输出不乱码）
- `agent interactive` 勾了 **Emulate terminal in output console** —— 交互模式用
  `prompt_toolkit`（`nanobot/cli/terminal.py:187` 建 `PromptSession`），没有真 tty 会报错

手工新建的话：Run → Edit Configurations → `+` → Python → 把左上角 Script path 切成
**Module name**，填 `nanobot`，Parameters 填上表内容。

跑 gateway 前先停掉后台实例，否则端口冲突：

```bash
./.venv/Scripts/python.exe -m nanobot gateway stop
```

后台实例信息记录在 `~/.nanobot/run/gateway.json`（含 pid / port / 启动命令）。

## 四、主线：一次 turn 的七个阶段

理解这个项目的钥匙在 `nanobot/agent/loop.py:1680-1687`，一次对话被切成七段顺序执行：

```
restore → compact → command → build → run → save → respond
```

| stage | 实现 | 干什么 |
|---|---|---|
| restore | `loop.py:1749` `_restore_turn` | 恢复 checkpoint、处理附件 |
| compact | `loop.py:1805` `_compact_session` | 上下文超限时压缩历史 |
| command | `loop.py:1813` `_dispatch_command` | 斜杠命令拦截（命中就不走 LLM） |
| build | `loop.py:1862` `_build_turn` | **拼上下文**：system prompt、skills、历史、工具 schema |
| run | `loop.py:1978` `_run_turn` | 调 `AgentRunner`，多轮 LLM + 工具 |
| save | `loop.py:2016` `_persist_turn` | 落盘会话历史 |
| respond | `loop.py:2063` `_prepare_outbound` | 组装出站消息 |

**万能断点**：`nanobot/agent/loop.py:1697`（`_run_turn_stage` 里的 `result = await handler(ctx)`），
设成日志断点并打印 `name`，一次对话就能看到七个阶段按序输出。先用它建立全局观，再往下钻。

## 五、场景断点清单

### 场景 A：单条消息（oneshot）— 最小闭环，先跑这个

运行配置 `agent oneshot (debug)`，Shift+F9。

| # | 断点 | 观察点 |
|---|---|---|
| 1 | `nanobot/cli/agent.py:247` | 入口 `agent_loop.process_direct(...)` |
| 2 | `nanobot/agent/loop.py:2282` | 组 `InboundMessage`、拿 session 锁（与总线 turn 串行化） |
| 3 | `nanobot/agent/loop.py:1862` | `_build_turn`：上下文怎么拼出来的 |
| 4 | `nanobot/agent/loop.py:1158` | `runner.run(AgentRunSpec(...))`，看 `initial_messages` / `max_iterations` |
| 5 | `nanobot/agent/runner.py:508` | `for iteration in range(spec.max_iterations)` —— **闭环的心脏** |
| 6 | `nanobot/agent/runner.py:530` | `_request_model(...)` 发请求 |
| 7 | `nanobot/providers/openai_compat_provider.py:2079` | 真正的 HTTP（流式）；非流式在 `1975` |
| 8 | `nanobot/agent/loop.py:1717` | `_assemble_outbound` 组装 `OutboundMessage` |

### 场景 B：交互式 classic — 完整总线闭环

在 A 的基础上加下面几个，看清「投递总线 → 被消费 → 回到渲染器」整圈：

| 断点 | 观察点 |
|---|---|
| `nanobot/cli/agent.py:397` | `bus.publish_inbound(...)` |
| `nanobot/agent/loop.py:1244` | `AgentLoop.run` 消费端（1 秒轮询，**用日志断点**） |
| `nanobot/agent/loop.py:1408` | `_dispatch` 中 `await self._process_message(...)` |
| `nanobot/agent/turn_delivery.py:245` | `bus.publish_outbound(response)` 回程 |
| `nanobot/cli/agent.py:314` | CLI 消费出站消息并交给 renderer |

### 场景 C：WebUI / gateway — 真实 channel 闭环

这条路才是 Telegram / 飞书 / WebSocket 等真实渠道走的路。

| 断点 | 观察点 |
|---|---|
| `nanobot/channels/base.py:255` | `_handle_message`：所有 channel 的统一入口（鉴权、pairing 在此之前） |
| `nanobot/channels/base.py:321` | `bus.publish_inbound(msg)` |
| `nanobot/channels/manager.py:709` | 出站消费循环（1 秒轮询，用日志断点） |
| `nanobot/channels/manager.py:858` | `channel.send(msg)` 最终投递 |
| `nanobot/channels/websocket/runtime.py:922` | 推给浏览器 |

### 场景 D：带工具调用的闭环（最能体现"多轮"）

发一句会触发工具的话，例如「读一下工作区里的 HEARTBEAT.md」。

| 断点 | 技巧 / 观察点 |
|---|---|
| `nanobot/agent/runner.py:508` | **条件断点** `iteration > 0`：只在第二轮停，直接看到工具结果回灌后的 messages |
| `nanobot/agent/runner.py:553` | `response.should_execute_tools` 判定 |
| `nanobot/agent/runner.py:587` | `execute_tool_calls(...)` 批量执行 |
| `nanobot/agent/tools/execution.py:141` | `await tool.execute(**params)` 单个工具落地 |
| `nanobot/agent/tools/registry.py:196` | 注册表路径的 `tool.execute` |

这是理解 agent 本质的地方：LLM 返回 tool_calls → 本地执行 → 结果作为消息追加 →
**同一个 for 循环再发一次请求**，直到模型不再要求工具为止。

### 场景 E：流式输出闭环

| 断点 | 观察点 |
|---|---|
| `nanobot/agent/runner.py:995` | `if wants_streaming:` 分支选择 |
| `nanobot/agent/runner.py:1003` | `hook.on_stream(context, delta)` 每个 token（**必须日志断点**，否则停几百次） |
| `nanobot/agent/turn_delivery.py:294` | delta 包成 `StreamDeltaEvent` 发上总线 |
| `nanobot/agent/progress_hook.py:67` | hook 侧实现 |
| `nanobot/channels/manager.py:828` | `channel.send_delta(...)` 推给渠道 |

### 场景 F：记忆与压缩闭环

| 断点 | 观察点 |
|---|---|
| `nanobot/agent/loop.py:2016` | `_persist_turn`：本轮消息如何入库 |
| `nanobot/agent/loop.py:2054` | `self.sessions.save(session)` 落盘 |
| `nanobot/agent/loop.py:1805` | `_compact_session`：上下文超限时的自动压缩 |

落盘结果可以直接看：`~/.nanobot/sessions/<hash>/` 下的 `.jsonl`，文件名是会话 key 的
base64（例如 `cli:pycharm` → `Y2xpOnB5Y2hhcm0`），第一行是 metadata（含 token 用量）。

## 六、三个必须掌握的技巧

**1）日志断点优先于暂停断点。** 断点上右键 → 取消勾选 `Suspend` → 勾 `Evaluate and log`
填表达式。观察"完整流程"靠这个；靠单步会被 HTTP 超时和 1 秒轮询搅乱。常用表达式：

- 万能断点处：`name`
- `runner.py:508`：`f"iter={iteration} msgs={len(messages)}"`
- `runner.py:1003`：`delta`

**2）条件断点隔离自己的会话。** 两个消费循环会被无关消息不断触发。给断点加条件
`session_key == "cli:pycharm"`（运行配置里固定了这个 session id），噪音立刻消失。

**3）Evaluate Expression (Alt+F8) 看 messages。** 停在 `runner.py:530` 时求值：

```python
[(m.get('role'), str(m.get('content'))[:80]) for m in messages]
```

一眼看清发给模型的完整上下文——这是理解本框架最快的方式。

## 七、常见坑

| 现象 | 原因 / 处理 |
|---|---|
| 断点进不去、只看到进程启动 | 少了 `--classic`，跑到 bun TUI 子进程去了 |
| 停久了流式内容为空 / 请求报错 | 断点期间 HTTP 超时。看请求体就停在 `runner.py:530`，别在 provider 内部久留 |
| 交互模式启动即报错 | 没开 Emulate terminal，`prompt_toolkit` 拿不到 tty |
| gateway 启动失败 | 后台实例占用 18790，先 `nanobot gateway stop` |
| 断点疯狂触发 | 命中的是 1 秒轮询循环（`loop.py:1244` / `manager.py:709`），改条件断点或日志断点 |
| 退出时 httpcore2 报 GeneratorExit | 解释器退出噪音，可忽略 |

## 八、一次完整闭环的调用链（速查）

```
channel / CLI
  ├─ CLI oneshot:  cli/agent.py:247  process_direct
  ├─ CLI 交互:     cli/agent.py:397  publish_inbound ─┐
  └─ 真实 channel: channels/base.py:321 publish_inbound ─┤
                                                        ↓
                                    MessageBus ── loop.py:1244 AgentLoop.run
                                                        ↓
                                             loop.py:1371 _dispatch（按会话加锁）
                                                        ↓
                                            loop.py:1570 _process_message
                                                        ↓
              loop.py:1680-1687  restore → compact → command → build → run → save → respond
                                                                          │
                                                        loop.py:1158  runner.run(AgentRunSpec)
                                                                          ↓
                                    runner.py:508  for iteration ──┬─ 530 _request_model
                                                                   │      ↓ providers/openai_compat_provider.py:2079
                                                                   └─ 587 execute_tool_calls ──┐
                                                                          ↑                    │
                                                                          └── 结果回灌 messages ┘
                                                        ↓
                            loop.py:1717 _assemble_outbound → turn_delivery.py:245 publish_outbound
                                                        ↓
                       CLI: cli/agent.py:314 renderer   /   channel: manager.py:858 channel.send
```

**推荐学习顺序**：万能日志断点看七阶段 → 场景 A 走通最小闭环 → 场景 D 理解工具多轮 →
场景 B/C 看总线与真实渠道 → 场景 E/F 看流式与持久化。

