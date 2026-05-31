# Nginx 服务企业应用 知识点梳理

> 源文档:`yuque/nginx服务企业应用.md`
> 整理目标:把原文按"软件选型 → Nginx 安装与目录结构 → 配置文件结构 → 虚拟主机 → status/日志 → location 匹配优先级 → rewrite → 访问认证"重新组织,跨文件 include、location 匹配优先级、日志切割等"多文件/多组件协作"的内容一律配 ASCII 流程图,末尾每节有【纠错】。

---

## 1. 软件选型

### 1.1 静态服务软件

| 软件 | 定位 |
|---|---|
| **Apache** | 中小型 Web 主流;插件丰富、稳定;高并发偏弱(prefork/select 模型) |
| **Nginx** | 大型网站 Web 主流;高并发、低资源占用 |
| **Tengine** | Nginx 的淘宝分支,加入一致性 hash、动态健康检查、并发限流等 |
| **Lighttpd** | 不温不火,静态解析高效,曾被百度贴吧、豆瓣使用 |

### 1.2 动态服务软件

| 软件 | 用途 |
|---|---|
| **PHP-FPM**(FastCGI) | Nginx 推荐:`nginx + php-fpm` |
| **Tomcat** | Java 中小型应用主流 |
| **Resin** | Java 大型应用 |
| **IIS** | Windows 平台,ASP/ASPX |

### 1.3 业务选型表

| 业务类型 | 推荐 |
|---|---|
| 高并发静态 | Nginx(优先) / Lighttpd |
| 动态(JSP/Do) | Tomcat / Resin(前面叠 Nginx 做反代) |
| 动静混合 | LNMP(Linux + Nginx + MySQL + PHP),Nginx 前端、PHP-FPM 后端 |

### 【纠错】

- 原文 §1.2 把 `LNMP / LEMP` 写作"L**E**MP 是因为 Nginx 的 N 取自 engine-x"——**应澄清**:LEMP 中的 E 实际就是 "**E**ngine x" 的首字母,所以也叫 LEMP;两者指同一套技术栈,只是首字母选择不同。
- 原文 §1.1 "Tomcat 中小企业动态 Web 服务主流,**互联网 Java 容器主流(如 jsp、do)**" —— 准确说法:Tomcat 是 **Servlet/JSP 容器**(Web 容器),不是完整 Java EE 应用服务器(JBoss/WildFly/WebLogic 才是);`.do` 是早期 Struts 框架的约定后缀,不是 Tomcat 的固有特性。

---

## 2. Nginx 的核心优势

### 2.1 主要特性

- 支持高并发(数万级长连接)
- 资源占用极低(3 万并发 10 个 worker 内存 < 200MB)
- 支持反向代理 + 负载均衡 + 加速缓存
- 内置上游健康检查
- 异步 I/O 事件模型:**epoll(Linux 2.6+) / kqueue(BSD/macOS)**

### 2.2 与 Apache 的核心区别:select vs epoll

```
select 模型(类比"宿管大妈挨家问")
    监听 N 个 fd 时:
       for fd in fds:
           if fd 可读: 处理
    复杂度 O(N),fd 多了线性增长
    FD_SETSIZE 默认 1024,超过要重编译

epoll 模型(类比"事先约定的地方等待")
    epoll_wait() 只返回真正就绪的 fd
    内核维护红黑树 + 就绪链表,回调 callback
    复杂度 O(就绪数),fd 增加性能基本不掉
```

| 指标 | select | epoll |
|---|---|---|
| 性能 | 连接数增多急剧下降 | 基本平稳 |
| 连接数 | 受 FD_SETSIZE 限制(默认 1024) | 无硬限制(受 fs.file-max / ulimit) |
| 机制 | 线性轮询 | 事件回调 |
| 开发复杂度 | 低 | 中 |

### 【纠错】

- 原文 §2.4.3 "select 处理的最大连接数不超过 1024" —— **更精确**地说:1024 是 `FD_SETSIZE` 默认值,可重编译调高,但 `select` 仍是 O(N) 线性扫描,1024 只是"额外的伤上加伤"。
- 原文称 Apache "**基于传统的 select 模型**" —— 不全对:Apache 默认用 **MPM(prefork/worker/event)**,**event MPM** 在 2.4 之后基于 `kqueue`/`epoll`/`port` 事件机制,跟 Nginx 已比较接近。`select` 这个说法只对旧版本/低端模块成立。

