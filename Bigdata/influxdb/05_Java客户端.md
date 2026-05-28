# 05 Java 客户端(influxdb-client-java)

> 覆盖原文档:第 11 章 JAVA 操作 InfluxDB
> 前置:已读 `04_HTTP_API_与命令行工具.md`(SDK 内部就是 HTTP)
> 官方仓库:https://github.com/influxdata/influxdb-client-java

---

## 一、依赖与项目结构

### 1.1 Maven 依赖

```xml
<dependencies>
    <dependency>
        <groupId>com.influxdb</groupId>
        <artifactId>influxdb-client-java</artifactId>
        <version>6.5.0</version>
    </dependency>
</dependencies>
```

> 💡 选版本时注意:`influxdb-client-java` 的版本跟 InfluxDB **服务端版本不必严格一致**,但要在 client 的兼容范围内。截至 2026,该 client 已迭代到 7.x,**支持 OSS 2.x / Cloud 2 / Cloud 3**。

### 1.2 SDK 本质

```
┌──────────────────────────────────────────────────────┐
│       influxdb-client-java(6.5.0)                    │
│                                                       │
│   ┌──────────────────────────────────────────────┐   │
│   │   InfluxDBClient(顶层入口)                  │   │
│   │   ├─ getQueryApi() / getQueryApi()           │   │
│   │   ├─ getWriteApiBlocking() / makeWriteApi() │   │
│   │   ├─ getBucketsApi() / getOrganizationsApi()│   │
│   │   ├─ ping() / health()                       │   │
│   │   └─ ...                                     │   │
│   └──────────────────────────────────────────────┘   │
│                       │                              │
│                       ▼                              │
│              OkHttp + Retrofit                       │
│                       │                              │
│                       ▼                              │
│              HTTP Authorization: Token …             │
└──────────────────────────────────────────────────────┘
                       │
                       ▼
                  InfluxDB HTTP API
```

---

## 二、创建客户端的 4 种重载

```java
// 1) 仅 URL — 只能调用不需要鉴权的接口(如 ping)
InfluxDBClientFactory.create("http://localhost:8086");

// 2) URL + Token — 全权操作,需在调用具体 API 时指定 bucket/org
InfluxDBClientFactory.create(url, token);

// 3) URL + Token + Org + Bucket — 锁定到某个 bucket,最常用
InfluxDBClientFactory.create(url, token, org, bucket);

// 4) createV1(url, username, password, database, retentionPolicy)
//    兼容 InfluxDB 1.x 风格的接入(用户名密码 + database)
InfluxDBClientFactory.createV1(url, "tony", "11111111".toCharArray(),
                               "my_db", "autogen");
```

> ⚠️ **重要**:Token 参数类型是 `char[]`,**不是 String**。原因是 JVM 中 String 进入常量池后无法被显式清零,而 `char[]` 用完可以 `Arrays.fill(token, '\0')` 抹除。生产代码应抓住这一点,减少 Token 在堆内存中残留时间。

```java
private static char[] token = "ZA8u...PT3w==".toCharArray();   // 注意 .toCharArray()
```

---

## 三、健康检查:`ping()` vs `health()`

```java
InfluxDBClient client = InfluxDBClientFactory.create("http://localhost:8086");

System.out.println(client.ping());     // 返回 boolean,推荐
System.out.println(client.health());   // 返回 HealthCheck 对象,已废弃
```

| 方法 | 返回 | 状态 |
| --- | --- | --- |
| `ping()` | `boolean` | ✅ 推荐 |
| `health()` | `HealthCheck` | ⚠️ Deprecated(2.0+) |

> ⚠️ 原文 11.4.5 说"返回 HealthCheck 对象,相对而言对这个对象的处理比直接处理布尔值要麻烦很多" — 还要再补一点:**`health()` 对应的服务端端点 `/health` 已经从 2.x 中移除/废弃**,某些版本直接 404。生产代码必须用 `ping()`。

---

## 四、查询数据

### 4.1 两种查询方法

```java
QueryApi queryApi = client.getQueryApi();

// 方式 A:结构化对象(List<FluxTable>)
List<FluxTable> result = queryApi.query(
    "from(bucket:\"test_init\") |> range(start:-2m)");

// 方式 B:原始 CSV 字符串
String csv = queryApi.queryRaw(
    "from(bucket:\"test_init\") |> range(start:-2m)");
```

| 方法 | 返回 | 适用 |
| --- | --- | --- |
| `query(...)` | `List<FluxTable>` 或自定义 POJO(`query(flux, MyPojo.class)`) | 结构化处理,大多数场景 |
| `queryRaw(...)` | `String`(原始 CSV) | 透传给前端 / 写文件 / 调试 |

### 4.2 FluxTable 模型

