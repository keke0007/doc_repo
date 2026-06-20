# Nginx 反向代理与负载均衡 知识点梳理

> 源文档:`yuque/Nginx的反向代理与负载均衡.md`
> 整理目标:把原文按"集群基础 → 反向代理与正向代理的本质区别 → upstream 调度算法 → proxy_set_header 与真实 IP 透传 → 基于 URI/UA/扩展名的路由分流"重新组织,跨主机/跨模块交互一律配 ASCII 流程图,涉及命令、配置示例都做了纠正与补全,末尾以【纠错】小节列出原文出入。

---

## 1. 集群基础

### 1.1 什么是集群

> 一组相互独立的计算机,通过高速网络组成"对外像一台机器"的服务系统。

特点:成员独立运行各自服务,彼此通信、协同对外提供应用、资源、数据,统一管理。

### 1.2 为什么要集群

| 目标 | 说明 |
|---|---|
| 高性能(Performance) | 横向扩展计算能力 |
| 性价比(Price/Perf) | 多台普通机器替代单台高端机 |
| 可伸缩性(Scalability) | 弹性增加/缩减节点 |
| 高可用性(Availability) | 单节点宕机不影响整体 |
| 透明性 | 用户看不到集群细节 |
| 可管理性、可编程性 | 自动化运维 |

### 1.3 集群分类

| 类型 | 中文 | 解决的核心问题 | 代表实现 |
|---|---|---|---|
| **LB** | 负载均衡集群 | 流量调度 | Nginx / LVS / HAProxy / F5 |
| **HA** | 高可用集群 | 单点故障 | Keepalived / Pacemaker |
| **HPC** | 高性能计算集群 | 大规模并行计算 | MPI / Slurm |
| **GC**(Grid Computing) | 网络计算 | 分布式任务 | — |

### 1.4 LB 软硬件

- 硬件:**F5 BIG-IP**、**A10**(性能强、贵)。
- 软件:
  - **Nginx**:7 层为主,**1.9 版本之后**通过 `stream` 模块支持 4 层(TCP/UDP)。
  - **LVS**:纯 4 层。
  - **HAProxy**:4 层、7 层都支持。

### 1.5 负载均衡 vs 反向代理 vs 数据转发

- **负载均衡**:对请求进行调度与压力分担(更偏目标)。
- **反向代理**:代理服务器**代替用户**去访问后端(更偏实现方式)。
- **数据转发**:网络层(NAT/路由)按规则转发包,不解析 7 层。

#### 正向代理 vs 反向代理

```
正向代理 (Forward Proxy)
   ┌────────┐  请求(目标 = 真实服务器)  ┌──────────┐  请求  ┌──────────┐
   │ 客户端 │ ─────────────────────────▶│ 代理服务器│──────▶│真实服务器│
   └────────┘                             └──────────┘        └──────────┘
   "客户端知道目标是谁,代理替它出去"

反向代理 (Reverse Proxy)
   ┌────────┐  请求(目标 = 网站域名)   ┌──────────┐  请求  ┌──────────┐
   │ 客户端 │ ─────────────────────────▶│ 代理服务器│──────▶│真实服务器│
   └────────┘   (客户端不知道有几台后端)  └──────────┘        └──────────┘
   "客户端只知道域名,代理替服务集群挡在前面"
```

### 【纠错】

- 原文"nginx (7 层 1.9 版本之后支持 4 层)"措辞简略,**完整事实**:Nginx 1.9.0 引入 `ngx_stream_core_module`(实验性),1.9.13 之后默认编译;编译时仍需显式 `--with-stream`。
- 原文 §1.5 把"反向代理与数据转发"放在一起对比却没展开二者本质区别——**关键差异**是反向代理工作在应用层(7 层),会建立两个独立 TCP 连接(client↔proxy、proxy↔upstream);数据转发(LVS-DR/TUN/NAT)工作在传输层(4 层),不建立 7 层连接,常常只改包头。

---

## 2. 实验拓扑与基础环境

### 2.1 地址规划

| HOSTNAME | IP | 角色 |
|---|---|---|
| lb01 | 10.0.0.5 | Nginx 主负载均衡 |
| lb02 | 10.0.0.6 | Nginx 备负载均衡 |
| web01 | 10.0.0.7 | Web 节点 |
| web02 | 10.0.0.8 | Web 节点 |
| web03 | 10.0.0.9 | Web 节点 |

### 2.2 常用网络查看命令

