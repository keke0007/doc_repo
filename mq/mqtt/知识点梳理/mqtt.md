好的,直接在回复中展示。我把两份文档完整整理在下面,你复制到 markdown 即可。原文的错误我用 ⚠️ 标注并给出正确说法。

------

# 文档一:第二章 MQTT 进阶 知识梳理

## 一、QoS 服务质量

### 1. QoS 概念

MQTT 设备多运行在网络受限环境下,只依赖 TCP 无法完全保证消息可靠到达。MQTT 通过 QoS 机制提供不同等级的可靠性保证。

| 等级  | 语义     | 特征                    |
| ----- | -------- | ----------------------- |
| QoS 0 | 最多一次 | 可能丢失,不存储不重传   |
| QoS 1 | 至少一次 | 不会丢失,可能重复       |
| QoS 2 | 仅一次   | 不丢失也不重复,开销最大 |

QoS 等级由发布者在 PUBLISH 报文中指定。Broker 转发时一般保持原 QoS,但若订阅者声明的最大 QoS 低于消息原有等级,则会**降级**转发(取两者较小值)。

### 2. QoS 0 通信流程

```
发送方                          接收方
  |                              |
  |-------- PUBLISH ----------->|
  |       (即发即弃)            |
  |                              |
  (无确认、无存储、无重传)
```

消息丢失场景:连接关闭/重置时,处于网络链路或 OS 缓冲区中的消息会丢失。

### 3. QoS 1 通信流程

```
发送方                                  接收方
  |                                      |
  |--- PUBLISH (Packet ID=N, DUP=0) --->|
  |   [本地存储,等待 PUBACK]            ||<------ PUBACK (Packet ID=N) ---------|
  |   [收到后删除本地缓存]               |
  |                                      |

若超时未收到 PUBACK,重传:
  |--- PUBLISH (Packet ID=N, DUP=1) --->|
```

消息重复原因:

- 情况A:PUBLISH 未到达接收方 → 重传后接收方实际只收到一次
- 情况B:PUBLISH 已到达,PUBACK 丢失 → 重传后接收方会收到两次

DUP=1 仅表示这是重传报文,接收方不会因此自动去重。

### 4. QoS 2 通信流程(四步握手)

```
发送方--- PUBLISH  (Packet ID=N, QoS=2) -->|
  |   [本地存储 PUBLISH]                |
  |                                      |
  |<------ PUBREC  (Packet ID=N) -------|
  |   [删除 PUBLISH,转存 PUBREL]        |
  |                                      |
  |--- PUBREL   (Packet ID=N) --------->|
  |                                      |    [收到 PUBREL 才视为
  |                                      |     新消息,之前的视为重复]
  |                                      |
  |<------ PUBCOMP (Packet ID=N) -------|
  |   [Packet ID 释放,可复用]            |
```

去重核心:**以 PUBREL 为界**。PUBREL 之前到达的相同 Packet ID 的 PUBLISH 视为重复,之后到达的视为新消息。 重传规则:发送方仅在收到 PUBREC **之前**可以重传 PUBLISH;一旦发出 PUBREL,直到收到 PUBCOMP 之前都不能用此 Packet ID 重传或发新消息。

### 5. QoS 适用场景

| 等级  | 场景                                             |
| ----- | ------------------------------------------------ |
| QoS 0 | 高频但不那么重要的数据,如传感器周期性采样        |
| QoS 1 | 较重要、有实时性要求的指令或状态(需自行处理重复) |
| QoS 2 | 金融、航空等不允许丢失也不允许重复的场景         |

------

## 二、主题详解

### 1. 主题基础

- UTF-8 字符串,使用 `/` 分层,如 `chat/room/1`
- 不建议以 `/` 开头或结尾(语义易歧义)
- 客户端订阅或发布时主题自动创建,无需手动创建/删除

### 2. 通配符规则

**单层通配符 `+`** 必须独占一个层级:

```
+              ✓
test/+         ✓
test/+/temp    ✓
test+          ✗  未独占整个层级
```

`test/+/temperature` 匹配 `test/1/temperature`,不匹配 `test/temperature`(层级缺失)和 `test/bedroom/1/temperature`(层级超出)。

**多层通配符 `#`** 必须独占层级且位于主题末尾:

```
#                  ✓  匹配所有
test/#             ✓
test/bedroom#      ✗  未独占
test/#/temp        ✗  不在末尾
```

`test/#` 可匹配 `test`、`test/temperature`、`test/1/temperature`。

### 3. 系统主题