```
List<FluxTable>            ← 一个表流
   │
   └─ FluxTable            ← 一张子表 = 一个 series 组
        │
        ├─ getGroupKey()   → List<FluxColumn>  ← 哪些列定义了分组
        │                                       (通常是 _start,_stop,_field,_measurement)
        │
        ├─ getColumns()    → List<FluxColumn>  ← 全部列定义
        │
        └─ getRecords()    → List<FluxRecord>  ← 行
              │
              └─ FluxRecord
                   ├─ getTime()
                   ├─ getValue()
                   ├─ getMeasurement()
                   ├─ getField()
                   ├─ getValueByKey("tagName")
                   └─ getValues() → Map<String, Object>
```

### 4.3 完整查询示例(流程图)

```
┌──────────────────────────────────────────────────────────────┐
│  ExampleQuery.main()                                         │
│      │                                                        │
│      ▼                                                        │
│  InfluxDBClientFactory.create(url, token, org, bucket)       │
│      │                                                        │
│      ▼                                                        │
│  InfluxDBClient.getQueryApi() ──▶ QueryApi                   │
│      │                                                        │
│      ▼                                                        │
│  queryApi.query(fluxScript)                                  │
│      │                                                        │
│      │ 内部                                                   │
│      ▼                                                        │
│  POST /api/v2/query?org=xxx                                  │
│  Authorization: Token xxx                                    │
│  Content-Type: application/json                              │
│  Body: { "query": "...", "type": "flux" }                    │
│      │                                                        │
│      ▼                                                        │
│  返回 application/csv                                        │
│      │                                                        │
│      ▼                                                        │
│  influxdb-client-java 解析 CSV                               │
│      │                                                        │
│      ▼                                                        │
│  List<FluxTable> (一张子表 = 一个 series 组)                │
└──────────────────────────────────────────────────────────────┘
```

---

## 五、写入数据 — 同步(WriteApiBlocking)

### 5.1 三种写入入口

```java
WriteApiBlocking writeApi = client.getWriteApiBlocking();

// (A) 行协议字符串
writeApi.writeRecord(WritePrecision.NS,
    "temperature,location=west value=60.0");

// (B) Point 对象(SDK 提供的 builder)
Point p = Point.measurement("temperature")
    .addTag("location", "west")
    .addField("value", 55D)
    .time(Instant.now(), WritePrecision.MS);
writeApi.writePoint(p);

// (C) POJO 注解
Temperature t = new Temperature();
t.location = "west"; t.value = 40D; t.time = Instant.now();
writeApi.writeMeasurement(WritePrecision.NS, t);
```

每个方法都有带 `s` 的批量版本:`writeRecords` / `writePoints` / `writeMeasurements`。

### 5.2 POJO 注解模型

```java
@Measurement(name = "temperature")
public class Temperature {

    @Column(tag = true)         // 这个字段是 tag
    String location;

    @Column                     // 默认为 field
    Double value;

    @Column(timestamp = true)   // 这个字段是时间戳
    Instant time;
}
```

`@Column` 的可用属性:

| 属性 | 含义 |
| --- | --- |
| `tag = true` | 该字段作为 tag |
| `timestamp = true` | 该字段作为时间戳 |
| `name = "..."` | 自定义列名(默认用 Java 字段名) |
| (都不设) | 该字段作为 field |

### 5.3 同步写入的执行流程

```
ExampleWriteSync.main()
      │
      ▼
InfluxDBClient ─▶ WriteApiBlocking
      │
      ▼
writeApi.writePoint(p)   ─┐
                          │ 同步阻塞
                          ▼
                 POST /api/v2/write?org=xxx&bucket=xxx&precision=ns
                 Authorization: Token xxx
                 Content-Type: text/plain
                 Body: <Line Protocol 文本>
                          │
                          ▼
                    HTTP 204 No Content
                          │
                          ▼
                    方法返回(成功)
                          │
                          ▼
                    继续主线程
```

**特性**:
- 调用线程**阻塞**到 HTTP 响应回来
- 失败立即抛异常(`InfluxException`)
- 适合**对写入实时性要求高 / 数据量小**的场景

---

## 六、写入数据 — 异步(WriteApi)

### 6.1 异步模型

```
应用线程                                  守护线程(由 SDK 维护)
   │                                             │
   │  writeApi.writeRecord(...)                 │
   │  ────────▶ 放入本地 Buffer  ────▶          │
   │            (立即返回,不阻塞)               │
   │                                             │ 触发条件:
   ▼                                             │   • Batch Size 满(默认 1000)
应用继续干别的                                   │   • Flush Interval 到(默认 1s)
                                                  │   • 显式调用 flush() / close()
                                                  ▼
                                          HTTP POST /api/v2/write
                                                  │
                                                  ▼
                                          失败 → 自动重试(指数退避)
                                                  │
                                                  ▼
                                          OK / 永久失败
```

