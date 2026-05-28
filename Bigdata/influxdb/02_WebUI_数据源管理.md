# 02 Web UI 与数据源管理

> 覆盖原文档:第 3.1 章(Load Data / Buckets / Telegraf / Scraper / API Token)
> 前置概念:已读 `01_概念与架构.md`

---

## 一、Load Data 总览

InfluxDB 2.x 的左侧导航栏中,向上箭头图标对应 **Load Data**(加载数据)页面,这是所有"把数据写进 InfluxDB"路径的入口。

```
┌────────────── Load Data ──────────────┐
│                                        │
│  ┌────────┐  ┌────────┐  ┌────────┐  │
│  │SOURCES │  │BUCKETS │  │TELEGRAF│  │  ← 选项卡
│  └────────┘  └────────┘  └────────┘  │
│  ┌──────────┐  ┌────────┐             │
│  │ SCRAPERS │  │API Tok │             │
│  └──────────┘  └────────┘             │
└────────────────────────────────────────┘
```

写入数据的途径共 **4 类**:

| 方式 | 模型 | 适用 |
| --- | --- | --- |
| 文件上传(CSV / Line Protocol / Annotated CSV) | 手动一次性 | 测试、补数 |
| 客户端 SDK(Java/Go/Python/JS…) | 应用代码主动 push | 业务集成 |
| Telegraf(独立 Agent) | Push 模型,Agent 主动推 | 主机/中间件监控 |
| Scraper(内置抓取任务) | Pull 模型,InfluxDB 主动拉 | 暴露 `/metrics` 的服务 |

---

## 二、Bucket 管理

### 2.1 概念再回顾

- **Bucket ≈ database**(逻辑容器),自带**数据保留策略**(retention)
- **重命名 bucket 是高风险操作** — 因为大量代码/Task/Telegraf 配置是按 bucket 名引用的

### 2.2 创建与设置

| 字段 | 含义 | 备注 |
| --- | --- | --- |
| Name | bucket 名 | 字符串,在 Org 内唯一 |
| Delete Data Older Than | 过期时间 | NEVER / 1h / 6h / 24h / 7d / 30d / 自定义 |
| Schema Type | 模式 | 默认 implicit(隐式,无模式);**explicit 模式**支持声明 measurement schema |

> ⚠️ 原文 3.1.3.4 写"InfluxDB 是一个无模式的数据库,你甚至可以前后在同一个 measurement 下插入 field **不同**的数据"。
>
> **这在 InfluxDB 2.x 中只有 `implicit` 模式下成立**。2.x 引入了 `explicit` 模式 bucket,可以为 measurement 预定义 schema,此时插入不符合 schema 的数据会被拒绝。原讲义没有提到这一点。

### 2.3 Label

每个 Bucket 卡片左下方有 `Add a label`,用于打标签做分组/筛选。实际工程中**很少用到**。

---

## 三、数据写入(以 Line Protocol 为例)

### 3.1 行协议格式快速回顾(完整版见附录笔记)

```
people,name=tony age=12
└─────┘└────────┘└────┘
  ↑       ↑        ↑
  │       │        └─ field set(数值字段,逗号分隔)
  │       └────────── tag set(索引维度,逗号分隔)
  └────────────────── measurement
```

> ⚠️ 原文 3.1.3.3 给出的 5 条样本数据写作:
> ```
> people, name=tony age=12      ← measurement 后面多了空格
> ```
> **InfluxDB 行协议中,measurement 和第一个 tag 之间必须用逗号 `,` 连接,且不能有多余空格**。
> 正确写法应该是 `people,name=tony age=12`(没有空格)。
> 原文的样本如果照搬到行协议解析器里会**报错**。如果是 Web UI 表单贴入,Web UI 可能会忽略前导空格,但作为示例代码这是 bug,需要纠正。

### 3.2 Web UI 写入流程

```
Load Data ─▶ Line Protocol
    │
    ▼
选择目标 Bucket
    │
    ▼
ENTER MANUALLY(手动) 或 UPLOAD FILE(文件)
    │
    ▼
指定时间精度(ns / us / ms / s)
    │
    ▼
点击 WRITE DATA
    │
    ▼
Data Written Successfully ✓
```

---

## 四、Telegraf 数据源

### 4.1 Telegraf 在生态里的位置

- 是独立部署的**采集 Agent**,**不是 InfluxDB 服务的一部分**
- 插件化:**input plugin** 采数据,**output plugin** 推数据
- 现已成为通用采集组件,**很多其它时序库**(Prometheus, OpenTSDB...)也接入它

### 4.2 配置文件的来源

InfluxDB 2.x Web UI **只是帮你生成 Telegraf 配置文件**,并通过自身 HTTP API 暴露给 Telegraf 读取。**它不能管 Telegraf 的启停**。