以 `$SYS/` 开头,用于获取 Broker 自身状态。EMQX 常用示例:

| 主题                                         | 说明           |
| -------------------------------------------- | -------------- |
| `$SYS/brokers`                               | 集群节点列表   |
| `$SYS/brokers/<node>/version`                | EMQX 版本      |
| `$SYS/brokers/<node>/uptime`                 | 运行时间       |
| `$SYS/brokers/<node>/clients/+/connected`    | 客户端上线事件 |
| `$SYS/brokers/<node>/clients/+/disconnected` | 客户端下线事件 |

订阅系统主题需要在 Broker 端开通对应权限。

------

## 三、会话(Session)

### 1. 会话与连接的区别

**会话** ≠ **连接**。会话可以跨多次连接存在,用于保留客户端的订阅、未确认的消息和离线期间到达的消息。

### 2. 关键参数

**Clean Start**(MQTT 5.0,旧版叫 Clean Session)

- `0` / false:复用与该 Client ID 关联的旧会话(订阅、离线消息恢复)
- `1` / true:丢弃旧会话,开启全新会话

**Session Expiry Interval**(秒)

- 未指定或 0:网络断开时会话立即结束
- 大于 0:断开后保留指定秒数,超时则过期
- 0xFFFFFFFF:永不过期

会话由 **Client ID** 唯一标识,复用会话必须使用相同的 Client ID。

### 3. 会话恢复 ASCII 流程(跨多客户端调用)

```
   Publisher        Broker          Subscriber(同一 Client ID)
       |              |                       |
       |              |<-- CONNECT(Clean=1, Expiry=300) --|
       |              |--- CONNACK ----------->|
       |              |<-- SUBSCRIBE topic ----|
       |              |--- SUBACK ------------>|
       |              |                       |
       |--PUBLISH---->|                       |
       |              |--- PUBLISH(转发) ---->|
       |              |                       |
       |              |   [Subscriber 断开]   X
       |--PUBLISH---->|                       
       |              |  [缓存到该会话队列]   
       |--PUBLISH---->|                       
       |              |                       
       |              |<-- CONNECT(Clean=0, 同 ClientID) --|
       |              |--- CONNACK(SP=1) ----->|
       |              |--- 推送离线消息 ------>|- 推送离线消息 ------>|
```

CONNACK 中的 SP(Session Present)标志位表示 Broker 是否找到了已有会话。

------

## 四、消息特性

### 1. 保留消息(Retained Message)

普通消息若发布时无订阅者会被直接丢弃;保留消息会留在 Broker 中,**任何后续的新订阅者**只要订阅匹配的主题,都会立即收到这条最新的保留消息。

发布:PUBLISH 报文中将 Retain 标志设为 1。 注意事项:

- Broker 为每个主题只存一条**最新**的保留消息
- 保留消息**不属于会话状态**,发布者断开后保留消息仍然存在
- 删除方式:发送 Payload 为空的同主题保留消息;或在 Dashboard 删除;或通过 MQTT 5.0 的消息过期间隔自动清理

⚠️ **原文错误纠正**:原文说"在保留消息发布前订阅主题,将不会收到保留消息,需要待保留消息发布后重新订阅"。这个表述有歧义。**正确**说法应是:订阅建立时,只会收到 Broker 中**当时已存在**的保留消息;若订阅在前、保留消息发布在后,该保留消息会作为普通消息被实时推送给已订阅的客户端,无需重新订阅。

### 2. 消息过期间隔(MQTT 5.0)

发布端为有时效性的消息设置 Message Expiry Interval(秒)。若消息在 Broker 中停留超过该值,则不再分发给订阅端。Broker 转发时会更新过期间隔为 `原值 - 已停留时间`。

### 3. 遗嘱消息(Will Message)

作用:感知客户端**意外断开**。客户端在 CONNECT 报文中注册遗嘱消息,当 Broker 检测到该客户端意外断连时,向遗嘱主题发布该消息。

可设置字段:Will Topic、Will Retain、Will QoS、Will Properties、Will Payload。 专属属性:**Will Delay Interval**(秒) — 网络断开后延迟多久发布遗嘱;若客户端在延迟期内重连成功,遗嘱不会发布。 若未指定 Will Delay Interval 或为 0,断开时立即发布。 遗嘱消息属于会话状态;若延迟期内会话过期或客户端用 Clean Start=1 重连,Broker 必须立即发布遗嘱(不再等待)。