```bash
ip address show          # 查看 IP 地址(简写 ip a)
ip route show            # 查看路由(简写 ip r)
```

### 2.3 整体调用流程

```
                  浏览器
                     │  Host: bbs.etiantian.org
                     ▼
              ┌─────────────┐
              │ lb01 (10.0.0.5) │  Nginx 反向代理 + upstream
              └──────┬──────┘
                     │ 按调度算法分发
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
 ┌──────────┐  ┌──────────┐  ┌──────────┐
 │web01:80  │  │web02:80  │  │web03:80  │
 │10.0.0.7  │  │10.0.0.8  │  │10.0.0.9  │
 └──────────┘  └──────────┘  └──────────┘
```

---

## 3. Nginx 反向代理实践

### 3.1 安装(三台 web + 两台 lb 同配置)

```bash
yum install -y pcre-devel openssl-devel
mkdir -p /server/tools && cd /server/tools
wget -q http://nginx.org/download/nginx-1.10.3.tar.gz
useradd -M -s /sbin/nologin www                # 统一用 www 用户
tar xf nginx-1.10.3.tar.gz && cd nginx-1.10.3
./configure --user=www --group=www \
            --prefix=/application/nginx-1.10.3 \
            --with-http_stub_status_module \
            --with-http_ssl_module
make && make install
ln -s /application/nginx-1.10.3 /application/nginx
```

### 3.2 三台 Web 节点的相同 nginx.conf(基于域名的虚拟主机)

```nginx
worker_processes  1;
events { worker_connections  1024; }
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile      on;
    keepalive_timeout 65;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    server {
        listen 80;
        server_name bbs.etiantian.org;
        location / { root html/bbs;  index index.html index.htm; }
        access_log logs/access_bbs.log main;
    }
    server {
        listen 80;
        server_name www.etiantian.org;
        location / { root html/www;  index index.html index.htm; }
        access_log logs/access_www.log main;
    }
}
```

### 3.3 准备测试页(三台机器同时执行)

```bash
mkdir -p /application/nginx/html/{www,bbs}
for name in www bbs; do
  echo "$name `hostname`" > /application/nginx/html/$name/xiaoxinxin.html
done
```

### 3.4 直连测试(在 lb01 上发起,绕过负载,直打后端)

```
curl -H 'host:bbs.etiantian.org' 10.0.0.7/xiaoxinxin.html   →  bbs web01
curl -H 'host:bbs.etiantian.org' 10.0.0.8/xiaoxinxin.html   →  bbs web02
curl -H 'host:bbs.etiantian.org' 10.0.0.9/xiaoxinxin.html   →  bbs web03
curl -H 'host:www.etiantian.org' 10.0.0.7/xiaoxinxin.html   →  www web01
...
```

> 这一步的意义:把"后端正常"与"负载均衡正常"解耦,出问题时先排除上游。

### 3.5 lb01 上的负载均衡配置(最小可用版)

```nginx
http {
    upstream server_pools {
        server 10.0.0.7:80;
        server 10.0.0.8:80;
        server 10.0.0.9:80;
    }
    server {
        listen 80;
        server_name bbs.etiantian.org;
        location / { proxy_pass http://server_pools; }
    }
}
```

### 3.6 经 lb01 测试(轮询)

```
curl -H 'host:bbs.etiantian.org' 10.0.0.5/xiaoxinxin.html   →  bbs web03
curl -H 'host:bbs.etiantian.org' 10.0.0.5/xiaoxinxin.html   →  bbs web02
curl -H 'host:bbs.etiantian.org' 10.0.0.5/xiaoxinxin.html   →  bbs web01
```

每次落到不同 web,验证默认 rr 轮询生效。

### 【纠错】

- 原文 §3.3 在 `./configure` 时用了 `--user=nginx --group=nginx`,但前一行 `useradd www -s /sbin/nologin -M` 创建的是 **www** 用户。直接照抄会导致 `getpwnam("nginx") failed`。**正确做法**:用户名要和 `--user` 一致(本文统一改成 `www`),或者前面 `useradd nginx`。
- 原文最后一行软链 `ln -s /application/nginx-1.10.3 /application/ngin`(末尾 "ngin",**漏一个 `x`**),会让 `/application/nginx/sbin/nginx` 这个路径找不到。
- 原文 §3.4 测试结果里 `10.0.0.7 → web02`、`10.0.0.8 → web01`,与 §3.1 地址规划里 `web01=10.0.0.8 / web02=10.0.0.7` 反着。原表与原测试自相矛盾。**本文已统一规划为 web01=10.0.0.7 / web02=10.0.0.8 / web03=10.0.0.9**,避免后续踩坑。

