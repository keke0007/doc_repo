# MQTT 知识点梳理

> 源文档:`yuque/MQTT.md`
> 整理目标:把原文中庞杂(且夹杂不少 OCR/手写笔误)的 MQTT 知识点按"协议本身 → Broker 与会话特性 → 客户端实现 → 与 Web 集成"重新组织,跨文件/跨组件的交互一律配 ASCII 流程图,知识点末尾若与正确技术事实有出入,放【纠错】块。

---

## 1. MQTT 协议总览

### 1.1 定义与定位

- 全称:**MQTT**(MQ Telemetry Transport)。在 OASIS 标准化后官方说明 **MQTT 已不再展开为缩写**,仅作专有名词使用;早期文档常展开为 *Message Queuing Telemetry Transport*。
- 传输层:构建于 **TCP/IP** 之上的应用层协议(Web 场景下可再叠加 WebSocket)。
- 模式:**发布 / 订阅**(Publish / Subscribe)。
- 历史:IBM 于 1999 年发布,2014 年成为 OASIS 标准;主流版本 3.1 / 3.1.1 / 5.0。

### 1.2 设计原则(为 IoT 量身定做)

1. 精简,不添加可有可无的功能
2. Pub/Sub 模式,便于消息在大量设备间传递
3. 主题动态创建,零运维成本
4. 把传输量降到最低
5. 把低带宽、高延迟、不稳定网络作为常态
6. 支持连续会话控制
7. 假设客户端算力有限
8. 提供 QoS 服务质量管理
9. 不强求传输数据的类型与格式,保持灵活性

### 1.3 与 HTTP 对比(为什么 IoT 偏爱 MQTT)

| 维度 | HTTP | MQTT |
|---|---|---|
| 通信模型 | 请求/响应,客户端主动拉 | 发布/订阅,服务器可推 |
| 连接形态 | 短连接为主 | 长连接 |
| 头部开销 | 大(文本头) | 极小(2 字节固定头) |
| 服务端推送 | 不原生支持(需轮询/SSE/WS) | 原生支持 |
| 多设备分发 | 需逐个请求 | 一次发布,Broker 扇出 |

### 1.4 基本概念

- **Publisher**:发布者,发出消息。
- **Subscriber**:订阅者,接收并处理消息。
- **Broker**:消息代理,位于发布者与订阅者之间(mosquitto / EMQX / HiveMQ 等)。
- **Topic**:主题,路由依据,层级用 `/` 分隔,通配符:`+` 单层、`#` 多层。
- **Payload**:消息负载(消息体)。
- **QoS**:服务质量,0/1/2 三档(详见 §1.5)。
- 一个客户端既可作发布者也可作订阅者,可通过通配符一次订阅多个 topic。

### 1.5 QoS 三个等级

| 等级 | 英文名 | 中文 | 行为 |
|---|---|---|---|
| QoS 0 | **At most once** | 至多一次 | "fire and forget",可能丢失,不会重复 |
| QoS 1 | **At least once** | 至少一次 | 保证到达,可能重复 |
| QoS 2 | **Exactly once** | 只有一次 | 保证到达且不重复,握手最多 |

发布与订阅最终生效的 QoS = `min(pub_qos, sub_qos)`。

### 1.6 发布/订阅基本流程

```
 ┌──────────┐  PUBLISH(topic=t/x, payload, qos)  ┌────────┐  PUBLISH 扇出   ┌──────────┐
 │ Publisher│ ────────────────────────────────▶ │ Broker │ ──────────────▶ │Subscriber│
 └──────────┘  ◀──PUBACK / PUBREC···─── (QoS1/2) └────────┘   (按订阅匹配)  └──────────┘
                                                     │
                                              SUBSCRIBE(topic=t/+)
                                                     ▲
                                              ┌──────────┐
                                              │Subscriber│
                                              └──────────┘
```

### 【纠错】