```
┌─────────────── 配置生成阶段 ───────────────┐
│ 用户 ─▶ Web UI ─▶ 选择 input(cpu/mem/...)  │
│         点 CREATE AND VERIFY               │
│              │                              │
│              ▼                              │
│   InfluxDB 把配置存到 _internal             │
│   暴露 URL:                                │
│   http://host:8086/api/v2/telegrafs/<id>    │
│   同时生成一个 WRITE 权限的 Token           │
└──────────────────────────────────────────────┘

┌─────────────── 运行阶段(用户负责) ───────────┐
│  export INFLUX_TOKEN=<生成的写入Token>      │
│                                              │
│  ./telegraf --config http://host:8086/api/v2/telegrafs/<id>
│      │                                       │
│      ▼                                       │
│  Telegraf 启动 → 拉配置 → 加载 input/output  │
│      │                                       │
│      ▼                                       │
│  每 10s(默认) 采集 host 指标               │
│      │                                       │
│      ▼                                       │
│  influxdb_v2 output 插件                    │
│      │                                       │
│      │ HTTP POST + Bearer Token             │
│      ▼                                       │
│  InfluxDB Bucket(example02)                 │
└──────────────────────────────────────────────┘
```

### 4.3 关键命令与脚本

讲义给出的启停脚本 `host_tel.sh`(放在 `~/bin/`,加 `755` 权限)逻辑:

```
start ─▶ 检查 telegraf 是否已运行 ─▶ 未运行则 export TOKEN + 启动
stop  ─▶ ps -ef | grep telegraf 取 pid ─▶ kill <pid>
status ▶ 查询当前进程状态
```

> ✅ 关键点:
> 1. `INFLUX_TOKEN` **必须**在同一 shell 会话中导出,否则 telegraf 启动会因鉴权失败而无法写入
> 2. 启动脚本中 `--config` 后的 URL 是 Web UI 复制的、不要改最后的 `<id>`
> 3. InfluxDB **不会监控 Telegraf 的存活**,生产中需要 systemd / supervisord 等做守护

> ⚠️ 原文示例 3.1.5.6 命令里有一个**排版问题**:
> ```
> telegraf --config http://localhost:8086/api/v2/telegrafs/09dcf4afcfd90000telegraf
> ```
> URL 末尾多粘上了 `telegraf` 三个字母,这是文档的笔误,实际执行时**不要带**这后缀,否则路径错误会 404。

---

## 五、Scraper(InfluxDB 主动拉取)

### 5.1 Scraper 与 Telegraf 的区别

| 维度 | Telegraf | Scraper |
| --- | --- | --- |
| 模型 | **Push**(Agent 推) | **Pull**(InfluxDB 拉) |
| 配置位置 | 独立 Agent | InfluxDB 内置 |
| 抓取间隔 | 可配,默认 10s | **固定 10s,不可改** |
| 数据格式 | 任意(由 input 插件决定) | **必须是 Prometheus 文本格式** |
| 适合场景 | 主机/复杂系统 | 已暴露 `/metrics` 的服务 |

### 5.2 创建流程

```
Load Data ─▶ SCRAPERS ─▶ CREATE SCRAPER
    │
    ▼
┌──────────────────────────────────────┐
│ Name           : example03_scraper   │
│ Target Bucket  : example03           │
│ Target URL     : http://host:8086/metrics │
└──────────────────────────────────────┘
    │
    ▼
InfluxDB 每 10s 发起一次 HTTP GET <URL>
    │
    ▼
解析 Prometheus 文本格式 ─▶ 写入 Bucket
```

### 5.3 自身 `/metrics`

InfluxDB 自己暴露 `http://localhost:8086/metrics`(Prometheus 格式),包含:

- `boltdb_reads_total` / `boltdb_writes_total`(元数据库读写)
- `go_gc_duration_seconds_*`(GC 暂停时间分布)
- `go_goroutines`(协程数)
- `go_memstats_alloc_bytes`(内存分配)
- `http_api_requests_total{handler,method,path,...}`(各 API 调用次数)

> 💡 这就是"为什么 Setup 时会自动出现一个 Scraper" — 它指向 `localhost:8086/metrics`,把自身指标写入初始 Bucket。

---

## 六、API Token 管理

### 6.1 鉴权模型

InfluxDB 2.x **所有客户端最终都是 HTTP 请求**(CLI、SDK、Web UI、Telegraf、Scraper 都不例外),鉴权统一靠 Token:

```
┌──────────┐   HTTP Header                ┌──────────┐
│ Client   │── Authorization: Token xxx ─▶│ influxd  │
└──────────┘                               └──────────┘
                                                │
                                                ▼
                            ┌─────────────────────────────────┐
                            │ Token 校验:                     │
                            │  • token 是否存在 & Active       │
                            │  • token 是否有目标操作的 scope  │
                            │    (read/write × bucket/org/...) │
                            └─────────────────────────────────┘
```

### 6.2 Token 类型对照

| 类型 | 来源 | 权限范围 |
| --- | --- | --- |
| **Operator Token** | `influxd` 进程级,启动时生成或显式创建 | **全实例**(跨所有 Org) |
| **All Access Token** | Web UI 选 "All Access API Token" | 当前 Org 内全权限 |
| **Read/Write Token** | Web UI 选 "Read/Write API Token" | 指定 bucket 的读/写 |
| **自定义 Token** | API / CLI 创建 | 任意细粒度组合 |

