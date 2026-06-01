# Nginx 知识点汇总整理

> 基于原始文档（01～12）梳理，已标注原文错误并给出正确内容；多组件交互场景附 ASCII 流程图。

---

## 目录

1. [Nginx 应用概述（01）](#1-nginx-应用概述)
2. [Nginx 基本配置（02）](#2-nginx-基本配置)
3. [Nginx 静态服务（03）](#3-nginx-静态服务)
4. [Nginx 代理与负载均衡（04）](#4-nginx-代理与负载均衡)
5. [Nginx 缓存服务（05）](#5-nginx-缓存服务)
6. [Nginx Rewrite（06）](#6-nginx-rewrite)
7. [Nginx HTTPS（07）](#7-nginx-https)
8. [LNMP 动态网站（08）](#8-lnmp-动态网站)
9. [LNMT 动态网站（09）](#9-lnmt-动态网站)
10. [Nginx + Lua 实战（10）](#10-nginx--lua-实战)
11. [Nginx 性能优化（11）](#11-nginx-性能优化)
12. [Nginx 常见问题（12）](#12-nginx-常见问题)

---

## 1. Nginx 应用概述

### 1.1 基本简述

- Nginx 是开源、高性能、可靠的 HTTP 中间件与代理服务。
- 主要应用场景：静态处理、反向代理、负载均衡、资源缓存。
- 常见同类 HTTP 服务：HTTPD（Apache）、IIS（微软）、GWS（Google）、OpenResty、Tengine（淘宝）。

### 1.2 优秀特性

#### IO 多路复用（epoll）

传统串行 IO 会阻塞，多线程方案消耗大；IO 多路复用允许多个文件描述符在**同一个线程内**并发交替顺序完成，复用同一线程。

实现方式对比：

| 方式   | 特点                              |
|--------|----------------------------------|
| select | 有最大文件描述符数量限制；线性遍历，效率低 |
| poll   | 无最大连接数限制；仍为线性遍历         |
| epoll  | FD 就绪时由系统回调直接放入；最大连接无限制；效率最高 |

> **原文错误**：原文将 epoll 写为 "Epool"，正确拼写为 **epoll**。

#### 轻量级

- 功能模块少，代码模块化，启动速度快。

#### CPU 亲和（affinity）

将每个 worker 进程绑定到固定 CPU 核心，减少 CPU cache miss，提升性能。

#### sendfile 零拷贝

传统文件传输经过 4 次拷贝（硬盘 → 内核 buf → 用户 buf → socket 缓冲区 → 协议引擎），需要用户态/内核态切换。

sendfile 在内核直接完成两个文件描述符间的数据传输，流程如下：

```
传统文件传输：
  硬盘 ──DMA──▶ 内核buf ──CPU──▶ 用户buf ──CPU──▶ Socket内核buf ──DMA──▶ 协议栈
              (4次拷贝，2次上下文切换)

sendfile 零拷贝：
  硬盘 ──DMA──▶ 内核buf ──CPU──▶ Socket内核buf ──DMA──▶ 协议栈
              (2次拷贝，0次用户态/内核态切换)
```

### 1.3 安装目录结构

| 路径 | 类型 | 作用 |
|------|------|------|
| `/etc/nginx/nginx.conf` | 配置文件 | Nginx 主配置文件 |
| `/etc/nginx/conf.d/*.conf` | 配置文件 | 虚拟主机配置 |
| `/etc/nginx/mime.types` | 配置文件 | Content-Type 与扩展名映射 |
| `/usr/lib/systemd/system/nginx.service` | 配置文件 | systemd 守护进程管理 |
| `/etc/logrotate.d/nginx` | 配置文件 | 日志轮切配置 |
| `/usr/sbin/nginx` | 命令 | Nginx 管理命令 |
| `/usr/lib64/nginx/modules` | 目录 | 模块目录 |
| `/usr/share/nginx/html` | 目录 | 默认站点目录 |
| `/var/cache/nginx` | 目录 | 缓存目录 |
| `/var/log/nginx` | 目录 | 日志目录 |

### 1.4 常用内置变量

| 变量 | 说明 |
|------|------|
| `$uri` | 当前请求 URI（不含参数） |
| `$request_uri` | 请求完整 URI（含参数） |
| `$host` | 请求报文中 Host 首部 |
| `$remote_addr` | 客户端 IP |
| `$remote_port` | 客户端端口 |
| `$request_method` | 请求方法（GET/POST/PUT） |
| `$server_addr` | 服务器地址 |
| `$server_name` | 服务器名称 |
| `$server_port` | 服务器端口 |
| `$scheme` | 请求协议（http/https） |
| `$http_user_agent` | 客户端 User-Agent |
| `$http_x_forwarded_for` | 经过代理时的客户端真实 IP 链 |
| `$document_root` | 当前请求映射到的 root 目录 |

> **原文错误**：原文写的是 `depict_root`，正确变量名为 **`$document_root`**。

---

## 2. Nginx 基本配置

### 2.1 配置文件层级

```
nginx.conf
├── Main 层（全局配置）
│   ├── user
│   ├── worker_processes
│   ├── error_log
│   └── pid
├── events 层
│   ├── worker_connections
│   └── use（epoll/select/poll）
└── http 层
    └── server 层（可多个，虚拟主机）
        └── location 层（可多个，路径匹配）
```

### 2.2 日志配置

```nginx
# 日志格式定义（http 层）
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

# 使用日志格式（server/location 层）
access_log /var/log/nginx/access.log main;
```

> **原文错误**：原文日志变量中写了 `iktatus`，正确变量名为 **`$status`**（HTTP 响应状态码）。

主要日志变量：

| 变量 | 说明 |
|------|------|
| `$remote_addr` | 客户端 IP |
| `$time_local` | 本地时间 |
| `$request` | 请求行（方法 + URI + 协议版本） |
| `$status` | HTTP 响应状态码 |
| `$body_bytes_sent` | 响应 body 大小（字节） |
| `$http_referer` | 上一页面地址，可用于防盗链 |
| `$http_user_agent` | 客户端浏览器/设备信息 |

### 2.3 状态监控（stub_status）

```nginx
location /mystatus {
    stub_status on;
    access_log off;
}
```

输出字段说明：
- `Active connections`：当前活跃连接数
- `server`：处理握手总次数
- `accepts`：接收总连接数
- `handled requests`：处理总请求数
- `请求丢失数 = 握手数 - 连接数`
- `Reading`：Nginx 正在读取请求的连接数
- `Writing`：Nginx 正在写响应的连接数
- `Waiting`：Keep-alive 长连接下等待中的连接数

### 2.4 访问限制

#### 连接频率限制（limit_conn_module）

```nginx
# http 层定义共享内存
limit_conn_zone $binary_remote_addr zone=conn_zone:10m;

# server/location 层限制同一IP并发连接数
limit_conn conn_zone 1;
```

#### 请求频率限制（limit_req_module）

```nginx
# http 层定义
limit_req_zone $binary_remote_addr zone=req_zone:10m rate=1r/s;

# location 层应用
limit_req zone=req_zone;                         # 超出则拒绝
limit_req zone=req_zone burst=3 nodelay;         # 超出可缓冲，缓冲满再拒绝
```

> **原文错误**：原文请求限制语法写成 `Syntax: limit_conn zone number [burst=number] [nodelay];`，这是错误的。**连接限制**和**请求限制**是两个不同指令：
> - 连接限制：`limit_conn zone number;`
> - 请求限制：`limit_req zone=zone [burst=number] [nodelay];`（正确语法）

**为什么请求限制比连接限制更有效？**

HTTP 1.1 支持在一个 TCP 连接上发送多个请求（Keep-alive 复用），因此限制连接数不能防止单连接内的请求轰炸；请求限制直接针对请求维度，颗粒度更细。

### 2.5 访问控制

#### 基于 IP 的访问控制

```nginx
# 拒绝特定 IP，允许其余
location ~ ^/1.html {
    deny 192.168.56.1;
    allow all;
}

# 允许特定网段，拒绝其余
location / {
    allow 192.168.56.0/24;
    deny all;
}
```

**局限性**：当客户端使用代理时，`$remote_addr` 获取的是代理 IP，无法获取真实客户端 IP。

解决方案：
1. 代理服务器开启 `X-Forwarded-For`，后端读取该头部。
2. 结合 `geo` 模块处理。
3. 通过 HTTP 自定义变量传递。

#### 基于用户认证

```nginx
auth_basic "Auth access Blog Input your Passwd!";
auth_basic_user_file /etc/nginx/auth_conf;
```

生成密码文件：`htpasswd -c /etc/nginx/auth_conf username`

局限性：密码依赖文件，管理不便。可改用 Nginx + LDAP 或 Nginx + Lua 实现高效验证。

### 2.6 虚拟主机

```nginx
# 基于域名
server {
    listen 80;
    server_name www.example.com;
    root /soft/code/www;
}

# 基于端口
server { listen 8001; ... }
server { listen 8002; ... }

# 域名别名（多个域名指向同一站点）
server {
    listen 80;
    server_name www.example.com example.com;
}
```

---

## 3. Nginx 静态服务

### 3.1 静态资源类型

| 类型 | 格式 |
|------|------|
| 浏览器渲染 | HTML、CSS、JS |
| 图片 | JPEG、GIF、PNG |
| 视频 | FLV、MP4 |
| 文件 | TXT 及任意可下载文件 |

### 3.2 传输性能优化

```nginx
sendfile on;         # 零拷贝传输，提升文件读取效率
tcp_nopush on;       # 在 sendfile 开启时生效，批量发送数据包，提高传输效率
tcp_nodelay on;      # 在 keepalive 长连接下，禁用 Nagle 算法，提高实时性
```

三者关系：`sendfile` 是基础，`tcp_nopush` 提升批量传输效率（适合静态资源），`tcp_nodelay` 提升实时性（适合动态资源）。

### 3.3 gzip 压缩

```nginx
gzip on;                       # 开启压缩
gzip_comp_level 2;             # 压缩级别（1-9），越高越耗 CPU
gzip_http_version 1.1;         # 压缩所用 HTTP 版本（主流 1.1）
gzip_types text/plain application/json text/css application/javascript
           application/xml image/jpeg image/gif image/png;
gzip_static on;                # 预读已压缩的 .gz 文件
```

> **原文错误**：原文 location 正则 `~.* $jpg|gif|png$ $` 写法错误，正确应为 `~.*\.(jpg|gif|png)$`，同理 `~.* $.txt|xml$ $` 应为 `~.*\.(txt|xml)$`。

### 3.4 浏览器缓存

缓存校验流程：

```
浏览器请求
    │
    ▼
有本地缓存？
  │           │
  否          是
  │           ▼
  │     缓存是否过期？（Expires / Cache-Control: max-age）
  │       │           │
  │       否          是
  │       │           ▼
  │       │     向服务器验证（ETag / Last-Modified）
  │       │       │           │
  │       │    未修改(304)  已修改(200)
  │       │       │           │
  ▼       ▼       ▼           ▼
请求服务器  使用缓存  使用缓存    更新缓存并展示
```

```nginx
# 静态资源缓存配置
location ~ .*\.(js|css|html)$ {
    root /soft/code/js;
    expires 1h;
}
location ~ .*\.(jpg|gif|png)$ {
    root /soft/code/images;
    expires 7d;
}

# 开发环境禁止缓存
location ~ .*\.(css|js|json|mp4|html)$ {
    add_header Cache-Control no-store;
    add_header Pragma no-cache;
}
```

### 3.5 跨域访问（CORS）

浏览器出于安全默认禁止跨域请求（防 CSRF 攻击）。

```nginx
location ~ .*\.(html|htm)$ {
    add_header Access-Control-Allow-Origin http://www.example.com;
    add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
    root /soft/code;
}
```

### 3.6 防盗链

原理：通过检测 HTTP `Referer` 请求头，判断请求来源是否合法。

```nginx
location ~ .*\.(jpg|gif|png)$ {
    valid_referers none blocked www.example.com;
    if ($invalid_referer) {
        return 403;
    }
    root /soft/code/images;
}
```

- `none`：允许 Referer 为空（直接访问）
- `blocked`：允许 Referer 不以 http/https 开头
- `www.example.com`：允许指定域名

---

## 4. Nginx 代理与负载均衡

### 4.1 正向代理 vs 反向代理

| 类型 | 代理对象 | 场景 |
|------|----------|------|
| 正向代理 | 客户端 | 客户端访问受限资源（翻墙、内网上网） |
| 反向代理 | 服务端 | 隐藏后端服务器，统一入口 |

```
正向代理：  客户端 ←→ [代理] ──→ 服务端
反向代理：  客户端 ──→ [代理] ←→ 服务端
```

### 4.2 代理配置语法

```nginx
# 基本反向代理
location / {
    proxy_pass http://127.0.0.1:8080;
    include proxy_params;
}
```

`proxy_params` 推荐配置（保存为独立文件复用）：

```nginx
proxy_redirect default;
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_connect_timeout 30;
proxy_send_timeout 60;
proxy_read_timeout 60;

proxy_buffer_size 32k;
proxy_buffering on;
proxy_buffers 4 128k;
proxy_busy_buffers_size 256k;
proxy_max_temp_file_size 256k;
```

### 4.3 负载均衡

Nginx 属于**七层（应用层）SLB**（对比四层的 LVS）。

```nginx
upstream backend {
    server 192.168.1.10:8080 weight=5;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080 backup;
}
server {
    location / {
        proxy_pass http://backend;
        include proxy_params;
    }
}
```

#### 后端服务器状态参数

| 状态 | 说明 |
|------|------|
| `down` | 暂时不参与负载均衡 |
| `backup` | 备用服务器，主服务器不可用时启用 |
| `max_fails` | 允许请求失败的次数 |
| `fail_timeout` | max_fails 失败后暂停服务时间 |
| `max_conns` | 限制最大接收连接数 |

#### 调度策略

| 策略 | 配置 | 说明 |
|------|------|------|
| 轮询（默认） | 无需配置 | 按时间顺序依次分配 |
| 加权轮询 | `weight=N` | weight 越大，分配比例越高 |
| ip_hash | `ip_hash;` | 同一 IP 固定到同一后端（解决 session 问题，但代理访问会失衡） |
| url_hash | `hash $request_uri;` | 相同 URL 固定到同一后端（缓存命中率高） |
| least_conn | `least_conn;` | 连接数最少的后端优先 |
| 自定义 hash | `hash $key;` | 自定义 key 做 hash |

#### 四层负载均衡（stream 模块）

```nginx
stream {
    upstream ssh_proxy {
        hash $remote_addr consistent;
        server 192.168.56.103:22;
    }
    upstream mysql_proxy {
        hash $remote_addr consistent;
        server 192.168.56.103:3306;
    }
    server {
        listen 6666;
        proxy_pass ssh_proxy;
    }
    server {
        listen 5555;
        proxy_pass mysql_proxy;
    }
}
```

> `stream` 块必须在 `main` 层（与 `http` 平级），不能嵌套在 `http` 内。

### 4.4 动静分离

**流程图：**

```
客户端请求
     │
     ▼
  代理 Nginx (192.168.69.112:80)
     │
     ├──── 匹配 *.png|jpg|gif ───────▶ upstream static (Nginx:80)
     │                                        │
     │                                        ▼
     │                                   /soft/code/images/
     │
     └──── 匹配 *.jsp ─────────────────▶ upstream java (Tomcat:8080)
                                               │
                                               ▼
                                          Java 动态处理
```

配置示例：

```nginx
upstream static { server 192.168.69.113:80; }
upstream java   { server 192.168.69.113:8080; }

server {
    listen 80;
    server_name 192.168.69.112;

    location / {
        root /soft/code;
        index index.html;
    }
    location ~ .*\.(png|jpg|gif)$ {
        proxy_pass http://static;
        include proxy_params;
    }
    location ~ .*\.jsp$ {
        proxy_pass http://java;
        include proxy_params;
    }
}
```

动静分离优点：动态服务宕机不影响静态资源访问；静态服务器可独立扩容。

---

## 5. Nginx 缓存服务

### 5.1 缓存类型

| 类型 | 位置 | 说明 |
|------|------|------|
| 服务端缓存 | 后端服务器 | 应用层自身缓存（Redis/Memcached） |
| 代理缓存 | Nginx Proxy | 代理层缓存后端响应内容 |
| 客户端缓存 | 浏览器 | 浏览器本地缓存（expires/Cache-Control） |

### 5.2 proxy_cache 配置

**proxy_cache 请求流程：**

```
客户端请求
     │
     ▼
Nginx Proxy
     │
     ├─── 缓存命中（HIT）──────────────────▶ 直接返回缓存内容
     │
     └─── 缓存未命中（MISS）
               │
               ▼
          后端 Web 服务器
               │
               ▼
          响应内容写入缓存
               │
               ▼
          返回客户端
```

```nginx
# http 层定义缓存路径
proxy_cache_path /soft/cache
    levels=1:2                  # 两层目录分级
    keys_zone=code_cache:10m    # 共享内存名:大小（1m 可存约 8000 个 key）
    max_size=10g                # 缓存最大容量
    inactive=60m                # 60 分钟未访问则清理
    use_temp_path=off;          # 关闭临时文件（提升性能）

server {
    listen 80;
    location / {
        proxy_pass http://cache;
        proxy_cache code_cache;
        proxy_cache_valid 200 304 12h;     # 200/304 状态缓存 12 小时
        proxy_cache_valid any 10m;          # 其余状态缓存 10 分钟
        proxy_cache_key $host$uri$is_args$args;   # 缓存 key
        add_header Nginx-Cache "$upstream_cache_status";  # 响应头观察命中情况
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        include proxy_params;
    }
}
```

> **原文错误**：原文中 `proxy_cache_key $host$uri$is_argsisease;` 末尾 `isease` 是排版错误，正确应为 `$is_args$args`（`$is_args` 在有参数时输出 `?`，`$args` 为参数值）。

### 5.3 缓存清理

**方式一：直接删除缓存目录**

```bash
rm -rf /soft/cache/*
```

**方式二：ngx_cache_purge 模块（需编译安装）**

```nginx
location ~ /purge(/.*) {
    allow 127.0.0.1;
    allow 192.168.69.0/24;
    deny all;
    proxy_cache_purge code_cache $host$1$is_args$args;
}
```

访问 `http://domain/purge/path` 即可清理对应缓存。

### 5.4 部分页面不缓存

```nginx
server {
    # 匹配特定 URI 设置不缓存标志
    if ($request_uri ~ ^/(url3|login|register|password)) {
        set $cookie_nocache 1;
    }
    location / {
        proxy_pass http://cache;
        proxy_cache code_cache;
        proxy_cache_key $host$uri$is_args$args;
        proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
        proxy_no_cache $http_pragma $http_authorization;
        ...
    }
}
```

---

## 6. Nginx Rewrite

### 6.1 配置语法

```
rewrite regex replacement [flag];
```

常用正则：

| 符号 | 说明 |
|------|------|
| `.` | 匹配除换行符外的任意字符 |
| `?` | 前面元素重复 0 次或 1 次 |
| `+` | 前面元素重复 1 次或多次 |
| `*` | 前面元素重复 0 次或多次 |
| `\d` | 匹配数字 |
| `^` | 字符串开始 |
| `$` | 字符串结束 |
| `{n}` | 重复 n 次 |
| `{n,}` | 重复 n 次或更多 |
| `[c]` | 匹配单个字符 c |
| `[a-z]` | 匹配 a-z 任意小写字母 |
| `()` | 分组，可用 `$1`、`$2` 引用 |
| `\` | 转义字符 |

> **原文错误**：原文正则表达式表格中把 `*` 的描述写成"最少连接数，那个机器连接数少就分发"，这是 Nginx 负载均衡 `least_conn` 策略的描述，与正则 `*` 完全无关。正确描述：**`*` 表示前面的元素重复 0 次或更多次**。

### 6.2 Rewrite Flag

| Flag | 说明 |
|------|------|
| `last` | 停止当前 rewrite 检测，重新以新 URI 发起内部请求（会继续匹配 location） |
| `break` | 停止当前 rewrite 检测，不再匹配 location，在当前 root 下寻找资源 |
| `redirect` | 返回 302 临时重定向（地址栏改变，Nginx 重启后失效） |
| `permanent` | 返回 301 永久重定向（地址栏改变，浏览器缓存，Nginx 重启后仍生效） |

**last vs break 对比：**

```
请求 /break
     │
     ▼
rewrite ^/break /test/ break;
     │
     ▼
  不再匹配 location，直接在 root 目录寻找 /test/ 目录
  （若目录不存在则 404）

请求 /last
     │
     ▼
rewrite ^/last /test/ last;
     │
     ▼
  停止当前 rewrite，重新以 /test/ 发起内部请求
  │
  ▼
  匹配 location /test/ { return 200 '...'; }
  │
  ▼
  返回 {"status":"success"}
```

**redirect vs permanent 对比：**

- `redirect`（302）：Nginx 关闭后，浏览器不会缓存跳转目标，访问原 URL 会失败。
- `permanent`（301）：Nginx 关闭后，浏览器缓存 301，会直接跳转到目标 URL，不再访问 Nginx。

### 6.3 匹配优先级

1. 执行 `server` 块的 `rewrite` 指令
2. 执行 `location` 匹配
3. 执行选中 `location` 块内的 `rewrite` 指令

### 6.4 使用场景示例

```nginx
# URL 美化（将目录风格 URL 映射到实际文件）
location / {
    rewrite ^/course-(\d+)-(\d+)-(\d+)\.html /course/$1/$2/course_$3.html break;
}

# 根据 User-Agent 重定向
if ($http_user_agent ~* Chrome) {
    rewrite ^/nginx http://kt.example.com/index.html redirect;
}

# HTTP 强制跳转 HTTPS（推荐写法）
server {
    listen 80;
    server_name example.com;
    rewrite ^ https://$server_name$request_uri? redirect;
}
```

---

## 7. Nginx HTTPS

### 7.1 为什么需要 HTTPS

- HTTP 明文传输，中间人可窃取、篡改数据。
- HTTPS = HTTP + TLS/SSL，提供加密传输和身份验证。

### 7.2 自签证书配置

```bash
# 生成自签证书（不受浏览器信任，用于测试）
openssl req -days 36500 -x509 -sha256 -nodes \
    -newkey rsa:2048 -keyout server.key -out server.crt
```

```nginx
server {
    listen 443 ssl;                                  # 推荐写法（ssl on 已在 1.15.0+ 废弃）
    server_name localhost;
    ssl_certificate     ssl_key/server.crt;
    ssl_certificate_key ssl_key/server.key;
    ssl_session_cache   shared:SSL:10m;              # 注意：shared 不是 share
    ssl_session_timeout 10m;
    ssl_protocols       TLSv1.2 TLSv1.3;            # 建议去掉 TLSv1.0/1.1（不安全）
    ssl_ciphers         ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        root /soft/code;
    }
}

# HTTP 强制跳转 HTTPS
server {
    listen 80;
    server_name localhost;
    rewrite ^ https://$server_name$request_uri? redirect;
}
```

> **原文错误**：
> 1. 原文 `ssl_session_cache share:SSL:10m;` 中 `share` 应为 **`shared`**。
> 2. 原文 `rewrite^(.*)` 缺少空格，应为 `rewrite ^ ...` 或 `rewrite ^(.*) ...`。
> 3. 原文使用 `ssl on;` 搭配 `listen 443;`，在 Nginx 1.15.0+ 中 `ssl on` 已废弃，推荐使用 `listen 443 ssl;`。

### 7.3 苹果 ATS 要求

1. TLS 版本 ≥ 1.2（openssl ≥ 1.0.2）
2. 证书哈希算法 ≥ SHA256
3. 公钥算法：RSA ≥ 2048 位 或 ECC ≥ 256 位
4. 使用前向加密（ECDHE 等）

---

## 8. LNMP 动态网站

### 8.1 架构组成

LNMP = Linux + Nginx + MySQL/MariaDB + PHP

### 8.2 Nginx + PHP 请求流程

```
用户发送 HTTP 请求
        │
        ▼
   Nginx 服务器
        │
        ├─── 静态文件（*.html, *.jpg 等）────▶ 直接返回
        │
        └─── 动态文件（*.php）
                  │
                  ▼
            FastCGI 客户端
            (fastcgi_pass 127.0.0.1:9000)
                  │
                  ▼
            PHP-FPM 进程管理器
                  │
                  ▼
            PHP Worker（解析执行 PHP）
                  │
                  ├─── 查询博文/内容 ──▶ MySQL 数据库
                  │
                  └─── 查询图片/附件 ──▶ NFS 存储
                  │
                  ▼
            返回 HTML 给 Nginx
                  │
                  ▼
         Nginx 返回响应给用户
```

### 8.3 Nginx FastCGI 配置

```nginx
server {
    listen 80;
    server_name _;
    root /soft/code;
    index index.php index.html;

    location ~ \.php$ {
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### 8.4 PHP 配置优化（php.ini）

| 参数 | 建议值 | 说明 |
|------|--------|------|
| `expose_php` | `Off` | 隐藏 PHP 版本号 |
| `display_errors` | `Off` | 关闭错误输出（生产环境） |
| `log_errors` | `On` | 记录错误到日志 |
| `error_log` | `/var/log/php_error.log` | 错误日志路径 |
| `memory_limit` | `128M` 以上 | 单脚本最大内存 |
| `upload_max_filesize` | `16M` 或 `32M` | 文件上传大小限制 |
| `allow_url_fopen` | `Off` | 禁止远程执行 PHP Shell |
| `date.timezone` | `Asia/Shanghai` | 时区 |

> **原文错误**：
> 1. 原文写 `sql.safe_mode = Off`，`sql.safe_mode` 实际上是 MySQL 的配置项，PHP 的安全模式配置是 `safe_mode = Off`（但该指令在 PHP 5.4 中已移除）。现代 PHP 7.x+ 已无此配置，建议通过 `disable_functions` 禁用危险函数代替。
> 2. `mysql_connect()` 函数在 PHP 7.0 中已删除，测试代码中的 `mysql_connect` 应改为 **`mysqli_connect()`** 或使用 **PDO**。
> 3. 原文说"安装 Mariadb"，但实际安装步骤用的是 MySQL 官方 `mysql57-community-release` 源安装的 **MySQL 5.7**，不是 MariaDB（二者是不同的数据库）。

### 8.5 PHP-FPM 优化（推荐配置，4核16G）

```ini
[global]
pid = /var/run/php-fpm.pid
error_log = /var/log/php/php-fpm.log
log_level = warning
rlimit_files = 65535
events.mechanism = epoll

[www]
user = nginx
group = nginx
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 512      # 同一时刻最大子进程数
pm.start_servers = 10      # 启动时初始进程数
pm.min_spare_servers = 10  # 最小空闲进程数
pm.max_spare_servers = 30  # 最大空闲进程数
pm.max_requests = 2048     # 每个进程最大处理请求数（防内存泄漏）
pm.process_idle_timeout = 15s
request_slowlog_timeout = 5s
slowlog = /var/log/php/php-slow.log
```

---

## 9. LNMT 动态网站

### 9.1 架构组成

LNMT = Linux + Nginx + MySQL + Tomcat（Java 应用）

### 9.2 Nginx 反向代理 Tomcat

```
客户端请求 (HTTP 80)
        │
        ▼
   Nginx (反向代理)
        │
        ▼
upstream java_prod {
    server 192.168.56.20:8080;
}
        │
        ▼
   Tomcat (8080)
```

```nginx
upstream java_prod {
    server 192.168.56.20:8080;
}
server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://java_prod;
        include proxy_params;
    }
}
```

### 9.3 JVM 故障排查思路

```
1. jps                              → 获取 Java 进程 PID
2. jstack <PID> >> java.txt         → 导出线程栈
3. top -H -p <PID>                  → 找出占用 CPU 高的线程 ID（十进制）
4. echo "obase=16; <TID>" | bc      → 将线程 TID 转为十六进制
5. grep <十六进制TID> java.txt       → 在线程栈中定位问题线程
6. 分析对应业务逻辑，优化代码
```

### 9.4 Tomcat 线程池参数

| 参数 | 说明 |
|------|------|
| `maxThreads` | 最大线程数（建议 600） |
| `minSpareThreads` | 初始化时创建的线程数 |
| `maxSpareThreads` | 线程空闲超过此数则关闭多余线程 |
| `acceptCount` | 请求队列容量，超出则拒绝 |

> **原文错误**：原文参数名写成 `maxSpareHtreads`，正确应为 **`maxSpareThreads`**（多写了一个 H）。

---

## 10. Nginx + Lua 实战

### 10.1 Nginx 调用 Lua 方式

需编译安装 `ngx_devel_kit` + `lua-nginx-module`（或直接使用 OpenResty）。

| 指令 | 说明 |
|------|------|
| `set_by_lua` / `set_by_lua_file` | 设置 Nginx 变量，复杂赋值逻辑 |
| `access_by_lua` / `access_by_lua_file` | 访问控制阶段（鉴权、IP 过滤） |
| `content_by_lua` / `content_by_lua_file` | 内容处理，生成响应体 |
| `init_by_lua` / `init_by_lua_file` | 全局初始化（加载 WAF 规则等） |

常用 Nginx Lua API：

| API | 说明 |
|-----|------|
| `ngx.var.xxx` | 读写 Nginx 变量 |
| `ngx.req.get_headers()` | 获取请求头 |
| `ngx.req.get_uri_args()` | 获取 URL 参数 |
| `ngx.redirect(url)` | 重定向 |
| `ngx.print(str)` | 输出响应内容 |
| `ngx.say(str)` | 输出响应内容（末尾追加换行） |
| `ngx.header.xxx` | 设置响应头 |
| `ngx.exec(location)` | 内部跳转到指定 location |

### 10.2 灰度发布流程

```
用户请求 (IP: x.x.x.x)
        │
        ▼
Nginx + Lua (192.168.56.11)
        │
        ▼
dep.lua 脚本执行
        │
        ├─── 获取客户端 IP（X-Real-IP / x_forwarded_for / remote_addr）
        │
        ▼
Memcached 查询 (127.0.0.1:11211)
  key = clientIP
        │
        ├─── 存在且值为 "1" ──────▶ @java_test
        │                                │
        │                                ▼
        │                       Tomcat 测试集群 (192.168.56.13:9090)
        │                        新版代码
        │
        └─── 不存在 ──────────────▶ @java_prod
                                         │
                                         ▼
                                Tomcat 生产集群 (192.168.56.12:8080)
                                 旧版代码
```

控制灰度名单：通过 `telnet 127.0.0.1 11211` 向 Memcached 写入 IP → 值 `1`，该 IP 的请求即进入测试集群。

### 10.3 WAF 应用防火墙

**WAF 请求拦截流程：**

```
HTTP 请求
    │
    ▼
Nginx (access_by_lua_file /etc/waf/waf.lua)
    │
    ▼
WAF 检测（读取 wafconf/ 下的规则文件）
    │
    ├─── 规则匹配（SQL 注入 / 路径穿越 / CC 攻击等）
    │           │
    │           ▼
    │      返回 403 拦截页面
    │
    └─── 规则不匹配
               │
               ▼
          正常转发到后端
```

防护能力：
- SQL 注入：检测 `select`, `or`, `union` 等关键字
- 文件上传漏洞：禁止 upload 目录执行 PHP
- CC 攻击：`CCrate="100/60"` 限制单 IP 60 秒内最多 100 次请求
- 密码撞库：结合 IP 黑名单 + 预警机制

```nginx
# 文件上传目录禁止 PHP 执行
location ^~ /upload {
    root /soft/code/upload;
    if ($request_filename ~* (.*)\.php) {
        return 403;
    }
}
```

---

## 11. Nginx 性能优化

### 11.1 影响性能的层次

| 层次 | 关注点 |
|------|--------|
| 网络层 | 带宽、丢包率 |
| 系统层 | CPU、内存、磁盘 IO、文件句柄数 |
| 服务层 | 连接优化、请求处理 |
| 程序层 | 接口响应时间、代码效率 |
| 数据库层 | 查询效率、索引优化 |

### 11.2 文件句柄优化

```bash
# /etc/security/limits.conf
root  soft  nofile  65535
root  hard  nofile  65535
*     soft  nofile  25535
*     hard  nofile  25535
```

```nginx
# Nginx 进程级别
worker_rlimit_nofile 35535;
```

### 11.3 Nginx 通用性能配置

```nginx
user nginx;
worker_processes auto;           # 自动匹配 CPU 核心数
worker_cpu_affinity auto;        # 自动绑定 CPU（减少 cache miss）
worker_rlimit_nofile 35535;

events {
    use epoll;                   # 使用 epoll 模型
    worker_connections 10240;    # 单 worker 最大连接数
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    charset       utf-8;

    sendfile    on;
    tcp_nopush  on;              # 静态资源服务启用
    tcp_nodelay on;              # 动态/长连接服务启用
    keepalive_timeout 65;

    gzip on;
    gzip_disable "MSIE [1-6]\.";
    gzip_http_version 1.1;

    include /etc/nginx/conf.d/*.conf;
}
```

### 11.4 压力测试（ab 工具）

```bash
ab -n 2000 -c 200 http://127.0.0.1/index.html
# -n  总请求数
# -c  并发数
# -k  开启长连接
```

关键指标：

| 指标 | 含义 |
|------|------|
| `Requests per second` | QPS（每秒处理请求数） |
| `Time per request (mean)` | 客户端视角的平均响应时间 |
| `Time per request (mean, across all)` | 服务端处理单个请求的平均时间 |
| `Transfer rate` | 网络传输速率 |

---

## 12. Nginx 常见问题

### 12.1 Server 优先级

当多个 `server_name` 相同时，**按配置文件名字母顺序**加载，先加载的优先生效。

```bash
# 测试时 nginx -t 会有 warn：
# [warn] conflicting server name "192.168.69.113" on 0.0.0.0:80, ignored
```

### 12.2 Location 匹配优先级

| 优先级 | 语法 | 说明 |
|--------|------|------|
| 1（最高） | `location = /path` | 精确匹配 |
| 2 | `location ^~ /path` | 前缀匹配，匹配后不再检查正则 |
| 3 | `location ~ regex` | 正则匹配（区分大小写） |
| 4 | `location ~* regex` | 正则匹配（不区分大小写） |
| 5（最低） | `location /path` | 普通前缀匹配 |

### 12.3 try_files 使用

```nginx
location / {
    try_files $uri $uri/ @java_page;
    # 1. 检查 $uri 文件是否存在
    # 2. 检查 $uri/ 目录是否存在
    # 3. 上述均不存在，转发至 @java_page
}
location @java_page {
    proxy_pass http://127.0.0.1:8080;
}
```

### 12.4 alias vs root 区别

| 指令 | 实际文件路径 |
|------|-------------|
| `root /local/code;` + `location /req/path/` | `/local/code/req/path/index.html`（root 路径 + URI） |
| `alias /local/code/;` + `location /req/path/` | `/local/code/index.html`（alias 路径替换 URI） |

```nginx
# root：拼接 URI
location /request_path/code/ {
    root /local_path/code/;
    # 实际路径：/local_path/code/request_path/code/index.html
}

# alias：替换 URI
location /request_path/code/ {
    alias /local_path/code/;
    # 实际路径：/local_path/code/index.html
}
```

### 12.5 获取用户真实 IP

多级代理场景：

```
真实客户端 ──▶ 代理1 ──▶ 代理2 ──▶ Nginx
                                      │
                                      $remote_addr = 代理2 IP
                                      $http_x_forwarded_for = 真实IP, 代理1IP, 代理2IP
```

在最终 Nginx 上获取真实 IP：

```nginx
# 信任代理服务器的 IP，取最左侧真实 IP
set_real_ip_from 192.168.1.0/24;
real_ip_header X-Forwarded-For;
```

### 12.6 常见 HTTP 状态码

| 状态码 | 含义 |
|--------|------|
| 200 | 请求成功 |
| 301 | 永久重定向 |
| 302 | 临时重定向 |
| 304 | 资源未修改（使用缓存） |
| 400 | 请求参数错误 |
| 401 | 需要身份认证 |
| 403 | 禁止访问（Forbidden） |
| 404 | 资源不存在 |
| 413 | 上传文件过大（Request Entity Too Large） |
| 502 | 后端服务无响应（**Bad Gateway**） |
| 504 | 后端服务执行超时（Gateway Time-out） |

> **原文错误**：502 的英文写成 "boy gateway"，正确应为 **"Bad Gateway"**。

### 12.7 网站访问原理（完整链路）

```
用户输入 URL
    │
    ▼
1. DNS 解析
   本地 Hosts → 本地 DNS → 上级 DNS → 返回 IP

    │
    ▼
2. TCP 三次握手建立连接

    │
    ▼
3. HTTP 请求发送

    │
    ▼
4. CDN 层（静态资源缓存命中则直接返回）

    │
    ▼
5. 负载均衡层（Nginx/LVS/F5 调度）
    │
    ├── 静态请求 ──▶ 静态集群
    │
    └── 动态请求 ──▶ 动态集群（PHP/Java）
                        │
                        ├── Opcache 命中 ──▶ 返回
                        │
                        ├── Redis 缓存命中 ──▶ 返回
                        │
                        └── 查询数据库 ──▶ MySQL
                                │
                                ▼
                        结果写入 Redis 缓存
                                │
                                ▼
                          返回 WEB 节点

    │
    ▼
6. TCP 四次挥手断开连接
```

术语说明：
- **PV**（Page View）：页面浏览量
- **UV**（Unique Visitor）：唯一访客数（设备维度）
- **IP**：独立 IP 数（出口 IP 维度）

例：大楼 100 人共用 1 个公网 IP，每人刷新 5 次 → PV=500，UV=100，IP=1

### 12.8 Nginx 优化方案汇总

| 优化方向 | 措施 |
|----------|------|
| 压缩 | gzip 压缩 + gzip_static |
| 缓存 | expires 静态缓存 + proxy_cache 代理缓存 |
| 网络 IO | epoll + worker_connections |
| 进程 | worker_processes auto + CPU 亲和 |
| 安全 | 隐藏版本号 + 防盗链 + IP 访问控制 + 限速 |
| HTTPS | TLS 1.2/1.3 + HSTS + 证书链优化 |
| 连接 | keepalive + upstream keepalive |
| 上传限制 | `client_max_body_size` + upload 目录 PHP 执行拦截 |

---

## 附录：错误汇总

| 文档 | 位置 | 原文 | 正确内容 |
|------|------|------|----------|
| 01 | IO复用实现 | "Epool" | **epoll** |
| 01 | 内置变量 | `depict_root` | **`$document_root`** |
| 02 | 日志变量 | `iktatus` | **`$status`** |
| 02 | 请求限制语法 | `limit_conn zone number [burst=...]` | **`limit_req zone=zone [burst=number] [nodelay];`** |
| 03 | location 正则 | `~.* $jpg\|gif\|png$ $` | **`~.*\.(jpg\|gif\|png)$`** |
| 05 | proxy_cache_key | `$is_argsisease` | **`$is_args$args`** |
| 06 | 正则表达式表格 | `*` → "最少连接数" | **`*` → "重复 0 次或更多次"** |
| 07 | ssl_session_cache | `share:SSL:10m` | **`shared:SSL:10m`** |
| 07 | ssl 指令 | `ssl on;` + `listen 443;` | **`listen 443 ssl;`**（1.15.0+ 推荐） |
| 07 | rewrite | `rewrite^(.*)` | **`rewrite ^ ...`**（缺空格） |
| 08 | 安装描述 | "安装 Mariadb" | 实际安装的是 **MySQL 5.7**，非 MariaDB |
| 08 | PHP 安全模式 | `sql.safe_mode` | PHP 配置为 `disable_functions`，`safe_mode` 在 PHP 7.x 已移除 |
| 08 | PHP 测试代码 | `mysql_connect()` | **`mysqli_connect()`** 或 **PDO**（PHP 7.0+ 已删除 `mysql_*` 函数） |
| 09 | Tomcat 参数 | `maxSpareHtreads` | **`maxSpareThreads`** |
| 12 | 502 状态码 | "boy gateway" | **"Bad Gateway"** |

---

## 深度专题：生产实战补充

> 以下三个知识点在文档基础上结合实际工作场景做深度展开，补充文档未覆盖的生产细节。

---

## 专题一：proxy_params 生产实战

### 1. 它解决什么问题

`proxy_params` 是把反向代理通用参数抽成独立文件，通过 `include proxy_params;` 在多个 `location` 中复用，避免重复配置。但生产中不同业务对代理参数的需求差异很大，正确做法是**公共参数放 proxy_params，差异参数在 location 内覆盖**。

```
/etc/nginx/proxy_params        ← 公共基础参数（一处定义）
/etc/nginx/conf.d/api.conf     ← 各 location include 后按需覆盖
```

### 2. 每个参数的生产含义

```nginx
# /etc/nginx/proxy_params

# ── 跳转行为 ──────────────────────────────────────────────────────────────
proxy_redirect default;
# 后端返回 Location 头（30x 跳转）时，Nginx 把其中的后端地址替换成对外地址。
# 防止内部 IP:端口 泄露给客户端。

# ── 头信息透传 ────────────────────────────────────────────────────────────
proxy_set_header Host              $http_host;
# 不加此行，后端收到的 Host 是 upstream 地址，虚拟主机识别失败，常见 400/404 根因。

proxy_set_header X-Real-IP         $remote_addr;
# 把直连 Nginx 的 IP 写给后端，后端日志/鉴权可用。

proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
# 在已有 XFF 链后追加当前 $remote_addr，记录完整代理链路。

proxy_set_header X-Forwarded-Proto $scheme;
# 告知后端当前协议是 http 还是 https，后端生成跳转 URL 时需要此信息。
# 不加此行，https 站点后端生成的 Location 头可能变成 http://，引发跳转死循环。

# ── 超时控制 ──────────────────────────────────────────────────────────────
proxy_connect_timeout 10;   # Nginx 与后端建立 TCP 连接的超时（秒）
proxy_send_timeout    60;   # Nginx 向后端发送请求数据的超时
proxy_read_timeout    60;   # Nginx 等待后端返回响应的超时

# ── 缓冲区控制 ────────────────────────────────────────────────────────────
proxy_buffer_size        16k;   # 读取后端响应头的缓冲区
proxy_buffering          on;    # 开启响应体缓冲（默认 on）
proxy_buffers            4 64k; # 4 块 × 64k = 256k 响应体缓冲
proxy_busy_buffers_size  128k;  # 向客户端发送时最多占用的繁忙缓冲
proxy_max_temp_file_size 256k;  # 缓冲满后写磁盘临时文件的上限
```

**三个超时分别对应代理的三个阶段：**

```
Nginx ──[connect_timeout 10s]──▶ 与后端建立 TCP 连接
Nginx ──[send_timeout    60s]──▶ 向后端发送请求体数据
Nginx ──[read_timeout    60s]──▶ 等待后端返回响应数据

任一阶段超时 → 客户端收到 504 Gateway Time-out
```

**缓冲开启与关闭的差异：**

```
proxy_buffering on（默认，适合普通 HTTP 响应）：
  后端 ──写入──▶ Nginx 缓冲区（内存/磁盘）──慢慢发──▶ 客户端
  后端写完即可释放连接，Nginx 独立控制发送节奏

proxy_buffering off（适合流式响应 / SSE / 长轮询）：
  后端 ──▶ Nginx ──实时透传──▶ 客户端
  后端连接被占用直到客户端接收完毕，延迟最低
```

### 3. 微服务网关场景：公共 + 差异化覆盖

电商系统中 Nginx 作为统一 API 网关，不同服务对超时和缓冲需求差异极大：

```
客户端
  │
  ▼
Nginx API 网关 (api.shop.com)
  │
  ├── /api/order/**    → 订单服务（超时长，涉及支付回调）
  ├── /api/search/**   → 搜索服务（超时短，超时降级）
  ├── /api/upload/**   → 上传服务（关闭缓冲，流式直传）
  └── /api/push/**     → 推送服务（长连接，SSE 实时推送）
```

```nginx
# 订单服务：等待支付宝/微信回调最长 90s
location /api/order/ {
    proxy_pass http://order_cluster;
    include proxy_params;
    proxy_read_timeout 120;    # 覆盖默认 60s
    proxy_send_timeout 120;
}

# 搜索服务：快速失败，超时立即降级返回兜底数据
location /api/search/ {
    proxy_pass http://search_cluster;
    include proxy_params;
    proxy_connect_timeout 3;   # 3s 连不上就报错
    proxy_read_timeout    5;   # 5s 没结果就降级
}

# 文件上传：关闭请求缓冲，边接收边转发，避免大文件撑爆内存
location /api/upload/ {
    proxy_pass http://upload_cluster;
    include proxy_params;
    proxy_request_buffering off;   # 关闭请求体缓冲（默认 on）
    proxy_buffering         off;   # 关闭响应体缓冲
    client_max_body_size    2g;    # 允许最大上传 2GB
    proxy_read_timeout      300;
}

# SSE 推送：必须关闭响应缓冲，否则消息憋在缓冲区不发出去
location /api/push/ {
    proxy_pass http://push_cluster;
    include proxy_params;
    proxy_buffering         off;    # 关键：实时透传
    proxy_read_timeout      3600;   # 长连接保持 1 小时
    proxy_set_header Connection ''; # 兼容 HTTP/1.1 长连接
    add_header X-Accel-Buffering no; # 通知 CDN 也不缓冲
}
```

### 4. WebSocket 代理（proxy_params 必须额外补充）

普通 proxy_params 不包含 WebSocket 协议升级头，直接使用会导致握手失败：

```
客户端发起 WebSocket 握手：
  GET /ws HTTP/1.1
  Connection: Upgrade
  Upgrade: websocket

Nginx 默认代理时把 Connection 改成 close
  → 后端收不到 Upgrade → 握手失败 → 长期回退到 HTTP 轮询
```

```nginx
location /ws/ {
    proxy_pass http://ws_backend;
    include proxy_params;

    # WebSocket 专属头，必须额外加
    proxy_http_version  1.1;
    proxy_set_header    Upgrade    $http_upgrade;
    proxy_set_header    Connection "upgrade";
    proxy_read_timeout  3600;    # WS 长连接，不能被超时踢掉
}
```

### 5. 生产事故案例

**事故一：proxy_read_timeout 默认 60s 导致订单丢失**

```
用户下单 → Nginx 转发 → 订单服务调用支付宝接口（最长需要 90s）
                                    │
                   到第 60s，Nginx 超时 → 返回 504 给用户
                   用户看到"下单失败"，但实际支付宝已经扣款
                   → 钱扣了，订单没建成

根因：proxy_read_timeout 默认 60s，支付回调需要 90s
修复：针对支付 location 单独设置 proxy_read_timeout 120;
```

**事故二：proxy_buffering on 导致 SSE 消息堆积延迟**

```
后端每秒推送一条实时消息 → Nginx 缓冲区未满不发出去
用户 30s 后才一次性收到 30 条消息（缓冲区满才刷出）

根因：SSE location 未关闭 proxy_buffering
修复：SSE/长轮询 location 设置 proxy_buffering off;
```

**事故三：未透传 X-Forwarded-Proto 导致 HTTPS 死循环跳转**

```
配置：Nginx 443 → 后端 8080，后端框架自动把 http 请求 301 跳到 https
结果：Nginx 转发给后端时未带 X-Forwarded-Proto: https
     后端看到 $scheme = http → 触发 301 → 跳回 Nginx → 再次转发
     → 无限循环，客户端报 ERR_TOO_MANY_REDIRECTS

修复：proxy_set_header X-Forwarded-Proto $scheme;
     后端读取此头，判断原始协议已是 https，不再跳转
```

---

## 专题二：Rewrite 生产实战

### 1. 四种 Flag 的核心区别

```
                    last                              break
                 ┌──────────┐                     ┌──────────┐
请求进入 ──▶    │ rewrite  │ ──新URI──▶ 重新      │ rewrite  │ ──新URI──▶ 当前 root
                │          │            匹配所有   │          │            下找文件
                └──────────┘            location   └──────────┘
                                            │
                                            ▼
                                      可命中其他 location 块
                                      （如 /test/ 对应的 location）

                  redirect (302)                    permanent (301)
浏览器收到后：   不缓存跳转目标                    缓存跳转目标
Nginx 关闭后：   跳转失效                          跳转依然生效（浏览器缓存）
适用场景：       临时切流、维护、A/B 测试           域名永久迁移、SEO 权重转移
```

### 2. 域名迁移（301 永久跳转）

品牌升级，域名从 `old-brand.com` 迁移到 `new-brand.com`，SEO 权重需完整转移：

```
用户/搜索引擎 ──▶ old-brand.com ──301──▶ new-brand.com
                        │                       │
                   Nginx 处理              新站点服务
```

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name old-brand.com www.old-brand.com;

    # 保留完整路径和参数，搜索引擎权重完整转移
    return 301 https://www.new-brand.com$request_uri;
}
```

> 注意：301 会被浏览器和搜索引擎长期缓存，上线前必须在测试环境充分验证，线上一旦发出 301，短期内很难撤回。

### 3. 多端适配（User-Agent 分流）

同一域名，手机访问 H5 站，PC 访问 Web 站，微信内嵌走小程序页面：

```
用户请求 www.shop.com
         │
         ├── iPhone / Android (Mobile)   → m.shop.com
         ├── MicroMessenger (微信)        → miniapp.shop.com
         └── 其他 (PC/iPad)              → www.shop.com (PC 版)
```

```nginx
server {
    listen 80;
    server_name www.shop.com;

    set $mobile 0;

    # 微信内置浏览器优先判断（小程序 webview 也带此 UA）
    if ($http_user_agent ~* "MicroMessenger") {
        set $mobile 2;
    }

    # 移动设备（排除平板，平板走 PC 版）
    if ($http_user_agent ~* "(iPhone|Android.*Mobile|Windows Phone)") {
        set $mobile 1;
    }

    location / {
        if ($mobile = 2) {
            rewrite ^ https://miniapp.shop.com$request_uri? redirect;
        }
        if ($mobile = 1) {
            rewrite ^ https://m.shop.com$request_uri? redirect;
        }
        # 默认 PC 版
        root /var/www/pc;
        index index.html;
    }
}
```

### 4. 旧版 API 兼容（无感升级路径转换）

后端 API 从 v1 升级到 v2，但 App 老版本无法强制更新，由 Nginx 做路径转换，后端只维护 v2：

```
App 旧版本请求：GET /api/v1/user/profile
                       ↓ Nginx rewrite（break，不再匹配其他 location）
后端实际处理：  GET /api/v2/users/me/profile
```

```nginx
server {
    listen 80;
    server_name api.shop.com;

    # v1 用户接口 → v2 映射
    location /api/v1/user/ {
        rewrite ^/api/v1/user/(.*)$ /api/v2/users/$1 break;
        proxy_pass http://api_backend;
        include proxy_params;
    }

    # v1 商品接口 → v2 映射
    location /api/v1/goods {
        rewrite ^/api/v1/goods(.*)$ /api/v2/products$1 break;
        proxy_pass http://api_backend;
        include proxy_params;
    }

    # v2 直接透传
    location /api/v2/ {
        proxy_pass http://api_backend;
        include proxy_params;
    }
}
```

### 5. 防盗链 + Rewrite 返回水印图

直接返回 403 会让第三方页面出现"红叉"，改为返回水印图体验更好且起到品牌宣传作用：

```
第三方网站：<img src="https://cdn.shop.com/img/product.jpg">
                              │
               Referer 不在白名单，不返回 403
                              │
                              ▼
               重写到 /watermark.jpg 并在当前 root 下找文件
               第三方网站展示"禁止盗链"水印图
```

```nginx
location ~ .*\.(jpg|png|gif)$ {
    valid_referers none blocked *.shop.com shop.com;

    if ($invalid_referer) {
        rewrite ^ /watermark.jpg break;   # break：在当前 root 下找水印图
    }

    root /var/www/images;
    expires 7d;
}
```

### 6. 网站维护模式（一键切流）

发布期间把所有请求打到维护页，运维自己的 IP 可以绕过预览：

```nginx
server {
    listen 80;
    server_name www.shop.com;

    set $maintain 1;   # 开启维护模式：值为 1；注释此行恢复正常

    location / {
        # 运维 IP 绕过维护模式，直接预览新版本
        if ($remote_addr = "192.168.1.100") {
            set $maintain 0;
        }
        if ($maintain = 1) {
            rewrite ^ /maintain.html break;
        }
        proxy_pass http://backend;
        include proxy_params;
    }

    location = /maintain.html {
        root /var/www/static;
        add_header Retry-After 3600;   # 告知客户端 1 小时后重试
    }
}
```

切换操作只需修改配置后 `nginx -s reload`，秒级生效，无需重启。

### 7. HTTPS 强跳 + www 统一规范（SEO 必做）

避免 `shop.com` 和 `www.shop.com` 被搜索引擎识别为两个站点（重复内容惩罚）：

```nginx
# 所有 HTTP 请求统一跳 HTTPS
server {
    listen 80;
    server_name shop.com www.shop.com;
    return 301 https://www.shop.com$request_uri;
}

# HTTPS 裸域名跳 www
server {
    listen 443 ssl;
    server_name shop.com;
    return 301 https://www.shop.com$request_uri;
}

# 唯一正式入口
server {
    listen 443 ssl;
    server_name www.shop.com;
    # 业务逻辑...
}
```

**流量漏斗，四种访问方式最终汇聚到同一入口：**

```
http://shop.com/page      ──301──▶ https://www.shop.com/page
http://www.shop.com/page  ──301──▶ https://www.shop.com/page
https://shop.com/page     ──301──▶ https://www.shop.com/page
https://www.shop.com/page ─────▶  正常响应 ✓
```

---

## 专题三：获取用户真实 IP 生产实战

### 1. 问题根源

`$remote_addr` 只能拿到**与 Nginx 直接建立 TCP 连接的 IP**。一旦链路中有代理层，拿到的是代理 IP，而非真实客户端 IP。

```
真实用户 (1.1.1.1)
      │  TCP 连接
      ▼
 CDN 节点 (104.x.x.x)
      │  新建 TCP 连接
      ▼
后端 Nginx
      $remote_addr = 104.x.x.x  ← 是 CDN IP，不是用户 IP
```

### 2. 两个关键请求头

#### X-Forwarded-For（链式追加）

每一层代理在转发时，把上一层的 IP 追加到这个头的末尾：

```
真实用户 (1.1.1.1) 发出请求，无 X-Forwarded-For 头
          │
          ▼
代理1 (2.2.2.2) 追加 → X-Forwarded-For: 1.1.1.1
          │
          ▼
代理2 (3.3.3.3) 追加 → X-Forwarded-For: 1.1.1.1, 2.2.2.2
          │
          ▼
后端服务器读取：
  XFF 最左边 = 真实用户 IP（1.1.1.1）
  后面依次   = 代理链 IP
```

#### X-Real-IP（单值记录）

只记录一个 IP，通常由第一个可信代理写入，值为其直连的客户端 IP，不会被后续代理累加。比 XFF 更简单，但丢失了代理链信息。

### 3. 各 CDN 厂商的透传头差异

不同 CDN 除标准 XFF 之外还有各自的专属头，生产中需按厂商配置：

```
Cloudflare CDN：
  CF-Connecting-IP: 1.1.1.1         ← Cloudflare 独有，最可靠
  X-Forwarded-For: 1.1.1.1

阿里云 CDN：
  Ali-CDN-Real-IP: 1.1.1.1          ← 阿里云独有
  X-Forwarded-For: 1.1.1.1, CDN节点IP

腾讯云 CDN：
  X-Forwarded-For: 1.1.1.1, CDN节点IP
  （无专属头，依赖标准 XFF）
```

**多 CDN 兼容配置：**

```nginx
# 用 map 按优先级提取真实 IP
map $http_cf_connecting_ip $real_client_ip {
    default  $http_cf_connecting_ip;   # 有 CF 头优先用
    ""       $http_x_real_ip;          # 其次用 X-Real-IP
}

map $real_client_ip $final_client_ip {
    default  $real_client_ip;
    ""       $remote_addr;             # 兜底用直连 IP
}
```

### 4. realip_module（推荐方案）

配置后 `$remote_addr` 会被自动替换为真实客户端 IP，后续日志、限速、鉴权模块无需改动。

```nginx
# 声明哪些 IP 是可信代理（只信任这些代理传来的头）
set_real_ip_from  10.0.0.0/8;
set_real_ip_from  172.16.0.0/12;
set_real_ip_from  192.168.0.0/16;
# Cloudflare 的 IP 段（从官网获取，定期更新）
set_real_ip_from  103.21.244.0/22;
set_real_ip_from  104.16.0.0/12;

# 从哪个头提取真实 IP
real_ip_header    X-Forwarded-For;

# 递归去除可信代理 IP，找到第一个不可信的 IP（即真实用户）
real_ip_recursive on;
```

### 5. ip_hash 在代理环境下的失效问题

负载均衡用 `ip_hash` 本意让同一用户固定打到同一台后端（保持 session），但有代理时全部流量来自同一出口 IP，导致 ip_hash 退化成单机：

```
1000 个用户各不同 IP
        │
        ▼
公司代理 / CDN 出口 (10.0.0.1)  ← 所有流量出口都是同一个 IP
        │
        ▼
Nginx upstream ip_hash
        │
        hash(10.0.0.1) → 始终命中 server1
        │
        ├── server1 ← 所有请求（过载）
        ├── server2 ← 空闲
        └── server3 ← 空闲
```

**修复：改为对真实 IP 做 hash**

```nginx
upstream backend {
    # 不用 ip_hash，改用真实 IP 变量做 hash
    hash $http_x_forwarded_for consistent;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

### 6. XFF 头被客户端伪造的安全漏洞

XFF 是普通 HTTP 请求头，客户端可以随意伪造，直接信任会被绕过：

```
攻击者每次请求伪造不同 XFF 绕过基于 IP 的限速：

  limit_req_zone $http_x_forwarded_for zone=limit:10m rate=10r/s;  ← 错误写法

  请求1: X-Forwarded-For: 1.1.1.1  → 独立计数，绕过限速
  请求2: X-Forwarded-For: 2.2.2.2  → 独立计数，绕过限速
  请求3: X-Forwarded-For: 3.3.3.3  → 独立计数，绕过限速
  → 实际每秒发 1000 次，限速完全失效
```

**正确防御：公网入口 Nginx 覆盖 XFF，不追加**

```nginx
# 公网入口层（直接面向用户）：重置 XFF，防止伪造
proxy_set_header X-Forwarded-For $remote_addr;   # 覆盖，不追加

# 内部服务之间（可信环境）才使用追加
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

**配合 real_ip_recursive 从右向左剥离可信代理 IP：**

```
X-Forwarded-For: [伪造1.1.1.1], [真实5.5.5.5], [可信代理2.2.2.2], [可信代理3.3.3.3]

real_ip_recursive on 处理过程（从右向左）：
  3.3.3.3 → 在可信列表，跳过
  2.2.2.2 → 在可信列表，跳过
  5.5.5.5 → 不在可信列表，停止 → 这就是真实客户端 IP ✓
  1.1.1.1 → 未被采信（伪造值被丢弃）
```

### 7. 完整多级代理链路示意

```
真实用户 (1.1.1.1)
     │ 无任何自定义头
     ▼
┌──────────────────────────────────────────────────────────┐
│  CDN / 第一层代理 (2.2.2.2)                              │
│  proxy_set_header X-Real-IP       1.1.1.1                │
│  proxy_set_header X-Forwarded-For 1.1.1.1                │
└──────────────────────────────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────────────────────────┐
│  内部负载均衡 (3.3.3.3)                                  │
│  X-Real-IP 不变：1.1.1.1                                 │
│  X-Forwarded-For 追加：1.1.1.1, 2.2.2.2                 │
└──────────────────────────────────────────────────────────┘
     │
     ▼
┌──────────────────────────────────────────────────────────┐
│  后端 Nginx                                              │
│                                                          │
│  $remote_addr         = 3.3.3.3  （直连代理 IP，无效）  │
│  $http_x_real_ip      = 1.1.1.1  ✓                      │
│  $http_x_forwarded_for= 1.1.1.1, 2.2.2.2  ✓            │
│                                                          │
│  配置 set_real_ip_from + real_ip_recursive 后：          │
│  $remote_addr 自动替换为 1.1.1.1  ✓                     │
│  限速、日志、鉴权模块无需任何修改                        │
└──────────────────────────────────────────────────────────┘
```

---

## 三个知识点在生产架构中的协作关系

```
用户 (1.1.1.1)
     │
     ▼
┌──────────────────────────────────────────────────────────────────┐
│  公网入口 Nginx                                                   │
│                                                                   │
│  ① 获取真实 IP（专题三）                                          │
│     set_real_ip_from CDN IP 段                                    │
│     real_ip_header CF-Connecting-IP                               │
│     → $remote_addr = 1.1.1.1（用户真实 IP）                      │
│                                                                   │
│  ② Rewrite 路由决策（专题二）                                     │
│     HTTP  → HTTPS (301 permanent)                                 │
│     /v1/* → /v2/* (break，路径转换)                               │
│     维护模式 → /maintain.html (break)                             │
│     手机 UA → m.shop.com (redirect)                               │
│                                                                   │
│  ③ proxy_params 代理透传（专题一）                                │
│     X-Real-IP: 1.1.1.1          → 后端日志/鉴权使用              │
│     X-Forwarded-For: 1.1.1.1    → 审计/风控使用                  │
│     X-Forwarded-Proto: https     → 后端不误触发跳转               │
│     proxy_read_timeout 按业务差异化（订单 120s，搜索 5s）         │
│     proxy_buffering off（SSE/上传），on（普通接口）               │
└──────────────────────────────────────────────────────────────────┘
     │              │               │
     ▼              ▼               ▼
 订单服务        搜索服务         上传服务
 timeout:120s   timeout:5s      buffering:off
 (支付回调)     (快速降级)      (流式直传)
```

**三者分工：**

| 知识点 | 职责 | 出问题的典型症状 |
|--------|------|-----------------|
| proxy_params | 控制 Nginx 与后端通信的管道参数 | 504 超时、SSE 消息延迟、上传 OOM、HTTPS 跳转死循环 |
| Rewrite | 请求进入管道前的路由与 URL 决策 | 旧 API 404、手机访问 PC 版、SEO 权重流失、维护切流慢 |
| 获取真实 IP | 贯穿整个链路的用户身份标识 | 限速误伤正常用户、封禁打到代理 IP、ip_hash 单机热点 |
