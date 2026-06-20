# 第1章 MQTT 入门 - 知识点梳理

> 对应原文档：`第1章【MQTT入门】/第一章【MQTT入门】.md`

---

## 一、章节知识点总览

```
第1章 MQTT 入门
├── 1. MQTT 概述
│   ├── 1.1 MQTT 简介(协议来源、Pub/Sub 模型、Broker 角色)
│   ├── 1.2 MQTT 五大特性
│   └── 1.3 三大核心概念(Client / Broker / Topic)
├── 2. MQTT 快速入门
│   ├── 2.1 EMQX 简介
│   ├── 2.2 EMQX(Docker)部署 + 端口
│   ├── 2.3 Dashboard 控制台
│   └── 2.4 EMQX 客户端工具 MQTTX
└── 3. MQTT 控制报文
    ├── 3.1 报文概念
    ├── 3.2 15 种控制报文(三类:连接 / 发布 / 订阅)
    ├── 3.3 报文格式(固定报头 + 可变报头 + 有效载荷)
    └── 3.4 用 Wireshark 抓包验证
```

---

## 二、MQTT 概述

### 2.1 MQTT 是什么

MQTT(Message Queuing Telemetry Transport)是 IBM 于 1999 年提出的、基于 **发布/订阅(Pub/Sub)** 模式的轻量级消息传输协议,运行在 TCP/IP 之上,是 ISO 标准(ISO/IEC PRF 20922)与 OASIS 标准协议。

发布订阅模式的核心:发布者(Publisher)与订阅者(Subscriber)通过第三方组件 **Broker(代理)** 完成消息中转,二者无需直接通信。

```
                          ┌──────────────┐
   Publisher ── PUB ────► │              │ ── PUB ──► Subscriber A
                          │   Broker     │
   Publisher ── PUB ────► │              │ ── PUB ──► Subscriber B
                          └──────────────┘
                            (路由 / 过滤)
```

解耦的两个维度:

| 维度 | 含义 |
| ---- | ---- |
| 空间解耦 | Pub 与 Sub 不需知道对方的地址、身份是否存在 |
| 时间解耦 | Pub 与 Sub 不必同时在线(配合会话/保留消息可以异步通信) |

> 适用场景:**物联网(IoT)**、车联网、即时消息、低带宽 / 高延迟 / 弱网环境。

### 2.2 MQTT 五大特性

1. **轻量级**:固定报头最小仅 2 字节,适合资源受限设备。
2. **可靠**:提供 QoS 0/1/2 三个等级 + 会话(Session) + 持久连接 + 遗嘱消息。
3. **安全**:支持 TLS/SSL 加密,支持 用户名/密码、客户端证书 等多种认证手段。
4. **双向通信**:Broker 可主动向客户端推送消息(无需轮询),客户端既可发布也可订阅。
5. **多语言支持**:Java / Python / Go / Node.js / C / C++ / PHP 等几乎所有主流语言都有客户端库。

### 2.3 三大核心概念

| 概念 | 说明 |
| ---- | ---- |
| MQTT Client | 任何运行 MQTT 客户端库的应用或设备(传感器、手机 App、测试工具)。**既可以是 Publisher,也可以是 Subscriber**。 |
| MQTT Broker | 消息代理,负责连接管理、订阅管理、消息路由与转发。EMQX、Mosquitto、HiveMQ 都是 Broker 实现。 |
| Topic | UTF-8 字符串,用 `/` 分层(类似 URL 路径),是消息路由的依据。订阅或发布时由客户端动态创建,无需预先声明。 |

主题示例:

```text
chat/room/1
sensor/10/temperature
```

> 不建议用 `/` 开头或结尾(如 `/chat`、`chat/`),会形成不易察觉的空层级。

---

## 三、EMQX 与快速入门

### 3.1 EMQX 简介

EMQX 是国内厂商 EMQ 开源的、用 Erlang/OTP 实现的 MQTT Broker,同时支持 **MQTT 3.1 / 3.1.1 / 5.0** 三个版本协议。EMQX 5.x 默认以 MQTT 5.0 接入。

### 3.2 部署(Docker)

```bash
docker run -d --name emqx-enterprise \
  -p 1883:1883 -p 8083:8083 \
  -p 8084:8084 -p 8883:8883 \
  -p 18083:18083 \
  -v emqx_data:/opt/emqx/data \
  -v emqx_log:/opt/emqx/log \
  -v emqx_etc:/opt/emqx/etc \
  emqx/emqx-enterprise:5.6.1
```

### 3.3 默认端口(重要)