- 原文 §"mqtt 简介" 同时出现两种全称写法:`Message Queuing Telemetry Transport` 与 `Message Queue Telemetry Transport`,二者出处不同(早期文档与维基对照),**OASIS 标准之后官方明确"MQTT 不再展开"**,引用时应注明这点。
- 原文 QoS0 写为 `Almost Once`,**应为 `At most once`**(Almost ≠ At most)。
- 原文 QoS2 的说明文字直接复制了 QoS1 的解释(开头仍写"QoS1 代表..."),正确应是:"QoS2 代表 Sender 发送的一条消息,Receiver 有且只会收到一次;通过 PUBREC/PUBREL/PUBCOMP 四步握手保证不丢不重"。

---

## 2. Broker 关键特性

### 2.1 Keepalive(心跳)机制

- CONNECT 报文里设置 `keepalive`(秒)。
- 客户端在 keepalive 周期内若无任何消息,**必须主动发 PINGREQ**;Broker 回 PINGRESP。
- 若在 `1.5 × keepalive` 内仍未收到客户端报文,Broker 认为客户端"已挂",触发"异常断连"(可能进而发布遗嘱消息)。

### 2.2 保留消息(Retained Message)

- 发布时把 RETAIN 置 1,Broker 会**为该主题存储最新一条**(只存一条,新覆盖旧)。
- 之后任何**新订阅**该主题的客户端,会立刻收到这条保留消息。
- 删除保留消息:对该主题发布一条 **payload 为空且 RETAIN=1** 的消息。
- 存储介质(内存 / 磁盘)决定 Broker 重启后是否保留,与 Broker 配置有关。

```
时间轴
  t1   Publisher  ── PUBLISH(topic=s/t, payload="v1", retain=1) ──▶ Broker
                                                                       │
                                                                       │ 保存为 s/t 的保留消息
                                                                       ▼
                                                                  ┌────────┐
                                                                  │ Store  │ s/t = "v1"
                                                                  └────────┘
  t2   Subscriber ── SUBSCRIBE(s/t) ───────────────────────────▶ Broker
                                                                       │
        Subscriber ◀── PUBLISH(s/t, "v1", retain=1) ─────────────────  │
                                            (订阅成功立刻收到)
```

#### 重要细节

- 同一会话内,先订阅再发布,这条消息**不算保留消息发送给该订阅者**(它是一条普通的实时消息,即使 RETAIN=1)。要看到"作为保留消息推送"的效果,必须**先发布、后订阅**或**取消订阅再重新订阅**。
- MQTT 5.0 新增"消息过期间隔"属性,可让保留消息按时间自动清理。

### 2.3 持久会话与 Clean Session

#### MQTT 3.1.1 的语义

- CONNECT 报文里有 `Clean Session` 标志:
  - `true`:每次新会话,断开即清除订阅与未完成消息。
  - `false`:**持久会话**,Broker 为该 ClientID 保存订阅关系与未完成的 QoS1/2 消息,断线重连可恢复。
- 持久会话三大作用:
  1. 避免反复订阅。
  2. 接收离线期间消息。
  3. 让 QoS1/2 的"必到达"承诺跨过网络中断。
- **前提**:重连时使用**相同且固定的 ClientID**;若 ClientID 动态生成,等于新会话。

#### MQTT 5.0 的改进

- 把 `Clean Session` 拆成两个独立概念:
  - **Clean Start**:本次连接是否丢弃旧会话(true=丢弃)。
  - **Session Expiry Interval**:断开后会话保留多少秒(0=断开即销毁,`0xFFFFFFFF`=永不过期)。
- 客户端通过 CONNACK 的 **Session Present** 字段判断是否复用了已有会话,据此决定是否需要重新订阅。

```
设备                       Broker
 │  CONNECT(ClientID=c1, CleanStart=false, SessionExp=3600)
 │ ─────────────────────────▶
 │                              ┌─ 查 c1 是否有保留会话 ─┐
 │                              │  有 → SessionPresent=1│
 │                              │  无 → SessionPresent=0│
 │                              └─────────────────────┘
 │  CONNACK(SessionPresent=1)
 │ ◀─────────────────────────
 │
 │  (复用旧订阅,Broker 顺势推送离线期间累积的 QoS1/2 消息)
 │ ◀── PUBLISH(... offline msg ...)
```

#### 工程建议