---

## 4. Nginx 常用模块与负载调度算法

### 4.1 反向代理涉及的核心模块

| 模块 | 作用 |
|---|---|
| `ngx_http_stub_status_module` | 暴露当前连接/请求统计(给 Zabbix/Prometheus 用) |
| `ngx_http_ssl_module` | HTTPS / TLS |
| `ngx_http_log_module` | 访问/错误日志 |
| `ngx_http_upstream_module` | 定义后端服务池 |
| `ngx_http_proxy_module` | `proxy_pass` 等反代指令 |

### 4.2 两块协作:upstream + proxy

- **upstream 模块**:像"池塘",把一组后端服务器登记进去。
- **proxy 模块**:`proxy_pass http://池塘名;` 把请求转交进池塘,实际调度由 upstream 完成。

```
              ┌────────────────────────────────────────────┐
              │ http {                                     │
              │     upstream server_pools {                │
              │         server 10.0.0.7:80 weight=1;       │
              │         server 10.0.0.8:80 weight=2;       │
              │         server 10.0.0.9:80 backup;         │
              │     }                                      │
              │                                            │
              │     server {                               │
              │         listen 80;                         │
              │         location / {                       │
              │             proxy_pass http://server_pools;│  ← 把请求扔进池塘
              │             proxy_set_header Host $host;   │
              │             proxy_set_header X-Forwarded-For $remote_addr;
              │         }                                  │
              │     }                                      │
              │ }                                          │
              └────────────────────────────────────────────┘
```

**注意**:`upstream` 块只能写在 `http {}` 之内、`server {}` 之外。

### 4.3 五种调度算法

| 算法 | 关键字 | 行为 | 适用场景 |
|---|---|---|---|
| 轮询(默认) | rr | 按顺序轮流分发,可结合 weight | 后端等价 |
| 加权轮询 | wrr(weight=N) | 权重越大命中越多 | 后端配置不均 |
| ip_hash | `ip_hash;` | 按客户端 IP 做哈希,**同一 IP 落同一台** | 简单会话粘滞 |
| least_conn | `least_conn;` | 选当前活跃连接数最少的后端 | 后端处理时间差异大 |
| fair(第三方) | `fair;` | 按后端响应时间分发 | 需 `nginx-upstream-fair` 模块 |
| url_hash / 一致性哈希 | 第三方 / Tengine | 同 URL 落同一台 | 静态/缓存命中率优化 |

#### 加权轮询效果

```
upstream server_pools{
    server 10.0.0.7:80 weight=1;
    server 10.0.0.8:80 weight=2;
}
```

```
请求序号  1  2  3  4  5  6  7  8  9
落点      08 07 08 08 07 08 08 07 08    (大致按 1:2 比例)
```

#### ip_hash 工作示意

```
 客户端A(IP1) ──┐                       ┌─→ web01
                ├─► hash(IP) → 选 web   ├─→ web02
 客户端B(IP2) ──┘                       └─→ web03

 同一 IP 的后续请求一直命中同一台,从而"伪"实现 Session 共享。
```

> 局限:NAT 出口下大量用户共享同一公网 IP,会全部压向同一台后端,负载严重不均。生产中**更推荐用集中式会话(Redis)** 解决 Session 共享。

#### least_conn

```
 收到新请求 → 看 upstream 里 active connections 最少的那台 → 投递
```

### 4.4 upstream `server` 行参数

| 参数 | 含义 | 默认 |
|---|---|---|
| `weight=N` | 权重 | 1 |
| `max_fails=N` | 多少次连续失败认定该后端"暂时不可用" | 1(企业建议 2–3) |
| `fail_timeout=Ns` | 探测失败之后,等多久再重试 / 失败窗口期 | 10s |
| `backup` | 备份节点,**只有所有非 backup 节点都失败时**才接管 | — |
| `down` | 标记下线,等同于"注释掉" | — |

#### backup 行为流程

```
        正常情况:
   请求 ─▶ rr 在 web01/web02 间轮转
            web03(backup) 静默,不接流量

        web01、web02 都被判定 fail 后:
   请求 ─▶ web03 接管全部流量
```

> ⚠️ 当 upstream 用 `ip_hash` 时,**节点不能配 `backup`,也不能直接配 `weight`**;`down` 仍可用。

### 4.5 健康检查参数(Tengine / `nginx_upstream_check_module`)

