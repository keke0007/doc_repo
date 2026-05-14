# Nginx 与 Web 架构学习手册

> 本文档由 `nginx` 目录下原有 Markdown 笔记归纳、去重、重排后整理而成。内容不只是 Nginx 本身，也包含 Web 集群常用的时间同步、SSH、Rsync、NFS、Sersync、HTTPS、负载均衡和 Keepalived 高可用。每个知识点都补充了代码示例和应用场景，方便按实验逐步学习。

## 目录

- [1. Web 架构学习路线](#1-web-架构学习路线)
- [2. Chrony 时间同步](#2-chrony-时间同步)
- [3. SSH 远程管理](#3-ssh-远程管理)
- [4. Rsync 备份服务](#4-rsync-备份服务)
- [5. NFS 网络文件系统](#5-nfs-网络文件系统)
- [6. Sersync 实时同步](#6-sersync-实时同步)
- [7. HTTP 协议基础](#7-http-协议基础)
- [8. 网络 IO 模型](#8-网络-io-模型)
- [9. Nginx 快速入门](#9-nginx-快速入门)
- [10. Nginx 配置结构](#10-nginx-配置结构)
- [11. Nginx 常用模块](#11-nginx-常用模块)
- [12. Nginx 反向代理](#12-nginx-反向代理)
- [13. Nginx 七层负载均衡](#13-nginx-七层负载均衡)
- [14. Nginx 四层负载均衡](#14-nginx-四层负载均衡)
- [15. Nginx HTTPS 实践](#15-nginx-https-实践)
- [16. 常见 Web 架构场景](#16-常见-web-架构场景)
- [17. Keepalived 高可用](#17-keepalived-高可用)
- [18. Nginx 平滑升级与优化](#18-nginx-平滑升级与优化)
- [19. 综合实验](#19-综合实验)

## 1. Web 架构学习路线

### 1.1 为什么 Nginx 目录里有很多周边服务

知识点：真实 Web 架构不是单独安装一个 Nginx，而是多个组件协作。

应用场景：

- `Chrony`：保证多台服务器时间一致，便于日志分析、证书校验、分布式任务。
- `SSH`：远程登录、文件传输、免密运维。
- `Rsync`：备份、同步、灾备。
- `NFS`：多台 Web 共享上传文件。
- `Sersync`：监听文件变化并实时同步到备份节点。
- `Nginx`：静态资源、反向代理、负载均衡、HTTPS、限流、缓存。
- `Keepalived`：为 Nginx 负载均衡层提供 VIP 漂移和高可用。

推荐学习顺序：

1. 先学 HTTP 协议和 Nginx 基础。
2. 再学静态站点、虚拟主机、日志、访问控制、代理。
3. 然后学习负载均衡、HTTPS、会话共享、真实 IP。
4. 最后学习高可用、平滑升级、内核和 Nginx 性能优化。

### 1.2 常用实验主机规划

应用场景：搭建 Nginx 反向代理、负载均衡、NFS、Rsync、Keepalived 实验环境。

示例规划：

| 主机 | IP | 角色 |
| --- | --- | --- |
| `m01` | `10.0.0.61` | 管理机、跳板机 |
| `backup` | `10.0.0.41` | Rsync 备份服务器 |
| `nfs` | `10.0.0.31` | NFS 共享存储 |
| `web01` | `10.0.0.7` | Web 节点 |
| `web02` | `10.0.0.8` | Web 节点 |
| `proxy01` | `10.0.0.5` | Nginx 负载均衡 |
| `proxy02` | `10.0.0.6` | Nginx 负载均衡备用 |
| `vip` | `10.0.0.3` | Keepalived 虚拟 IP |

代码示例：

```bash
# 每台机器设置主机名
hostnamectl set-hostname web01

# 配置 hosts 方便实验
cat >> /etc/hosts << EOF
10.0.0.5 proxy01
10.0.0.6 proxy02
10.0.0.7 web01
10.0.0.8 web02
10.0.0.31 nfs
10.0.0.41 backup
10.0.0.61 m01
EOF
```

## 2. Chrony 时间同步

### 2.1 时间同步的作用

知识点：时间同步用于保证多台服务器时间一致。服务器时间不一致会导致日志顺序混乱、证书校验失败、分布式任务异常、备份校验困难。

应用场景：Web 集群、数据库集群、负载均衡、HTTPS 证书、定时任务、日志审计。

代码示例：

```bash
# 查看当前时间
date

# 查看时区
timedatectl

# 设置上海时区
timedatectl set-timezone Asia/Shanghai
```

### 2.2 Chrony 服务端

知识点：Chrony 是 CentOS 7/8 常用时间同步服务，适合做内网时间服务器。

应用场景：内网机器统一从一台 Chrony 服务器同步时间，减少公网访问和时间偏差。

代码示例：

```bash
yum install -y chrony

# /etc/chrony.conf 核心配置
cat > /etc/chrony.conf << EOF
server ntp.aliyun.com iburst
server ntp.tencent.com iburst
allow 10.0.0.0/24
local stratum 10
EOF

systemctl enable --now chronyd
systemctl restart chronyd

# 查看同步源
chronyc sources -v
```

### 2.3 Chrony 客户端

知识点：客户端只需要把时间源指向内网 Chrony 服务器。

应用场景：Web、NFS、Rsync、Proxy 节点统一向 `m01` 或专用时间服务器同步。

代码示例：

```bash
yum install -y chrony

cat > /etc/chrony.conf << EOF
server 10.0.0.61 iburst
EOF

systemctl enable --now chronyd
systemctl restart chronyd

chronyc tracking
chronyc sources -v
```

## 3. SSH 远程管理

### 3.1 SSH 基础

知识点：SSH 用于加密远程登录和文件传输，比 Telnet 安全。常见端口是 `22`。

应用场景：远程管理 Linux 服务器、执行命令、传输配置和脚本。

代码示例：

```bash
# 远程登录
ssh root@10.0.0.7

# 指定端口
ssh -p 2222 root@10.0.0.7

# 远程执行命令
ssh root@10.0.0.7 "hostname; uptime"
```

### 3.2 scp 文件传输

知识点：`scp` 基于 SSH 加密传输，可推送或拉取文件。

应用场景：分发 Nginx 配置、上传脚本、拉取日志。

代码示例：

```bash
# 推送文件到远端
scp nginx.conf root@10.0.0.7:/etc/nginx/nginx.conf

# 推送目录
scp -r html root@10.0.0.7:/usr/share/nginx/

# 从远端拉取日志
scp root@10.0.0.7:/var/log/nginx/access.log ./
```

### 3.3 SSH 密钥登录

知识点：密钥登录由私钥和公钥组成。私钥保存在客户端，公钥放到服务器 `~/.ssh/authorized_keys`。

应用场景：自动化运维、批量分发配置、跳板机管理。

代码示例：

```bash
# 在管理机生成密钥
ssh-keygen -t rsa -b 4096 -C "m01"

# 推送公钥
ssh-copy-id root@10.0.0.7
ssh-copy-id root@10.0.0.8

# 测试免密登录
ssh root@10.0.0.7 "hostname"
```

### 3.4 SSH 安全优化

知识点：公网服务器应减少暴露面，避免 root 直接登录，优先使用密钥认证和防火墙限制来源。

应用场景：云服务器、跳板机、生产服务器 SSH 加固。

代码示例：

```bash
# /etc/ssh/sshd_config 建议项
Port 2222
PermitRootLogin no
PasswordAuthentication no
UseDNS no
GSSAPIAuthentication no

# 重启 sshd 前先测试配置
sshd -t
systemctl restart sshd
```

## 4. Rsync 备份服务

### 4.1 Rsync 是什么

知识点：Rsync 是高效文件同步工具，支持增量传输、本地同步、SSH 远程同步、守护进程模式。

应用场景：站点备份、配置同步、日志归档、灾备服务器。

代码示例：

```bash
# 本地同步
rsync -av /data/ /backup/data/

# 通过 SSH 推送
rsync -avz /etc/ root@10.0.0.41:/backup/web01/etc/

# 从远端拉取
rsync -avz root@10.0.0.7:/var/www/html/ ./html/
```

### 4.2 Rsync 常用参数

知识点：常用组合是 `-avz`，生产同步常配合 `--delete`、`--exclude`、`--bwlimit`。

应用场景：无差异同步、限速同步、排除临时文件。

代码示例：

```bash
# 保持目标目录和源目录一致，目标多余文件会被删除
rsync -av --delete /data/ /backup/data/

# 排除日志和缓存
rsync -av --exclude='*.log' --exclude='cache/' /app/ /backup/app/

# 限速，单位 KB/s
rsync -av --bwlimit=1024 /data/ root@10.0.0.41:/backup/data/
```

### 4.3 Rsync 守护进程服务端

知识点：守护进程模式使用模块名暴露目录，客户端通过 `rsync://` 或 `user@host::module` 访问。

应用场景：集中备份服务器，允许多个客户端推送备份。

代码示例：

```bash
yum install -y rsync
useradd -r -s /sbin/nologin rsync
mkdir -p /backup
chown -R rsync:rsync /backup

cat > /etc/rsyncd.conf << EOF
uid = rsync
gid = rsync
port = 873
fake super = yes
use chroot = no
max connections = 200
timeout = 600
ignore errors
read only = false
list = false
auth users = rsync_backup
secrets file = /etc/rsync.passwd
log file = /var/log/rsyncd.log

[backup]
comment = backup directory
path = /backup
EOF

echo "rsync_backup:123456" > /etc/rsync.passwd
chmod 600 /etc/rsync.passwd

systemctl enable --now rsyncd
ss -lntup | grep 873
```

### 4.4 Rsync 客户端

知识点：客户端密码文件只保存密码，不保存用户名，权限必须严格。

应用场景：Web 节点每日推送配置和代码备份到备份服务器。

代码示例：

```bash
echo "123456" > /etc/rsync.pass
chmod 600 /etc/rsync.pass

# 推送
rsync -avz /etc/ rsync_backup@10.0.0.41::backup/web01/etc/ --password-file=/etc/rsync.pass

# 拉取
rsync -avz rsync_backup@10.0.0.41::backup/web01/etc/ /tmp/etc/ --password-file=/etc/rsync.pass
```

### 4.5 自动备份脚本

知识点：备份脚本一般包含打包、校验、推送、保留周期、日志输出。

应用场景：每天凌晨备份系统配置、应用配置和站点目录。

代码示例：

```bash
mkdir -p /server/scripts /backup

cat > /server/scripts/client_backup.sh << 'EOF'
#!/bin/bash
set -e
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

HOST=$(hostname)
DATE=$(date +%F)
LOCAL_DIR=/backup/${HOST}_${DATE}
PASS_FILE=/etc/rsync.pass
RSYNC_USER=rsync_backup
RSYNC_SERVER=10.0.0.41
RSYNC_MODULE=backup

mkdir -p "${LOCAL_DIR}"
tar -czf "${LOCAL_DIR}/etc.tar.gz" /etc
tar -czf "${LOCAL_DIR}/www.tar.gz" /var/www/html 2>/dev/null || true
find "${LOCAL_DIR}" -type f -exec md5sum {} \; > "${LOCAL_DIR}/md5sum.txt"

rsync -az "${LOCAL_DIR}/" ${RSYNC_USER}@${RSYNC_SERVER}::${RSYNC_MODULE}/${HOST}/${DATE}/ --password-file=${PASS_FILE}
find /backup -type d -mtime +7 -name "${HOST}_*" -exec rm -rf {} \;
EOF

chmod +x /server/scripts/client_backup.sh

# crontab -e
# 0 1 * * * /server/scripts/client_backup.sh >> /var/log/client_backup.log 2>&1
```

## 5. NFS 网络文件系统

### 5.1 NFS 是什么

知识点：NFS 允许 Linux 主机通过网络挂载远端目录，像使用本地目录一样读写文件。

应用场景：多台 Web 节点共享用户上传的图片、附件、静态资源。

代码示例：

```bash
# 服务端安装
yum install -y nfs-utils rpcbind

# 客户端安装
yum install -y nfs-utils
```

### 5.2 NFS 服务端配置

知识点：`/etc/exports` 定义共享目录、允许访问的客户端和权限参数。

应用场景：让 `10.0.0.0/24` 网段的 Web 节点读写 `/data`。

代码示例：

```bash
mkdir -p /data
useradd -u 666 -M -s /sbin/nologin www
chown -R www:www /data

cat > /etc/exports << EOF
/data 10.0.0.0/24(rw,sync,all_squash,anonuid=666,anongid=666)
EOF

systemctl enable --now rpcbind nfs
exportfs -rv
showmount -e 127.0.0.1
```

常用参数：

| 参数 | 含义 | 应用场景 |
| --- | --- | --- |
| `rw` | 允许读写 | Web 上传目录 |
| `ro` | 只读 | 静态资源分发 |
| `sync` | 同步写入磁盘 | 数据安全优先 |
| `async` | 异步写入 | 性能优先但风险更高 |
| `all_squash` | 所有用户映射为匿名用户 | 统一权限 |
| `anonuid/anongid` | 指定匿名用户 UID/GID | 映射为 `www` 用户 |

### 5.3 NFS 客户端挂载

知识点：客户端通过 `mount -t nfs` 挂载服务端共享目录。

应用场景：Web 节点把上传目录挂载到 NFS。

代码示例：

```bash
showmount -e 10.0.0.31
mkdir -p /var/www/html/upload
mount -t nfs 10.0.0.31:/data /var/www/html/upload
df -h

# 永久挂载
echo "10.0.0.31:/data /var/www/html/upload nfs defaults,_netdev 0 0" >> /etc/fstab
mount -a
```

### 5.4 NFS 排错

知识点：NFS 问题通常与服务状态、导出配置、权限映射、防火墙、挂载参数有关。

应用场景：客户端无法挂载、能读不能写、写入文件属主异常。

代码示例：

```bash
# 服务端检查
systemctl status rpcbind nfs
exportfs -v
showmount -e 10.0.0.31

# 客户端检查
mount | grep nfs
touch /var/www/html/upload/test.txt
ls -ln /var/www/html/upload/test.txt

# 重新加载导出
exportfs -rv
```

## 6. Sersync 实时同步

### 6.1 实时同步原理

知识点：Sersync 基于 inotify 监听文件变化，再调用 rsync 同步变更数据。

应用场景：NFS 数据实时备份到备份服务器，图片上传后自动同步到灾备节点。

代码示例：

```bash
# 查看系统 inotify 限制
sysctl fs.inotify.max_user_watches

# 临时调大
sysctl -w fs.inotify.max_user_watches=524288
```

### 6.2 Sersync 典型架构

应用场景：用户上传文件到 NFS，Sersync 将 NFS 的 `/data` 实时同步到 backup 的 rsync 模块。

配置思路：

1. backup 部署 rsync 守护进程，新增 `data` 模块。
2. nfs 服务器安装 Sersync。
3. Sersync 监听 `/data`，变更后推送到 `backup::data`。

Rsync 服务端模块示例：

```bash
cat >> /etc/rsyncd.conf << EOF

[data]
comment = nfs data backup
path = /backup/data
read only = false
auth users = rsync_backup
secrets file = /etc/rsync.passwd
EOF

mkdir -p /backup/data
chown -R rsync:rsync /backup/data
systemctl restart rsyncd
```

Sersync 同步命令示例：

```bash
# 先做一次全量同步
rsync -az /data/ rsync_backup@10.0.0.41::data --password-file=/etc/rsync.pass

# 启动 sersync，具体二进制路径按实际安装目录调整
/usr/local/sersync/sersync2 -dro /usr/local/sersync/confxml.xml
```

## 7. HTTP 协议基础

### 7.1 URL、HTML、HTTP

知识点：

- URL 是资源地址，例如 `https://www.example.com/index.html`。
- HTML 是浏览器展示的页面内容。
- HTTP 是浏览器和服务器之间传输请求与响应的协议。

应用场景：理解 Nginx 的 `server_name`、`location`、反向代理、状态码和请求头。

代码示例：

```bash
# 查看 HTTP 响应头
curl -I http://example.com

# 查看完整请求过程
curl -v http://example.com
```

### 7.2 HTTP 请求

知识点：HTTP 请求由请求行、请求头、空行、请求体组成。常见方法：GET、POST、PUT、DELETE、HEAD。

应用场景：区分页面访问、表单提交、接口请求、文件上传。

代码示例：

```bash
# GET 请求
curl "http://127.0.0.1/index.html"

# POST 请求
curl -X POST -d "user=alice&password=123" http://127.0.0.1/login

# 自定义请求头
curl -H "Host: www.example.com" http://10.0.0.7/
```

### 7.3 HTTP 响应

知识点：HTTP 响应包含状态码、响应头、响应体。常见状态码：`200` 成功，`301/302` 跳转，`403` 禁止，`404` 不存在，`500/502/504` 服务端或代理错误。

应用场景：排查 Nginx 访问失败。

代码示例：

```bash
curl -I http://127.0.0.1/

# 只输出状态码
curl -o /dev/null -s -w "%{http_code}\n" http://127.0.0.1/
```

### 7.4 HTTP 请求头

知识点：请求头用于携带客户端、域名、连接、缓存、代理链路等信息。

应用场景：反向代理转发真实 IP、按 User-Agent 调度移动端页面。

代码示例：

```bash
curl -H "User-Agent: iPhone" http://www.example.com/
curl -H "X-Forwarded-For: 1.1.1.1" http://www.example.com/
```

## 8. 网络 IO 模型

### 8.1 阻塞、非阻塞、同步、异步

知识点：阻塞/非阻塞描述调用者等待数据的方式；同步/异步描述事件完成后由谁继续处理。Nginx 使用事件驱动、非阻塞 IO，适合高并发连接。

应用场景：理解为什么 Nginx 相比传统多进程 Web 服务更适合静态资源和高并发代理。

代码示例：

```bash
# 查看 Nginx worker 进程
ps -ef | grep nginx

# 查看连接数量
ss -ant | awk 'NR>1 {print $1}' | sort | uniq -c
```

### 8.2 Nginx 事件模型

知识点：Linux 下 Nginx 通常使用 `epoll` 事件模型，一个 worker 可以处理大量连接。

应用场景：配置高并发 Nginx 时，需要关注 `worker_processes`、`worker_connections` 和系统文件描述符限制。

代码示例：

```nginx
worker_processes auto;

events {
    use epoll;
    worker_connections 65535;
}
```

```bash
# 查看最大打开文件数
ulimit -n

# 查看 Nginx 编译参数
nginx -V
```

## 9. Nginx 快速入门

### 9.1 Nginx 是什么

知识点：Nginx 是高性能 Web 服务器、反向代理服务器、负载均衡器，也支持 TCP/UDP 四层代理。

应用场景：静态网站、API 网关、反向代理、负载均衡、HTTPS 入口、缓存加速。

代码示例：

```bash
yum install -y epel-release
yum install -y nginx
systemctl enable --now nginx
nginx -v
curl -I http://127.0.0.1
```

### 9.2 源码编译安装

知识点：源码编译可以自定义安装路径、模块、运行用户和编译参数。

应用场景：需要启用特定模块、使用指定版本、做平滑升级实验。

代码示例：

```bash
yum install -y gcc gcc-c++ make pcre-devel openssl-devel zlib-devel
useradd -r -s /sbin/nologin nginx

tar -xzf nginx-1.24.0.tar.gz
cd nginx-1.24.0
./configure \
  --prefix=/usr/local/nginx \
  --user=nginx \
  --group=nginx \
  --with-http_ssl_module \
  --with-http_stub_status_module \
  --with-stream
make
make install

/usr/local/nginx/sbin/nginx -t
/usr/local/nginx/sbin/nginx
```

### 9.3 Nginx 常用命令

知识点：Nginx 配置变更前要先检查语法，再重载。

应用场景：修改虚拟主机、代理、HTTPS 配置后安全生效。

代码示例：

```bash
# 检查配置
nginx -t

# 启动、停止、重载
systemctl start nginx
systemctl stop nginx
systemctl reload nginx
systemctl restart nginx

# 源码安装常用方式
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -s reload
/usr/local/nginx/sbin/nginx -s quit
```

## 10. Nginx 配置结构

### 10.1 主配置结构

知识点：Nginx 配置分为全局块、events 块、http 块、server 块、location 块。

应用场景：理解配置指令应该写在哪一层。

代码示例：

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;

events {
    worker_connections 10240;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    sendfile on;
    keepalive_timeout 65;

    include /etc/nginx/conf.d/*.conf;
}
```

### 10.2 虚拟主机 server

知识点：一个 Nginx 可以通过多个 `server` 提供多个站点，常按域名或端口区分。

应用场景：一台服务器部署多个网站。

代码示例：

```nginx
server {
    listen 80;
    server_name www.example.com;
    root /usr/share/nginx/html/example;
    index index.html index.htm;

    access_log /var/log/nginx/www.example.com.access.log main;
    error_log  /var/log/nginx/www.example.com.error.log warn;
}
```

### 10.3 location 匹配

知识点：`location` 用于按 URI 路径选择处理规则。常见匹配：精确匹配 `=`、前缀匹配、正则匹配 `~`。

应用场景：静态资源走本地目录，接口请求转发给后端。

代码示例：

```nginx
server {
    listen 80;
    server_name www.example.com;

    location = /health {
        return 200 "ok\n";
    }

    location /static/ {
        root /data/www;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        include fastcgi_params;
    }
}
```

## 11. Nginx 常用模块

### 11.1 静态资源 root 与 alias

知识点：`root` 会把 URI 拼接到指定目录；`alias` 会把匹配到的路径替换成指定目录。

应用场景：发布静态网站、图片目录、下载目录。

代码示例：

```nginx
server {
    listen 80;
    server_name static.example.com;

    location /images/ {
        root /data/www;
    }

    location /download/ {
        alias /data/files/;
        autoindex on;
    }
}
```

### 11.2 autoindex 目录浏览

知识点：`autoindex on` 可显示目录文件列表。

应用场景：内网软件下载站、临时文件共享。

代码示例：

```nginx
location /repo/ {
    alias /data/repo/;
    autoindex on;
    autoindex_localtime on;
    autoindex_exact_size off;
}
```

### 11.3 访问控制 allow/deny

知识点：基于 IP 允许或拒绝访问。

应用场景：后台管理页面只允许办公网访问。

代码示例：

```nginx
location /admin/ {
    allow 10.0.0.0/24;
    deny all;
    proxy_pass http://127.0.0.1:8080;
}
```

### 11.4 用户认证 auth_basic

知识点：通过 HTTP Basic Auth 增加用户名密码认证。

应用场景：临时保护测试站点、下载目录、内部管理页。

代码示例：

```bash
yum install -y httpd-tools
htpasswd -bc /etc/nginx/auth_basic admin '123456'
```

```nginx
location /private/ {
    auth_basic "private site";
    auth_basic_user_file /etc/nginx/auth_basic;
    root /data/www;
}
```

### 11.5 状态监控 stub_status

知识点：`stub_status` 输出 Nginx 当前连接和请求状态。

应用场景：监控 Nginx 活跃连接、请求量、读写等待情况。

代码示例：

```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    allow 10.0.0.0/24;
    deny all;
}
```

```bash
curl http://127.0.0.1/nginx_status
```

### 11.6 日志配置

知识点：访问日志记录请求，错误日志记录异常。自定义日志格式可记录真实 IP、耗时、上游状态。

应用场景：访问统计、故障排查、性能分析。

代码示例：

```nginx
log_format proxy '$remote_addr $http_x_forwarded_for [$time_local] '
                 '"$request" $status $body_bytes_sent '
                 '$request_time $upstream_response_time '
                 '$upstream_addr $upstream_status';

access_log /var/log/nginx/proxy_access.log proxy;
error_log /var/log/nginx/proxy_error.log warn;
```

日志分析示例：

```bash
# 访问最多的 IP
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head

# 状态码统计
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -nr
```

### 11.7 限速与限流

知识点：`limit_req` 限制请求频率，`limit_conn` 限制连接数。

应用场景：保护登录接口、下载接口，缓解恶意请求。

代码示例：

```nginx
http {
    limit_req_zone $binary_remote_addr zone=req_limit:10m rate=5r/s;
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

    server {
        listen 80;
        server_name www.example.com;

        location /login {
            limit_req zone=req_limit burst=10 nodelay;
            proxy_pass http://app;
        }

        location /download/ {
            limit_conn conn_limit 2;
            limit_rate 1m;
            root /data/www;
        }
    }
}
```

### 11.8 rewrite 与 return

知识点：`return` 适合简单跳转或返回状态码；`rewrite` 适合 URI 重写。

应用场景：旧链接跳转新链接、HTTP 跳 HTTPS、伪静态。

代码示例：

```nginx
# 简单跳转
server {
    listen 80;
    server_name old.example.com;
    return 301 http://www.example.com$request_uri;
}

# URI 重写
location /old/ {
    rewrite ^/old/(.*)$ /new/$1 permanent;
}
```

## 12. Nginx 反向代理

### 12.1 正向代理与反向代理

知识点：正向代理代理客户端，服务端不知道真实客户端；反向代理代理服务端，客户端不知道真实后端。

应用场景：

- 正向代理：内网客户端访问外部资源、统一出口。
- 反向代理：Nginx 接收用户请求，再转发给后端 Web/API。

### 12.2 基础反向代理

知识点：`proxy_pass` 将请求转发到后端服务。

应用场景：Nginx 代理 Java、PHP、Node.js、Go 服务。

代码示例：

```nginx
server {
    listen 80;
    server_name app.example.com;

    location / {
        proxy_pass http://10.0.0.7:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 12.3 代理超时与缓冲

知识点：代理连接、发送、读取都有超时时间；缓冲区影响大响应和慢客户端场景。

应用场景：后端接口慢、上传下载大文件、502/504 排查。

代码示例：

```nginx
location /api/ {
    proxy_pass http://10.0.0.7:8080;
    proxy_connect_timeout 5s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;

    proxy_buffering on;
    proxy_buffer_size 64k;
    proxy_buffers 8 64k;
    proxy_busy_buffers_size 128k;
}
```

### 12.4 proxy_pass 结尾斜杠

知识点：`proxy_pass` 是否带 `/` 会影响 URI 转发结果。

应用场景：代理接口路径时避免后端收到错误路径。

代码示例：

```nginx
# 不带斜杠：/api/user -> http://backend/api/user
location /api/ {
    proxy_pass http://backend;
}

# 带斜杠：/api/user -> http://backend/user
location /api/ {
    proxy_pass http://backend/;
}
```

## 13. Nginx 七层负载均衡

### 13.1 upstream 基础

知识点：七层负载均衡基于 HTTP 请求进行转发，通过 `upstream` 定义后端节点。

应用场景：多台 Web 服务器分摊访问压力。

代码示例：

```nginx
upstream web_cluster {
    server 10.0.0.7:80;
    server 10.0.0.8:80;
}

server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://web_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 13.2 调度算法

知识点：Nginx 常用调度方式包括轮询、权重、ip_hash、least_conn。

应用场景：不同机器配置不同、需要会话粘滞、长连接服务。

代码示例：

```nginx
# 权重
upstream web_weight {
    server 10.0.0.7:80 weight=3;
    server 10.0.0.8:80 weight=1;
}

# 按客户端 IP 粘滞
upstream web_iphash {
    ip_hash;
    server 10.0.0.7:80;
    server 10.0.0.8:80;
}

# 最少连接
upstream web_least {
    least_conn;
    server 10.0.0.7:80;
    server 10.0.0.8:80;
}
```

### 13.3 后端健康与容错参数

知识点：开源 Nginx 没有主动健康检查，但可以通过失败次数和超时时间做被动摘除。

应用场景：后端节点异常时降低用户访问失败概率。

代码示例：

```nginx
upstream web_cluster {
    server 10.0.0.7:80 max_fails=3 fail_timeout=10s;
    server 10.0.0.8:80 max_fails=3 fail_timeout=10s;
    server 10.0.0.9:80 backup;
}

location / {
    proxy_pass http://web_cluster;
    proxy_next_upstream error timeout http_500 http_502 http_503 http_504;
}
```

### 13.4 动静分离

知识点：静态资源由 Nginx 直接处理，动态请求转发给后端应用。

应用场景：减少后端压力，提高静态资源访问速度。

代码示例：

```nginx
server {
    listen 80;
    server_name www.example.com;

    location ~* \.(jpg|jpeg|png|gif|css|js|ico)$ {
        root /data/www;
        expires 7d;
    }

    location / {
        proxy_pass http://app_cluster;
    }
}
```

### 13.5 根据 URI 调度

知识点：不同 URI 可转发到不同后端集群。

应用场景：`/api` 走接口服务，`/admin` 走后台服务，`/static` 走静态服务器。

代码示例：

```nginx
upstream api_cluster {
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
}

upstream admin_cluster {
    server 10.0.0.21:8080;
}

server {
    listen 80;
    server_name www.example.com;

    location /api/ {
        proxy_pass http://api_cluster;
    }

    location /admin/ {
        proxy_pass http://admin_cluster;
    }
}
```

### 13.6 根据设备调度

知识点：可基于 `User-Agent` 判断移动端或 PC 端。

应用场景：移动端访问 `m.example.com` 或移动端后端集群。

代码示例：

```nginx
server {
    listen 80;
    server_name www.example.com;

    if ($http_user_agent ~* "(Android|iPhone|iPad)") {
        return 302 http://m.example.com$request_uri;
    }

    location / {
        proxy_pass http://pc_cluster;
    }
}
```

### 13.7 多级代理获取真实 IP

知识点：多级代理会让后端看到上一层代理 IP，需要通过 `X-Forwarded-For` 或 `real_ip` 模块还原真实客户端 IP。

应用场景：日志审计、限流、风控、访问统计。

代码示例：

```nginx
# 代理层传递真实 IP
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

```nginx
# 后端 Nginx 使用 realip 模块
set_real_ip_from 10.0.0.0/24;
real_ip_header X-Forwarded-For;
real_ip_recursive on;

log_format realip '$remote_addr $http_x_forwarded_for "$request" $status';
access_log /var/log/nginx/access.log realip;
```

## 14. Nginx 四层负载均衡

### 14.1 四层负载均衡是什么

知识点：四层负载均衡基于 TCP/UDP 连接转发，不理解 HTTP 内容。Nginx 通过 `stream` 模块实现。

应用场景：MySQL、Redis、SSH、TCP 服务代理，也可用于四层代理七层 Nginx。

代码示例：

```bash
# 确认是否支持 stream
nginx -V 2>&1 | grep -- --with-stream
```

### 14.2 TCP 负载均衡配置

应用场景：用 Nginx 代理后端 MySQL 或 TCP 服务。

代码示例：

```nginx
stream {
    upstream mysql_cluster {
        server 10.0.0.51:3306;
        server 10.0.0.52:3306;
    }

    server {
        listen 3306;
        proxy_connect_timeout 3s;
        proxy_timeout 30s;
        proxy_pass mysql_cluster;
    }
}
```

### 14.3 四层代理 HTTP

知识点：四层代理只转发 TCP，不处理 HTTP header，也无法做 URI 路由。

应用场景：四层 Nginx 负责入口端口转发，后面接七层 Nginx 集群。

代码示例：

```nginx
stream {
    upstream proxy_cluster {
        server 10.0.0.5:80;
        server 10.0.0.6:80;
    }

    server {
        listen 80;
        proxy_pass proxy_cluster;
    }
}
```

## 15. Nginx HTTPS 实践

### 15.1 HTTPS 基础

知识点：HTTPS = HTTP + TLS。TLS 通过证书验证身份，并协商对称密钥加密传输数据。

应用场景：登录、支付、后台管理、API、所有公网网站。

代码示例：

```bash
# 查看证书信息
openssl x509 -in server.crt -noout -text

# 测试 HTTPS
curl -Iv https://www.example.com
```

### 15.2 自签证书

知识点：自签证书适合测试，不被浏览器信任；生产应使用正规 CA 证书。

应用场景：内网测试 HTTPS、学习 TLS 配置。

代码示例：

```bash
mkdir -p /etc/nginx/ssl
openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/server.key \
  -out /etc/nginx/ssl/server.crt \
  -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Example/OU=IT/CN=www.example.com"
```

### 15.3 Nginx HTTPS 配置

应用场景：单台 Nginx 提供 HTTPS 网站。

代码示例：

```nginx
server {
    listen 443 ssl http2;
    server_name www.example.com;

    ssl_certificate     /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    root /usr/share/nginx/html;
    index index.html;
}
```

### 15.4 HTTP 强制跳转 HTTPS

应用场景：公网网站统一加密访问。

代码示例：

```nginx
server {
    listen 80;
    server_name www.example.com;
    return 301 https://$host$request_uri;
}
```

### 15.5 HTTPS 反向代理集群

知识点：证书通常部署在负载均衡层，由 Nginx 解密后转发给后端 HTTP 服务。

应用场景：统一 HTTPS 入口，后端 Web 节点走内网 HTTP。

代码示例：

```nginx
upstream web_cluster {
    server 10.0.0.7:80;
    server 10.0.0.8:80;
}

server {
    listen 443 ssl http2;
    server_name www.example.com;

    ssl_certificate     /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        proxy_pass http://web_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

## 16. 常见 Web 架构场景

### 16.1 LNMP 架构

知识点：LNMP = Linux + Nginx + MySQL/MariaDB + PHP，Nginx 通过 FastCGI 转发 PHP 请求给 php-fpm。

应用场景：WordPress、Discuz、phpMyAdmin、传统 PHP 网站。

代码示例：

```bash
yum install -y nginx php-fpm php-mysqlnd mariadb-server
systemctl enable --now nginx php-fpm mariadb
```

```nginx
server {
    listen 80;
    server_name php.example.com;
    root /usr/share/nginx/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### 16.2 会话共享

知识点：负载均衡下，如果 session 保存在单台 Web 本地，用户请求被调度到其他节点会导致登录丢失。常见方案是 session 存 Redis。

应用场景：多台 PHP Web 节点共享登录状态。

代码示例：

```bash
yum install -y redis
systemctl enable --now redis

# PHP session 示例配置，按实际环境修改
grep -n "session.save" /etc/php.ini
```

```ini
session.save_handler = redis
session.save_path = "tcp://10.0.0.51:6379"
```

### 16.3 静态资源共享

知识点：用户上传文件要么使用共享存储 NFS，要么上传到对象存储/CDN。实验环境常用 NFS。

应用场景：多台 Web 节点访问同一份上传图片。

代码示例：

```bash
# Web 节点挂载 NFS 上传目录
mkdir -p /usr/share/nginx/html/upload
mount -t nfs 10.0.0.31:/data /usr/share/nginx/html/upload

# 验证
echo "upload test" > /usr/share/nginx/html/upload/test.txt
curl http://127.0.0.1/upload/test.txt
```

## 17. Keepalived 高可用

### 17.1 高可用与 VRRP

知识点：高可用用于减少单点故障。Keepalived 基于 VRRP 实现 VIP 漂移，主节点故障时 VIP 自动切换到备节点。

应用场景：两台 Nginx 负载均衡器做主备，用户只访问 VIP。

### 17.2 Keepalived 安装

代码示例：

```bash
yum install -y keepalived
systemctl enable keepalived
```

### 17.3 Master 配置

应用场景：`proxy01` 持有 VIP `10.0.0.3`。

代码示例：

```conf
# /etc/keepalived/keepalived.conf
global_defs {
    router_id proxy01
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.3/24
    }
}
```

### 17.4 Backup 配置

应用场景：`proxy02` 在 Master 故障后接管 VIP。

代码示例：

```conf
global_defs {
    router_id proxy02
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.3/24
    }
}
```

验证：

```bash
systemctl start keepalived
ip addr show eth0 | grep 10.0.0.3
tcpdump -i eth0 vrrp
```

### 17.5 Keepalived 监控 Nginx

知识点：只监控服务器存活不够，还要监控 Nginx 服务是否正常。

应用场景：Master 机器没宕机但 Nginx 停了，VIP 应漂移到 Backup。

代码示例：

```bash
cat > /etc/keepalived/check_nginx.sh << 'EOF'
#!/bin/bash
systemctl is-active nginx >/dev/null 2>&1
EOF

chmod +x /etc/keepalived/check_nginx.sh
```

```conf
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 2
    weight -50
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.0.0.3/24
    }
    track_script {
        check_nginx
    }
}
```

### 17.6 脑裂

知识点：脑裂是主备节点都认为自己是 Master，同时持有 VIP，可能导致请求异常和数据风险。

应用场景：Keepalived 故障排查。

常见原因：

- 防火墙拦截 VRRP。
- 主备网络不通。
- 网卡或交换机异常。
- `virtual_router_id`、认证配置不一致。

排查命令：

```bash
ip addr | grep 10.0.0.3
systemctl status keepalived
tcpdump -i eth0 vrrp
firewall-cmd --state
```

## 18. Nginx 平滑升级与优化

### 18.1 平滑升级原理

知识点：Nginx 通过信号控制 master 和 worker，替换二进制后可启动新 master，再逐步退出旧 worker，实现不中断升级。

应用场景：生产环境升级 Nginx 版本、增加新编译模块。

常用信号：

| 信号 | 作用 |
| --- | --- |
| `HUP` | 重载配置 |
| `USR2` | 启动新二进制 master |
| `WINCH` | 优雅关闭旧 worker |
| `QUIT` | 优雅退出旧 master |
| `TERM/INT` | 快速退出 |

### 18.2 平滑升级步骤

代码示例：

```bash
# 查看旧版本编译参数
nginx -V

# 编译新版本，configure 参数尽量与旧版本一致
tar -xzf nginx-1.26.0.tar.gz
cd nginx-1.26.0
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-stream
make

# 备份旧二进制并替换
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
cp objs/nginx /usr/local/nginx/sbin/nginx

# 检查新二进制
/usr/local/nginx/sbin/nginx -t

# 启动新 master
kill -USR2 $(cat /usr/local/nginx/logs/nginx.pid)

# 关闭旧 worker
kill -WINCH $(cat /usr/local/nginx/logs/nginx.pid.oldbin)

# 确认无异常后退出旧 master
kill -QUIT $(cat /usr/local/nginx/logs/nginx.pid.oldbin)
```

### 18.3 平滑回滚

应用场景：升级后发现异常，快速切回旧版本。

代码示例：

```bash
cp /usr/local/nginx/sbin/nginx.old /usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -t
kill -USR2 $(cat /usr/local/nginx/logs/nginx.pid)
kill -WINCH $(cat /usr/local/nginx/logs/nginx.pid.oldbin)
kill -QUIT $(cat /usr/local/nginx/logs/nginx.pid.oldbin)
```

### 18.4 Nginx 性能优化

知识点：优化需要结合系统资源、连接数、文件描述符、缓存、压缩、日志和内核参数。

应用场景：高并发静态站点、反向代理入口、下载站。

代码示例：

```nginx
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    use epoll;
    worker_connections 65535;
    multi_accept on;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    server_tokens off;

    gzip on;
    gzip_comp_level 5;
    gzip_types text/plain text/css application/json application/javascript application/xml;

    open_file_cache max=10000 inactive=60s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 2;
}
```

系统参数示例：

```bash
cat >> /etc/security/limits.conf << EOF
nginx soft nofile 65535
nginx hard nofile 65535
EOF

cat >> /etc/sysctl.conf << EOF
net.core.somaxconn = 4096
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_tw_reuse = 1
EOF

sysctl -p
```

## 19. 综合实验

### 19.1 两台 Web + 一台 Nginx 负载均衡

应用场景：最基础的 Web 集群。

Web 节点：

```bash
yum install -y nginx
echo "web01 $(hostname)" > /usr/share/nginx/html/index.html
systemctl enable --now nginx
```

负载均衡节点：

```nginx
upstream web_cluster {
    server 10.0.0.7:80;
    server 10.0.0.8:80;
}

server {
    listen 80;
    server_name www.example.com;

    location / {
        proxy_pass http://web_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

验证：

```bash
nginx -t
systemctl reload nginx
for i in {1..6}; do curl http://10.0.0.5; done
```

### 19.2 Nginx + NFS 共享上传目录

应用场景：多台 Web 节点共享用户上传内容。

代码示例：

```bash
# NFS 服务端
mkdir -p /data/upload
chown -R www:www /data/upload
echo "/data/upload 10.0.0.0/24(rw,sync,all_squash,anonuid=666,anongid=666)" >> /etc/exports
exportfs -rv

# Web 节点
mkdir -p /usr/share/nginx/html/upload
mount -t nfs 10.0.0.31:/data/upload /usr/share/nginx/html/upload
```

### 19.3 Nginx HTTPS + 负载均衡

应用场景：公网 HTTPS 入口，后端内网 HTTP。

代码示例：

```nginx
upstream web_cluster {
    server 10.0.0.7:80;
    server 10.0.0.8:80;
}

server {
    listen 80;
    server_name www.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name www.example.com;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;

    location / {
        proxy_pass http://web_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

### 19.4 Keepalived + 双 Nginx 入口

应用场景：消除单台负载均衡故障点。

验证流程：

```bash
# proxy01 和 proxy02 都启动 nginx 与 keepalived
systemctl enable --now nginx keepalived

# 查看 VIP 在哪台机器
ip addr | grep 10.0.0.3

# 客户端访问 VIP
curl http://10.0.0.3

# 停止 Master 的 keepalived 或 nginx，观察 VIP 漂移
systemctl stop keepalived
```

### 19.5 常见故障排查清单

应用场景：Nginx 访问异常时按顺序定位。

代码示例：

```bash
# 1. 配置语法
nginx -t

# 2. 服务状态
systemctl status nginx

# 3. 端口监听
ss -lntup | grep nginx

# 4. 本机访问
curl -I http://127.0.0.1

# 5. 域名解析
dig www.example.com

# 6. 代理后端连通性
curl -I http://10.0.0.7

# 7. 日志
tail -n 100 /var/log/nginx/error.log
tail -n 100 /var/log/nginx/access.log
```

状态码排查：

| 状态码 | 常见原因 | 排查方向 |
| --- | --- | --- |
| `403` | 权限不足、缺少首页、访问被 deny | 文件权限、index、allow/deny |
| `404` | 文件不存在、root/alias 配错 | URI、root、alias |
| `500` | 后端应用异常 | 应用日志 |
| `502` | 后端不可用、端口不通、FastCGI 异常 | upstream、php-fpm、端口 |
| `504` | 后端响应超时 | proxy_read_timeout、后端性能 |

## 学习建议

1. 先用单台 Nginx 跑通静态站点、虚拟主机和日志。
2. 再加入两台 Web 节点，练习反向代理和七层负载均衡。
3. 接着加入 NFS、Rsync、Sersync，理解静态资源共享和备份。
4. 最后加入 HTTPS 和 Keepalived，形成接近生产的高可用 Web 架构。
5. 每次修改 Nginx 配置都先执行 `nginx -t`，通过后再 `systemctl reload nginx`。