```
   Client A         Broker          Client B(订阅遗嘱主题)
      |               |                    |
      |--CONNECT (Will Topic, Will Delay=5)-->|
      |               |--CONNACK---------->|
      |               |<-- SUBSCRIBE ------|
      |               |                    |
      |   X 意外断开  |                    |
      |               | [启动 5s 倒计时]   |
      |               | [5s 内未重连]      |
      |               |---发布遗嘱消息---->|

正常断开(DISCONNECT)不会触发遗嘱消息。
```

### 4. 延迟发布(EMQX 扩展)

主题格式:`$delayed/{DelayInterval}/{TopicName}`,Broker 收到后延迟指定秒数再转发。

举例:

- `$delayed/15/x/y` :15 秒后发布到 `x/y`
- `$delayed/60/a/b` :60 秒后发布到 `a/b`

最大延迟 4294967 秒。如果 `{DelayInterval}` 不能被解析为整数,EMQX 直接丢弃该消息。

⚠️ **原文错误纠正**:

- 原文"使用 `$delay` 作为主题前缀" → **正确前缀是 `$delayed`**(多了个 `ed`)
- 原文"单位为妙" → **错别字**,应为"秒"

### 5. 用户属性(User Property,MQTT 5.0)

Key-Value 形式的附加元数据,可在 PUBLISH、SUBSCRIBE、CONNECT、DISCONNECT 等报文中携带,语义上类似 HTTP 请求头。常用于日志记录、消息分类与标签。

------

## 五、订阅详解

### 1. 订阅四个选项(MQTT 5.0)

| 选项                | 取值  | 含义                               |
| ------------------- | ----- | ---------------------------------- |
| QoS                 | 0/1/2 | 服务端转发消息时允许使用的最大 QoS |
| No Local            | 0/1   | 是否禁止把消息回投给发布者本人     |
| Retain As Published | 0/1   | 转发时是否保留消息原 Retain 标志   |
| Retain Handling     | 0/1/2 | 订阅建立时是否发送已有保留消息     |

#### 1.1 QoS 选项

- 服务端支持的最大 QoS < 客户端请求 QoS:Broker 在 SUBACK 中告知最终授予的 QoS,订阅方自行判断是否继续
- 客户端订阅 QoS < 消息发布 QoS:Broker 转发时降级

#### 1.2 No Local

- `0`(默认):允许把消息转发给发布者
- `1`:不转发给发布者 主要用于**桥接场景**,避免两个 Broker 互相订阅 `#` 主题时形成转发风暴。

#### 1.3 Retain As Published

- `0`(默认):转发时**清除**消息中的 Retain 标志
- `1`:转发时**保持**原 Retain 标志

⚠️ **原文错误纠正**:原文写"0(默认值):服务端在向此订阅转发应用消息时需要`清除`消息中的 Retain 标识**不变**" — 措辞自相矛盾(同时出现"清除"和"不变")。正确含义:`0` 是清除,`1` 是保持。

桥接场景下应将该选项设为 1,否则跨桥接的保留消息会丢失"保留"语义。

#### 1.4 Retain Handling

- `0`(默认):订阅建立时立即发送已有保留消息
- `1`:**仅在新订阅(非重复订阅)** 建立时发送
- `2`:订阅建立时**不**发送保留消息

### 2. 共享订阅($share / $queue)

普通订阅每个匹配客户端都会收到消息副本;共享订阅在订阅组内**负载均衡**地分发。

| 形式     | 前缀              | 真实主题 | 说明                         |
| -------- | ----------------- | -------- | ---------------------------- |
| 带群组   | `$share/<group>/` | `t/1`    | 不同 group 各收一份,组内分流 |
| 不带群组 | `$queue/`         | `t/1`    | 等价于所有订阅者在同一组     |

```
                    +--------------+
                    |  Publisher   |
                    +------+-------+
                           | publish msg → t/1
                           v
                    +--------------+
                    |   Broker     |
                    +------+-------+
                           |
              +------------+------------+
              |                         |
       group: g1 (副本×1)        group: g2 (副本×1)
              |                         |
        +-----+-----+             +-----+-----+
        v           v             v           v
       sub1        sub2          sub3        sub4
       (二选一接收)              (二选一接收)
```

负载均衡算法(Dashboard 可配):

1. random — 随机
2. round_robin — 轮询(默认)
3. hash — 基于字段哈希
4. sticky — 粘性,选定后维持
5. local — 优先选与发布者同节点的会话

⚠️ **原文错误纠正**:原文负载均衡算法编号是 1、2、3、4、**6**,跳过了 5。本地优先(Local)应为第 5 项。

