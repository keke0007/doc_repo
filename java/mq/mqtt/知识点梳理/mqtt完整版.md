好,把两份都展开补全。下面是完整加强版,直接看就行。

------

# 文档一:第二章 MQTT 进阶 完整版

## 一、QoS 服务质量

### 1.1 QoS 概念与背景

MQTT 设备多运行在网络受限、不稳定的环境下(蜂窝网络、卫星链路、低功耗 IoT)。仅依赖底层 TCP 协议无法完全保证消息可靠到达——TCP 仅在连接稳定不关闭的前提下保证字节流的有序到达,一旦断连,处于网络栈缓冲区中的消息会丢失。

为此,MQTT 在应用层设计了 **QoS(Quality of Service)** 机制,通过不同的报文交互流程,提供三种消息可靠性等级。

| 等级  | 名称                    | 投递语义          | 报文交互                            | 开销 |
| ----- | ----------------------- | ----------------- | ----------------------------------- | ---- |
| QoS 0 | 至多一次(At most once)  | 可能丢失          | 1 次单向 PUBLISH                    | 最小 |
| QoS 1 | 至少一次(At least once) | 不会丢失,可能重复 | PUBLISH + PUBACK                    | 中   |
| QoS 2 | 仅一次(Exactly once)    | 不丢失也不重复    | PUBLISH + PUBREC + PUBREL + PUBCOMP | 最大 |

### 1.2 QoS 等级是谁指定的

- **发布端 → Broker**:由发布端在 PUBLISH 报文中指定
- **Broker → 订阅端**:取 `min(消息的 QoS, 订阅时声明的最大 QoS)`,即只可降级,不可升级

举例:发布端发 QoS 2 消息,订阅端订阅时声明最大 QoS 为 1,则 Broker 转发该消息时降级为 QoS 1。

### 1.3 QoS 0 详解

**报文交互:**

```
 Sender                              Receiver
   |                                    |
   |--- PUBLISH (QoS=0, no Packet ID)-->|
   |    payload, dup=0, retain=?         |
   |                                    |
        (即发即弃,无确认)
```

**特点:**

- 不需要 Packet ID(因为没有重传)
- 不需要本地存储
- 不需要重传
- 接收端不会有重复消息

**消息丢失场景:**

1. 发送端调用 publish 后,消息进入 OS TCP 发送缓冲区,此时连接断开 → 缓冲区数据丢失
2. 消息已发出但接收方在 TCP 层尚未将其传给 MQTT 层时连接断开
3. 网络链路上发生丢包,TCP 还未来得及重传

**适用场景:**

- 周期性传感器采样(温湿度、心率)
- 实时位置上报
- 高频但允许丢失个别样本的数据流

### 1.4 QoS 1 详解

**报文交互:**

```
 Sender                                 Receiver
   ||
   |--- PUBLISH (QoS=1, Packet ID=N) ---->|
   |    [本地存储 PUBLISH,启动定时器]     |
   |                                       |
   |<------ PUBACK (Packet ID=N) ---------|
   |    [删除本地缓存,定时器停止]         |
   |                                       |
   ===== 若超时未收到 PUBACK =====
   |--- PUBLISH (Packet ID=N, DUP=1) ---->|
   |    [DUP 标志位置 1 表示这是重传]      |
   |                                       |
   |<------ PUBACK (Packet ID=N) ---------|
```

**Packet ID 的作用:**

- 是 PUBLISH 与 PUBACK 配对的唯一标识
- 只在两个 MQTT 端点之间生效,不是全局唯一
- 在收到对应的确认报文之前,Packet ID 被占用,不能用来发送新消息

**重复消息成因(两种场景):**

场景 A:PUBLISH 在网络中丢失Receiver |--- PUBLISH(N) ----X 网络丢包 |    [超时] |--- PUBLISH(N, DUP=1) ----------->|  收到 1 次 |<------ PUBACK(N) ----------------| 结果:接收方收到 1 次,无重复

```
场景 B:PUBACK 在网络中丢失
```

Sender                              Receiver |--- PUBLISH(N) ------------------>|  收到 1 次 |<--- PUBACK(N) ----X 网络丢包 |    [超时] |--- PUBLISH(N, DUP=1) ----------->|  又收到 1 次 |<------ PUBACK(N) ----------------| 结果:接收方收到 2 次,重复

```
**关键点:**
- 接收方**不能**仅凭 DUP=1 自动去重(因为接收方在场景 A 中也会看到 DUP=1)
- 业务侧需要自己设计幂等逻辑(如以业务流水号去重)

**消息重复带来的危害:**
若用 QoS 1 下发"开/关灯"指令,顺序 1, 2 可能在订阅端看到 1, 2, 1, 2,造成设备状态来回翻转。

### 1.5 QoS 2 详解

QoS 2 通过 4 次握手(2 次请求/响应流程)实现"仅一次"语义。

**完整报文交互:**
```

Sender                                   Receiver |                                         | |--- PUBLISH (QoS=2, Packet ID=N) ------>| |    [发送方存储 PUBLISH]                  [接收方存储 Packet ID |                                          标记"已收到 N"] |                                         | |<-------- PUBREC (Packet ID=N) ---------| |    [发送方删除 PUBLISH, |     转存 PUBREL, |     从此不能再重传 PUBLISH]              | |                                         | |--- PUBREL (Packet ID=N) -------------->| |                                          [接收方收到 PUBREL, |                                           确认本次传输完成, |                                           丢弃 Packet ID 记录] |                                         | |<-------- PUBCOMP (Packet ID=N) --------| |    [发送方删除 PUBREL, |     释放 Packet ID,可再使用]            |

```
**四个报文的角色:**

| 报文 | 方向 | 作用 |
|------|------|------|
| PUBLISH | S → R | 发起消息,携带 Packet ID 和 payload |
| PUBREC(Receive) | R → S | 确认收到 PUBLISH |
| PUBREL(Release) | S → R | 通知接收端可以释放 Packet ID 了 |
| PUBCOMP(Complete) | R → S | 双方释放完成 |

**为什么 QoS 2 不会重复(去重原理):**

QoS 1 之所以重复,是因为接收方一回完 PUBACK 就立刻"忘记"了这个 Packet ID,无法分辨后来同 ID 的 PUBLISH 是重传还是新消息。

QoS 2 引入了 PUBREL,把"何时可以释放 Packet ID"这一动作从单方决定改成**双方协商**:
- 接收方在收到 **PUBREL 之前**:相同 Packet ID 的 PUBLISH 都视为重复消息(丢弃)
- 接收方在收到 **PUBREL 之后**:才允许释放该 Packet ID,后续相同 Packet ID 的 PUBLISH 是新消息

**重传规则(关键约束):**

| 状态 | 发送方可否重传 PUBLISH |
|------|---------------------|
| 已发 PUBLISH,未收 PUBREC | **可以**重传 PUBLISH |
| 已收 PUBREC,已发 PUBREL | **不可**重传 PUBLISH(只能重传 PUBREL) |
| 已收 PUBCOMP | Packet ID 释放,可发新消息 |

### 1.6 三种 QoS 适用场景对比

| QoS | 典型场景 | 业务考虑 |
|-----|---------|---------|
| 0 | 传感器周期采样、位置上报、监控数据流 | 数据量大、能容忍偶尔丢失 |
| 1 | 设备指令下发、状态变更通知、IM 聊天消息 | 需要业务侧做幂等去重 |
| 2 | 金融交易、计费扣费、航空管制、医疗指令 | 不允许丢失也不允许重复 |

---

## 二、主题(Topic)详解

### 2.1 主题基础

**本质:** UTF-8 编码的字符串,作为 MQTT 消息路由的依据,使用 `/` 分层。
**特点:**
- 大小写敏感:`a/B` 与 `a/b` 不同
- 不需要预创建,客户端订阅或发布时主题自动出现,无需手动删除
- 不建议以 `/` 开头或结尾(语义易引发误解,如 `/a/b` 会比 `a/b` 多一个空层级)
- 单层级允许包含字符 `a-z A-Z 0-9 / _ -`,中文也可,但生产中不推荐
- 主题层级数量有上限(EMQX 默认 7 层,可调)

### 2.2 通配符

**单层通配符 `+`:**
- 必须独占一个层级
- 可以出现在任意位置(开头、中间、末尾)

| 模式 | 是否合法 |
|------|---------|
| `+` | ✓ 匹配第一层任意值 |
| `test/+` | ✓ |
| `test/+/temperature` | ✓ |
| `test+` | ✗ 没独占整个层级 |
| `+test` | ✗ |

匹配示例(`test/+/temperature`):
- ✓ `test/1/temperature`
- ✓ `test/abc/temperature`
- ✗ `test/temperature`(层级数不够)
- ✗ `test/1/2/temperature`(层级数过多)

**多层通配符 `#`:**
- 必须独占一个层级
- 必须是主题的最后一个字符
- 表示其父级及任意数量(0 个或多个)的子层级