- 普通设备:`CleanSession=true`,通过"QoS1 + Broker 消息持久化"保证可靠。
- 关键设备:才开持久会话,并配合**有限的离线缓存时长**(如 1 小时),防止 Broker 存储溢出。

### 2.4 遗嘱消息 (Last Will and Testament, LWT)

- CONNECT 时设置 Will Topic / Will Payload / Will QoS / Will Retain。
- **正常 DISCONNECT 不会触发**;只有 Broker 检测到"异常断连"(网络、Keepalive 超时、Crash)才发布。
- 经典模式:**上线主动发 retain="online",遗嘱设为 retain="offline"**,任何订阅者一次订阅即可拿到当前在线状态。

```
设备 A                                        Broker                              监控端
  │ CONNECT(Will=topic:a/will, msg:offline, retain=true)
  │ ─────────────────────────────────────▶
  │
  │ PUBLISH(a/will, "online", retain=true)
  │ ─────────────────────────────────────▶  (覆盖保留消息为 online)
  │
  │ × 异常断连(网络丢失)
  │                                           │
  │                                           │ Keepalive 超时
  │                                           ▼
  │                                  发布遗嘱 PUBLISH(a/will,"offline",retain=true)
  │                                                                          │
  │                                                                          ▼
  │                                                              监控端收到 offline
```

### 2.5 MQTT 5.0 消息过期间隔(Message Expiry Interval)

- 发布时携带"过期间隔"属性。
- 若消息在 Broker 上停留超过这个秒数,**不再分发给任何后续订阅者**。
- 典型场景:实时交通、限时促销、传感器即时读数、自动清理过时的保留消息。

### 【纠错】

- 原文标题写作 `Lass Will and Testament`,**应为 `Last Will and Testament`**。
- 原文若干处把"保留消息只有一条"与"会话恢复时推保留消息"混在一起,这是两件事:**任何新订阅都会立即推**当前的保留消息,**会话恢复**则推的是"离线期间漏掉的 QoS1/2 普通消息"。区分:保留消息**不属于会话状态**,会话销毁也不会删除保留消息。

---

## 3. MQTT over WebSocket

### 3.1 为什么要叠 WebSocket

- 浏览器**不能直接开 TCP 端口**(`mqtt://` 走不通),需要走 `ws://` / `wss://`。
- 一次叠加获得:登录鉴权、消息分发、QoS、Retain、Will,这些 MQTT 已自带,纯 WebSocket 都得自己写。

### 3.2 浏览器 ↔ Broker ↔ 后端的整体拓扑

```
   浏览器                        Nginx(可选)                   MQTT Broker             后端服务
 (mqtt.js /                  (反向代理 ws://)                 (mosquitto/EMQX)
  Paho JS)
     │                              │                              │                      │
     │  HTTP Upgrade: websocket     │                              │                      │
     │ ───────────────────────────▶ │ ──── Upgrade 透传 ─────────▶│                      │
     │                              │                              │                      │
     │  MQTT CONNECT (over ws)      │                              │                      │
     │ ───────────────────────────────────────────────────────────▶│                      │
     │                                                              │
     │  SUBSCRIBE / PUBLISH (over ws frame)                         │
     │ ────────────────────────────────────────────────────────────▶│
     │                                                              │
     │                                                              │ 路由匹配
     │                                                              ▼
     │                                                       ┌──────────┐
     │                                                       │后端订阅端│ paho-mqtt(python/java)
     │                                                       └──────────┘
     │                                                              │
     │ ◀──── 后端反向推送(PUBLISH 同主题) ────────────────────────  │
```

### 3.3 关键 URL/端口约定

- 协议: `ws://` 明文 / `wss://` TLS;`mqtt://` 明文 TCP / `mqtts://` TLS。
- 公共 Broker 示例:
  - EMQX 公共:`ws://broker.emqx.io:8083/mqtt`、`wss://broker.emqx.io:8084/mqtt`、`tcp://broker.emqx.io:1883`、`tls://broker.emqx.io:8883`。
  - HiveMQ 公共:`ws://broker.hivemq.com:8000/mqtt`。
  - mqtt.eclipseprojects.io:WebSocket 走 80/443,**URL 必须是 `/mqtt`**。