| 参数 | 说明 | 默认 |
|---|---|---|
| `check` | 开启主动健康检查 | — |
| `inter` | 两次健康检查的间隔(ms) | 2000 |
| `rise` | 连续成功多少次判为可用 | — |
| `fall` | 连续失败多少次判为宕机 | 3 |
| `maxconn` | 单后端最大并发 | — |

> 原生 Nginx 只在请求来了时被动探测;主动健康检查需要 Tengine 或额外模块。一致性 hash 同理,详见 `tengine.taobao.org`。

### 4.6 提升用户体验:`proxy_next_upstream`

```nginx
location / {
    proxy_pass http://server_pools;
    proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
}
```

含义:当上游返回上述状态码或连接出错,Nginx 自动把请求转给池中下一台。

### 【纠错】

- 原文表格里 fail_timeout 描述里出现 "如果 max_fails 是 5,它就检测 5 次,如果 5 次都是 502..." —— 实际上 `max_fails` 是**统计失败次数的阈值**,在 `fail_timeout` 内累计失败达到 `max_fails` 即判该后端不可用并暂停 `fail_timeout` 秒后再试,**不是简单的"检测 5 次"**。
- 原文 §4.5 "backup 备份节点:所有的节点都挂掉后**数据**才会请求 web01"—— 句意混乱。正确表达:**所有非 backup 节点都被判定为失败后,Nginx 才把请求转发到 backup 节点**。
- 原文 §4.11 关于 fair 的"Nginx 本身不支持 fair"——对的,**但还需要补充**:它依赖的是 `ngx_http_upstream_fair_module` 第三方模块,不是 Tengine 自带。
- 原文 §4.13 字段名 `proxy_set_header X-ForwardedFor`(漏了一个 `-`)是排版错误,正确为 **`X-Forwarded-For`**;HTTP 头实际还有更新的标准 `Forwarded`(RFC 7239),云原生场景下一并设置。

---

## 5. 透传真实信息:proxy_set_header

### 5.1 反向代理"丢失"的两类信息

1. **Host 头**:不显式传递,后端的 `server_name` 命中默认 server,多虚拟主机场景必错。
2. **客户端真实 IP**:不传,后端日志全是 lb01 的 IP。

### 5.2 经典模板

```nginx
location / {
    proxy_pass http://server_pools;

    # 让后端知道原始 Host
    proxy_set_header Host              $host;

    # 真实客户端 IP(单层代理)
    proxy_set_header X-Real-IP         $remote_addr;

    # 多级代理的链路记录
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;

    # 透传原始协议(http/https),让后端识别 HTTPS 是否被卸载
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### 5.3 数据流示意

```
 用户 IP=1.2.3.4
       │  Host: www.etiantian.org
       ▼
   ┌──────────┐  生成头:
   │   lb01   │     Host: www.etiantian.org
   │ 10.0.0.5 │     X-Real-IP: 1.2.3.4
   └────┬─────┘     X-Forwarded-For: 1.2.3.4
        │
        ▼
   ┌──────────┐    后端日志中可拿到真实 IP:
   │  web01   │     log_format 里 $http_x_forwarded_for
   │ 10.0.0.7 │
   └──────────┘
```

后端 `log_format` 里把 `"$http_x_forwarded_for"` 加在末尾,看到的形如:

```
10.0.0.5 - - [30/Oct/2017:12:36:10 +0800] "GET / HTTP/1.0" 200 10 "-"
   "Mozilla/5.0 ..." "1.2.3.4"
```

### 5.4 `$proxy_add_x_forwarded_for` 的细节

- 该变量等价于:**入站的 `X-Forwarded-For` 头(若有) + 逗号 + `$remote_addr`**。
- 多级代理场景能保留完整链路:`真实 IP, 边缘代理 IP, ...`。
- 直接写 `proxy_set_header X-Forwarded-For $remote_addr;` 会**丢失中间链路**,只剩"上一跳"。

### 【纠错】

- 原文 §5.2 直接写 `proxy_set_header X-Forwarded-For $remote_addr;`,该写法**会覆盖原 XFF**;**推荐改为** `$proxy_add_x_forwarded_for`,在多层代理(WAF→CDN→边缘 LB→内部 LB→app)拓扑里才不会丢链路。

---

## 6. 反向代理相关 proxy 参数速查

| 参数 | 含义 |
|---|---|
| `proxy_pass http://pool;` | 把请求转给 upstream 池 |
| `proxy_set_header` | 改/加请求头给后端 |
| `client_body_buffer_size` | 客户端请求体缓冲区大小 |
| `proxy_connect_timeout` | 与后端建立 TCP / 握手超时 |
| `proxy_send_timeout` | 给后端发完一次数据的最大间隔 |
| `proxy_read_timeout` | 等后端响应数据的最大间隔(常被忽视,WebSocket 必调) |
| `proxy_buffer_size` | 单个响应头缓冲块 |
| `proxy_buffers` | 响应体缓冲块的数量与大小 |
| `proxy_busy_buffers_size` | 高负载时允许已忙的缓冲区,推荐 `proxy_buffers * 2` |
| `proxy_temp_file_write_size` | 写入磁盘临时文件的块大小 |