---

## 3. 编译安装(企业规范)

### 3.1 流程总览

```
 ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 │ 系统检查 │ →  │ 装依赖   │ →  │ 建用户   │ →  │ ./configure│ → │ make &   │
 │ uname -r │    │ pcre/ssl │    │ www      │    │ --prefix  │   │ make install│
 └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                                       │
                                                                       ▼
                                          ln -s 软链 → /application/nginx
                                                                       │
                                                                       ▼
                                                   nginx 启动 + 配置精简 + 测试
```

### 3.2 命令清单(可直接复用)

```bash
# 1. 系统检查
cat /etc/redhat-release
uname -r

# 2. 依赖(规律:所有依赖都加 -devel)
yum install -y pcre-devel openssl-devel gcc gcc-c++ make

# 3. 用户(无登录、无家目录)
useradd -M -s /sbin/nologin www

# 4. 下载解压
cd /server/tools
wget http://nginx.org/download/nginx-1.10.2.tar.gz
tar xf nginx-1.10.2.tar.gz && cd nginx-1.10.2

# 5. 配置 → 编译 → 安装(顺序不可乱)
./configure --prefix=/application/nginx-1.10.2 \
            --user=www --group=www \
            --with-http_stub_status_module \
            --with-http_ssl_module
echo $?           # 应为 0,确认 configure 通过
make
make install

# 6. 软链(方便升级时切换版本)
ln -s /application/nginx-1.10.2 /application/nginx

# 7. 精简配置
cd /application/nginx/conf
egrep -v '#|^$' nginx.conf.default > nginx.conf

# 8. 启动 + 检查
/application/nginx/sbin/nginx
ps -ef | grep nginx
netstat -lntup | grep 80

# 9. PATH 简化(可选)
echo 'export PATH=/application/nginx/sbin:$PATH' >> /etc/profile
source /etc/profile
which nginx
```

### 3.3 编译参数含义

| 参数 | 作用 |
|---|---|
| `--prefix=DIR` | 安装目录(不存在会自动创建) |
| `--user=USER --group=GROUP` | worker 进程运行身份 |
| `--with-http_stub_status_module` | 启用基础状态模块(`/status` 看连接数) |
| `--with-http_ssl_module` | 启用 HTTPS |
| `--with-http_v2_module` | (扩展)HTTP/2 |
| `--with-stream` | (扩展)4 层 TCP/UDP 负载 |
| `--with-pcre` | 正则支持(rewrite 必需) |

### 3.4 目录结构

```
/application/nginx/
├── conf/      # 配置文件(nginx.conf、mime.types、fastcgi.conf …)
├── html/      # 默认站点目录
├── logs/      # 访问日志 / 错误日志 / pid 文件
└── sbin/      # 服务命令(只有一个 nginx 可执行文件)
```

### 3.5 常用命令

| 用途 | 命令 |
|---|---|
| 启动 | `/application/nginx/sbin/nginx` |
| 停止 | `nginx -s stop` |
| 平滑重启(reload) | `nginx -s reload` |
| 平滑停止(让 worker 处理完再退出) | `nginx -s quit` |
| 语法检查 | `nginx -t` |
| 显示编译参数 | `nginx -V`(大写 V,小写 v 只显版本) |
| 列出全部配置(含 include) | `nginx -T` |

### 【纠错】

- 原文最后给的"软链命令"在原文反代笔记里出现 **`ln -s ... /application/ngin`** 拼写错误,本节已统一为 `nginx`。
- 原文 `nginx -V` 的描述说"-V 大写,显示原有编译参数",**正确补充**:小写 `-v` 仅显示版本号,大写 `-V` 同时显示版本和编译参数。

---

## 4. nginx.conf 结构

### 4.1 三层嵌套

```
main 全局区
 └─ events {}                    # 网络模型 / worker 连接数
 └─ http {}                      # HTTP 服务总区
      ├─ include / log_format / sendfile / keepalive_timeout
      └─ server {}               # 一个虚拟主机
           ├─ listen / server_name / root / index
           ├─ access_log / error_log
           └─ location {} ...    # 路径路由
```

### 4.2 注释版示例