- 浏览器端必须用 `ws://` / `wss://`,**不能用 `mqtt://`**,这是原文给前端工程师的核心提醒。

### 3.4 Nginx 反向代理 WebSocket(MQTT 共用)

```nginx
location /mqtt {
    proxy_pass http://emqx_ws_pool;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;          # 关键:协议升级
    proxy_set_header Connection "upgrade";           # 关键
    proxy_set_header Host $host;
    proxy_read_timeout  600s;                        # 防止 1 分钟左右断
    proxy_send_timeout  600s;
}
```

- 现象:不加 `proxy_read_timeout` 时,默认 60s 没数据就被 Nginx 切掉。
- 兜底方案:客户端发 ping/心跳,主动刷新空闲计时(也可直接用 MQTT 的 keepalive)。

### 【纠错】

- 原文有一段公共 Broker 列表把 EMQX 公共服务器写为 `mqtt://47.94.220.165:1833`,**端口拼写错误**,常规 MQTT 明文端口是 `1883`;该 IP 也不是 EMQX 官方公共 Broker,使用前应核实。
- 原文 SSE 一节描述 "SSE 默认支持断线重连;WebSocket 则需要自己实现",这点正确;**但同节"SSE 实现成本低,无需引入其他组件"应限定"在支持 SSE 的浏览器"**,IE/老 Edge 不支持 EventSource,需要 polyfill。

---

## 4. paho-mqtt(Python 客户端)

### 4.1 安装

```bash
pip install paho-mqtt
```

### 4.2 三种"骨架"用法

| 用法 | 适用 | 说明 |
|---|---|---|
| `Client` 类 + 回调 | 长连接订阅、双向通信 | 主流方式 |
| `publish.single` / `publish.multi` | 一次性发布 | 内部自动 connect/disconnect |
| `subscribe.simple` / `subscribe.callback` | 一次性抓 N 条消息 | 调试、脚本 |

### 4.3 回调函数总表(常用)

| 回调 | 触发时机 | 关键参数 |
|---|---|---|
| `on_connect(client, userdata, flags, rc[, properties])` | 收到 CONNACK | `rc=0` 成功,其他为错误码 |
| `on_disconnect(client, userdata, rc)` | 断开连接 | `rc=0` 主动断开,其他为异常 |
| `on_subscribe(client, userdata, mid, granted_qos)` | 收到 SUBACK | `granted_qos` Broker 实际授予的 QoS |
| `on_unsubscribe(client, userdata, mid)` | 收到 UNSUBACK | — |
| `on_publish(client, userdata, mid)` | QoS0:消息离开客户端;QoS1/2:与 Broker 握手完成 | `mid` 与 `publish()` 返回的 mid 对应 |
| `on_message(client, userdata, msg)` | 收到任意未被特定回调匹配的 PUBLISH | `msg.topic / msg.payload / msg.qos / msg.retain` |
| `message_callback_add(sub, cb)` | 指定 topic 过滤器的专用回调,**优先级高于** `on_message` | 多主题分流,推荐 |
| `on_log(client, userdata, level, buf)` | 任意日志 | level 映射 logging 包 |

`on_connect` 的 rc 取值:`0 成功 / 1 协议版本错 / 2 ClientID 无效 / 3 服务器不可用 / 4 用户名密码错 / 5 未授权`。

### 4.4 重要选项函数

- `username_pw_set(u, p)`:在 `connect()` 之前调用。
- `tls_set(ca_certs=..., certfile=..., keyfile=..., cert_reqs=ssl.CERT_REQUIRED, tls_version=..., ciphers=None)`:启用 TLS,**必须在 connect 之前**。
- `tls_insecure_set(True)`:跳过主机名校验,**仅供测试**。
- `will_set(topic, payload, qos, retain)`:遗嘱。
- `reconnect_delay_set(min_delay, max_delay)`:自动重连退避。
- `max_inflight_messages_set(n)`:同时在途的 QoS>0 消息上限,默认 20。
- `max_queued_messages_set(n)`:发送队列上限,0=无限。

### 4.5 网络循环三种调用

