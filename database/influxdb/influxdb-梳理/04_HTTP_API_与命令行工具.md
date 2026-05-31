# 04 HTTP API、CLI 与运维

> 覆盖原文档:第 8 章交互总览、第 9 章 HTTP API、第 10 章 influx-cli、
> 第 15 章 influxd 命令、第 16 章 BoltDB、第 17 章 数据迁移
> 前置:已读 `02_WebUI_数据源管理.md`(Token 鉴权)

---

## 一、总体交互模型

**InfluxDB 2.x 唯一的对外接口就是 HTTP API**。所有"客户端"本质都是 HTTP 客户端:

```
        ┌──────────────────────────────────────────┐
        │                                          │
        │             InfluxDB(influxd)            │
        │  ┌────────────────────────────────────┐  │
        │  │  HTTP API (/api/v2/*, /query, ...) │  │
        │  └────────────────────────────────────┘  │
        │                  ▲                       │
        └──────────────────┼───────────────────────┘
                           │
            ┌──────────────┼──────────────┬───────────────┬────────────────┐
            │              │              │               │                │
       ┌────▼────┐   ┌─────▼──────┐ ┌────▼────┐  ┌───────▼──────┐  ┌──────▼──────┐
       │ Web UI  │   │ influx CLI │ │ SDK     │  │  Telegraf    │  │  Scraper   │
       │ (浏览器)│   │ (命令行)   │ │ (Java...)│  │ (内嵌 out 插件)│  │ (内部)      │
       └─────────┘   └────────────┘ └─────────┘  └──────────────┘  └─────────────┘
```

**安全核心**:所有这些请求都需要鉴权,几乎都靠 **Token Header** 完成。

---

## 二、HTTP API 鉴权方式对比

### 2.1 两种鉴权方式

```
┌────────────────────────────────────────────────────────────────────────┐
│  方式 A: Token 鉴权 (推荐,生产首选)                                    │
│  ┌─────────┐   Authorization: Token <token-value>      ┌──────────┐    │
│  │ Client  │ ──────────────────────────────────────▶  │ influxd  │    │
│  └─────────┘                                          └──────────┘    │
└────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────┐
│  方式 B: 登录(Basic Auth) — 给 Web UI 用                              │
│                                                                        │
│  第 1 步: POST /api/v2/signin                                          │
│  ┌─────────┐   Authorization: Basic Base64(user:pass)  ┌──────────┐    │
│  │ Client  │ ──────────────────────────────────────▶  │ influxd  │    │
│  │         │   ◀────── Set-Cookie: influxdb-oss-session=... ──     │    │
│  └─────────┘                                          └──────────┘    │
│                                                                        │
│  第 2 步: 后续请求自动携带 Cookie                                       │
│  ┌─────────┐   Cookie: influxdb-oss-session=...        ┌──────────┐    │
│  │ Client  │ ──────────────────────────────────────▶  │ influxd  │    │
│  └─────────┘                                          └──────────┘    │
└────────────────────────────────────────────────────────────────────────┘
```

### 2.2 两种方式对比

| 维度 | Token | Basic Auth + Cookie |
| --- | --- | --- |
| 头部 | `Authorization: Token <value>` | 登录时 `Authorization: Basic ...`,之后 `Cookie:` |
| 过期 | 默认不过期 | **~30 分钟自动失效** |
| 用途 | **应用集成、自动化** | **Web UI 用户登录** |
| 风险 | Token 泄露需立即作废 | Basic Auth 把 `user:pass` 用 Base64 编码,**等同明文**,极易被中间人解码 |

> ⚠️ 原文 9.3.2.5 描述 Base64 时只说"编码不是加密",**还可以进一步强调**:Base64 是公开算法,可逆解码,**任何 HTTP 拦截工具都能直接还原原文**。所以**Basic Auth 必须配合 HTTPS**,否则等于明文传账号密码。

### 2.3 失败响应

```
状态码 401 Unauthorized
{
  "code": "unauthorized",
  "message": "unauthorized access"
}
```

---

## 三、HTTPS 配置

### 3.1 完整启用流程