| 模式 | 是否合法 |
|------|---------|
| `#` | ✓ 匹配所有主题 |
| `test/#` | ✓ 匹配 `test`、`test/a`、`test/a/b` 等 |
| `test/bedroom#` | ✗ 没独占整个层级 |
| `test/#/temperature` | ✗ 不在末尾 |

匹配示例(`test/#`):
- ✓ `test`
- ✓ `test/temperature`
- ✓ `test/1/2/3/temperature`

### 2.3 系统主题 $SYS/

EMQX 等 Broker 通过 `$SYS/` 开头的主题对外暴露集群运行状态。MQTT 协议本身未强制规定 `$SYS/` 的具体子主题,但社区有事实标准。

EMQX 常用系统主题:

| 主题 | 用途 |
|------|------|
| `$SYS/brokers` | 集群节点列表 |
| `$SYS/brokers/<node>/version` | EMQX 版本 |
| `$SYS/brokers/<node>/uptime` | 节点运行时长 |
| `$SYS/brokers/<node>/datetime` | 当前系统时间 |
| `$SYS/brokers/<node>/clients/+/connected` | 客户端上线事件 |
| `$SYS/brokers/<node>/clients/+/disconnected` | 客户端下线事件 |
| `$SYS/brokers/<node>/stats/connections/count` | 当前连接数 |
| `$SYS/brokers/<node>/metrics/messages/sent` | 消息发送统计 |

**注意:**
- 订阅 `$SYS/#` 可获取全部系统消息
- 系统主题在 Broker 端默认有访问限制,需在授权配置中开通对应权限
- `+` 与 `#` 不会匹配以 `$` 开头的主题(MQTT 规范),需精确指定 `$SYS/...`

---

## 三、会话(Session)

### 3.1 会话与连接的关系
```

+----------------- 会话 (Session) -----------------+ |                                                  | |   +-- 连接1 --+   断开    +-- 连接2 --+          | |   |  CONNECT |  -------> |  CONNECT |  ...     | |   +----------+           +----------+          | |                                                  | |   会话状态:订阅、未确认的 QoS1/2 消息、         | |             离线缓存的 QoS1/2 消息、遗嘱信息    | +--------------------------------------------------+

```
**关键事实:**
- 一个会话可以跨多次 TCP 连接存在
- 会话由 **Client ID** 唯一标识
- 同一 Client ID 同时只能有一个活跃连接(后来的连接会踢掉前一个)
- 会话状态包含订阅、QoS 1/2 投递中的消息、离线消息、Will 等

### 3.2 Clean Start(MQTT 5.0)/ Clean Session(MQTT 3.1.1)

| 取值 | 含义 |
|------|------|
| 0 / false | Broker 若有同 Client ID 的旧会话,**复用**它(订阅恢复、离线消息推送) |
| 1 / true | Broker 必须**丢弃**旧会话,以全新会话开始 |

**MQTT 5.0 与 3.1.1 区别:**
- 3.1.1 中 `Clean Session=0` 表示既要复用旧会话,也要把当前会话保留下来
- 5.0 把这两件事拆开:`Clean Start` 控制开始时是否复用,`Session Expiry Interval` 控制结束后保留多久

### 3.3 Session Expiry Interval(MQTT 5.0)

| 取值 | 含义 |
|------|------|
| 未指定 / 0 | 网络断开时立即结束会话 |
| 大于 0 | 断开后保留 N 秒,N 秒内重连可恢复 |
| 0xFFFFFFFF | 永不过期 |

### 3.4 会话恢复完整流程(跨多端调用)
```

Publisher          Broker                Subscriber (Client ID = "sub01") |                |                          | |                |<-- CONNECT ---|                |    Clean Start = 1 |                |    Session Expiry = 300         | |                |--- CONNACK (SP=0) ---->| |                |                          | |                |<-- SUBSCRIBE topic/X ---| |                |--- SUBACK ------------>| |                |                          | |---PUBLISH----->|---PUBLISH(转发)-------->| |                |       Subscriber 断开    X
 |---PUBLISH----->|  存入会话队列
 |---PUBLISH----->|  存入会话队列

```
 |                |<-- CONNECT (同 ClientID,Clean Start=0) ---|
 |                |--- CONNACK (SP=1) ---->|     SP=1 表示找到旧会话
 |                |--- 推送离线消息 N----->|
 |                |--- 推送离线消息 N+1 -->|
**SP(Session Present)** 在 CONNACK 中:
- 0:Broker 没找到旧会话,本次开启全新会话
- 1:成功复用了旧会话

### 3.5 会话演示步骤(MQTTX 操作)

1. 关闭 MQTTX 的"自动重订阅"功能(否则 sub 重连后会自动 SUBSCRIBE,看不到会话恢复的效果)
2. 创建 sub 客户端:MQTT 5.0 / Clean Start=true / Session Expiry=300,连接后订阅 `mqttx_xxx/test`
3. 创建 pub 客户端,向同主题发布消息;sub 应收到
4. 断开 sub,继续用 pub 发布;此时 Broker 把消息缓存到 sub 的会话队列中
5. 把 sub 的 Clean Start 改为 false,保持 Session Expiry=300,再次连接
6. sub 应陆续收到离线期间发布的消息

---

## 四、消息特性

### 4.1 保留消息(Retained Message)

#### 4.1.1 概念
- **普通消息:** 发布时若无订阅者,Broker 直接丢弃
- **保留消息:** Broker 会持久化(内存或磁盘),后续任何**新订阅者**订阅匹配主题时,**立即**收到这条最新保留消息

#### 4.1.2 使用场景
1. 智能家居设备状态上报:状态变更频次低,新上线的控制端需要立刻拿到当前状态
2. 传感器版本号、序列号等基本不变的属性:上线后发一条保留消息
3. 上报间隔较长的传感器:新订阅者立刻拿到最近一次的值

#### 4.1.3 发布与生命周期
- 发布时将 `Retained` 标志设为 `true`
- Broker 为**每个主题**只保存**最新一条**保留消息(新发的同主题保留消息会覆盖旧的)
- 保留消息**不属于会话状态**:发布它的客户端断开后,保留消息依然存在