### 排错四步走

1. 在 lb01 上直接 `curl -H 'Host:xxx' 后端IP`,验证后端独立服务。
2. 在 lb01 上 `curl http://127.0.0.1/`,验证 Nginx 本机配置。
3. 浏览器 + Host 解析,验证全链路。
4. 检查缓存 / DNS / hosts。

### 【纠错】

- 原文表格里 `proxy_trmp_file_write_size`,**拼写错误**,正确为 `proxy_temp_file_write_size`。

---

## 7. 按 URI 转发(7 层动静分离)

### 7.1 业务规划

| URI | 后端 | 目录 | 类型 |
|---|---|---|---|
| `/upload` | 10.0.0.8:80 | html/www/upload | upload 服务 |
| `/static` | 10.0.0.7:80 | html/www/static | 静态资源 |
| `/` | 10.0.0.9:80 | html/www | 默认(动态) |

### 7.2 完整配置

```nginx
http {
    upstream upload_pools  { server 10.0.0.8:80; }
    upstream static_pools  { server 10.0.0.7:80; }
    upstream default_pools { server 10.0.0.9:80; }

    server {
        listen 80;
        server_name www.etiantian.org;

        location /upload/ {
            proxy_pass http://upload_pools;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /static/ {
            proxy_pass http://static_pools;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location / {
            proxy_pass http://default_pools;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### 7.3 调用流程

```
              www.etiantian.org/...
                       │
                       ▼
                ┌────────────┐
                │   lb01     │
                └─────┬──────┘
                      │ 按 URI 匹配 location
        ┌─────────────┼──────────────┐
        │             │              │
        ▼             ▼              ▼
   /upload/        /static/          /
   upload_pools   static_pools    default_pools
   10.0.0.8        10.0.0.7        10.0.0.9
```

> 7 层负载的本质就是这种**按 URL/URI/HTTP 头来路由**的能力,LVS(4 层)做不到。

### 7.4 为什么这样做

- 同一域名对外:用户感知一个站点。
- 静态资源单独跑(可加 CDN/缓存),减压主业务。
- 上传/下载、API、页面可按特性分拆到不同后端池。

---

## 8. 按 User-Agent 转发(分流 PC/移动)

### 8.1 思路

通过 `if ($http_user_agent ~* '...')` 设置一个变量,然后 `proxy_pass` 到不同池。

### 8.2 配置示例

```nginx
upstream pc_pools     { server 10.0.0.7:80; }
upstream mobile_pools { server 10.0.0.8:80; }
upstream other_pools  { server 10.0.0.9:80; }

