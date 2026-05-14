# JumpServer 学习笔记归纳版

> 本文由 `Jumpserver` 目录下 2 份 Markdown 文档归纳整理而成，按学习路径重新编排。每个知识点都包含：核心概念、代码示例、应用场景。

## 目录

- [1. JumpServer 与堡垒机基础](#1-jumpserver-与堡垒机基础)
- [2. JumpServer 架构组件](#2-jumpserver-架构组件)
- [3. 基础环境准备](#3-基础环境准备)
- [4. 数据库与缓存安装](#4-数据库与缓存安装)
- [5. Core 后端安装](#5-core-后端安装)
- [6. Lina 与 Luna 前端安装](#6-lina-与-luna-前端安装)
- [7. KoKo 与 Lion 连接组件安装](#7-koko-与-lion-连接组件安装)
- [8. Nginx 统一入口配置](#8-nginx-统一入口配置)
- [9. JumpServer 基础使用流程](#9-jumpserver-基础使用流程)
- [10. OpenVPN 基础](#10-openvpn-基础)
- [11. OpenVPN 证书体系](#11-openvpn-证书体系)
- [12. OpenVPN 服务端部署](#12-openvpn-服务端部署)
- [13. OpenVPN 客户端接入](#13-openvpn-客户端接入)
- [14. VPN 访问内网路由](#14-vpn-访问内网路由)
- [15. OpenVPN 用户密码认证](#15-openvpn-用户密码认证)
- [16. 综合实践：VPN + JumpServer 安全接入](#16-综合实践vpn--jumpserver-安全接入)
- [17. 学习建议与排错清单](#17-学习建议与排错清单)

## 1. JumpServer 与堡垒机基础

### 1.1 什么是 JumpServer

JumpServer 是开源堡垒机系统，用于统一管理用户、资产、账号、授权、登录审计和操作录像。

应用场景：

- 运维人员通过统一入口登录服务器。
- 控制不同用户能访问哪些资产。
- 记录 SSH、RDP、数据库等操作过程。
- 满足等保、审计、权限收敛要求。

### 1.2 为什么需要堡垒机

没有堡垒机时，常见问题是：

- 服务器账号分散，难以统一回收。
- 多人共用 root，无法定位责任人。
- 无法记录用户执行了什么命令。
- 外部人员接入内网缺少审计。

堡垒机解决思路：

```text
用户 -> JumpServer -> 授权资产 -> 系统账号 -> 审计日志/录像
```

应用场景：

- 生产服务器禁止个人直连，只允许通过堡垒机。
- 离职人员只需禁用 JumpServer 用户，无需逐台清理账号。

### 1.3 JumpServer 管理对象

| 对象 | 说明 |
| --- | --- |
| 用户 | 登录 JumpServer 的人员 |
| 用户组 | 批量管理用户权限 |
| 资产 | Linux、Windows、数据库、网络设备等 |
| 资产节点 | 资产分组目录 |
| 系统用户/账号 | 连接资产时使用的远程账号 |
| 授权规则 | 用户、资产、账号之间的访问关系 |
| 会话审计 | 在线会话、历史命令、录像 |

应用场景：

- 按部门或项目划分用户组。
- 按环境划分资产节点，例如 `prod`、`test`、`db`。

## 2. JumpServer 架构组件

### 2.1 组件说明

| 组件 | 作用 |
| --- | --- |
| Core | JumpServer 后端核心服务 |
| Lina | Web 管理前端 |
| Luna | Web Terminal 前端 |
| KoKo | SSH/SFTP 连接组件 |
| Lion | RDP 连接组件 |
| Guacd | Apache Guacamole 服务，支持远程桌面协议 |
| MariaDB/MySQL | 存储配置数据 |
| Redis | 缓存、任务队列、会话相关数据 |
| Nginx | 统一反向代理入口 |

访问链路：

```text
浏览器/SSH客户端
  -> Nginx
  -> Lina/Luna/Core/KoKo/Lion
  -> 目标资产
```

应用场景：

- Web 管理走 Lina/Core。
- Web SSH 走 Luna + KoKo。
- SSH 客户端登录堡垒机走 KoKo。
- Windows RDP 连接走 Lion + Guacd。

### 2.2 端口规划示例

| 服务 | 端口示例 |
| --- | --- |
| Nginx Web | 80/443 |
| Core | 8080 |
| KoKo SSH | 2222 |
| KoKo HTTP/WebSocket | 5000 |
| Lion | 8081 |
| Guacd | 4822 |
| MariaDB | 3306 |
| Redis | 6379 |

应用场景：

- 部署前先规划端口，避免和已有服务冲突。
- 防火墙只暴露 Nginx 和 KoKo SSH，对后端组件做内网访问限制。

## 3. 基础环境准备

### 3.1 系统依赖

示例命令：

```bash
yum install -y epel-release
yum install -y git wget curl gcc make unzip tar vim \
  python36 python36-devel python36-pip \
  openssl-devel mysql-devel openldap-devel
```

应用场景：

- 源码部署 JumpServer 需要编译依赖。
- Python、Node、Go 分别用于 Core、前端、KoKo 编译。

### 3.2 创建服务目录

```bash
mkdir -p /opt/jumpserver
mkdir -p /opt/jumpserver/{core,lina,luna,koko,lion}
```

应用场景：

- 统一保存 JumpServer 组件。
- 后续备份、升级时路径清晰。

### 3.3 基础安全建议

```bash
# 查看防火墙状态
firewall-cmd --state

# 只开放必要端口示例
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=2222/tcp
firewall-cmd --reload
```

应用场景：

- Web 管理使用 80/443。
- SSH 终端入口使用 KoKo 的 2222。
- 数据库、Redis、Core 不直接暴露公网。

## 4. 数据库与缓存安装

### 4.1 安装 MariaDB

```bash
yum install -y mariadb mariadb-server mariadb-devel
systemctl enable --now mariadb
```

初始化数据库：

```bash
mysql -uroot <<'SQL'
create database jumpserver default charset 'utf8';
create user 'jumpserver'@'127.0.0.1' identified by 'jumpserver_password';
grant all on jumpserver.* to 'jumpserver'@'127.0.0.1';
flush privileges;
SQL
```

应用场景：

- Core 使用数据库保存用户、资产、授权、审计等元数据。
- 生产环境建议数据库独立部署并定期备份。

### 4.2 安装 Redis

```bash
yum install -y redis
systemctl enable --now redis
```

配置密码示例：

```bash
sed -i 's/^# requirepass .*/requirepass redis_password/' /etc/redis.conf
systemctl restart redis
```

测试：

```bash
redis-cli -a redis_password ping
```

应用场景：

- Core 任务队列。
- 会话、缓存、异步任务。

## 5. Core 后端安装

### 5.1 下载 Core

```bash
cd /opt/jumpserver
git clone https://github.com/jumpserver/jumpserver.git core
cd core
```

应用场景：

- 源码部署需要直接拉取 Core 项目。
- 如果生产部署，建议固定版本 tag，而不是直接使用最新分支。

### 5.2 安装 Python 依赖

```bash
cd /opt/jumpserver/core
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip setuptools
pip install -r requirements/requirements.txt
```

应用场景：

- 使用虚拟环境隔离 Python 依赖。
- 便于升级和回滚。

### 5.3 配置 Core

```bash
cd /opt/jumpserver/core
cp config_example.yml config.yml
```

核心配置示例：

```yaml
SECRET_KEY: "change_me_to_random_string"
BOOTSTRAP_TOKEN: "change_me_bootstrap_token"

DB_ENGINE: mysql
DB_HOST: 127.0.0.1
DB_PORT: 3306
DB_USER: jumpserver
DB_PASSWORD: jumpserver_password
DB_NAME: jumpserver

REDIS_HOST: 127.0.0.1
REDIS_PORT: 6379
REDIS_PASSWORD: redis_password
```

生成随机密钥：

```bash
openssl rand -hex 32
```

应用场景：

- `SECRET_KEY` 用于安全加密，生产环境必须修改。
- `BOOTSTRAP_TOKEN` 用于组件注册，Core、KoKo、Lion 需保持一致。

### 5.4 启动 Core

前台测试：

```bash
cd /opt/jumpserver/core
source venv/bin/activate
./jms start
```

后台运行：

```bash
./jms start -d
```

查看状态：

```bash
./jms status
tail -f logs/jumpserver.log
```

应用场景：

- 首次启动先前台观察报错。
- 确认数据库、Redis、依赖都正常后再后台运行。

## 6. Lina 与 Luna 前端安装

### 6.1 安装 Node.js

```bash
curl -sL https://rpm.nodesource.com/setup_14.x | bash -
yum install -y nodejs
node -v
npm -v
```

应用场景：

- Lina、Luna 是前端项目，需要 Node 构建。
- 生产部署建议使用项目要求的 Node 版本。

### 6.2 安装 Lina

```bash
cd /opt/jumpserver
git clone https://github.com/jumpserver/lina.git
cd lina
npm install
```

配置示例：

```bash
cp .env.development.example .env.development
```

`.env.development` 示例：

```ini
VUE_APP_CORE_HOST=http://127.0.0.1:8080
VUE_APP_ENV=development
```

开发运行：

```bash
npm run serve
```

构建：

```bash
npm run build:prod
```

应用场景：

- Lina 提供 JumpServer 管理后台页面。
- 生产环境一般构建成静态文件后交给 Nginx。

### 6.3 安装 Luna

```bash
cd /opt/jumpserver
git clone https://github.com/jumpserver/luna.git
cd luna
npm install
```

配置示例：

```bash
cp .env.development.example .env.development
```

构建：

```bash
npm run build:prod
```

应用场景：

- Luna 提供 Web Terminal 页面。
- 用户通过浏览器连接 Linux 资产时会使用 Luna。

## 7. KoKo 与 Lion 连接组件安装

### 7.1 安装 Go

```bash
wget https://go.dev/dl/go1.20.13.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.20.13.linux-amd64.tar.gz
echo 'export PATH=/usr/local/go/bin:$PATH' >/etc/profile.d/go.sh
source /etc/profile.d/go.sh
go version
```

应用场景：

- KoKo 使用 Go 编写，源码部署需要 Go 环境。

### 7.2 安装 KoKo

```bash
cd /opt/jumpserver
git clone https://github.com/jumpserver/koko.git
cd koko
make
```

配置：

```bash
cp config_example.yml config.yml
```

配置示例：

```yaml
CORE_HOST: http://127.0.0.1:8080
BOOTSTRAP_TOKEN: "change_me_bootstrap_token"
SSHD_PORT: 2222
HTTPD_PORT: 5000
```

启动：

```bash
./koko
```

应用场景：

- SSH 客户端登录 JumpServer。
- Web Terminal 后端连接 Linux 资产。
- SFTP 文件管理。

### 7.3 安装 Guacd

```bash
yum install -y cairo-devel libjpeg-turbo-devel libpng-devel uuid-devel freerdp-devel pango-devel libssh2-devel
```

编译安装示例：

```bash
wget https://downloads.apache.org/guacamole/1.5.5/source/guacamole-server-1.5.5.tar.gz
tar xf guacamole-server-1.5.5.tar.gz
cd guacamole-server-1.5.5
./configure --with-init-dir=/etc/init.d
make
make install
ldconfig
systemctl enable --now guacd
```

应用场景：

- Lion 通过 Guacd 支持 RDP 远程桌面协议。

### 7.4 安装 Lion

```bash
cd /opt/jumpserver
git clone https://github.com/jumpserver/lion-release.git lion
cd lion
```

配置：

```yaml
CORE_HOST: http://127.0.0.1:8080
BOOTSTRAP_TOKEN: "change_me_bootstrap_token"
GUA_HOST: 127.0.0.1
GUA_PORT: 4822
```

启动：

```bash
./lion
```

应用场景：

- 通过浏览器连接 Windows RDP 资产。
- 审计 Windows 远程桌面会话。

## 8. Nginx 统一入口配置

### 8.1 安装 Nginx

```bash
yum install -y nginx
systemctl enable --now nginx
```

### 8.2 反向代理配置示例

`/etc/nginx/conf.d/jumpserver.conf`：

```nginx
server {
    listen 80;
    server_name jump.example.com;

    client_max_body_size 4096m;

    location /ui/ {
        alias /opt/jumpserver/lina/dist/;
        try_files $uri /ui/index.html;
    }

    location /luna/ {
        alias /opt/jumpserver/luna/dist/;
        try_files $uri /luna/index.html;
    }

    location /koko/ {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

检查并重载：

```bash
nginx -t
systemctl reload nginx
```

应用场景：

- 对外只暴露一个 Web 入口。
- 将前端静态资源、Core API、WebSocket 连接统一代理。

### 8.3 HTTPS 配置建议

```nginx
server {
    listen 443 ssl;
    server_name jump.example.com;

    ssl_certificate /etc/nginx/ssl/jump.example.com.crt;
    ssl_certificate_key /etc/nginx/ssl/jump.example.com.key;
}
```

应用场景：

- 生产环境必须使用 HTTPS。
- 防止登录凭据和会话信息明文传输。

## 9. JumpServer 基础使用流程

### 9.1 创建用户与用户组

Web 操作路径：

```text
用户管理 -> 用户列表 -> 创建用户
用户管理 -> 用户组 -> 创建用户组
```

应用场景：

- 按团队划分用户，例如 `ops`、`dev`、`dba`。
- 不同用户组授权不同资产范围。

### 9.2 添加资产

资产字段示例：

```text
名称: web01
地址: 10.0.0.11
平台: Linux
协议: SSH 22
节点: /Default/prod/web
```

命令验证：

```bash
ssh root@10.0.0.11
```

应用场景：

- 将 Linux、Windows、数据库资产纳入统一管理。
- 按环境、业务、机房建立资产节点。

### 9.3 创建账号与授权

授权关系：

```text
用户/用户组 + 资产/资产节点 + 账号 = 资产授权
```

应用场景：

- 运维组可以用 `root` 管理生产 Linux。
- 开发组只能使用普通账号访问测试环境。
- DBA 只授权数据库资产。

### 9.4 登录与审计

SSH 客户端登录 KoKo：

```bash
ssh -p 2222 username@jump.example.com
```

Web 登录：

```text
http://jump.example.com
```

审计路径：

```text
会话管理 -> 在线会话
审计台 -> 命令记录
审计台 -> 会话录像
```

应用场景：

- 实时中断异常会话。
- 回放用户操作过程。
- 查询某个用户在某台机器执行过的命令。

## 10. OpenVPN 基础

### 10.1 什么是 VPN

VPN 是通过公网建立加密隧道，让远程用户或远程站点安全访问内网资源的技术。

常见场景：

- 点对点连接。
- 站点到站点互联。
- 远程办公访问内网。

应用场景：

- 运维人员先连接 VPN，再访问 JumpServer。
- 分支机构访问总部内网服务。

### 10.2 什么是 OpenVPN

OpenVPN 是基于 SSL/TLS 的开源 VPN 方案，支持证书认证、用户名密码认证、路由推送和跨平台客户端。

应用场景：

- 小中型企业远程接入。
- 云上 VPC 和本地环境临时互通。
- 作为堡垒机前置网络访问控制。

### 10.3 VPN 与 JumpServer 的关系

推荐访问链路：

```text
用户电脑 -> OpenVPN -> 内网 -> JumpServer -> 目标资产
```

应用场景：

- JumpServer 不直接暴露公网。
- 外部用户必须先通过 VPN 进入受控网络。

## 11. OpenVPN 证书体系

### 11.1 安装 easy-rsa

```bash
yum install -y epel-release
yum install -y easy-rsa openvpn
```

准备 PKI 目录：

```bash
mkdir -p /etc/openvpn/easy-rsa
cp -r /usr/share/easy-rsa/3/* /etc/openvpn/easy-rsa/
cd /etc/openvpn/easy-rsa
```

应用场景：

- easy-rsa 用于创建 CA、服务端证书、客户端证书和 DH 参数。

### 11.2 初始化 PKI 与 CA

```bash
./easyrsa init-pki
./easyrsa build-ca nopass
```

应用场景：

- CA 用于签发服务端和客户端证书。
- `nopass` 适合实验环境；生产环境建议保护 CA 私钥。

### 11.3 签发服务端证书

```bash
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-dh
openvpn --genkey --secret ta.key
```

应用场景：

- 服务端证书用于 OpenVPN Server 身份认证。
- `ta.key` 用于 TLS Auth，减少恶意连接探测。

### 11.4 签发客户端证书

```bash
./easyrsa gen-req client01 nopass
./easyrsa sign-req client client01
```

客户端所需文件：

```text
ca.crt
client01.crt
client01.key
ta.key
client.ovpn
```

应用场景：

- 每个用户独立客户端证书，便于吊销和审计。
- 不建议多人共用同一证书。

## 12. OpenVPN 服务端部署

### 12.1 安装服务

```bash
yum install -y openvpn easy-rsa
```

### 12.2 拷贝证书

```bash
cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/
cp /etc/openvpn/easy-rsa/pki/issued/server.crt /etc/openvpn/
cp /etc/openvpn/easy-rsa/pki/private/server.key /etc/openvpn/
cp /etc/openvpn/easy-rsa/pki/dh.pem /etc/openvpn/
cp /etc/openvpn/easy-rsa/ta.key /etc/openvpn/
```

### 12.3 服务端配置

`/etc/openvpn/server.conf`：

```ini
port 1194
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0

server 10.8.0.0 255.255.255.0
push "route 172.16.1.0 255.255.255.0"

keepalive 10 120
persist-key
persist-tun

cipher AES-256-CBC
status /var/log/openvpn-status.log
log-append /var/log/openvpn.log
verb 3
```

应用场景：

- `10.8.0.0/24` 是 VPN 客户端地址池。
- `push route` 告诉客户端如何访问内网网段。

### 12.4 开启内核转发

```bash
echo 'net.ipv4.ip_forward = 1' >/etc/sysctl.d/openvpn.conf
sysctl -p /etc/sysctl.d/openvpn.conf
```

应用场景：

- VPN Server 需要转发客户端到内网的流量。

### 12.5 启动 OpenVPN

```bash
systemctl enable --now openvpn@server
systemctl status openvpn@server
ss -lntup | grep 1194
```

查看日志：

```bash
tail -f /var/log/openvpn.log
```

应用场景：

- 首次连接失败时优先看服务端日志。

## 13. OpenVPN 客户端接入

### 13.1 Linux 客户端

安装：

```bash
yum install -y openvpn
```

客户端配置 `client.ovpn`：

```ini
client
dev tun
proto udp
remote vpn.example.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun

ca ca.crt
cert client01.crt
key client01.key
tls-auth ta.key 1

cipher AES-256-CBC
verb 3
```

连接：

```bash
openvpn --config client.ovpn
```

后台连接：

```bash
openvpn --daemon --config client.ovpn --log-append /var/log/openvpn-client.log
```

应用场景：

- Linux 运维机接入企业内网。
- 自动化任务通过 VPN 访问内网堡垒机。

### 13.2 Windows 客户端

步骤：

1. 安装 OpenVPN GUI。
2. 将 `client.ovpn`、证书和密钥放入配置目录。
3. 右键 OpenVPN GUI，以管理员身份运行。
4. 选择配置并连接。

验证：

```powershell
ipconfig
ping 10.8.0.1
ping 172.16.1.10
```

应用场景：

- Windows 用户远程办公接入内网。
- 运维人员通过 VPN 访问 JumpServer Web 页面。

### 13.3 macOS 客户端

可使用 Tunnelblick 或 OpenVPN Connect。

验证：

```bash
ifconfig | grep -A3 tun
netstat -rn | grep 172.16
```

应用场景：

- macOS 用户接入企业内网。

## 14. VPN 访问内网路由

### 14.1 常见问题

VPN 客户端能连上，但访问不了内网机器，通常原因是：

- OpenVPN Server 没开启 IP 转发。
- 内网主机没有到 VPN 网段的回程路由。
- 防火墙或安全组未放行。
- NAT 未配置。

应用场景：

- 排查“VPN 已连接但无法访问 JumpServer/内网资产”。

### 14.2 添加内网主机路由

在内网主机或网关上添加回程路由：

```bash
ip route add 10.8.0.0/24 via 172.16.1.10
```

持久化示例：

```bash
echo '10.8.0.0/24 via 172.16.1.10' >> /etc/sysconfig/network-scripts/route-eth0
systemctl restart network
```

应用场景：

- 内网主机需要知道 VPN 客户端网段从哪里返回。
- 适合有内网路由器或可控网关的环境。

### 14.3 使用 NAT 转换

Firewalld：

```bash
firewall-cmd --permanent --add-masquerade
firewall-cmd --permanent --add-port=1194/udp
firewall-cmd --reload
```

iptables：

```bash
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE
```

应用场景：

- 虚拟机实验环境。
- 无法修改内网网关路由时，让内网主机看到的来源变成 VPN Server。

### 14.4 路由排错命令

```bash
# 客户端查看地址和路由
ip addr show tun0
ip route

# 服务端查看转发和 NAT
sysctl net.ipv4.ip_forward
iptables -t nat -L -n -v

# 抓包定位
tcpdump -i tun0 icmp
tcpdump -i eth0 host 172.16.1.10
```

应用场景：

- 判断流量是否进入 VPN 隧道。
- 判断回包是否回到 VPN Server。

## 15. OpenVPN 用户密码认证

### 15.1 为什么需要用户名密码

证书认证适合设备身份识别，用户名密码适合人员身份识别。两者结合可以提高安全性。

应用场景：

- 每个人使用自己的 VPN 账号。
- 离职时禁用账号，不一定要重新分发所有证书。

### 15.2 服务端配置

在 `server.conf` 添加：

```ini
script-security 3
auth-user-pass-verify /etc/openvpn/check_user.sh via-env
username-as-common-name
```

如果只使用用户名密码而不要求客户端证书，部分版本可配置：

```ini
verify-client-cert none
```

应用场景：

- 证书 + 密码双因素。
- 或在实验环境简化为账号密码认证。

### 15.3 用户校验脚本

`/etc/openvpn/check_user.sh`：

```bash
#!/bin/bash

user_file="/etc/openvpn/openvpn_passwd"

if grep -q "^${username}:${password}$" "$user_file"; then
  exit 0
else
  exit 1
fi
```

权限：

```bash
chmod 700 /etc/openvpn/check_user.sh
chown root:root /etc/openvpn/check_user.sh
```

用户文件：

```bash
cat >/etc/openvpn/openvpn_passwd <<'EOF'
alice:alice_password
bob:bob_password
EOF
chmod 600 /etc/openvpn/openvpn_passwd
```

重启：

```bash
systemctl restart openvpn@server
```

应用场景：

- 小规模用户管理。
- 实验环境理解认证流程。

注意：生产环境不建议明文保存密码，应接入 PAM、LDAP 或更安全的认证系统。

### 15.4 客户端配置

`client.ovpn` 添加：

```ini
auth-user-pass
```

也可以指定密码文件：

```ini
auth-user-pass pass.txt
```

`pass.txt`：

```text
alice
alice_password
```

应用场景：

- 手动输入账号密码。
- 自动化客户端连接时使用密码文件。

## 16. 综合实践：VPN + JumpServer 安全接入

### 16.1 推荐网络架构

```text
公网用户
  -> OpenVPN Server:1194/udp
  -> 内网 JumpServer:80/443,2222
  -> 目标资产:22/3389/3306
```

安全原则：

- JumpServer Web 不直接暴露公网。
- 目标资产不暴露公网。
- 用户必须先通过 VPN，再通过堡垒机访问资产。
- 所有资产操作由 JumpServer 审计。

应用场景：

- 远程办公访问生产环境。
- 第三方运维人员临时接入。

### 16.2 防火墙放行示例

OpenVPN Server：

```bash
firewall-cmd --permanent --add-port=1194/udp
firewall-cmd --permanent --add-masquerade
firewall-cmd --reload
```

JumpServer：

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=2222/tcp
firewall-cmd --reload
```

目标资产：

```bash
# 只允许 JumpServer 访问 SSH
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="172.16.1.20" port protocol="tcp" port="22" accept'
firewall-cmd --reload
```

应用场景：

- 收敛资产访问入口。
- 避免用户绕过堡垒机直连服务器。

### 16.3 最小权限授权示例

```text
用户组: dev
资产节点: /test/web
系统账号: appuser
权限: SSH 登录、SFTP 禁用

用户组: ops
资产节点: /prod
系统账号: root
权限: SSH 登录、SFTP 允许
```

应用场景：

- 开发只能访问测试环境。
- 运维按职责访问生产环境。
- 权限变更集中在 JumpServer 授权规则中完成。

## 17. 学习建议与排错清单

### 17.1 推荐学习顺序

1. 先理解堡垒机的用户、资产、账号、授权、审计模型。
2. 再掌握 JumpServer 各组件职责：Core、Lina、Luna、KoKo、Lion。
3. 完成数据库、Redis、Core、前端、连接组件、Nginx 的部署链路。
4. 学会创建用户、资产、账号和授权规则。
5. 学会使用 SSH 客户端和 Web Terminal 登录资产。
6. 再学习 OpenVPN，理解证书、路由、NAT、用户认证。
7. 最后组合 VPN + JumpServer，形成安全接入闭环。

### 17.2 JumpServer 常用排错命令

```bash
# Core
cd /opt/jumpserver/core
./jms status
tail -f logs/jumpserver.log

# Nginx
nginx -t
tail -f /var/log/nginx/error.log

# 数据库
mysql -ujumpserver -p -h 127.0.0.1 jumpserver

# Redis
redis-cli -a redis_password ping

# 端口
ss -lntup | grep -E '80|443|8080|2222|5000|4822'
```

### 17.3 OpenVPN 常用排错命令

```bash
systemctl status openvpn@server
tail -f /var/log/openvpn.log
ss -lunp | grep 1194

ip addr show tun0
ip route
sysctl net.ipv4.ip_forward
iptables -t nat -L -n -v
```

### 17.4 常见问题

| 问题 | 常见原因 | 处理方式 |
| --- | --- | --- |
| JumpServer Web 打不开 | Nginx 配置错误、Core 未启动 | 检查 `nginx -t`、Core 日志 |
| 登录资产失败 | 账号密码错误、资产网络不可达、授权缺失 | 测试 SSH，检查授权规则 |
| KoKo SSH 不通 | 2222 未监听或防火墙未放行 | 检查 KoKo 配置和端口 |
| Web Terminal 黑屏 | WebSocket 代理错误 | 检查 Nginx `Upgrade` 相关配置 |
| RDP 连接失败 | Guacd/Lion 未启动或目标 3389 不通 | 检查 guacd、Lion 和网络 |
| VPN 能连接但访问不了内网 | 缺路由、未开转发、无 NAT | 检查 `ip_forward`、路由、iptables |
| OpenVPN 认证失败 | 证书不匹配、账号密码错误 | 看 OpenVPN 服务端日志 |

### 17.5 生产建议

- JumpServer 和 OpenVPN 都使用 HTTPS/加密访问。
- JumpServer 不直接暴露公网，放在 VPN 后面更稳妥。
- 目标资产只允许 JumpServer 访问管理端口。
- 用户按组授权，避免给个人零散授权。
- 禁止多人共用同一个堡垒机账号。
- 定期备份 JumpServer 数据库和配置文件。
- OpenVPN 每人独立证书和账号，离职及时吊销。
- 自建部署前确认版本兼容性，生产环境优先使用官方推荐安装方式。