#### 4.1.4 删除三种方式
1. 发送 Payload 为空的同主题保留消息
2. Dashboard 手动删除
3. 配合 MQTT 5.0 的 Message Expiry Interval,过期自动清理

#### 4.1.5 存储方式
- **内存存储**(默认):快,但 Broker 重启会丢失
- **磁盘存储**:重启不丢,但写入性能下降

### 4.2 消息过期间隔(Message Expiry Interval,MQTT 5.0)

#### 4.2.1 概念
为有时效性的消息设置过期时间(秒)。若该消息在 Broker 中**等待投递**的时间超过这个间隔,Broker 不再分发。

**典型场景:** 联网汽车的"建议车速,绿灯可通过"提示,只在车到达下一个路口前有效。

#### 4.2.2 转发时的间隔更新
```

发布端 → Broker:Message Expiry = 60s [消息在 Broker 等待 25s] Broker → 订阅端:Message Expiry = 60 - 25 = 35s

```
这能避免桥接到下一个 Broker 时时效性丢失。

#### 4.2.3 演示步骤
1. pub 与 sub 都连上 Broker;sub 订阅主题 `mqttx_xxx/demo`,Session Expiry=300
2. sub 断开
3. pub 发两条:Message Expiry 分别为 5 秒、60 秒
4. 等待至少 5 秒,sub 重连(Clean Start=false)
5. 结果:sub 只收到过期 60 秒的那条;5 秒的已被丢弃

### 4.3 遗嘱消息(Will Message / LWT - Last Will and Testament)

#### 4.3.1 作用
让其他客户端**感知到某客户端意外断开**。客户端在 CONNECT 时把遗嘱注册给 Broker,意外断连时由 Broker 代为发布。

#### 4.3.2 触发与不触发
| 断开方式 | 是否触发遗嘱 |
|---------|------------|
| 客户端发送 DISCONNECT 报文(正常断开) | **不触发** |
| TCP 连接异常断开 | **触发** |
| Keep Alive 超时 | **触发** |
| 客户端被 Broker 踢下线(协议错误等) | 视场景,通常**触发** |

#### 4.3.3 遗嘱字段
和普通消息类似:
- Will Topic
- Will Payload
- Will QoS
- Will Retain
- Will Properties(MQTT 5.0)
- **Will Delay Interval**(MQTT 5.0,遗嘱专属)

#### 4.3.4 Will Delay Interval
- 默认 0:连接异常断开时立即发布
- 大于 0:延迟 N 秒发布;若客户端在 N 秒内成功重连(并恢复同会话),遗嘱**不会**发布
- 解决"短暂断网导致遗嘱被无谓发送"的问题

#### 4.3.5 遗嘱与会话的纠葛

遗嘱属于会话状态。两种边界情况:
- 在 Will Delay 倒计时期间,Session 过期 → 必须立即发布遗嘱(避免遗嘱跟着会话一起被丢弃)
- 客户端用 Clean Start=1 重连 → 旧会话被丢弃 → 必须立即发布旧会话的遗嘱

总结:**Will Delay 到期** 与 **会话结束** 哪个先发生,Broker 就在那个时机发布遗嘱。

#### 4.3.6 演示步骤
```

1. Client A 连接,指定: Will Topic = mqttx_xxx/status Will Payload = "offline" Will Delay Interval = 5s Session Expiry = 300s
2. Client B 连接,订阅 mqttx_xxx/status
3. Client A 正常 DISCONNECT → Client B 收不到遗嘱
4. Client A 关闭 TCP(模拟掉线)→ 5 秒后 Client B 收到 "offline"

```
### 4.4 延迟发布(EMQX 扩展)

#### 4.4.1 概念
EMQX 自定义的能力,通过特殊主题前缀让消息在 Broker 中"睡一会儿"再转发。

**主题格式:** `$delayed/{DelayInterval}/{TopicName}`
- `$delayed`:固定前缀(注意原文有错别字 `$delay`,**正确是 `$delayed`**)
- `{DelayInterval}`:延迟秒数(整数),最大 4294967 秒;若解析失败,EMQX 直接丢弃该消息
- `{TopicName}`:真实目标主题

**示例:**
```

$delayed/15/x/y    → 15 秒后发布到 x/y $delayed/60/a/b    → 60 秒后发布到 a/b $delayed/3600/sys  → 1 小时后发布到 sys

```
#### 4.4.2 应用场景
1. 农业:清晨/傍晚定时启动灌溉
2. 能源:回家前 30 分钟开空调
3. 公共设施:夜间统一开关路灯
4. 医疗:固定时间提醒服药

#### 4.4.3 演示步骤
1. Dashboard:**管理 → 延迟发布 → 启用**(默认已启用),设置最大延迟消息数
2. sub 订阅 `delay/msg`(注意是真实主题,**不要**带 `$delayed/` 前缀)
3. pub 向 `$delayed/10/delay/msg` 发布消息
4. 等 10 秒,sub 收到该消息

### 4.5 用户属性(User Property,MQTT 5.0)

Key-Value 形式的扩展元数据,可在 PUBLISH、SUBSCRIBE、CONNECT、DISCONNECT 等多种报文中携带。**类似 HTTP Header**。

**特点:**
- 一个报文中可有多个用户属性,Key 可重复
- Broker 转发 PUBLISH 时会把用户属性原样转发给订阅者

**应用场景:**
1. 审计日志:在 PUBLISH 中携带 `operator=alice`、`reason=manual_override`
2. 消息分类:`type=alarm`、`level=critical` 用于订阅端过滤或路由
3. 链路追踪:`trace_id=...`、`span_id=...`

---

## 五、订阅详解

### 5.1 订阅选项(MQTT 5.0)

订阅由两部分组成:
- **主题过滤器(Topic Filter)**:决定哪些主题的消息会被路由过来
- **订阅选项(Subscription Options)**:进一步定制 Broker 的转发行为

四个订阅选项:

| 选项 | 取值 | 默认 |
|------|------|------|
| QoS | 0 / 1 / 2 | — |
| No Local | 0 / 1 | 0 |
| Retain As Published(RAP) | 0 / 1 | 0 |
| Retain Handling(RH) | 0 / 1 / 2 | 0 |

#### 5.1.1 QoS 选项

订阅时声明的 QoS = **服务端转发消息时允许使用的最大 QoS**。Broker 转发时取 `min(消息原始 QoS, 订阅时声明的最大 QoS)`。

两种场景:
- 服务端支持的最大 QoS < 客户端请求 QoS:Broker 在 SUBACK 中**告知最终授予的 QoS**,订阅方决定是否继续
- 订阅请求的 QoS < 消息发布的 QoS:Broker 在转发时**降级**

**演示:**
```

sub 订阅 sub/qos/demo,声明 QoS=0 pub 向 sub/qos/demo 发 QoS=1 消息 结果:sub 收到的消息 QoS=0(降级)

```
#### 5.1.2 No Local

| 取值 | 含义 |
|------|------|
| 0(默认) | 允许把消息回投给发布者本人 |
| 1 | 不把消息回投给发布者本人 |

**典型应用:** 桥接(Bridge)场景。
```

Server A <----- 互订阅 # ----- Server B (无 No Local 时)

A 的客户端发消息 → A 路由到 B → B 路由到 A → A 又路由到 B → ... 形成无限转发风暴

```
设置 No Local=1 后,B 不会把"来自 A 的消息"再投回 A,风暴消失。