server {
    listen 80;
    server_name www.etiantian.org;

    location / {
        if ($http_user_agent ~* "MSIE|Chrome|Firefox") {
            proxy_pass http://pc_pools;
            break;
        }
        if ($http_user_agent ~* "iPhone|Android|iPad") {
            proxy_pass http://mobile_pools;
            break;
        }
        proxy_pass http://other_pools;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 8.3 测试

```bash
curl -A "iPhone"  http://www.etiantian.org/   →  落到 mobile_pools
curl -A "Chrome"  http://www.etiantian.org/   →  落到 pc_pools
curl -A "curl/x"  http://www.etiantian.org/   →  落到 other_pools
```

### 【纠错】

- 原文 §7.1 用 `user_agent` 做转发只给了占位,这里补完整。**注意**:`if` 在 `location` 中是"邪恶的"(参考 Nginx Wiki *If is Evil*),复杂判断推荐用 `map`:

  ```nginx
  map $http_user_agent $backend_pool {
      default                 other_pools;
      ~*MSIE|Chrome|Firefox   pc_pools;
      ~*iPhone|Android|iPad   mobile_pools;
  }
  server {
      location / { proxy_pass http://$backend_pool; }
  }
  ```

---

## 9. 按扩展名转发(动静分离另一种写法)

```nginx
server {
    listen 80;
    server_name www.etiantian.org;

    location ~* \.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {
        proxy_pass http://static_pools;
    }
    location ~* \.(php|jsp|cgi|do)$ {
        proxy_pass http://dynamic_pools;
    }
    location / {
        proxy_pass http://default_pools;
    }
}
```

执行优先级(摘录,详见 nginx 企业应用文档):

```
精确匹配  =        →  ^~ 前缀     →  正则 ~ / ~*  →  普通前缀   →  /
```

### 【纠错】

- 原文 §8 只放了占位代码,这里给出工程上常见的可直接复用的版本。

---

## 10. 两台 lb 与备份(为高可用做铺垫)

负载均衡器本身就是单点。生产中 lb01 + lb02 + Keepalived 形成 HA:

```
                              ┌────────────────┐
                          VIP │  10.0.0.100    │
                              └──────┬─────────┘
              ┌──────────────────────┴──────────────────────┐
              │                                              │
        ┌─────────┐    Keepalived (VRRP)    ┌─────────┐
        │  lb01   │ ◀────────────────────▶ │  lb02   │
        │ Master  │                          │ Backup  │
        └────┬────┘                          └────┬────┘
             │              (同样的 upstream/proxy 配置)         │
             └────────────────┬─────────────────────────────────┘
                              ▼
                ┌──────────────────────────┐
                │  web01 / web02 / web03   │
                └──────────────────────────┘
```

要点:

- Master/Backup 共享 VIP,客户端只面对 VIP。
- Master 宕机后 VRRP 把 VIP 漂到 Backup。
- 与 Nginx `backup`(upstream 级别)是**两层备份**,不要混淆:
  - upstream `backup` = 后端 Web 的备份。
  - Keepalived `BACKUP` = 负载均衡器自身的备份。

---

## 11. 7 层 vs 4 层 负载均衡

| 维度 | 7 层(Nginx HTTP / HAProxy http) | 4 层(LVS / Nginx stream / HAProxy tcp) |
|---|---|---|
| 工作层 | 应用层(HTTP) | 传输层(TCP/UDP) |
| 能否按 URL/Host/Cookie/UA 路由 | 可以 | 不可以 |
| TLS 终止 | 通常在此完成 | 一般直接透传 TLS |
| 连接 | client↔lb、lb↔web 各一条 | 主要做包转发,效率更高 |
| 性能 | 中(单机几十 K QPS) | 极高(单机百万 QPS) |
| 健康检查粒度 | HTTP 状态码、URL 探活 | TCP 握手成功即认为存活 |
| 典型搭配 | Nginx 反代 + 应用集群 | LVS-DR 外层 + Nginx 7 层 |

> 实际大型网站常见组合:**LVS(4 层 + Keepalived)** → **Nginx(7 层)** → **后端应用** 三层架构。

---

## 12. 整篇主要纠错汇总

| 位置 | 原文 | 应为 |
|---|---|---|
| §3.3 `./configure` 用户名 | `--user=nginx --group=nginx` 但 `useradd www` | 二者保持一致(本文统一 `www`) |
| §3.3 软链 | `ln -s ... /application/ngin` | `ln -s ... /application/nginx` |
| §3.4 IP 与 web 编号 | 表格与测试结果自相矛盾 | 本文统一 web01=10.0.0.7 / web02=10.0.0.8 / web03=10.0.0.9 |
| §4.5 backup 解释 | "所有节点都挂掉后**数据**才会请求 web01" | "所有非 backup 节点失败后,请求才转给 backup 节点" |
| §4.13 表格 | `X-ForwardedFor` | `X-Forwarded-For` |
| §4.13 表格 | `proxy_trmp_file_write_size` | `proxy_temp_file_write_size` |
| §5.2 透传真实 IP | `X-Forwarded-For $remote_addr;` | 推荐 `$proxy_add_x_forwarded_for` |
| §4.5 max_fails 描述 | "检测 5 次" | 是阈值,在 fail_timeout 窗口内累计达阈值即下线 |
| §1.4 nginx 4 层支持 | "1.9 之后支持 4 层" | 1.9.0 引入 stream,需编译时 `--with-stream`,1.9.13 起默认启用 |
| §1.5 反向代理 vs 数据转发 | 未明确区别 | 7 层应用代理 vs 4 层转发(LVS 等) |