| 端口  | 协议                                | 用途                       |
| ----- | ----------------------------------- | -------------------------- |
| 1883  | MQTT over TCP                       | 普通 MQTT 接入(明文)     |
| 8883  | MQTT over TLS/SSL                   | 加密 MQTT 接入             |
| 8083  | MQTT over WebSocket                 | 浏览器/H5 接入(明文)     |
| 8084  | MQTT over WebSocket Secure(WSS)   | 浏览器/H5 加密接入         |
| 18083 | HTTP                                | Dashboard 管理后台         |

### 3.4 Dashboard

- 访问地址:`http://<EMQX-IP>:18083`
- 默认账号密码:`admin / public`
- 重置密码 CLI:

```bash
./bin/emqx ctl admins passwd <Username> <Password>
```

主要功能:监控管理、访问控制(认证/授权)、数据集成(规则引擎)、在线热更新配置。

### 3.5 客户端工具 MQTTX

EMQ 出品,三种形态:

- **MQTTX Desktop**:跨平台桌面客户端(Win/macOS/Linux)
- **MQTTX CLI**:命令行客户端
- **MQTTX Web**:浏览器客户端,可通过 Docker 部署:

```bash
docker pull emqx/mqttx-web
docker run -d --name mqttx-web -p 80:80 emqx/mqttx-web
```

CLI 常用命令:

```bash
# 订阅
mqttx-cli sub -t 'test/1' -h 192.168.136.147 -p 1883 -v
# 发布
mqttx-cli pub -t 'test/1' -q 0 -h 192.168.136.147 -p 1883 -m "hello"
```

参数:`-t` 主题、`-h` Broker IP、`-p` 端口、`-v` 显示主题、`-q` QoS、`-m` 消息内容。

---

## 四、MQTT 控制报文(核心)

### 4.1 报文分类

MQTT 5.0 共定义 **15** 种控制报文(MQTT 3.1.1 为 14 种,缺 AUTH),按功能分三类:

```
连接类:CONNECT、CONNACK、DISCONNECT、AUTH(5.0 新增)、PINGREQ、PINGRESP
发布类:PUBLISH、PUBACK、PUBREC、PUBREL、PUBCOMP
订阅类:SUBSCRIBE、SUBACK、UNSUBSCRIBE、UNSUBACK
```

### 4.2 报文统一结构

所有控制报文都由三部分组成,但 **可变报头 / 有效载荷可选**:

```
┌──────────────────────────────────────────────────┐
│  固定报头 Fixed Header (1~5 字节,所有报文必有)  │
├──────────────────────────────────────────────────┤
│  可变报头 Variable Header (可选,因报文类型而异) │
├──────────────────────────────────────────────────┤
│  有效载荷 Payload (可选,因报文类型而异)         │
└──────────────────────────────────────────────────┘
```

举例:

- `PINGREQ`:仅有固定报头(2 字节)
- `PUBLISH`:三部分都有

### 4.3 固定报头(Fixed Header)

第 1 字节 = 报文类型(高 4 bit) + 标志位(低 4 bit);后跟剩余长度(1~4 字节,可变长编码)。

```
 第1字节  ┌─────────────┬─────────────┐
          │ 报文类型(4) │ 标志位(4)   │
          └─────────────┴─────────────┘
 第2~5字节 [ 剩余长度 (Remaining Length) ]
```

- **报文类型**(4 bit):取值 1~15,标识 CONNECT、PUBLISH 等。
- **标志位**(4 bit):多数报文为保留位;**只有 PUBLISH 报文** 这 4 位有实际含义:
  - Bit 3:**DUP** — 是否是重传
  - Bit 2~1:**QoS** — 服务质量等级 0/1/2
  - Bit 0:**RETAIN** — 是否为保留消息
- **剩余长度**:表示「可变报头 + 有效载荷」的字节数;采用 **可变长编码**(每字节高位为延续标志),最多 4 字节,故单条 MQTT 报文最大 256 MB。

> 报文总长度 = 固定报头长度(含剩余长度字段本身)+ 剩余长度值。

### 4.4 可变报头(Variable Header)

内容因报文而异,常见示例:

| 报文     | 可变报头字段顺序                                                      |
| -------- | --------------------------------------------------------------------- |
| CONNECT  | 协议名 + 协议级别 + **连接标志位(Connect Flags)** + Keep Alive + 属性 |
| PUBLISH  | 主题名 + 报文标识符(Packet ID,QoS>0 时才有)+ 属性                  |
| SUBSCRIBE| 报文标识符 + 属性                                                     |