```
loop()           手动驱动一次 select(),自己控制节奏
loop_start()     后台开线程跑 loop(),非阻塞,适合嵌进应用
loop_forever()   阻塞主线程跑 loop(),自动重连,脚本场景
```

任选其一,**不要混用**。

### 4.6 完整的发布/订阅模板

```python
# publisher.py
import paho.mqtt.client as mqtt

def on_connect(client, userdata, flags, rc):
    print("connect rc=", rc)

cli = mqtt.Client(client_id="pub-001", clean_session=True)
cli.username_pw_set("u", "p")
cli.will_set("device/pub-001/status", payload="offline", qos=1, retain=True)
cli.on_connect = on_connect
cli.connect("broker.emqx.io", 1883, keepalive=60)
cli.loop_start()
cli.publish("device/pub-001/status", "online", qos=1, retain=True)
cli.publish("sensor/temp", "23.5", qos=1)
# ... 业务 ...
cli.loop_stop()
cli.disconnect()
```

```python
# subscriber.py
import paho.mqtt.client as mqtt

def on_connect(client, userdata, flags, rc):
    client.subscribe([("sensor/+", 1), ("device/+/status", 1)])

def on_message(client, userdata, msg):
    print(msg.topic, msg.payload.decode(), "qos=", msg.qos, "retain=", msg.retain)

def on_temp(client, userdata, msg):           # 专用主题回调
    print("[temp]", msg.payload.decode())

cli = mqtt.Client(client_id="sub-001", clean_session=False)   # 持久会话
cli.username_pw_set("u", "p")
cli.on_connect = on_connect
cli.on_message = on_message
cli.message_callback_add("sensor/temp", on_temp)
cli.connect("broker.emqx.io", 1883, keepalive=60)
cli.loop_forever()
```

### 4.7 回调里"不要做重活"

**原则**:回调只做"中转",别在 `on_message` 里跑 10 秒的业务,否则后续消息会被阻塞。
**正确做法**:把消息丢进队列(`queue.Queue` / Redis / Kafka),业务线程/进程独立消费。

```
                  paho 网络线程(loop_start)
                  ┌──────────────────────┐
 Broker ── PUB ──▶│ on_message 仅入队     │── put ─▶ Queue
                  └──────────────────────┘                 │
                                                            ▼
                                                  ┌────────────────┐
                                                  │ 业务线程/进程池 │── 处理 10s
                                                  └────────────────┘
```

### 【纠错】

- 原文 `Client()` 参数说明里写 `protocol: 可以是 MQTTv3.1 或 MQTTv3.11`,**应为 `MQTTv311`(即 3.1.1) 或加上 `MQTTv5`**;库常量名是 `mqtt.MQTTv31 / mqtt.MQTTv311 / mqtt.MQTTv5`。
- 原文把 `on_disconnect` 写成 `ondisconnect`(漏下划线),实际签名是 `on_disconnect(client, userdata, rc)`(v1 系列),v2 含 reasonCode/properties。
- 原文称"在消息回调中,`message_callback_add()` 的优先级要高于 `on_message` 默认回调",**这是对的**,补充一点:**注册顺序无关,精确度高的 sub(更具体的 topic filter)优先**。
- 原文 `publish()` 内的"有效负载长度大于 268435455 字节"应为 **268435455 = 256 MB - 1** 即 `2^28-1`,这是 MQTT 报文长度的硬限制。

---

## 5. 与 Django/Spring 等后端集成的 7 种 Web 实时推送方案

| 方案 | 通道 | 实时性 | 服务端推送 | 复杂度 | 典型场景 |
|---|---|---|---|---|---|
| 短轮询 | HTTP | 低 | 否(拉) | 极低 | 数据变化极少、对实时性无要求 |
| 长轮询 | HTTP | 中 | 半推 | 中 | Nacos/Apollo/MQ 内部用 |
| iframe 流 | HTTP | 高 | 是 | 低,但 UI 体验差 | 历史方案,不推荐 |
| SSE | HTTP/`text/event-stream` | 高 | 是(单向) | 低 | 站内信、未读数、行情、监控 |
| WebSocket | 独立协议 | 极高 | 是(双向) | 中 | IM、游戏、协作编辑 |
| MQTT(over ws) | WebSocket | 极高 | 是(双向 + Pub/Sub) | 中 | 物联网、跨终端推送、设备 + Web 共用 |
| 第三方推送 | 外部 SDK | 高 | 是 | 低 | 推送、极光、goEasy |