**演示:**
```

sub 订阅 sub/local/demo,No Local=1 sub 自己向 sub/local/demo 发消息 结果:sub 收不到自己的消息

```
#### 5.1.3 Retain As Published(RAP)

| 取值 | Broker 转发时 |
|------|--------------|
| 0(默认) | **清除**消息中的 Retain 标志 |
| 1 | **保留**原 Retain 标志 |

**问题场景:** 桥接时,如果 A 把保留消息转发到 B,Broker B 默认会看到 Retain=0(被 A 清掉了),不会把它当保留消息存储。

**解决方案:** 在桥接订阅中设置 RAP=1,让 Retain 标志原样穿透。

> ⚠️ 原文写 "0(默认值):服务端在向此订阅转发应用消息时需要`清除`消息中的 Retain 标识不变" — "清除"和"不变"自相矛盾。**正确含义**:0 = 清除,1 = 保持。

#### 5.1.4 Retain Handling(RH)

控制订阅建立时,Broker **是否要立即把已有的保留消息推送给订阅方**。

| 取值 | 含义 |
|------|------|
| 0(默认) | 只要订阅建立成功,就发送保留消息 |
| 1 | 仅在**新建订阅**时发送;**重复订阅**(同主题、同会话)不发送 |
| 2 | 订阅建立时不发送保留消息(只接收后续新发布的) |

**取值 0 的演示:**
```

1. 开启自动重订阅
2. sub 连上(Clean Start=1, Session Expiry=300),向 sub/rh/demo 发保留消息
3. sub 订阅 sub/rh/demo, RH=0 → 立即收到保留消息
4. sub 断开,Clean Start=0 重连,自动重订阅 → 又收到保留消息

```
**取值 1 的演示:**
```

1. 删订阅,重订 sub/rh/demo,RH=1 → 收到保留消息(新订阅)
2. 断开,Clean Start=0 重连(自动复用之前订阅) → 不收到(重复订阅)

```
**取值 2 的演示:**
```

订阅 sub/rh/demo,RH=2 → 不收到保留消息(只能等新发的)

```
### 5.2 共享订阅(Shared Subscription,MQTT 5.0)

#### 5.2.1 普通订阅 vs 共享订阅
```

普通订阅: +------+ | pub  | +--+---+ | publish msg v +-----+         msg | Broker | ----------> sub1 (收 msg) +--+--+--+ ---------->  sub2 (收 msg) sub3 (收 msg) 每个订阅者都收到副本

共享订阅: +------+ | pub  | +--+---+ | publish msg v +-----+ | Broker | +--+--+--+ | 负载均衡选其中一个 v sub1   (收 msg) sub2   (不收) sub3   (不收)

```
#### 5.2.2 优势
- **横向扩展**:多个消费端分担消息处理压力
- **高可用**:一个消费端故障,其他还能继续处理
- **平滑接管**:故障端的未确认消息可被组内其他端重新接管

#### 5.2.3 两种格式

| 格式 | 前缀 | 真实主题示例 |
|------|------|------------|
| 带群组 | `$share/<group>/` | `$share/g1/t/1` 真实主题为 `t/1`,组名 `g1` |
| 不带群组 | `$queue/` | `$queue/t/1` 真实主题为 `t/1` |

**带群组的核心规则:**
- **不同群组 各收一份**(类似消息队列里"消费者组")
- **同一群组内 负载均衡**(只有一个订阅者收到)

例:
```

sub1, sub2, sub3 ∈ g1 → 都订阅 $share/g1/t1 sub4, sub5 ∈ g2 → 都订阅 $share/g2/t1

pub 向 t1 发一条消息:

- g1 收一份 → sub1/sub2/sub3 中**一个**收
- g2 收一份 → sub4/sub5 中**一个**收

```
**不带群组(`$queue/`):** 等价于所有订阅者都在同一个组里。

#### 5.2.4 共享订阅完整 ASCII
            +-----------+
            | publisher |
            +-----+-----+
                  | publish to t/1
                  v
            +-----------+
            |  Broker   |
            +-----+-----+
                  |
   +--------------+--------------+
   |                             |
```

group g1 (收一份)              group g2 (收一份) |                             | +---+----+----+              +----+----+ v        v    v              v         v sub1    sub2  sub3           sub4      sub5 (按算法只有一个收到)         (按算法只有一个收到)

```
#### 5.2.5 负载均衡算法(Dashboard 配置)

| 算法 | 行为 |
|------|------|
| Random | 组内随机选一个 |
| Round Robin(默认) | 组内顺序循环 |
| Hash | 基于某字段哈希(如 clientid)分配,同一来源固定到同一会话 |
| Sticky | 首次随机,之后保持选择;直到该会话结束 |
| Local | 优先选与发布者同节点的会话,无则退化为随机 |