```
1. 用 openssl 生成自签证书
   ┌──────────────────────────────────────────────┐
   │ openssl req -x509 -nodes -newkey rsa:2048 \  │
   │   -keyout selfsigned.key \                   │
   │   -out   selfsigned.crt \                    │
   │   -days  60                                  │
   └──────────────────────────────────────────────┘
                    │
                    ▼
2. 给 influxd 启动加 TLS 参数
   ┌──────────────────────────────────────────────┐
   │ ./influxd \                                  │
   │   --tls-cert=.../selfsigned.crt \            │
   │   --tls-key=.../selfsigned.key               │
   └──────────────────────────────────────────────┘
                    │
                    ▼
3. 客户端访问改 https://
                    │
                    ▼
4. 更新所有依赖的 URL:
   • Telegraf 配置 URL
   • Scraper 任务 URL
   • Java/Python SDK 客户端
   • influx CLI config 的 host-url
```

> ⚠️ 容易遗漏第 4 步。讲义提到了 Telegraf 和 Scraper,但 **influx-cli 的 config 也需要改**,且 SDK 客户端如果用了 `OkHttpClient` 默认 trust manager,**自签证书会被拒**,通常要么导入到 truststore,要么用 `--skip-verify`(仅测试)。

### 3.2 命令参数详解(原文 9.4.1)

| 参数 | 含义 |
| --- | --- |
| `req -x509` | 生成自签名证书(不发往 CA) |
| `-nodes` | 不加密私钥(也写作 no-DES);**真实生产应去掉,密钥要加密** |
| `-newkey rsa:2048` | 同时生成 2048 位 RSA 密钥 |
| `-keyout` | 私钥输出路径 |
| `-out` | 证书输出路径 |
| `-days <N>` | 有效期(天,默认 30,**不是 365**) |

> ⚠️ 原文 9.4.1 命令解释中写"`-days` 默认是 365 天" — **错误**。`openssl req` 的 `-days` 默认值是 **30 天**(不同 openssl 版本可能略有差异,但绝不是 365)。

---

## 四、其他生产安全机制

### 4.1 IP 白名单(实际是"出站白名单")

```
┌──────────────────────────────────────────────────────────┐
│ influxd 配置 (flux-only-host-whitelist 类似的项)         │
│                                                          │
│ 限制的是: FLUX 脚本(via http.post / sql.from / ...)     │
│           可以访问哪些"外部 host"                        │
│                                                          │
│ 不是: 谁可以访问 influxd                                 │
└──────────────────────────────────────────────────────────┘
```

> ⚠️ **极易理解错**。原文 9.5.1 已经强调"不是限制谁访问我",但很多读者一眼瞄过去会想成"防火墙白名单"。这是**给 FLUX 出站请求**用的护栏,防止 FLUX 脚本被注入后用 InfluxDB 当跳板攻击内网。

### 4.2 机密(Secrets)管理

```
┌────────────────────────────────────────────┐
│  influx secret set --key POSTGRES_USERNAME │
│  influx secret set --key POSTGRES_PASSWORD │
└────────────────────────────────────────────┘
                    │
                    ▼
            存到 BoltDB(加密)
                    │
                    ▼
FLUX 脚本中:
    import "influxdata/influxdb/secrets"
    username = secrets.get(key: "POSTGRES_USERNAME")
    password = secrets.get(key: "POSTGRES_PASSWORD")
    sql.from(
        dataSourceName: "mysql://${username}:${password}@localhost",
        ...
    )
```

**好处**:用户名密码不再硬编码在 FLUX 脚本中,即使脚本被外泄也不暴露凭据。

### 4.3 Token 的三种类型(再次梳理,与 02 章对应)

| 类型 | 创建途径 | 范围 |
| --- | --- | --- |
| **Operator Token** | `influxd recovery auth create-operator` 或 Setup 时自动生成 | **整个 InfluxDB 实例**,跨 Org |
| **All Access Token** | Web UI / `influx auth create --all-access` | **当前 Org** 内全部资源 |
| **Read/Write Token** | Web UI / `influx auth create --read-bucket xxx --write-bucket xxx` | 指定 bucket 的读/写 |

> 💡 **生产实践**:每个 Org 创建独立 All Access Token,避免不同业务误操作彼此资源;细粒度只读/只写 Token 留给具体应用。**Operator Token 仅用于运维**。

### 4.4 禁用部分端点

可在 `influxd` 配置中关闭:

- `/metrics` — Prometheus 自监控指标
- `/debug/pprof` — Go pprof
- Web UI — 完全关掉图形界面