### 5.1 长轮询(DeferredResult 实现)

```
浏览器           Nginx           Tomcat(Spring MVC)             业务变更端
   │  GET /watch/{id}                     │                          │
   │ ──────────────────────────────────▶  │ DeferredResult 注册      │
   │                                      │ Multimap.put(id, dr)     │
   │                                      │   ── 容器线程立刻释放 ──▶ (返回 servlet 池)
   │  (TCP 一直挂着)                       │                          │
   │                                      │                          │  POST /publish/{id}
   │                                      │ ◀──────────────────────  │
   │                                      │ Multimap.get(id).forEach │
   │                                      │   dr.setResult("data")   │
   │  ◀── 响应 200 + 数据 ──────────────  │                          │
   │                                                                  │
   │  立刻再 GET /watch/{id} ······ 循环                              │
```

超时则抛 `AsyncRequestTimeoutException`,用 `@ControllerAdvice` 统一返回约定状态码(原文用 304),前端识别后立刻再发起一次长轮询。

### 5.2 SSE(Server-Sent Events)

```
浏览器(EventSource)                    后端(SseEmitter)
   │  GET /sse/sub/{userId}             │
   │ ────────────────────────────────▶  │ new SseEmitter(0L)
   │                                    │ map.put(userId, emitter)
   │ ◀── 200 OK, text/event-stream ───  │ (单向流,长连接)
   │                                    │
   │ ◀── data: ... \n\n ─────────────── │ emitter.send(...)
   │ ◀── data: ... \n\n ─────────────── │
   │ ...                                │
   │ (断开自动重连)                       │ emitter.onCompletion/Timeout/Error 回调清理
```

要点:`Content-Type: text/event-stream`,只能服务端 → 浏览器,**单向**;断线浏览器自动重连;不支持 IE。

### 5.3 WebSocket

```
浏览器                              Spring(@ServerEndpoint)
  │  HTTP Upgrade: websocket           │
  │ ──────────────────────────────▶    │
  │  101 Switching Protocols           │ 建立 WS 会话,加入 Map<userId,Session>
  │ ◀──────────────────────────────    │
  │                                    │
  │  发 / 收任意帧 (text/binary)        │  session.getBasicRemote().sendText(...)
  │ ◀──────────────────────────────▶   │
```

要点:**全双工**;协议非 HTTP,需要在 Nginx 上配置 Upgrade;无内置鉴权、QoS、分发——都得自己写。

### 5.4 MQTT(同时给 Web 与设备用)

```
   设备(TCP mqtt://)
       │
       ▼
   ┌─────────┐                      ┌─────────────────┐
   │ Broker  │ ◀── PUBLISH ──────▶ │ 后端 paho-mqtt   │ (订阅业务主题,做持久化/审计)
   └─────────┘                      └─────────────────┘
       ▲ ▲
       │ │ ws/wss
       │ └──────────────────────── 浏览器(mqtt.js / paho JS)
       │
       └──────────────────────── 其他终端(手机 SDK / 桌面)
```

相比纯 WebSocket,Web 端额外得到:鉴权、QoS、Retain、Will、Topic 通配,且**消息天然就能扇出给设备/移动端**,真正的"一套消息总线打通三端"。

### 【纠错】

- 原文 SSE 章节说 "SSE 只能传送文本消息,二进制数据需要经过编码后传送",这是对的;补充:文本以 `data:` 行承载,空行 `\n\n` 作为消息分隔,Content-Type 必须为 `text/event-stream`。
- 原文长轮询例子把 `@ResponseStatus(HttpStatus.NOT_MODIFIED)` 配合 304 用于"长轮询超时"——这是为了让前端识别 → 立即重连,而**不是 HTTP 304 的真正语义(缓存命中)**,生产中建议用自定义状态码或 200 + 空体,避免被中间代理误判缓存。