**属性(Properties)是 MQTT 5.0 引入的概念**,位于可变报头末尾:

```
┌────────────────┬─────────────────────────────────┐
│ 属性长度       │ 属性 1 │ 属性 2 │ ... │ 属性 N │
└────────────────┴─────────────────────────────────┘
```

属性长度 = 后续所有属性的总字节数;无属性时长度为 0。

### 4.5 有效载荷(Payload)

承载报文核心业务数据:

| 报文      | Payload 内容                          |
| --------- | ------------------------------------- |
| CONNECT   | Client ID、遗嘱主题/内容、用户名、密码 |
| PUBLISH   | 应用消息内容(业务数据)             |
| SUBSCRIBE | 想订阅的主题列表 + 订阅选项           |

### 4.6 抓包验证(Wireshark)

- 在 Wireshark 中筛选 `mqtt`,可观察 CONNECT / CONNACK / SUBSCRIBE / SUBACK / PUBLISH / PUBACK / PUBREC / PUBREL / PUBCOMP / DISCONNECT 全过程。
- QoS 0:只有一个 PUBLISH。
- QoS 1:PUBLISH → PUBACK。
- QoS 2:PUBLISH → PUBREC → PUBREL → PUBCOMP(完整四次握手)。

---

## 五、原文档错误与勘误

| # | 位置 | 原文 | 问题 | 正确说法 |
| - | ---- | ---- | ---- | -------- |
| 1 | 2.3 节 Dashboard 默认密码 | `admin/pubic` | **拼写错误**,缺 `l` | `admin/public` |
| 2 | 端口表 8084 行 | "WebSocket Secure 端口" | 描述含糊,易误解为另一种 WSS 协议 | 应明确为 **MQTT over WebSocket Secure (WSS)**,即基于 TLS 的 WebSocket MQTT 接入 |
| 3 | 端口表 1883 行 | "TCP 端口" | 表述不严谨 | **MQTT over TCP 的默认端口(明文)** |
| 4 | 2.2.3 节标题(MQTTX CLI) | `### 2.2.3 MQTTX CLI 的使用` | 章节编号错误,父级是 2.4 | 应为 `### 2.4.3 MQTTX CLI 的使用` |
| 5 | 2.2.4 节标题 | `### 2.2.4 MQTTX Web 的使用` | 同上,编号错误 | 应为 `### 2.4.4 MQTTX Web 的使用` |
| 6 | 2.2.3 节代码块语言标识 | ` ```sell ` | 拼写错误 | 应为 ` ```shell ` |
| 7 | 1.2 节"双向通信"描述 | "客户端既可以向主题发布消息,也可以订阅接收特定主题上的消息" | 这是 Pub/Sub 的一般特征,不能精确刻画"双向" | MQTT 的"双向"重点在于 **Broker 可以主动向客户端推送消息(服务器推送)**,客户端无需轮询 |
| 8 | 1.2 节语言支持列表 | "PHP、Node.js、Python、Golang、Node.js、java" | **Node.js 重复列出** | 去重为 PHP、Node.js、Python、Golang、Java、C/C++ 等 |
| 9 | 3.3.2 节 CONNECT 可变报头字段 | "连接标识" | 翻译不规范 | 协议规范叫 **Connect Flags(连接标志位)**,内含 Clean Start、Will Flag、Will QoS、Will Retain、Password Flag、User Name Flag 等 |
| 10 | 3.2 节 | "MQTT 目前定义了 15 种控制报文类型" | 未区分版本 | **MQTT 3.1.1 是 14 种**(无 AUTH),**MQTT 5.0 才是 15 种**(新增 AUTH 用于增强认证) |
| 11 | 3.3.1 节剩余长度公式 | "MQTT 控制报文的总长度 = 固定报头的长度 + 剩余长度" | 表述容易让人误以为"剩余长度"指字节数本身,但它本身也算在固定报头中 | 准确表述:**总长度 = 固定报头(含剩余长度字段)+ 剩余长度字段所表示的值(即可变报头 + Payload 长度)** |

---

## 六、本章重点速查

- Pub/Sub 解耦 = 空间解耦 + 时间解耦
- 报文 3 段式结构:固定报头(必有)+ 可变报头(可选)+ 有效载荷(可选)
- 只有 **PUBLISH** 报文的固定报头标志位才有 DUP / QoS / RETAIN 含义
- MQTT 5.0 = 15 种控制报文(比 3.1.1 多 AUTH)
- EMQX 默认端口需牢记:**1883(TCP)/ 8883(TLS)/ 8083(WS)/ 8084(WSS)/ 18083(Dashboard)**