适用场景:生产对外服务时,只暴露 `/api/v2/*`,关闭管理面。

---

## 五、API 文档与 OpenAPI

### 5.1 三个文档入口

| 入口 | 内容 |
| --- | --- |
| `http(s)://<host>:8086/docs` | **跟随当前版本**的交互式文档(Swagger UI) |
| `https://docs.influxdata.com/influxdb/v2.4/api/` | 在线官方文档 |
| `http(s)://<host>:8086/api/v2/swagger.json` | **OpenAPI 3.0 定义文件**,可导入 Postman/ApiPost |

> ⚠️ 原文 9.6.2 写"访问 `http://localhost:8086` 还能下载 OpenAPI 文档" — 准确路径是 `http://localhost:8086/api/v2/swagger.json`(或在 `/docs` 页面顶部点 Download)。

### 5.2 导入 Postman/ApiPost 流程

```
浏览器: /docs ── 点 Download ──▶ swagger.json
                                    │
                                    ▼
Postman: Import ── 选 swagger.json
                                    │
                                    ▼
左侧出现完整的 API 目录树,可立即调用
```

---

## 六、influx CLI(命令行工具)

### 6.1 重要变化:与 influxd 解耦

| 版本 | 包结构 |
| --- | --- |
| **≤ 2.0** | `influx` 二进制随 `influxd` 一起发布 |
| **≥ 2.1** | **`influx` 单独打包,源码也分仓库** |

```
        github.com/influxdata/influxdb             github.com/influxdata/influx-cli
                  │                                          │
                  ▼                                          ▼
            influxd 二进制                              influx 二进制
```

下载时要分别在两个仓库 Release 下抓对应版本的 client tarball。

### 6.2 配置(避免每次粘 Token)

```
./influx config create --config-name influx.conf \
    --host-url http://localhost:8086 \
    --org atguigu \
    --token <token-value> \
    --active
```

**写到 `~/.influxdbv2/configs`**(TOML 格式):

```toml
["influx.conf"]
url    = "http://localhost:8086"
token  = "ZA8u...PT3w=="
org    = "atguigu"
active = true
```

### 6.3 多配置切换

```
list      ./influx config list
create    ./influx config create --config-name xxx ...
update    ./influx config update --config-name xxx ...   ← 修改用 update
switch    ./influx config <name>                         ← 切换默认
remove    ./influx config remove <name>
```

> 📝 同一份配置文件中,`["name"]` 必须全局唯一;`active=true` 只能存在一个,旧的会被自动改为 `previous=true`。

### 6.4 子命令一览(选重要)

| 命令 | 用途 |
| --- | --- |
| `setup` | 初始化(等同 Web UI Setup) |
| `auth` | Token 管理(create/list/active/inactive/delete) |
| `bucket` | Bucket 管理 |
| `org` | Org 管理 |
| `user` | 用户管理 |
| `task` | 定时任务管理 |
| `query` | 执行 FLUX 查询 |
| `write` | 写入行协议数据 |
| `delete` | 按时间/谓词删除数据 |
| `export` / `apply` / `stacks` | InfluxDB 模板相关(见第 12 章笔记) |
| `backup` / `restore` | 备份/恢复(仅 OSS) |
| `v1` | InfluxDB v1 兼容 API |
| `ping` | 健康检查 |

---

## 七、influxd 服务进程参数

### 7.1 子命令列表

| 子命令 | 用途 |
| --- | --- |
| `run` | 默认,启动服务进程 |
| `inspect` | **底层文件检查/导出**(TSI/TSM/WAL) |
| `recovery` | **离线恢复操作 token / org / user** |
| `upgrade` | 1.x → 2.x 升级 |
| `downgrade` | 元数据格式回退 |
| `version` | 打印版本 |
| `print-config` | 打印当前生效配置(2.4 已 deprecated,改用 `influx server-config`) |

### 7.2 inspect:看底层存储

`inspect` 下还有一层子命令,直接操作存储引擎文件:

| 子命令 | 用途 |
| --- | --- |
| `report-tsm` | TSM 文件报告(序列数、时间范围) |
| `report-tsi` | TSI 索引基数报告 |
| `dump-tsm` / `dump-tsi` / `dump-wal` | 转储原始内容 |
| `export-lp` | **将存储桶数据导出为行协议** |
| `build-tsi` | 重建 TSI 索引 |
| `verify-tsm` / `verify-wal` / `verify-tombstone` / `verify-seriesfile` | 完整性校验 |