### 3. 排它订阅($exclusive)

EMQX 扩展,主题前缀 `$exclusive/`。同一时刻只允许一个订阅者,后来的订阅请求会失败,直到原订阅者取消。 **注意**:`$exclusive/t/1` 和 `t/1` 不是同一个主题,其他客户端仍可通过 `t/1` 订阅。 默认在 Dashboard 中是关闭的,需要手动开启。误纠正**:原文 5.3.2 演示步骤 3 和 4 内容混乱重复("3、在sub2客户端连接中重新添加..."、"4、创建sub2客户端连接...")。**正确流程**应为:

```
1. 在 Dashboard 开启排他订阅功能
2. sub1 订阅 $exclusive/t/1     → 成功
3. sub2 订阅 $exclusive/t/1     → 失败(主题已被独占)
4. sub2 改为订阅 t/1            → 成功(普通主题不受影响)
5. sub1 取消订阅后,sub2 才能再订阅 $exclusive/t/1
```

### 4. 自动订阅

EMQX 在 Dashboard 配置规则,客户端连接成功后自动订阅指定主题,无需客户端主动发起 SUBSCRIBE。配置入口:**管理 → MQTT 高级特性 → 自动订阅 → 添加**。

⚠️ **原文错误纠正**:原文 5.4.2 演示步骤不完整,缺少配置自动订阅规则的关键步骤。**正确演示流程**:

```
1. Dashboard 配置自动订阅规则:主题 = a/1
2. sub 客户端连接到 Broker(无需主动订阅)
3. pub 客户端连接到 Broker
4. pub 向 a/1 发布消息
5. sub 自动收到该消息(因连接时已被自动订阅 a/1)
```

------

## 第二章 错误清单汇总

| 序号 | 章节       | 原文表述                               | 修正                                                    |
| ---- | ---------- | -------------------------------------- | ------------------------------------------------------- |
| 1    | 4.4.1      | `$delay` 作为主题前缀                  | 应为 `$delayed`                                         |
| 2    | 4.4.1      | "单位为妙"                             | 错别字,应为"秒"                                         |
| 3    | 5.1.4      | "清除消息中的 Retain 标识不变"         | 自相矛盾,0 = 清除,1 = 保持                              |
| 4    | 5.1.2 演示 | 步骤2 "通过 sub 客户端...发布消息"     | 应为 pub 客户端发布                                     |
| 5    | 5.2.5      | 算法编号 1/2/3/4/6                     | 缺少 5,Local 应是 5                                     |
| 6    | 5.3.2 演示 | 步骤 3、4 重复混乱                     | 见上文修正流程                                          |
| 7    | 5.4.2 演示 | 缺少配置自动订阅规则的步骤             | 见上文修正流程                                          |
| 8    | 4.1.2      | "需要重新订阅才会收到保留消息"         | 仅指订阅时已有保留消息;先订阅后发布时会作为普通消息推送 |
| 9    | 3.2.2      | 突然出现的"4、服务端使用 Client ID..." | 编号体系不一致,应单独成段或归并                         |

------

# 文档二:第三章 MQTT Dashboard 知识梳理

## 一、Dashboard 概述

EMQX 内置 Web 管理控制台,默认地址 `http://<ip>:18083`,首次登录账号 `admin / public`。

CLI 重置密码:

```shell
./bin/emqx ctl admins passwd <Username> <Password>
```

------

## 二、访问控制

### 1. 认证(Authentication)

**作用**:验证客户端身份。EMQX 提供三种认证方式:

| 方式                     | 数据源                                                       |
| ------------------------ | ------------------------------------------------------------ |
| Password-Based           | Built-in Database / MySQL / PostgreSQL / MongoDB / Redis / HTTP Server |
| JWT                      | 无需额外数据源                                               |
| SCRAM(MQTT 5.0 增强认证) | Built-in Database                                            |

#### 认证链(多文件/多源调用)流程

```
         +-------------------+
Client → |  CONNECT 报文     |
         +---------+---------+
                   |
                   v
            +------+------+
            | 认证器 #1   |  (例如 Built-in DB)
            +------+------+
                   |
        +----------+----------+
        |                     |
   命中 user                 未命中
        |                     |
        v                     v
   密码匹配?           +------+------+
        |              | 认证器 #2   |  (例如 HTTP Server)
   是 / 否                              
   允许 / 拒绝          +-----+-----+
                        | 是末位?    |
                        +-----+-----+
                              |
                       是 → 拒绝连接
                       否 → 移交下一认证器
```