> ⚠️ 原文 3.1.8.3 把初始的 `tony's Token` 描述为"权限很高"。**它实际是 All Access Token(组织内全权限)**,**不是 Operator Token**。Operator Token 在 Setup 时不会显式暴露在 Web UI 上,它属于 `influxd` 启动初始化阶段的产物。这两个概念在生产环境的权限审计里完全不一样,需区分。

### 6.3 Web UI 提供的 Token 模板只是常用模板

Web UI 上只有两个模板按钮:

- Read/Write API Token(可选 bucket)
- All Access API Token

**实际 API 支持的权限组合远比这两个细**:

```
权限维度举例:
  resource type:  buckets / tasks / dashboards / users / orgs ...
  action     :  read  /  write
  scope      :  全 Org  /  指定 ID
```

要做细粒度 Token,需要走 `POST /api/v2/authorizations` 或 `influx auth create --read-bucket <id> ...`。

### 6.4 Token 生命周期

| 操作 | 影响 |
| --- | --- |
| **重命名** | 安全,客户端按 token 值访问,与名字无关 |
| **关停(Active=Off)** | 立即失效,但保留记录;可随时打开 |
| **删除** | 永久不可恢复;依赖此 token 的所有客户端立即 401 |

---

## 七、典型示例的执行流程汇总

### 示例 1:文件导入(Line Protocol)

```
用户 ─▶ Web UI ─▶ Load Data ─▶ Line Protocol
       ▼
     选 Bucket(example01) + 输入行协议文本
       ▼
     InfluxDB 行协议解析器(进程内)
       ▼
     写入 TSM(per-series storage)
```

### 示例 2:Telegraf 推送(Push)

```
            ┌─────────────────┐
            │  Telegraf 主机  │
            │                 │
            │  ┌───────────┐  │       ┌──────────────────┐
            │  │ inputs    │  │       │   InfluxDB       │
            │  │  cpu/mem  │  │       │                  │
            │  │  /disk    │  │       │  ┌────────────┐  │
            │  └─────┬─────┘  │       │  │ 写入引擎   │  │
            │        │ 10s    │       │  │            │  │
            │        ▼        │       │  └─────┬──────┘  │
            │  ┌───────────┐  │ POST  │        │         │
            │  │ output    │  │──────▶│        ▼         │
            │  │ influx_v2 │  │ +Tok  │  ┌────────────┐  │
            │  └───────────┘  │       │  │example02   │  │
            └─────────────────┘       │  │  bucket    │  │
                                       │  └────────────┘  │
                                       └──────────────────┘
```

### 示例 3:Scraper 拉取(Pull)

```
┌──────────────────┐                ┌──────────────────────┐
│  目标服务         │                │   InfluxDB           │
│  /metrics        │◀── HTTP GET ───│  Scraper 调度器      │
│  (Prom format)   │   每 10s        │  (固定周期)          │
│                  │ ─── data ──────▶│                      │
└──────────────────┘                │  Prom 解析 → 行协议   │
                                     │      ↓                │
                                     │  写入 example03 Bucket│
                                     └──────────────────────┘
```

---

## 八、本章勘误小结

| # | 原文位置 | 原文说法 | 实际情况 |
| --- | --- | --- | --- |
| 1 | 3.1.3.3 示例数据 | `people, name=tony age=12`(measurement 后带空格) | 行协议规范要求 measurement 与第一个 tag 用逗号连接,**不允许空格**。正确:`people,name=tony age=12` |
| 2 | 3.1.3.4 | "在同一 measurement 下插入 field 不同的数据" | 仅在 **implicit 模式 bucket** 下成立,2.x 引入的 **explicit schema bucket** 会拒绝偏离 schema 的写入 |
| 3 | 3.1.5.6 命令行 | `.../telegrafs/09dcf4afcfd90000telegraf` | URL 末尾**误粘了 `telegraf` 字串**,实际执行应去掉,否则 404 |
| 4 | 3.1.6.1 | "InfluxDB 1.x 时候,类似的任务**只能**由 Telegraf 来实现" | 1.x 也有 `[[scrape]]` 配置块(虽然不如 2.x 内置 Scraper 简洁),原文"只能"略绝对 |
| 5 | 3.1.7.4-2 | "InfluxDB 的抓取任务都是 10 秒一次,无法自定义设置" | 截至 2.4 在 Web UI 上确实**不可改**,但**通过 API/template 可以**显式声明 `interval`(在 2.6+ 才完全打开了 Web UI 的可见性)。原书所述对 2.4 GA 是事实,但不是"内核限制"。 |
| 6 | 3.1.8.3 | 把 `tony's Token` 等价于"超高权限 Token" | 它是 **All Access Token(Org 内全权)**,**不是 Operator Token(实例级)**,两者区别巨大 |
| 7 | 3.1.8.4 | "Web UI 上给的只是生成 Token 的模板,但不代表它的全部功能" | ✅ 完全正确,值得加粗。实际细粒度权限需走 `influx auth` 或 `/api/v2/authorizations` |