### 6.2 获取 API

```java
// 已废弃,内部委托给 makeWriteApi
WriteApi writeApi = client.getWriteApi();

// 推荐
WriteApi writeApi = client.makeWriteApi();
WriteApi writeApi = client.makeWriteApi(writeOptions);   // 自定义参数
```

### 6.3 容易踩的坑:**进程立刻退出,数据没刷出**

```java
// ❌ 错误示范:进程立刻退出,缓冲区数据被丢
public static void main(String[] args) {
    WriteApi writeApi = client.makeWriteApi();
    writeApi.writeRecord(WritePrecision.NS, "temperature,location=north value=60.0");
    // 没 flush 也没 close,缓冲区里那一条数据来不及发就被主线程退出 → JVM 退出 → 数据丢
}
```

**正确做法**(两种):

```java
// 方式 1:显式 flush
writeApi.flush();

// 方式 2:close(更稳妥,会先 flush 再关连接池)
client.close();        // 关 InfluxDBClient → 触发 WriteApi.close() → flush
```

> 💡 最佳实践:`try (InfluxDBClient client = InfluxDBClientFactory.create(...)) { ... }` 用 try-with-resources。

### 6.4 默认配置(WriteOptions)

| 配置 | 默认值 | 含义 |
| --- | --- | --- |
| `DEFAULT_BATCH_SIZE` | 1000 | 单批次最大点数 |
| `DEFAULT_FLUSH_INTERVAL` | 1000(ms) | 距离上次刷写超过 1s 也刷出 |
| `DEFAULT_BUFFER_LIMIT` | 10000 | 整个缓冲区上限,**超出会丢点或阻塞**(取决于 backpressure 策略) |
| `DEFAULT_RETRY_INTERVAL` | 5000(ms) | 重试初始等待 |
| `DEFAULT_MAX_RETRIES` | 5 | 最多重试 5 次 |
| `DEFAULT_EXPONENTIAL_BASE` | 2 | 指数退避基数 |
| `DEFAULT_MAX_RETRY_DELAY` | 125,000(ms) | 单次重试最大等待 ~2 分钟 |
| `DEFAULT_MAX_RETRY_TIME` | 180,000(ms) | 总重试时间上限 ~3 分钟 |
| `DEFAULT_JITTER_INTERVAL` | 0 | 抖动,避免多 client 同时刷出 |

自定义:

```java
WriteOptions options = WriteOptions.builder()
    .batchSize(5000)
    .flushInterval(2000)
    .bufferLimit(50_000)
    .build();

WriteApi writeApi = client.makeWriteApi(options);
```

### 6.5 错误处理(异步)

异步写入失败**不会抛到调用线程**(因为调用早已返回)。需要监听:

```java
writeApi.listenEvents(WriteErrorEvent.class,
    event -> log.error("write failed", event.getThrowable()));

writeApi.listenEvents(WriteSuccessEvent.class,
    event -> log.info("wrote {} bytes", event.getLineProtocol().length()));
```

> ⚠️ 这一点原讲义没有提到 — **生产环境异步写入必须挂监听器**,否则失败默默丢点。

### 6.6 同步 vs 异步对比

```
                   ┌──────────────────────────────────────┐
       ┌─────────▶ │  WriteApiBlocking(同步)             │
       │           │   • 阻塞,失败立即抛异常              │
       │           │   • 一次请求一个点 / 一批             │
       │           │   • 适合:补数、调试、低吞吐写入     │
       │           └──────────────────────────────────────┘
   选择 ─┤
       │           ┌──────────────────────────────────────┐
       └─────────▶ │  WriteApi(异步)                     │
                   │   • 立即返回,后台守护线程批量发送   │
                   │   • 自动重试 + 退避                  │
                   │   • 失败需挂监听器才能感知           │
                   │   • 适合:高吞吐生产写入             │
                   │   • 进程退出前必须 flush / close     │
                   └──────────────────────────────────────┘
```

---

## 七、V1 兼容 API

```java
// InfluxDB 2.x 兼容 1.x 客户端的 createV1 入口
InfluxDBClient v1 = InfluxDBClientFactory.createV1(
    "http://localhost:8086",
    "tony",                           // 用户名
    "11111111".toCharArray(),         // 密码
    "my_db",                          // 数据库名(2.x 中映射到 bucket)
    "autogen"                         // 保留策略
);
```

底层映射:

```
1.x:  database + retention policy        2.x:  bucket
      ↓                                          ↑
                  DBRP 映射表(/api/v2/dbrps)
                  (在 2.x 中定义 database/rp → bucket 的映射)
```