```
TSI = Time-Series Index (倒排索引)
TSM = Time-Structured Merge tree (列存数据)
WAL = Write-Ahead Log (写前日志)
```

### 7.3 recovery:离线找回 Token

**典型场景**:Operator Token 丢了或被误删,Web UI 进不去。

```
./influxd recovery auth create-operator --username tony --org atguigu
```

- **离线执行**:必须先停 `influxd`(它会锁住 BoltDB)
- 不依赖任何现有 Token
- 直接基于磁盘文件操作

子命令:
- `recovery auth list / create-operator`
- `recovery org`
- `recovery user`

### 7.4 常用配置项

| 配置 | 含义 |
| --- | --- |
| `http-bind-address` | HTTP 监听地址(默认 `:8086`) |
| `bolt-path` | BoltDB 文件路径(默认 `~/.influxdbv2/influxd.bolt`) |
| `engine-path` | TSM/TSI/WAL 引擎数据目录(默认 `~/.influxdbv2/engine`) |
| `sqlite-path` | SQLite 元数据库路径(任务运行历史用) |
| `flux-log-enabled` | 是否记录 FLUX 查询日志,默认 false |
| `log-level` | 日志级别 debug/info/warn/error,默认 info |
| `tls-cert` / `tls-key` | HTTPS 证书 |

### 7.5 三种配置方式 + 优先级

```
┌─────────────────────────────────────────────────────────┐
│   优先级(高 → 低):                                    │
│                                                         │
│   命令行参数  ──────────▶  最高                         │
│       │                                                 │
│       ▼                                                 │
│   环境变量(INFLUXD_XXX)                                │
│       │                                                 │
│       ▼                                                 │
│   配置文件(config.json/toml/yaml)                      │
│       │                                                 │
│       ▼                                                 │
│   内置默认值     ──────▶  最低                          │
└─────────────────────────────────────────────────────────┘
```

**对应示例**(都是改 HTTP 端口为 8088/8089/9090):

```bash
# 方式 1:命令行参数
./influxd --http-bind-address=:8088

# 方式 2:环境变量(下划线 + 全大写)
export INFLUXD_HTTP_BIND_ADDRESS=:8089
./influxd

# 方式 3:配置文件
echo '{"http-bind-address": ":9090"}' > config.json
./influxd     # 自动检测同目录下的 config.{json,toml,yaml}
```

### 7.6 杀掉 influxd 的安全姿势

```bash
ps -ef | grep influxd | grep -v grep | awk '{print $2}' | xargs kill
```

**避免 `kill -9`** — 它会跳过优雅关闭,可能造成 WAL 文件未刷新落盘。

---

## 八、InfluxDB 内部存储(BoltDB)

```
~/.influxdbv2/
├─ influxd.bolt          ← BoltDB,存:用户/密码哈希/Token/Org/Bucket 元数据
├─ engine/
│   ├─ data/<bucket-id>/<shard>/*.tsm    ← 时序数据(列存)
│   ├─ wal/<bucket-id>/<shard>/*.wal     ← 写前日志
│   └─ index/<bucket-id>/<shard>/...     ← TSI 倒排索引
├─ configs              ← influx CLI 的配置
└─ influxd.sqlite       ← 任务执行历史/通知/检查
```

> ℹ️ BoltDB 是 Go 的纯内存映射 KV 数据库,**单进程独占文件锁** — 这就是为什么不能同时跑两个 `influxd` 指向同一目录。
>
> InfluxDB 用 BoltDB 存"元数据 + 密码哈希 + Token"。**用户密码不是明文存的**,而是 bcrypt 哈希。

> ⚠️ 原文第 16 章说"InfluxDB 内部自带了一个用 Go 语言写的 BlotDB" — **拼写应是 BoltDB(没有第二个 b)**,讲义全文都拼错为 BlotDB。

---

## 九、数据迁移/导出

### 9.1 关键命令

```
./influxd inspect export-lp \
    --bucket-id <bucket-id> \
    --engine-path ~/.influxdbv2/engine \
    --output-path ./export.lp \
    --start 2024-01-01T00:00:00Z \
    --end   2024-01-31T23:59:59Z \
    --compress
```