```nginx
worker_processes  1;                              # ← worker 进程数,生产建议 = CPU 核数
events {
    worker_connections  1024;                     # ← 每个 worker 最大并发连接
}
http {
    include       mime.types;                     # ← 媒体类型映射
    default_type  application/octet-stream;
    sendfile        on;                           # ← 零拷贝传输
    keepalive_timeout 65;

    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html { root html; }
    }
}
```

### 4.3 常见安装/启动错误

| 现象 | 原因 | 解决 |
|---|---|---|
| `the HTTP rewrite module requires the PCRE library` | 缺 pcre-devel | `yum install pcre pcre-devel -y` |
| `SSL modules require the OpenSSL library` | 缺 openssl-devel | `yum install openssl openssl-devel -y` |
| `bind() to 0.0.0.0:80 failed (98: Address already in use)` | 已有 Nginx 实例 / 其他进程占用 80 | `nginx -s stop` 或 `fuser -k 80/tcp` |
| `getpwnam("nginx") failed` | `--user=nginx` 但系统无该用户 | `useradd -M -s /sbin/nologin nginx` 或改 `--user=www` |
| `make` 报 `-DHAVE_CONFIG_H` 错 | 缺 `gcc-c++` | `yum -y install gcc-c++` |
| 找不到 /application | 配置/编译/安装步骤没走通 | 重新执行 configure → make → make install |

### 4.4 排错三步

```
ping  域名 / IP       → 链路通?
telnet 域名 端口      → 端口开?
curl  域名/URL        → 业务通?
```

### 4.5 状态码 403 的两类成因

1. 服务端 IP 限制 / `deny` 命中。
2. 站点目录下**没有 index 首页文件**且未开 `autoindex on;`。

### 【纠错】

- 原文 §3.2 "首页文件"那一段用全角空格做了缩进(`　　`),**全角空格在 nginx.conf 中合法**(被当成普通空白),但容易和半角混淆,生产建议统一使用半角空格。

---

## 5. 虚拟主机(Virtual Host)

### 5.1 概念

虚拟主机:一台物理机/一组 nginx 进程对外承担多个独立站点。Apache 用 `<VirtualHost>...</VirtualHost>`,Nginx 用 `server {}`。

### 5.2 三种类型与对比

| 类型 | 区分依据 | 监听 | 配置关键字 | 典型场景 |
|---|---|---|---|---|
| 域名 | `Host` 头 | 同 IP 同端口 | `server_name xxx.com;` | 公网网站(主流) |
| 端口 | TCP 端口 | 同 IP 不同端口 | `listen 81;` | 内部后台 |
| IP | 服务器 IP | 不同 IP 同端口 | `listen 10.0.0.8:80;` | 多 IP 主机隔离 |

### 5.3 基于域名(最常用)

```nginx
server { listen 80; server_name www.nmtui.com;  location / { root html/www;  index index.html; } }
server { listen 80; server_name bbs.nmtui.com;  location / { root html/bbs;  index index.html; } }
server { listen 80; server_name blog.nmtui.com; location / { root html/blog; index index.html; } }
```

### 5.4 基于端口

```nginx
server { listen 80; server_name bbs.nmtui.com; location / { root html/bbs; index index.html; } }
server { listen 81; server_name bbs.nmtui.com; location / { root html/bbs; index index.html; } }
```

测试:`curl bbs.nmtui.com:81`。

### 5.5 基于 IP

```nginx
server { listen 10.0.0.7:80; server_name www.nmtui.com; ... }
server { listen 10.0.0.8:80; server_name www.nmtui.com; ... }
```

> **重要**:涉及 `listen` 改 IP 的修改,**不能 reload**,要 `nginx -s stop` 后再 `nginx`。原因:reload 是热更新 worker,**监听 socket 是 master 继承下来的**,绑定 IP 变化必须重启 master。

### 5.6 多虚拟主机的请求匹配流程

```
            收到一个 HTTP 请求
                  │
                  ▼
   ┌──────────────────────────────────┐
   │ 1. 看到了哪个 (IP:Port)?         │  匹配 listen
   └──────────────────────────────────┘
                  │
                  ▼
   ┌──────────────────────────────────┐
   │ 2. 该 listen 下有几个 server?    │
   │    若 1 个:直接命中               │
   │    若 多个:用 Host 头匹配         │
   │        a. server_name 精确匹配    │
   │        b. *.xx 通配               │
   │        c. xx.* 通配               │
   │        d. 正则匹配                │
   │    都不命中 → 取该 listen 的"默认"│
   │       server(default_server)      │
   └──────────────────────────────────┘
                  │
                  ▼
   ┌──────────────────────────────────┐
   │ 3. 在选中的 server 内匹配 location│  详见 §7
   └──────────────────────────────────┘
```