---

## 6. Mosquitto Broker

### 6.1 Mosquitto 是什么

Eclipse Mosquitto:开源 MQTT Broker,支持 3.1 / 3.1.1 / 5.0;轻量、易部署、跨平台、支持 TLS 和 ACL。

### 6.2 安装(Linux 源码编译要点)

```bash
yum install -y openssl-devel c-ares-devel libuuid-devel
wget https://mosquitto.org/files/source/mosquitto-2.x.tar.gz
tar xf mosquitto-2.x.tar.gz && cd mosquitto-2.x
make && sudo make install
sudo ldconfig                       # 避免 libmosquitto.so.1 找不到
```

> 编译报错 `cjson/cJSON.h: 没有那个文件或目录` 时,需要先安装 `cjson` / `cjson-devel`,或显式禁用 `WITH_CJSON=no` 重新编译。

### 6.3 配置文件 `mosquitto.conf` 核心字段

```conf
# 多端口、协议混跑
listener 1883                      # MQTT over TCP
listener 8883                      # MQTT over TLS
  certfile /etc/mosquitto/certs/server.crt
  keyfile  /etc/mosquitto/certs/server.key
listener 8083                      # MQTT over WebSocket
  protocol websockets

# 鉴权
allow_anonymous false              # 关闭匿名
password_file /etc/mosquitto/passwd
acl_file      /etc/mosquitto/acl

# 持久化
persistence true
persistence_location /var/lib/mosquitto/
```

- 同一 Broker 可以**一些 listener 允许匿名、另一些强制鉴权**,通过 `per_listener_settings true` 开启,然后把 `allow_anonymous` / `password_file` 写在对应 listener 块**之后**。
- 修改 `password_file` 后,向 mosquitto 进程发 **SIGHUP** 即可重载,无需重启。

### 6.4 密码文件管理 `mosquitto_passwd`

```bash
# 新建密码文件并加入用户(-c 会覆盖现有文件,谨慎)
mosquitto_passwd -c /etc/mosquitto/passwd admin
# 向已有文件追加 / 改密
mosquitto_passwd /etc/mosquitto/passwd device001
# 删除用户
mosquitto_passwd -D /etc/mosquitto/passwd device001
# 免交互(密码会进入命令历史,慎用)
mosquitto_passwd -b /etc/mosquitto/passwd device001 'pass'
```

### 6.5 常用 CLI

```bash
mosquitto -c /etc/mosquitto/mosquitto.conf -d           # 后台启动
mosquitto -v                                            # 前台详细日志
mosquitto_sub -h broker.emqx.io -t 'sensor/+' -v        # 订阅,-v 同时打印主题
mosquitto_pub -h broker.emqx.io -t 'sensor/t1' -m 'hi' -q 1 -r   # 发布,-r 保留
mosquitto_sub -h host -t t -u user -P pass              # 带鉴权
```

### 6.6 Nginx 反代 mosquitto WebSocket 完整模板

```nginx
upstream mqtt_ws_backend {
    server 10.0.0.10:8083;
    server 10.0.0.11:8083 backup;
}

server {
    listen 443 ssl;
    server_name mqtt.example.com;
    ssl_certificate     /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location /mqtt {
        proxy_pass http://mqtt_ws_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 3600s;
        proxy_send_timeout 3600s;
    }
}
```

### 【纠错】

- 原文 `mosquitto -p 1884` 启动语法:**`-p` 选项已在新版本中标记弃用**,推荐通过配置文件里的 `listener 1884` 来指定。
- 原文 `mosquitto_ctrl status / start / stop / restart` 这一节描述与实际不符:**`mosquitto_ctrl` 不是用来启停 Broker 的服务管理器**,它是用来给运行中的 Broker 发指令(管理动态安全插件、远程控制等),启停应使用 `systemctl start/stop/restart mosquitto` 或 `kill` 信号。

---

## 7. 前端方案

### 7.1 mqttws31.js(已不推荐)

- 来自 Eclipse Paho 早期版本,API 风格陈旧(Paho.MQTT.Client / Paho.MQTT.Message)。
- 仅推荐用于维护遗留项目;新项目用 `mqtt.js`。