> ⚠️ 原文编号是 1/2/3/4/**6**(跳了 5),正确应是 1-5,Local 排第 5。

#### 5.2.6 演示步骤
1. 创建 4 个连接 sub1-sub4
2. sub1, sub2 订阅 `$share/g1/t/1`;sub3, sub4 订阅 `$share/g2/t/1`
3. pub 向 `t/1` 发两条消息
4. 默认轮询:每条消息 g1、g2 各收一份;g1 内 sub1 与 sub2 轮流收;g2 同理
5. 删订阅,改订 `$queue/t/1`,再发消息观察(所有 sub 在同一组内分流)

### 5.3 排他订阅($exclusive,EMQX 扩展)

允许对一个主题进行**互斥订阅**:同一时刻只允许一个订阅者,直到该订阅者取消订阅。

| 形式 | 前缀 | 真实主题 |
|------|------|---------|
| `$exclusive/t/1` | `$exclusive/` | `t/1` |

**注意:** `$exclusive/t/1` 和 `t/1` **不是同一个主题**。其他客户端仍可通过 `t/1` 普通订阅。

**默认禁用,**需在 **Dashboard → 管理 → MQTT 配置** 中开启。

订阅失败时常见错误码(SUBACK Reason Code):
- `0x97` Quota exceeded
- `0x8F` Topic Filter invalid
- `0x91` Packet Identifier in use

> ⚠️ 原文 5.3.2 演示步骤 3、4 重复且混乱。**正确步骤:**
```

1. Dashboard 中开启排他订阅功能
2. sub1 订阅 $exclusive/t/1 → 成功
3. sub2 订阅 $exclusive/t/1 → 失败(主题已独占)
4. sub2 改订 t/1 → 成功(普通主题不受影响)
5. sub1 取消订阅后,sub2 才能再订 $exclusive/t/1

```
### 5.4 自动订阅(EMQX 扩展)

EMQX 中可在 Dashboard 配置自动订阅规则,客户端连接成功后,Broker **代客户端**进行 SUBSCRIBE,客户端不需要主动发送订阅报文。

**配置入口:** 管理 → MQTT 高级特性 → 自动订阅 → 添加

> ⚠️ 原文演示步骤不完整。**正确流程:**
```

1. Dashboard 配置自动订阅规则,主题 = a/1(可用占位符如 client/${clientid})
2. sub 客户端连接,无需主动 SUBSCRIBE
3. pub 客户端连接
4. pub 向 a/1 发消息
5. sub 自动收到(因为连接成功后已被自动订阅)

```
---

## 第二章 错误清单

| # | 章节 | 原文表述 | 正确说法 |
|---|------|---------|---------|
| 1 | 4.4.1 | `$delay` 作为主题前缀 | 应为 `$delayed` |
| 2 | 4.4.1 | "单位为妙" | 错别字,应为"秒" |
| 3 | 5.1.4 | "清除消息中的 Retain 标识不变" | 自相矛盾,0=清除,1=保持 |
| 4 | 5.1.2 演示 | "通过 sub 客户端...发布消息" | 应为 pub 客户端发布 |
| 5 | 5.2.5 | 算法编号 1/2/3/4/6 | 缺 5,Local 应是第 5 项 |
| 6 | 5.3.2 演示 | 步骤 3、4 重复 "创建 sub2" | 应为先开启功能、再 sub1 订阅成功、sub2 同主题失败、sub2 普通主题成功 |
| 7 | 5.4.2 演示 | 缺少 Dashboard 配置自动订阅规则的步骤 | 必须先在 Dashboard 配置规则 |
| 8 | 4.1.2 | "需重新订阅才会收到保留消息" | 仅指订阅时已存在的保留消息;先订阅后发布会作为普通消息推送 |
| 9 | 3.2.2 | 突然出现的"4、服务端使用 Client ID..." | 编号体系不一致 |
| 10 | 1.4.2 | 只说"消息不丢失原因:与 QoS 1 相同" | 应补充:QoS 2 在 PUBREL/PUBCOMP 阶段同样依赖应答与重传保证可靠性 |

---

# 文档二:第三章 MQTT Dashboard 完整版

## 一、Dashboard 概述

EMQX 内置 Web 管理控制台,功能涵盖集群管理、客户端监控、认证授权、规则引擎、数据集成、日志、告警等。

| 项目 | 默认值 |
|------|------|
| 访问地址 | `http://<ip>:18083` |
| 默认账号 | `admin / public`(首次登录强制修改) |

CLI 重置密码:
```shell
./bin/emqx ctl admins passwd <Username> <Password>
```

------

## 二、访问控制

### 2.1 认证(Authentication)

#### 2.1.1 认证三步走

1. 选择认证方式
2. 选择数据源
3. 配置数据源参数

#### 2.1.2 三种认证方式

| 方式           | 说明                                             | 数据源                                                       |
| -------------- | ------------------------------------------------ | ------------------------------------------------------------ |
| Password-Based | 客户端用 Client ID 或 username + password 认证   | Built-in DB / MySQL / PostgreSQL / MongoDB / Redis / HTTP Server |
| JWT            | 客户端在 username 或 password 字段携带 JWT Token | 无需额外数据源                                               |
| SCRAM          | MQTT 5.0 增强认证,客户端与服务端双向认证         | Built-in Database                                            |

#### 2.1.3 数据源详解

**Built-in Database**

- 简单、零依赖、开箱即用
- 配置:用户标识(用户名 / Client ID)、密码哈希算法(plain / md5 / sha / sha256 / sha512 / pbkdf2 / bcrypt)

**外部数据库(以 MySQL 为例)**

- 必须配置:服务器地址、数据库名、用户名密码

- 关键配置:从数据库取认证数据的 SQL,例如:

  ```sql
  SELECT password_hash, salt FROM mqtt_user WHERE username = ${username} LIMIT 1
  ```

- Docker 部署 MySQL 示例:

  ```bash
  docker run -d -p 3306:3306 \  -v mysql8_conf:/etc/mysql/conf.d \  -v mysql8_data:/var/lib/mysql \  -e MYSQL_ROOT_PASSWORD=1234 \  --name mysql8 --restart=always \  --privileged=true \  mysql:8.0.30
  ```

**HTTP Server**

- 把认证逻辑外包给一个 HTTP 服务
- 配置:请求方法(POST/GET)、URL(协议、端口齐全)、Headers、Body 模板(常用 JSON)
- EMQX 根据 HTTP 响应码与 Body 决定通过/拒绝

#### 2.1.4 认证链(Authentication Chain)

EMQX 允许配置多个认证器,组成"链",在认证列表中可拖动调整顺序。

```
        +-------------------+
Client→ |  CONNECT 报文     |
        +---------+---------+
                  |
                  v
           +------+------+
           | 认证器 #1   |   (例如 Built-in DB)
           +------+------+
                  |
       +----------+     |
    匹配到用户            未匹配到
       |                     |
       v                     v
  密码正确?         +-------+-------+
  +----+----+       | 是末位认证器? |
  |         |       +-------+-------+
  允许      拒绝            |
                  +---|                   |
                  是                  否
                  |                   |
                  v                   v
              拒绝连接          交给下一认证器
```

**关键规则(很多人在这里搞错):**

- 在某认证器**命中且密码不匹配** → **直接拒绝**,不会交给下一认证器
- 只有**未命中**(数据源中查不到)才会交给下一个
- 链尾仍未命中 → 拒绝连接

#### 2.1.5 用户管理

仅对 Built-in Database 数据源生效。**认证列表 → 用户管理** 页可:

- 添加/删除单个用户
- 下载模板,批量导入

#### 2.1.6 客户端连接失败提示

未携带凭证或凭证错误时:

```
Connection refused: Bad User Name or Password
```

### 2.2 授权(Authorization)

#### 2.2.1 概念

认证只验证"你是谁",授权则决定"你能做什么"——具体是对 publish 与 subscribe 的主题级权限控制。

#### 2.2.2 多种授权检查器

- ACL 文件(默认)
- Built-in Database
- MySQL / PostgreSQL / MongoDB / Redis
- HTTP

多检查器组成**授权链**,逻辑与认证链类似:**未命中向下传递,命中即定夺。**

```
Client 发起 publish/subscribe
              |
              v
       +------+-------+
       | 检查器 #1    |
       +------+-------+
              |
        命中规则?
        +----+----+
        是        否
        |         |
        v         v
   allow/deny  下一检查器
              (无下一个时按 no_match 默认值,
               常见 deny 或 allow,生产环境多用 deny)
```

#### 2.2.3 ACL 文件授权

**文件语法:**

- 一条规则用花括号包裹,各元素以逗号分隔
- 每条规则以 `.` 结尾
- `%%` 开头是注释

**规则四要素:**

```
{Permission, Who, Action, [TopicFilters]}.
```

**1) Permission**

- `allow`:允许
- `deny`:拒绝

**2) Who**

| 写法                                       | 含义               |
| ------------------------------------------ | ------------------ |
| `{username, "alice"}` 或 `{user, "alice"}` | 用户名等于 alice   |
| `{username, {re, "^dash"}}`                | 用户名正则匹配     |
| `{clientid, "c01"}` 或 `{client, "c01"}`   | Client ID 等于 c01 |
| `{clientid, {re, "^dev_"}}`                | Client ID 正则匹配 |
| `{ipaddr, "127.0.0.1"}`                    | 单 IP / CIDR       |
| `{ipaddrs, ["10.0.0.1", "10.0.0.2"]}`      | 多 IP 列表         |
| `all`                                      | 任意客户端         |
| `{'and', [...]}`                           | 全部满足           |
| `{'or', [...]}`                            | 任意满足           |

注意:若 EMQX 部署在负载均衡器后,需开启 `proxy_protocol` 才能拿到客户端真实 IP,否则会取到 LB 的 IP。

**3) Action**

| 写法                                     | 含义                    |
| ---------------------------------------- | ----------------------- |
| `publish`                                | 发布                    |
| `subscribe`                              | 订阅                    |
| `all`                                    | 都允许                  |
| `{publish, [{qos, 1}, {retain, false}]}` | 发布 QoS=1 且非保留消息 |
| `{publish, {retain, true}}`              | 发布保留消息            |
| `{subscribe, {qos, 2}}`                  | 以 QoS 2 订阅           |

(从 v5.1.1 起支持检查 QoS、Retain 标志)

**4) Topic Filter**

| 写法              | 含义                              |
| ----------------- | --------------------------------- |
| `"t/${clientid}"` | 占位符,替换为发起请求的 Client ID |
| `"$SYS/#"`        | 通配符匹配所有 $SYS 子主题        |
| `{eq, "foo/#"}`   | 精确匹配字面量 `foo/#`,不展开通配 |

**特殊兜底规则:**

- `{allow, all}`:放行所有
- `{deny, all}`:拒绝所有

**完整示例:**

```erlang
%% 用户名 dashboard 可订阅 $SYS/#
{allow, {username, "dashboard"}, subscribe, ["$SYS/#"]}.

%% 来自 127.0.0.1 的客户端可发布订阅 $SYS/# 与 #
{allow, {ipaddr, "127.0.0.1"}, all, ["$SYS/#", "#"]}.

%% 拒绝其他客户端订阅敏感主题
{deny, all, subscribe, ["$SYS/#", {eq, "#"}, {eq, "+/#"}]}.

%% 兜底:生产环境建议 {deny, all},并配置 authorization.no_match = deny
{allow, all}.
```

**演示:** 在 Dashboard 的 ACL 文件中添加

```
{deny, all, subscribe, ["test/#"]}.
```

保存后任意客户端都无法订阅 `test/#`。

#### 2.2.4 基于 Built-in Database 授权

**优势:** 零依赖,可在 Dashboard UI 中直接增删规则。

**配置步骤:**

1. **访问控制 → 授权 → 添加 → Built-in Database**
2. 创建后,在数据源对应的 **权限管理** 中添加规则
3. 维度可选:Client ID / 用户名 / 主题
4. 字段:权限(允许/拒绝)、操作(发布/订阅/发布与订阅)、主题
5. 同一对象多条规则,可上移/下移调整优先级

#### 2.2.5 ACL vs Built-in DB 对比

| 维度                    | ACL 文件 | Built-in DB                 |
| ----------------------- | -------- | --------------------------- |
| 修改方式                | 编辑文件 | Dashboard UI                |
| 规则查询                | 全表扫描 | 索引(按 Client ID/Username) |
| 大规模规则              | 不友好   | 适合                        |
| 复杂逻辑(正则、IP 范围) | 支持     | 不直接支持                  |
| 默认                    | 是       | 否                          |

------

## 三、黑名单与连接抖动检测

### 3.1 黑名单

EMQX 支持禁止特定客户端访问,封禁手段包括:

- Client ID(精确)
- 用户名(精确)
- IP(精确)
- Client ID 模式(正则)
- 用户名模式(正则)
- IP 范围(CIDR)

**创建步骤:**

1. **访问控制 → 黑名单 → 创建**
2. 选择禁用对象类型 + 值
3. 可选:到期时间、原因
4. 保存

被封禁的客户端在 CONNECT 时会被直接拒绝。

### 3.2 连接抖动检测(Flapping Detect)

#### 3.2.1 目的

**自动**封禁短时间内频繁登录/掉线的客户端,避免它们消耗服务器资源,影响正常客户端。

#### 3.2.2 关键限制

**只封禁 Client ID,不封禁用户名和 IP**。客户端只要更换 Client ID 就能继续登录(对恶意客户端的防护有限)。

#### 3.2.3 默认参数

| 参数         | 默认值 | 含义                     |
| ------------ | ------ | ------------------------ |
| 检测时间窗口 | 1 分钟 | 监测周期                 |
| 最大断连次数 | 15     | 周期内允许的断连次数上限 |
| 封禁时长     | 5 分钟 | 触发后被封时长           |

**默认关闭**,需在 **访问控制 → 连接抖动** 手动启用。

------

## 四、数据集成

### 4.1 数据集成概念

#### 4.1.1 解决的问题

**朴素方案**:每个外部业务系统都自己写一段 MQTT 客户端代码去消费数据 → 重复劳动、维护成本高、数据格式不统一。

**EMQX 方案**:在 Broker 内部内置数据集成能力,通过配置而不是写代码完成数据流转。

#### 4.1.2 整体架构

```
   外部源(Kafka/MQTT/PubSub)        外部目标(MySQL/Redis/HTTP/Kafka...)
            |                                      ^
            v                                      |
        +-------+        +----------+         +-------+
        |Source +-------> Rule      +-------->| Sink  |
        +-------+        | Engine   |         +-------+
                         | (SQL)    |
        MQTT 客户端 ---> | FROM     | -- 内置动作:
        (publish/event)  | SELECT   |    republish/console
                         | WHERE    |    
                         +----+-----+
                              |
                         转换/过滤/路由
```

#### 4.1.3 三个核心组件

| 组件                  | 职责                                                         |
| --------------------- | ------------------------------------------------------------ |
| **Source**            | 从外部数据系统接收消息(MQTT、Kafka、GCP PubSub)              |
| **Sink**              | 将消息发送到外部数据系统(MySQL、Kafka、HTTP、Redis、PostgreSQL...) |
| **连接器(Connector)** | 与外部系统建立连接,1 个连接器可被多个 Source/Sink 复用       |
| **规则引擎(Rule)**    | 用 SQL 描述数据来源、处理过程、处理结果去向                  |

#### 4.1.4 规则三要素

| 要素         | SQL 子句                        | 示例                      |
| ------------ | ------------------------------- | ------------------------- |
| 数据来源     | FROM                            | `FROM "t/#"`(主题或事件)  |
| 数据处理     | SELECT / WHERE / FOREACH / 函数 | `SELECT a, b WHERE c > 0` |
| 处理结果去向 | 动作(Action)                    | republish / 控制台 / Sink |

支持的动作:

- 消息重发布(republish):发布到指定 MQTT 主题
- 控制台输出(console):打到日志
- 发送到 Sink:外部系统

### 4.2 入门:控制台输出

**需求:** 把发往 `t/a` 的消息打印到 EMQX 控制台。

**步骤:**

1. **集成 → 规则 → 创建**

2. 编写 SQL:

   ```sql
   SELECT * FROM "t/a"
   ```

3. 在"动作输出"中添加 **控制台输出**

4. 保存,可点"测试"按钮注入虚拟数据调试

5. MQTTX 向 `t/a` 发消息,查看 EMQX 日志

### 4.3 案例一:MQTT → Redis

**需求:** 把 `t/b` 主题的消息写入 Redis(以 Hash 形式)。

**Redis 部署(Docker):**

```bash
docker run -d -p 6379:6379 --restart=always \
  -v redis_config:/etc/redis/config \
  -v redis_data:/data \
  --name redis redis:7.0.10 \
  redis-server /etc/redis/config/redis.conf
```

**redis.conf 关键内容:**

```conf
appendonly yes
port 6379
requirepass 1234
bind 0.0.0.0
```

**配置步骤:**

1. **集成 → 规则 → 创建**

2. SQL:

   ```sql
   SELECT * FROM "t/b"
   ```

3. 动作输出 → 添加 Sink → 类型选 Redis

4. 配置中点 **+** 添加 Redis 连接器(地址、密码 1234)

5. Redis 命令模板:

   ```
   HSET emqx_messages:${clientid} username ${username} payload ${payload} timestamp ${timestamp}
   ```

6. 保存,可在 Flow 设计器查看拓扑

7. MQTTX 向 

   ```
   t/b
   ```

    发消息,在 Redis 检查:

   ```
   HGETALL emqx_messages:<clientid>
   ```

#### 数据流 ASCII

```
 MQTTX           EMQX                                       Redis
   |               |                                          |
   |--PUBLISH t/b->|                                          |
   |               | Rule SQL: SELECT * FROM "t/b"           |
   |               |                                          |
   |               | render: HSET emqx_messages:${clientid}  |
   |               |       username ${username}              |
   |               |       payload ${payload}                |
   |               |       timestamp ${timestamp}            |
   |               ||---Redis Sink (via Connector)----------->|
   |               | OK
```

### 4.4 案例二:Kafka → EMQX → 控制台 + Redis

**需求:** 从 Kafka 的 `test_mqtt_topic` 取消息,输出到 EMQX 控制台和 Redis。

#### 4.4.1 完整跨组件数据流

```
 SpringBoot           Kafka          EMQX (Rule)          Redis
   App           test_mqtt_topic       Source/Sink
    |                  |                  |                  |
    |--KafkaTemplate-->|                  |                  |
    |   .send()        |--Source pull --->|                  |
    |                  |                  |                  |
    |                  |                  | exec SQL:          | SELECT topic,                  |   offset, value    |                  |
    |                  |                  |--Console log---+  |                  |
    |                  |                  |--Sink HSET------>  |  HSET kafka_mqtt:||  ${topic} offset ||  ${offset} value |
    |                  |                  |  ${value}        |
```

#### 4.4.2 配置步骤

1. Kafka 中创建 `test_mqtt_topic`

2. **集成 → 规则 → 创建**,在"数据输入"添加 Source

3. 添加连接器:Kafka 类型,bootstrap-server 等

4. 在"动作输出"添加两个 Sink:控制台输出、Redis

5. Redis 命令模板:

   ```
   HSET kafka_mqtt:${topic} offset ${offset} value ${value}
   ```

   字段名以控制台输出为准(确认 Source 输出的 schema)。

#### 4.4.3 Spring Boot 生产者(发消息进 Kafka)

**pom 关键依赖:**

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>3.0.5</version>
</parent>
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
  </dependency>
</dependencies>
```

**application.yml:**

```yaml
spring:
  kafka:
    producer:
      bootstrap-servers: 192.168.136.147:9092
```

**启动类:**

```java
@SpringBootApplication
public class MqttKafkaApplication {
  public static void main(String[] args) {
    SpringApplication.run(MqttKafkaApplication.class, args);
  }
}
```

**测试:**

```java
@SpringBootTest(classes = MqttKafkaApplication.class)
public class MqttKafkaProducerTest {
  @Autowired
  private KafkaTemplate<String, String> kafkaTemplate;

  @Test
  public void sendMsg() {
    kafkaTemplate.send("test_mqtt_topic", "mqtt kafka producer msg....");
  }
}
```

发完后在 Redis 中:

```
HGETALL kafka_mqtt:test_mqtt_topic
```

### 4.5 SQL 语法

#### 4.5.1 FROM / SELECT / WHERE

**基本语法:**

```sql
SELECT <字段名> FROM <主题> [WHERE <条件>]
```

**FROM** 选择事件来源:

- 消息发布:写主题(支持通配),如 `FROM "t/#"`
- 事件:写事件主题,如 `$events/client_connected`、`$events/message_dropped`

**SELECT 投影:**

```sql
-- 只输出 a 和 b 字段
SELECT a, b FROM "t/#"

-- 输出全部可用字段(clientid, username, payload, topic, timestamp...)
SELECT * FROM "#" WHERE username = 'abc'

-- as 重命名后,WHERE 中也可使用别名
SELECT clientid as cid FROM "#" WHERE cid = 'abc'

-- WHERE 中可以用所有事件字段,即使没有在 SELECT 投影
SELECT clientid as cid FROM "#" WHERE username = 'abc'

-- 错误:xyz 既非事件字段,也未在 SELECT 中定义
SELECT clientid as cid FROM "#" WHERE xyz = 'abc'  -- 不工作
```

**常用事件字段:**

| 字段        | 含义                        |
| ----------- | --------------------------- |
| `clientid`  | 发布者 Client ID            |
| `username`  | 发布者用户名                |
| `topic`     | 消息主题                    |
| `payload`   | 消息体(JSON 自动解析为对象) |
| `qos`       | QoS 等级                    |
| `retain`    | 是否保留消息                |
| `timestamp` | 消息时间戳                  |
| `node`      | EMQX 节点名                 |

#### 4.5.2 FOREACH / DO / INCASE(数组遍历)

针对 payload 中的数组字段,逐个元素触发动作。

**基本语法:**

```sql
FOREACH <数组字段> [DO <投影>] [INCASE <过滤>] FROM <主题> [WHERE <条件>]
```

| 子句    | 类比 SQL | 作用                              |
| ------- | -------- | --------------------------------- |
| FOREACH | for-each | 遍历数组,每个元素触发一次后续动作 |
| DO      | SELECT   | 对当前元素投影                    |
| INCASE  | WHERE    | 对当前元素过滤                    |

**示例消息(主题 `t/1`,clientid `c_steve`):**

```json
{
  "date": "2024-07-05",
  "sensors": [
    {"name": "a", "idx": 0},
    {"name": "b", "idx": 1},
    {"name": "c", "idx": 2}
  ]
}
```

**示例 1:把每个 sensor 重发布到 `sensors/${idx}`,内容为 `${name}`**

期望产出 3 条消息:

```
sensors/0 → "a"
sensors/1 → "b"
sensors/2 → "c"
```

动作配置:

- 类型:消息重发布
- 目的主题:`sensors/${idx}`
- QoS:0
- 内容模板:`${name}`

SQL:

```sql
FOREACH
  payload.sensors
FROM "t/#"
```

**示例 2:仅 idx ≥ 1 的元素,重发到 `sensors/${idx}`,内容含 clientid 和 date**

```sql
FOREACH
  payload.sensors as e
DO
  clientid,
  payload.date as date,
  e.idx as idx,
  e.name as name
INCASE
  e.idx >= 1
FROM "t/#"
```

输出:

```
sensors/1 → "clientid=c_steve,name=b,date=2024-07-05"
sensors/2 → "clientid=c_steve,name=c,date=2024-07-05"
```

#### 4.5.3 CASE-WHEN

类似 SQL 标准的 CASE。

**示例:把 payload.x 限定在 [0, 7]:**

```sql
SELECT
  CASE WHEN payload.x < 0 THEN 0
       WHEN payload.x > 7 THEN 7
       ELSE payload.x
  END as x
FROM "t/#"
```

输入 `{"x": 8}` → 输出 `{"x": 7}`。

#### 4.5.4 内置 SQL 函数

EMQX 规则引擎提供大量内置函数,常用类别:

| 类别      | 示例                                                         |
| --------- | ------------------------------------------------------------ |
| 数学      | `abs`, `floor`, `ceil`, `round`, `power`, `sqrt`             |
| 类型判断  | `is_null`, `is_array`, `is_map`, `is_str`, `is_num`          |
| 类型转换  | `str`, `str_to_int`, `int`, `float`, `bool`                  |
| 字符串    | `concat`, `substr`, `replace`, `upper`, `lower`, `trim`, `tokens` |
| Map(对象) | `map_get`, `map_put`, `map_keys`, `map_values`               |
| 数组      | `length`, `nth`, `contains`, `head`, `tail`                  |
| 哈希      | `md5`, `sha`, `sha256`, `hash`                               |
| 压缩      | `zip`, `unzip`, `gzip`                                       |
| 编解码    | `base64_encode`, `base64_decode`, `bin2hexstr`, `hexstr2bin` |
| 日期时间  | `now_timestamp`, `now_rfc3339`, `format_date_time`           |
| JSON      | `json_encode`, `json_decode`                                 |

**示例(综合应用):**

```sql
FOREACH
  payload.sensors as e
DO
  abs(-1) as abs_demo,
  concat(e.name, 'xian') as address,
  clientid,
  e.name as name,
  e.idx as idx
INCASE
  e.idx >= 1
FROM "t/1"
```

输入(主题 `t/1`):

```json
{
  "date": "2024-07-05",
  "sensors": [
    {"name": "a", "idx": 0},
    {"name": "b", "idx": 1},
    {"name": "c", "idx": 2}
  ]
}
```

输出(在控制台输出 sink 中可见):

```
{ abs_demo: 1, address: "bxian", clientid: "...", name: "b", idx: 1 }
{ abs_demo: 1, address: "cxian", clientid: "...", name: "c", idx: 2 }
```

### 4.6 Webhook

#### 4.6.1 概念

EMQX 内置的 HTTP 集成方式。客户端事件或消息触发后,EMQX 把数据 POST 到预设的 HTTP 服务器。

#### 4.6.2 数据流

```
 Client          EMQX              External HTTP Server
   ||
   |--PUBLISH a/1->|                       |
   |               | Webhook 触发           |
   |               |---HTTP POST---------->|
   |               |   /webHook/notify     |
   |               |   Body: {topic, payload, clientid, ...}
   |               |                       |
   |               |<--200 OK--------------|
```

#### 4.6.3 Spring Boot 接收端

```java
@RestController
@RequestMapping("/webHook")
public class WebHookController {

  @PostMapping("/notify")
  public void notify(@RequestBody Map<Object, Object> body) {
    System.out.println(body);
  }
}
```

#### 4.6.4 Dashboard 配置

1. **集成 → Webhook → 创建**(实际上 Webhook 在新版本是 HTTP Sink + 规则的组合)
2. 配置 URL、请求方法、Headers、Body 模板
3. 选择触发事件(消息发布/客户端连接/断开等)或主题
4. 保存

#### 4.6.5 测试

MQTTX 向 `a/1` 发消息,Spring Boot 控制台应打印请求体。

------

## 五、日志管理

### 5.1 日志简介

EMQX 日志用于排查客户端连接、网络异常、性能问题等。

**两种输出方式:**

1. 控制台(默认开启)
2. 文件输出

**日志级别(由低到高):**

```
debug < info < notice < warning < error < critical < alert < emergency
```

默认级别:`warning`。

> ⚠️ 原文 "EMQX 只会输出比配置日志级别**高**的日志数据"。**正确**:输出**等于或高于**配置级别的日志(默认 warning 时,warning 级别的日志也会输出,而不是只输出 error 及以上)。

**各级别典型用途:**

| 级别      | 用途                |
| --------- | ------------------- |
| debug     | 开发期详细跟踪      |
| info      | 正常运行信息        |
| notice    | 显著但非异常的事件  |
| warning   | 潜在问题(默认级别)  |
| error     | 一般错误            |
| critical  | 严重错误,功能受影响 |
| alert     | 必须立即处理的错误  |
| emergency | 系统不可用          |

### 5.2 日志配置

**修改保存后立即生效,无需重启节点。** 入口:**管理 → 日志**

#### 5.2.1 控制台日志配置

| 选项       | 默认值  | 可选值                 |
| ---------- | ------- | ---------------------- |
| 启用       | 开      | —                      |
| 日志级别   | warning | debug~emergency        |
| 日志格式   | text    | text / json            |
| 时间戳格式 | auto    | auto / epoch / rfc3339 |
| 时间偏移   | system  | system / 自定义        |

**时间戳格式说明:**

- `auto`:跟随日志格式 — text 用 rfc3339,json 用 epoch
- `epoch`:Unix 微秒精度,如 `1711451539777087`
- `rfc3339`:`2024-03-26T11:52:19.777087+00:00`

#### 5.2.2 文件日志配置

额外参数:

| 选项             | 默认值                   | 说明                              |
| ---------------- | ------------------------ | --------------------------------- |
| 日志文件名       | `/opt/emqx/log/emqx.log` | 路径                              |
| 最大日志文件数   | 10                       | 滚动保留份数                      |
| 日志文件轮换大小 | —                        | 达到该值后轮换;禁用则文件无限增长 |
| 大小单位         | MB                       | KB / MB / GB                      |

文件日志启用后,日志目录会出现:

- `emqx.log.N`:实际日志,如 `emqx.log.1`、`emqx.log.2`
- `emqx.log.siz` 与 `emqx.log.idx`:滚动元数据,**勿手动修改**

### 5.3 日志格式对比

**text 格式:**

```
2024-03-26T11:52:19.777087+00:00 [warning] msg=client_disconnected clientid=c01 reason=keepalive_timeout
```

**json 格式:**

```json
{"time":1711451539777087,"level":"warning","msg":"client_disconnected","clientid":"c01","reason":"keepalive_timeout"}
```

JSON 适合接入 ELK / Loki / Datadog 等日志平台进行结构化分析;text 适合人工查看。

------

## 第三章 错误清单

| #    | 章节         | 原文表述                                        | 正确说法                          |
| ---- | ------------ | ----------------------------------------------- | --------------------------------- |
| 1    | 5.1          | "只输出比配置级别**高**的日志"                  | 应为"等于或高于"(包含等于)        |
| 2    | 第 4 节      | `# ![image-...](...)4 数据集成`(标题与图片混排) | 应分行;标题独立                   |
| 3    | 4.4          | 案例一、案例二编号都是 4.4.1                    | 第二个应为 4.4.2                  |
| 4    | 4.4.1 案例二 | 子步骤编号 5.1 / 5.2 / 5.3 / 5.4 与上下文不连贯 | 应统一为 5.1.1~5.1.4 或重新编号   |
| 5    | 2.2.3        | "通过 Dashboard 配置"前后重复出现               | 段落组织应去重                    |
| 6    | 4.1.2        | 数据集成内部的"工作原理介绍"与图示位置          | 推荐先讲概念再讲架构图,提升可读性 |

------

# 总结建议

如果你后续要在团队内部分享或做面试准备,可以重点强调以下几个**容易踩坑**的点:

1. **QoS 不是发布者一锤定音**:Broker 转发时取 `min(消息 QoS, 订阅声明的最大 QoS)`,可降级
2. **DUP=1 不能用于自动去重**:接收方业务侧需要自己保证幂等
3. **保留消息 ≠ 离线消息**:保留消息是"为新订阅者准备的",离线消息是"会话状态里给同一客户端的"
4. **遗嘱消息不在正常 DISCONNECT 时发送**
5. **会话由 Client ID 唯一标识**,复用必须用相同 Client ID
6. **共享订阅:不同 group 各收一份,组内分流**
7. **认证链:命中后密码不对直接拒绝,不会下沉到下一认证器**
8. **连接抖动检测只封 Client ID**,不封用户名 / IP
9. **EMQX 数据集成的核心是规则引擎 SQL + Sink/Source + 连接器**三件套
10. **日志级别是"≥ 配置级别"**,不是"> 配置级别"

需要进一步展开哪部分(比如把规则引擎事件主题 `$events/...` 全列出来,或加一章 EMQX 集群与桥接),告诉我即可。