要点:

- 在某认证器命中且**密码匹配**则放行,**密码不匹配**则直接拒绝(**不会**继续走下一认证器)
- 在某认证器**未命中**才会移交下一认证器
- 链尾仍未命中 → 拒绝连接

#### 用户管理

对于使用 Built-in Database 的认证器,可在认证列表 → 用户管理批量导入/添加用户名与密码。

#### 客户端连接失败提示

未携带认证信息或认证不通过时返回:`Connection refused: Bad User Name or Password`。

### 2. 授权(Authorization)

**作用**:验证已认证客户端对指定主题的发布/订阅权限。

EMQX 支持的检查器:ACL 文件(默认)、Built-in Database、MySQL、PostgreSQL、MongoDB、Redis、HTTP。多个检查器组成**授权链**,顺序依次匹配。

#### ACL 文件规则结构

```
{Permission, Who, Action, [TopicFilters]}.
```

- Permission:`allow` | `deny`
- Who:`{username, "x"}` / `{clientid, "x"}` / `{ipaddr, "x"}` / `{ipaddrs, [...]}` / `all` / `{'and', [...]}` / `{'or', [...]}`
- Action:`publish` / `subscribe` / `all`,从 v5.1.1 起支持附带 `qos` 与 `retain` 限定
- TopicFilters:支持通配符与占位符 `${clientid}` `${username}`,使用 `{eq, "..."}` 表示精确匹配(不展开通配)

ACL 文件解析规则:`%%` 开头是注释,每条规则以 `.` 结尾,自上而下首条匹配生效。

```
%% 用户名 dashboard 可订阅 $SYS/#
{allow, {username, "dashboard"}, subscribe, ["$SYS/#"]}.

%% 来自 127.0.0.1 的客户端可发布订阅 $SYS/# 与 #
{allow, {ipaddr, "127.0.0.1"}, all, ["$SYS/#", "#"]}.

%% 拒绝其他客户端订阅敏感主题
{deny, all, subscribe, ["$SYS/#", {eq, "#"}, {eq, "+/#"}]}.

%% 兜底放行(生产环境通常改为 deny all)
{allow, all}.
```

#### 授权检查流程

```
Client 请求 publish/subscribe
         |
         v
+--------+---------+
|  授权检查器 #1   |
+--------+---------+
         |
   规则匹配?
   +-----+-----+
   |           |
  是          否
   |           |
   v           v
allow/deny   下一检查器
            (无下一个时按 no_match 默认值处理)
```

### 3. 黑名单

封禁手段:客户端 ID、用户名、IP、Client ID 模式(正则)、用户名模式(正则)、IP 范围(CIDR)。可设置到期时间和原因。 入口:**访问控制 → 黑名单 → 创建**。

### 4. 连接抖动检测

针对短时间内频繁连接/断开的客户端进行临时封禁。**只封禁 Client ID,不封禁用户名和 IP**(更换 Client ID 仍可登录)。 默认参数:

- 检测时间窗口:1 分钟
- 最大断连次数:15
- 封禁时长:5 分钟

------

## 三、数据集成

### 1. 整体架构(多组件协作)

```
            +---------------------+
            |  外部源 / MQTT 客户端 |
            +----------+----|
                       v
   +-------+      +----+----+      +-------+
   |Source |----> |  Rule   |----> | Sink  |
   +-------+      | Engine  |      +-------+
                  |  (SQL)  |          |
                  +----+----+          |
                       |               v
                  内置动作:       外部系统:
              +------+-------+   MySQL/Kafka
              | republish    |   Redis/HTTP
              | console log  |   PostgreSQL
              +--------------+   ...
```

- **Source**:从外部系统接收消息(MQTT、Kafka、GCP PubSub 等)
- **Sink**:将消息发送到外部系统(MySQL、Kafka、HTTP 等)
- **连接器**:负责与外部系统建立连接,一个连接器可被多个 Source/Sink 复用
- **规则引擎**:基于 SQL,描述数据来源、处理过程、处理结果去向

### 2. 规则三要素

- **数据来源**:消息主题或事件主题(SQL `FROM`),也可是 Source
- **数据处理**:`SELECT` / `WHERE` / `FOREACH` / SQL 函数
- **结果去向**:消息重发布、控制台输出、各类 Sink

### 3. 数据流跨组件 ASCII(以 Kafka → Redis 为例)