### 5.7 配置规范化:每虚拟主机一个文件

```
/application/nginx/conf/
├── nginx.conf                  # 仅保留全局/http 框架 + include
└── extra/
    ├── www.conf                # 一个 server {}
    ├── bbs.conf
    └── blog.conf
```

主文件 `nginx.conf`:

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile      on;
    keepalive_timeout 65;
    log_format main '...';

    include extra/www.conf;     # ← 推荐:显式列出,顺序可控
    include extra/bbs.conf;
    include extra/blog.conf;
    # 也可:include extra/*.conf;   ← 简洁但顺序按文件名字典序
}
```

**多文件 include 加载流程**:

```
   nginx -t / 启动时
        │
        ▼
   解析 nginx.conf
        │
        ▼
   遇到 include extra/www.conf  → 把内容嵌入到当前位置(http 块内)
        │
        ▼
   遇到 include extra/bbs.conf  → 同理
        │
        ▼
   解析完毕 → 形成内存中的完整配置树
        │
        ▼
   master 监听 80 → fork worker
```

使用 `nginx -T` 可看到最终展开后的完整配置(便于核对顺序与最终生效的 server 块)。

### 5.8 别名(server_name 多个)

```nginx
server {
    listen 80;
    server_name www.nmtui.com nmtui.cn;   # ← 两个域名指向同一站点
    location / { root html/www; index index.html; }
}
```

### 【纠错】

- 原文 §4.4.3 "基于 IP 的虚拟主机配置完要 `nginx -s stop` 后启动",原文给的理由仅是"涉及 IP 不能软重启"。**更准确的解释**:平滑 reload 时新 worker 会继承旧 master 的 listen socket,**listen 的 IP/端口变化无法热更新**,因此必须 stop + start(或 `nginx -s quit` + start)。
- 原文 §4.5.2 用 `sed -n '10,21p'` 切分配置文件——**脆弱**,行号一变就坏;**推荐**直接手动按 server 块拆分,文件命名 `站点名.conf`。

---

## 6. 状态模块与日志

### 6.1 stub_status

```nginx
server {
    listen 80;
    server_name status.nmtui.com;
    location / {
        stub_status on;
        access_log off;             # 状态页不需要日志
        allow 10.0.0.0/24;          # 限制内网访问
        deny all;
    }
}
```

返回示例:

```
Active connections: 291
server accepts handled requests
 16630948 16630948 31070465
Reading: 6  Writing: 179  Waiting: 106
```

| 指标 | 含义 |
|---|---|
| Active connections | 当前活动客户端连接数 |
| accepts | 进程启动以来接收的连接总数 |
| handled | 实际处理过的连接总数(若 = accepts 即未拒绝) |
| requests | 总请求数(同一连接可能多请求) |
| Reading | 正在读 HTTP 请求头的连接数 |
| Writing | 正在写响应的连接数 |
| Waiting | 已处理完、保持长连接等下一个请求的连接数 |

> Zabbix / Prometheus 抓这个页面就可以生成 QPS、并发、长连接占比等指标图。

### 6.2 日志类型与配置位置

| 日志 | 关键字 | 可放位置 |
|---|---|---|
| 错误日志 | `error_log path level;` | main / http / server / location |
| 访问日志 | `access_log path format;` | http / server / location |
| 日志格式 | `log_format name '...';` | **只能在 http** |

#### 错误日志 level(从详细到粗略)

```
debug → info → notice → warn → error → crit → alert → emerg
```

### 6.3 标准 main 格式与字段

```nginx
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
```

| 变量 | 含义 |
|---|---|
| `$remote_addr` | 客户端 IP(若有代理,看到的是代理 IP) |
| `$remote_user` | HTTP Basic Auth 用户名 |
| `$time_local` | 本地时间 |
| `$request` | 请求行(`GET /xxx HTTP/1.1`) |
| `$status` | 状态码 |
| `$body_bytes_sent` | 响应体大小(不含头) |
| `$http_referer` | 来源页(防盗链用) |
| `$http_user_agent` | 客户端 UA |
| `$http_x_forwarded_for` | 上游透传过来的真实链路 IP |

### 6.4 日志切割

#### 方法 1:脚本 + cron

```bash
#!/bin/bash
# /server/scripts/cut_nginx_log.sh
set -e
BASE=/application/nginx
DATE=$(date +%F)
LOGNAME=access_www