### 7.2 mqtt.js(推荐)

```javascript
import mqtt from 'mqtt'

const client = mqtt.connect('wss://broker.emqx.io:8084/mqtt', {
  clientId: 'web-' + Math.random().toString(16).slice(2),
  username: 'u',
  password: 'p',
  clean: false,            // 持久会话
  keepalive: 60,
  reconnectPeriod: 2000,   // 自动重连
  will: { topic: 'web/status', payload: 'offline', qos: 1, retain: true },
})

client.on('connect', () => {
  client.subscribe('sensor/+', { qos: 1 })
  client.publish('web/status', 'online', { qos: 1, retain: true })
})
client.on('message', (topic, payload) => console.log(topic, payload.toString()))
```

### 7.3 防消息丢失的两板斧

1. `clean: false` + 固定 `clientId` + QoS≥1:刷新页面时丢线,重连后 Broker 会把刷新期间的消息补发。
2. 应用层加序号:对收到的消息做去重(QoS1/2 可能重复)。

### 【纠错】

- 原文写"前端不能使用 `mqtt://`,这是 TCP 的方式"——表述要更准确:**浏览器**沙盒不允许直接打开任意 TCP 连接,**Node.js 端 mqtt.js** 是完全可以用 `mqtt://` 的。所以应区分"浏览器环境" vs "Node 环境"。

---

## 8. 速查与排错

### 8.1 端口速查

| 协议 | 端口(常见) |
|---|---|
| MQTT over TCP | 1883 |
| MQTT over TLS | 8883 |
| MQTT over WS (EMQX) | 8083 |
| MQTT over WSS (EMQX) | 8084 |
| MQTT over WS (HiveMQ) | 8000 |
| MQTT over WS (mosquitto 默认) | 9001 |
| EMQX Dashboard | 18083(默认账号 `admin / public`) |

### 8.2 常见错误码(EMQX 文档对齐)

| rc / reason | 含义 | 排查 |
|---|---|---|
| 0 | 成功 | — |
| 1 | 协议版本不被接受 | client `protocol` 与 Broker 配置不匹配 |
| 2 | ClientID 非法 | 含非法字符 / 超长 / 空且 CleanSession=false |
| 3 | 服务端不可用 | Broker 进程 / 网络 |
| 4 | 用户名密码错 | `password_file` / `username_pw_set` |
| 5 | 未授权 | ACL 拦截 |

### 8.3 排错思路

1. **链路**:`ping host` → `telnet host port` → `mosquitto_sub -h host -t '#' -v` 抓所有消息。
2. **鉴权**:换匿名 Broker 验证客户端代码本身。
3. **回调**:打开 `enable_logger()` 看 paho 内部日志。
4. **Nginx**:`nginx -t` + `tail -f error.log`,检查 Upgrade/Connection 是否到了上游。
5. **断线频繁**:keepalive 是否过短、Nginx `proxy_read_timeout` 是否过短、是否 LB 主动 reset。

---

## 9. 整篇主要纠错汇总(便于快速核对)

| 位置 | 原文 | 应为 |
|---|---|---|
| QoS 0 | `Almost Once` | `At most once` |
| QoS 2 描述 | 复制了 QoS1 | "保证有且只一次,PUBREC/PUBREL/PUBCOMP 四步握手" |
| 章节标题 | `Lass Will and Testament` | `Last Will and Testament` |
| EMQX 公共 IP | `mqtt://47.94.220.165:1833` | 端口应为 `1883`;且并非 EMQX 官方公共地址 |
| `Client(protocol=...)` | `MQTTv3.11` | 常量名为 `MQTTv311`(即 3.1.1),或新增 `MQTTv5` |
| 回调名 | `ondisconnect` | `on_disconnect` |
| `mosquitto_ctrl` | 用于 start/stop/restart Broker | 实际用于动态安全管理等,启停用 systemctl |
| `mosquitto -p` | "推荐用法" | 该参数已弃用,改用 `listener` 配置 |
| 前端"不能用 mqtt://" | 适用于所有环境 | 仅适用于**浏览器**,Node.js 端可以 |