> 💡 用 V1 兼容 API 之前,**要先在 2.x 中创建对应的 DBRP 映射**,否则 1.x 风格的 `database` 不知道指向哪个 bucket。

---

## 八、常用 API 速查

| InfluxDBClient 方法 | 返回 | 用途 |
| --- | --- | --- |
| `ping()` | `boolean` | 健康检查 |
| `getQueryApi()` | `QueryApi` | FLUX 查询 |
| `getInfluxQLQueryApi()` | `InfluxQLQueryApi` | InfluxQL 兼容查询 |
| `getWriteApiBlocking()` | `WriteApiBlocking` | 同步写 |
| `makeWriteApi()` / `makeWriteApi(opts)` | `WriteApi` | 异步写 |
| `getBucketsApi()` | `BucketsApi` | 创建/列出 bucket |
| `getOrganizationsApi()` | `OrganizationsApi` | Org 管理 |
| `getUsersApi()` | `UsersApi` | 用户管理 |
| `getAuthorizationsApi()` | `AuthorizationsApi` | Token 管理 |
| `getDeleteApi()` | `DeleteApi` | 按时间/谓词删除点 |
| `getTasksApi()` | `TasksApi` | 任务管理 |
| `close()` | `void` | **关闭客户端,务必调用** |

---

## 九、完整端到端流程示例(三种写法对照)

```
                            InfluxDB
┌──────────────────────────────────────────────────────────┐
│                                                          │
│        ┌──────────────────────────────────────┐          │
│        │  /api/v2/write?org&bucket&precision  │          │
│        └──────────────────┬───────────────────┘          │
│                           │                              │
│                           ▼                              │
│                ┌──────────────────────┐                  │
│                │  Bucket: example_java │                  │
│                │   measurement=temperature              │
│                └──────────────────────┘                  │
└──────────────┬─────────────┬─────────────┬───────────────┘
               │             │             │
       Line Proto       Point Builder    POJO + 注解
               │             │             │
       ┌───────┴─────┐ ┌─────┴────┐ ┌─────┴──────┐
       │ writeRecord │ │writePoint│ │writeMeasure│
       └───────┬─────┘ └─────┬────┘ └─────┬──────┘
               │             │             │
               └─────────────┴─────────────┘
                             │
                  WriteApiBlocking(同步阻塞)
                  WriteApi(异步攒批)
                             │
                             ▼
                       InfluxDBClient
                             │
                             ▼
                  InfluxDBClientFactory.create(url, token, org, bucket)
```

---

## 十、本章勘误小结

| # | 原文位置 | 原文说法 | 实际情况 |
| --- | --- | --- | --- |
| 1 | 11.4.5 | "health 接口已被标记为废弃" | ✅ 对的;**更准确说**:`/health` 端点在 2.x 早期就废弃,client 端 `health()` 也 deprecated。生产应用 `ping()` |
| 2 | 11.5.5 | 称两个查询方法 "queryRaw / query" | ✅ 对的;**补充**:`QueryApi.query(flux, MyPojo.class)` 可以直接把结果映射到 POJO,比手动遍历 FluxTable 简洁,讲义没提 |
| 3 | 11.5.6 示例 | `"from(bucket:\"test_init\") | > range(start:-2m)"` | FLUX 中是 `|>`(管道符),**中间不能有空格**;讲义代码块里写成 `| >` 是排版问题,真正运行的 Java 字符串里要拼对 |
| 4 | 11.7.6.5 | "`@Column` 注解只能用在成员变量上" | ✅ 对;**补充**:`@Column(name="..")` 可以让 Java 字段名和 InfluxDB 列名不同 |
| 5 | 11.8.4 | "我们的 java 程序没有报错" — 异步 + 立即退出导致数据丢 | ✅ 描述准确;**但缺一句关键建议**:生产代码用 try-with-resources 或 finally close,且**异步写入必须 `listenEvents` 监听失败**,否则错误吞噬 |
| 6 | 11.8.7 | "缓冲区默认 10000 条数据" | 是 `DEFAULT_BUFFER_LIMIT`;**`DEFAULT_BATCH_SIZE` 才是 1000(单次写出大小)**。讲义混淆了 "缓冲区大小"和"批大小" |
| 7 | 11.8.8 | "writeApi 写出数据并不是一次把整个缓冲区都写出去,而是按批" | ✅ 正确,这也是为什么 `BUFFER_LIMIT=10000` 而 `BATCH_SIZE=1000` |
| 8 | 11.5.3 | "token 类型还必须得是 `char[]`" | ✅ 对;**安全原因**(String 进入常量池后无法清零)讲义没解释清楚 |
| 9 | 11.4.2 | "检查 InfluxDB 的健康状态不需要任何权限和 token" | ✅ 对;但要注意 **生产可能配置了 IP 白名单或前置网关,`/ping` 也可能被拦** |