cd $BASE/logs
[ -f ${LOGNAME}.log ] || exit 0

mv ${LOGNAME}.log ${LOGNAME}_${DATE}.log
$BASE/sbin/nginx -s reopen        # 优于 reload:只重新打开日志文件,不重载配置
```

`crontab`:

```
59 23 * * * /bin/bash /server/scripts/cut_nginx_log.sh > /dev/null 2>&1
```

#### 方法 2:logrotate(系统自带)

```
/etc/logrotate.d/nginx
─────────────────────────────────────
/application/nginx/logs/*.log {
    daily
    rotate 30
    missingok
    notifempty
    sharedscripts
    postrotate
        [ -f /application/nginx/logs/nginx.pid ] && \
            kill -USR1 `cat /application/nginx/logs/nginx.pid`
    endscript
}
```

#### 切割流程示意

```
     22:00       23:59 cron                 0:00 开始的新请求
       │           │                                │
       │           ▼                                │
   旧 access.log   mv 改名 access_2025-05-29.log     │
                              │                     │
                              ▼                     │
                  nginx -s reopen(USR1)             │
                              │                     │
                              ▼                     │
                  新建 access.log 文件 ◀─────────── 新请求写入
```

> **注意**:`reload` 也会重新打开日志文件,但代价大(会重读配置、重启 worker);`reopen` / `USR1` 是只重开日志文件的轻量操作。

### 【纠错】

- 原文 §4.8.4 切割脚本里用 `nginx -s reload` 来"让新文件生效",**应改为** `nginx -s reopen`(对应信号 USR1),仅触发日志文件重打开,不浪费一次配置 reload。
- 原文里 `$body_bytes_sents` 多了一个 `s`,正确变量名为 `$body_bytes_sent`。

---

## 7. autoindex(目录列表)

### 7.1 用法

```nginx
location / {
    root   html/www;
    autoindex on;            # 没首页时直接列目录
    autoindex_exact_size off;# 文件大小用 K/M/G 显示
    autoindex_localtime on;  # 显示本地时间(默认 GMT)
}
```

### 7.2 行为

- Nginx **能解析**的(html、jpg 等)→ 在页面里显示。
- **不能解析**的(zip、iso、bin)→ 直接下载(浏览器行为)。
- 安全提醒:除了"开放下载"场景,**不要在生产开 autoindex**,容易暴露隐私目录。

---

## 8. location 路径匹配

### 8.1 语法

```nginx
location [= | ^~ | ~ | ~*] uri { ... }
```

| 修饰符 | 含义 |
|---|---|
| `=` | **精确匹配**,完全相等才命中,效率最高 |
| `^~` | 前缀匹配,**命中后不再做正则检查** |
| `~` | 区分大小写的**正则** |
| `~*` | 不区分大小写的**正则** |
| `!~` / `!~*` | 取反的正则 |
| (无) | 前缀匹配,**最长前缀**优先;若同时有正则匹配且正则命中,正则优先 |

### 8.2 匹配优先级流程图

```
  请求 URI 到达 server 块
            │
            ▼
   ┌────────────────────────────┐
   │ 1. 找所有 = 精确匹配?       │ 命中 → 终止
   └────────────────────────────┘
            │ 未命中
            ▼
   ┌────────────────────────────┐
   │ 2. 找最长前缀匹配           │
   │    其中如果是 ^~,记下并跳过 │
   │    正则检查直接使用         │
   └────────────────────────────┘
            │
            ▼
   ┌────────────────────────────┐
   │ 3. 按配置文件出现顺序检查   │
   │    正则(~ / ~*),命中即用 │
   └────────────────────────────┘
            │ 没有正则命中
            ▼
   ┌────────────────────────────┐
   │ 4. 回退使用第 2 步的最长前缀│
   └────────────────────────────┘
```

### 8.3 经典例子

```nginx
location = / {                       # [A] 精确 /
    [ config A ]
}
location / {                         # [B] 兜底
    [ config B ]
}
location /documents/ {               # [C] 前缀
    [ config C ]
}
location ^~ /images/ {               # [D] 前缀,屏蔽正则
    [ config D ]
}
location ~* \.(gif|jpg|jpeg)$ {      # [E] 正则
    [ config E ]
}
```

| 请求 | 命中 |
|---|---|
| `/` | A(精确) |
| `/index.html` | B(只有它能匹配) |
| `/documents/document.html` | C(前缀) |
| `/documents/x.jpg` | E(正则在 C 之上) |
| `/images/1.gif` | D(`^~` 跳过正则) |
| `/foo.jpg` | E |

### 8.4 测试 location 的小技巧

让每个 location 返回不同 HTTP 状态码,用 `curl` 一查便知命中哪个:

```nginx
location / {                          return 401; }
location = / {                        return 402; }
location /documents/ {                return 403; }
location ^~ /images/ {                return 404; }
location ~* \.(gif|jpg|jpeg)$ {       return 500; }
```

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://www.nmtui.com/                  → 402
curl -s -o /dev/null -w "%{http_code}\n" http://www.nmtui.com/anything          → 401
curl -s -o /dev/null -w "%{http_code}\n" http://www.nmtui.com/documents/        → 403
curl -s -o /dev/null -w "%{http_code}\n" http://www.nmtui.com/documents/a.jpg   → 500
curl -s -o /dev/null -w "%{http_code}\n" http://www.nmtui.com/images/1.gif      → 404
```

### 【纠错】

- 原文 §4.9.4.1 写正则用了**全角括号** `（gif|jpg|jpeg）`,在 PCRE 中全角括号是字面量,**不会按预期匹配**。本节已统一改为半角 `(gif|jpg|jpeg)`。

---

## 9. 内网访问控制(allow/deny)

### 9.1 需求

- 内网可以访问 `/AV`。
- 外网不能访问 `/AV`。

### 9.2 配置

```nginx
server {
    listen 80;
    server_name www.nmtui.com;

    location / {
        root   html/www;
        index  index.html index.htm;
    }
    location /AV {
        root  html/www;
        index index.html index.htm;
        allow 172.16.1.0/24;      # ← 内网网段
        deny  all;                # ← 其余全部拒绝
    }
}
```

### 9.3 调用流程

```
 用户请求 /AV/xxx
        │
        ▼
  匹配到 location /AV
        │
        ▼
  ngx_http_access_module
   ├─ 命中 172.16.1.0/24 → allow → 正常处理 root
   └─ 否则 → 403 Forbidden
```

---

## 10. rewrite(地址重写)

### 10.1 语法

```
rewrite  regex  replacement  [flag];
```

`flag`:

| 值 | 含义 |
|---|---|
| `last` | 完成本次 rewrite,**重新匹配 location** |
| `break` | 完成本次 rewrite,**不再匹配 location**,继续当前块 |
| `redirect` | 返回 **302** 临时重定向 |
| `permanent` | 返回 **301** 永久重定向 |

> 没有 flag → 默认相当于继续向下走,直到所有 rewrite 规则处理完。

### 10.2 应用场景

- URL 美化(动态地址伪静态)。
- 旧域名跳转到新域名(SEO 友好)。
- 强制 https。
- 根据特殊变量、目录、客户端做跳转。

### 10.3 两种"裸域跳 www"写法

#### 方法 1:在 `server` 里用 `if`

```nginx
server {
    listen 80;
    server_name  www.nmtui.com nmtui.cn;

    if ($host ~* "^nmtui\.cn$") {
        rewrite ^/(.*)$ http://www.nmtui.com/$1 permanent;
    }
    location / { root html/www; index index.html; }
}
```

#### 方法 2:独立 server(推荐)

```nginx
server {
    listen 80;
    server_name nmtui.cn;
    return 301 http://www.nmtui.com$request_uri;        # ← 比 rewrite 更高效
}
server {
    listen 80;
    server_name www.nmtui.com;
    root  html/www;
    location / { index index.html; }
}
```

### 10.4 强制 HTTPS

```nginx
server {
    listen 80;
    server_name www.nmtui.com;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl http2;
    server_name www.nmtui.com;
    ssl_certificate     /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ...
}
```

### 【纠错】

- 原文 §4.10.3 方法 1 的 `rewrite ^/(.*) http://www.nmtui.com/$1 permanent;` 写法**会被 if 包住**,*If is Evil*;**推荐用方法 2**(独立 server + `return 301`),性能更好、可读性更高、行为更确定。

---

## 11. 访问认证(HTTP Basic Auth)

### 11.1 整体流程

```
 浏览器                            Nginx                         密码文件
   │  GET /                          │                               │
   │ ───────────────────────────▶    │                               │
   │                                 │ 看到 auth_basic 指令          │
   │ ◀── 401 WWW-Authenticate:Basic  │                               │
   │                                 │                               │
   │  GET / Authorization: Basic xxx │                               │
   │ ───────────────────────────▶    │ 解 Base64,查 htpasswd 中是否 │
   │                                 │ 存在对应用户 + 密码哈希匹配  │
   │                                 │ ───────────────────────────▶ │
   │                                 │ ◀──────────────────────────  │
   │ ◀── 200 OK + 内容               │                               │
```

### 11.2 配置

```nginx
location / {
    root   html/www;
    index  index.html index.htm;
    auth_basic           "Restricted area";
    auth_basic_user_file /application/nginx/conf/htpasswd;
}
```

### 11.3 用 htpasswd 维护密码

```bash
# 首次:创建文件并加用户(-c 会覆盖文件)
htpasswd -c /application/nginx/conf/htpasswd clsn

# 后续:追加用户 / 改密(不带 -c)
htpasswd    /application/nginx/conf/htpasswd zhangsan

# 免交互(密码进入命令历史,慎用)
htpasswd -b /application/nginx/conf/htpasswd lisi 'P@ss'

# 删除用户
htpasswd -D /application/nginx/conf/htpasswd lisi
```

参数:

| 参数 | 含义 |
|---|---|
| `-c` | 创建新文件(已存在会覆盖!) |
| `-b` | 命令行指定密码,免交互 |
| `-D` | 删除用户 |
| `-m` | 强制 MD5 |
| `-s` | 强制 SHA |
| `-p` | 明文(危险) |
| `-n` | 不写文件,只打印结果 |

> htpasswd 来自 `httpd-tools`,需要先 `yum install -y httpd-tools`。

### 11.4 权限收紧

```bash
chown www:www /application/nginx/conf/htpasswd     # ← 让 worker 进程可读
chmod 400     /application/nginx/conf/htpasswd
```

### 11.5 验证

```bash
curl -u clsn:123456 http://www.nmtui.com           # 通过
curl http://www.nmtui.com                          # 返回 401
```

### 【纠错】

- 原文 §4.11.3 `chmod 400 ...` + `chown -R www.www ...` 的顺序应该先 `chown` 再 `chmod`,否则当前用户 chmod 400 后可能再没权 chown(若不是 root);本节给的顺序更稳妥。

---

## 12. 整篇主要纠错汇总

| 位置 | 原文 | 应为 |
|---|---|---|
| §2.4 Apache 模型 | "基于传统的 select 模型" | Apache 2.4 event MPM 已使用 epoll/kqueue/port,select 只是历史描述 |
| §3.5 reload 与 reopen | 切割日志后 `nginx -s reload` | 改为 `nginx -s reopen`,代价更小 |
| §4.4.3 IP 虚拟主机 | "涉及 IP 不能软重启,只能 stop+start" | 原因:listen socket 由 master 继承,IP/端口变化无法热更 |
| §4.5.2 sed 切配置 | 按行号 `sed -n '10,21p'` 拆 | 推荐按 server 块手工拆分,文件名 = 站点名 |
| §4.8.4 脚本 | `nginx -s reload` | 改为 `nginx -s reopen`(USR1) |
| §4.8 日志变量 | `$body_bytes_sents` | `$body_bytes_sent`(无尾 s) |
| §4.9.4 正则 | `\.（gif|jpg|jpeg）$`(全角括号) | `\.(gif|jpg|jpeg)$`(半角) |
| §4.10.3 if 重写 | `if + rewrite ... permanent` | 推荐独立 server + `return 301 ...$request_uri` |
| §3.4.5 `-V` | "显示编译参数" | 大写 `-V` 显示版本+参数,小写 `-v` 只显示版本 |
| §1.2 Tomcat | "Java 容器主流(如 jsp、do)" | Tomcat 是 Servlet/JSP 容器;`.do` 是 Struts 约定 |