```
Spring Boot App     Kafka          EMQX (Rule)         Redis
     |                |                |                  |
     |--produce------>|                |                  |
     |  test_mqtt_topic                |                  |
     |                |--Source pull-->|                  |
     |                |                |  exec SQL          SELECT topic,|  offset, value                   |---Console log--+
     |                |                |---Sink HSET----->  HSET kafka_mqtt:
     |                |                |   ${topic} offset${offset} value
     |                |                |   ${value}
```

### 4. SQL 语法

#### FROM/SELECT/WHERE

```sql
SELECT <字段名> FROM <主题> [WHERE <条件>]
```

要点:

- WHERE 中可用消息事件中的所有可用字段(clientid、username 等),即使没在 SELECT 中投影
- WHERE 中也可使用 SELECT 中通过 `as` 定义的别名
- WHERE 中不能使用既非事件字段、又未在 SELECT 定义的变量

#### FOREACH-DO-INCASE(数组遍历)

```sql
FOREACH <数组字段> [DO <投影>] [INCASE <过滤>] FROM <主题> [WHERE <条件>]
```

| 子句    | 类比   | 作用                              |
| ------- | ------ | --------------------------------- |
| FOREACH | for    | 遍历数组,每个元素触发一次后续动作 |
| DO      | SELECT | 对当前元素投影出感兴趣字段        |
| INCASE  | WHERE  | 对当前元素过滤                    |

示例消息:

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

把 idx ≥ 1 的对象重发布到 `sensors/${idx}`:

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

#### CASE-WHEN

```sql
SELECT
  CASE WHEN payload.x < 0 THEN 0
       WHEN payload.x > 7 THEN 7
       ELSE payload.x
  END as x
FROM "t/#"
```

#### 内置函数

数学、类型转换、字符串、数组、哈希、压缩、位操作、编解码、日期等。常用如 `abs(-1)`、`concat(s1, s2)`、`str_to_int(...)`。 官方参考:`https://docs.emqx.com/zh/emqx/latest/data-integration/rule-sql-builtin-functions.html`

### 5. Webhook

原理:客户端事件或消息触发后,EMQX 将事件数据 POST 到预设的 HTTP 服务器。

```
Client → topic a/1 → EMQX → Webhook 触发 → HTTP POST → /webHook/notify
```

Spring Boot 接收端示例:

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

------

## 四、日志管理

### 1. 输出方式

- **控制台输出**(默认开启)
- **文件输出**

### 2. 日志级别(由低到高)

```
debug < info < notice < warning < error < critical < alert < emergency
```

默认级别:`warning`。EMQX 会输出**等于或高于**配置级别的日志。

⚠️ **原文错误纠正**:原文"EMQX 只会输出比配置日志级别高的日志数据" → 应为"**等于或高于**配置日志级别的日志"(默认 warning 时,warning 本身的日志也会输出)。

### 3. 配置要点

- Dashboard 修改保存后**立即生效,无需重启**
- 控制台日志:级别、格式(text/json)、时间戳格式(auto/epoch/rfc3339)、时区偏移
- 文件日志额外项:文件名(默认 `/opt/emqx/log/emqx.log`)、最大文件数(默认 10)、轮换大小(MB/GB/KB)
- 时间戳格式 `auto`:text → rfc3339,json → epoch
- 文件日志启用后,日志目录会出现 `emqx.log.N`(实际日志)以及 `emqx.log.siz` 与 `emqx.log.idx`(滚动元数据,**不要手动改**)

------

## 第三章 错误清单汇总

| 序号 | 章节         | 原文表述                                            | 修正                               |
| ---- | ------------ | --------------------------------------------------- | ---------------------------------- |
| 1    | 5.1 日志简介 | "只会输出比配置日志级别**高**的日志"                | 应为"等于或高于配置级别"(包含等于) |
| 2    | 第 4 节标题  | `# ![image-...](...)4 数据集成`(标题与图片混在一行) | 标题与图片应各自独占一行           |
| 3    | 4.4 节编号   | "案例一" 4.4.1,"案例二" 也写为 4.4.1                | 第二个应为 **4.4.2**               |
| 4    | 4.4.1 案例二 | "5.1、创建一个基于spring boot 3.0.5..." 编号体系    | 应统一为 5.1 / 5.2 / 5.3 / 5.4     |
| 5    | 2.2.3        | "通过 Dashboard 配置:" 这句子前后重复出现两次       | 段落组织应去重                     |

------

需要我补充某一节(比如把 QoS 2 的 Packet ID 释放再细化、或把规则引擎 SQL 用更多例子展开),告诉我章节号即可。