| 参数 | 说明 |
| --- | --- |
| `--bucket-id` | **必需**;在 Web UI 或 `influx bucket list` 拿 ID |
| `--engine-path` | 默认 `~/.influxdbv2/engine`;非默认数据目录需指定 |
| `--output-path` | **必需** |
| `--start` / `--end` | 可选,默认全量 |
| `--compress` | gzip 压缩输出,**行协议是高度重复的文本,压缩比通常 10-30 倍** |

### 9.2 执行流程

```
┌─────────────────────────────────────────────────────┐
│  ./influxd inspect export-lp ...                    │
│                                                      │
│   1. 读 engine-path/data/<bucket-id>/*.tsm  ──┐     │
│                                                │     │
│   2. 读 engine-path/wal/<bucket-id>/*.wal  ──┤     │
│                                                ▼     │
│                                          解码为行协议│
│                                                │     │
│                                                ▼     │
│                                          (可选)gzip │
│                                                │     │
│                                                ▼     │
│                                       output-path 文件│
└─────────────────────────────────────────────────────┘
```

### 9.3 实测体感(原讲义)

| 操作 | 大小 |
| --- | --- |
| `test_init` bucket 不压缩导出 | 1.5 GB(纯文本行协议) |
| 同样数据 `--compress` | 通常 < 100 MB |

> ⚠️ 离线导出是从**磁盘文件**直接读,绕过 HTTP API,所以**不受 Token 鉴权约束**;但前提是**你有 OS 层面的文件读取权限**。这意味着保护 `~/.influxdbv2/engine` 的文件权限和保护 Token 同等重要。

### 9.4 完整迁移路径(到另一台 InfluxDB)

```
源主机                                       目标主机
┌──────────────────┐                       ┌──────────────────┐
│  influxd inspect │                       │  influx write   │
│  export-lp       │  ─── scp / rsync ──▶  │  --bucket xxx   │
│                  │                       │  -f export.lp   │
└──────────────────┘                       └──────────────────┘
        │                                          │
        ▼                                          ▼
   export.lp (行协议)                       数据落到目标 bucket
```

或者用 `./influx backup` / `./influx restore`(只 OSS,基于 BoltDB 整体备份)。

---

## 十、本章勘误小结

| # | 原文位置 | 原文说法 | 实际情况 |
| --- | --- | --- | --- |
| 1 | 9.4.1 命令解释 | "`-days`,默认是 365 天" | openssl `req` 的 `-days` 默认值是 **30 天**,而且不同版本有差异 |
| 2 | 9.3.2.5 | 说 cookie 过期"大概半小时" | 准确说由 `session-length` 配置决定,**默认 60 分钟**;但讲义用"半小时"也不错(取决于版本) |
| 3 | 9.4.1 | 用 `-nodes` 表示"不加密私钥" | ✅ 对的;**生产应去掉 `-nodes`** 让私钥本身加密。讲义没强调这一点 |
| 4 | 9.6.2 | "访问 `localhost:8086` 还能下载 OpenAPI 文档" | 实际是访问 `/docs` 页面顶部的 Download 按钮,文件路径 `/api/v2/swagger.json` |
| 5 | 第 16 章 | "**BlotDB**" | 拼写错误,应为 **BoltDB**(Bolt = 螺栓,Go 社区流行的纯 KV 库,作者 Ben Johnson) |
| 6 | 9.2.2 | "InfluxDB 的 API,请求头必须加上 token" | 准确说,**Authorization 头**,且格式是 `Authorization: Token <value>`,不是只放 token 字符串本身 |
| 7 | 15.4.2 | 环境变量名 `INFLUXD_HTTP_BIND_ADDRESS` | ✅ 正确(配置项名转大写并把 `-` 换 `_`,加 `INFLUXD_` 前缀);讲义示例命令里写错为 `INFLUXD HTTP BIND ADDRESS`(空格)只是终端拷贝问题 |
| 8 | 15.4.3 | "config.json,config.toml,config.yaml" | ✅ 都支持;**官方推荐 TOML**,因为 InfluxDB 配置项里有大量 `-` 连字符,在 JSON key 里要带引号会很丑 |
| 9 | 17.1 参数解释 "`--engine-path` 默认是 `~/.influxdbv2/engine`" | ✅ 正确,但**如果你改过 `bolt-path` 或 `engine-path`,必须显式指定**;否则导出空文件且不报错 |
