# 安装Jumpserver

## 目录

  - [2.1 基础环境准备](#2.1-基础环境准备)
  - [2.2 安装Mariadb](#2.2-安装mariadb)
- [1.准备mariadb yum源地址 mariadb官方网站](#1.准备mariadb-yum源地址-mariadb官方网站)
  - [2.3 安装Redis](#2.3-安装redis)
  - [2.4 安装Jumpserver](#2.4-安装jumpserver)
    - [2.4.1 安装依赖及Python](#2.4.1-安装依赖及python)
    - [2.4.1 下载Core源码包](#2.4.1-下载core源码包)
    - [2.4.2 安装Python依赖](#2.4.2-安装python依赖)
    - [2.4.3 修改配置文件](#2.4.3-修改配置文件)
    - [2.4.4 启动Core程序](#2.4.4-启动core程序)
- [1.前台运行测试](#1.前台运行测试)
- [2.测试没有问题后可以通过-d选项放入后台](#2.测试没有问题后可以通过-d选项放入后台)
  - [2.5 安装Lina](#2.5-安装lina)
    - [2.5.1 下载源码](#2.5.1-下载源码)
    - [2.5.1 安装Node](#2.5.1-安装node)
    - [2.5.2 安装依赖](#2.5.2-安装依赖)
    - [2.5.3 修改配置](#2.5.3-修改配置)
    - [2.5.4 运行Lina](#2.5.4-运行lina)
    - [2.5.5 构建Lina](#2.5.5-构建lina)
  - [2.6 安装Luna](#2.6-安装luna)
    - [2.6.1 下载源码](#2.6.1-下载源码)
    - [2.6.2 安装依赖](#2.6.2-安装依赖)
    - [2.6.3 修改配置文件](#2.6.3-修改配置文件)
    - [2.6.5 构建luna](#2.6.5-构建luna)
  - [2.7 安装KoKo](#2.7-安装koko)
    - [2.7.1 安装Go](#2.7.1-安装go)
    - [2.7.2 下载源码](#2.7.2-下载源码)
    - [2.7.3 编译程序](#2.7.3-编译程序)
    - [2.7.4 修改配置](#2.7.4-修改配置)
    - [2.7.5 启动KoKo](#2.7.5-启动koko)
  - [2.8 安装Lion](#2.8-安装lion)
    - [2.8.1 构建 Guacd](#2.8.1-构建-guacd)
    - [2.8.2 启动 Guacd](#2.8.2-启动-guacd)
    - [2.8.3 下载Lion](#2.8.3-下载lion)
    - [2.8.4 修改配置](#2.8.4-修改配置)
    - [2.8.5 启动Lion](#2.8.5-启动lion)
  - [2.9 安装Nginx](#2.9-安装nginx)
    - [2.9.1 安装nginx](#2.9.1-安装nginx)
    - [2.9.2 整合jumpserver](#2.9.2-整合jumpserver)
    - [2.9.3 访问Jumpserver](#2.9.3-访问jumpserver)

### 2.1 基础环境准备

### 2.2 安装Mariadb

## 1.准备mariadb yum源地址 mariadb官方网站

```
国外官方源（下载非常慢） [root@jumpserver ~]# cat ՎҴ /etc/yum.repos.d/mariadb.repo ՎՎӓEOF [mariadb] name = MariaDB baseurl = http:Վˌyum.mariadb.org/10.6/centos7- amd64 gpgkey=https:Վˌyum.mariadb.org/RPM-GPG-KEY- MariaDB gpgcheck=1 EOF # 国内加速器 [root@jumpserver ~]# cat > /etc/yum.repos.d/mariadb.repo ՎՎӓEOF

[mariadb] name = MariaDB baseurl = https:Վˌmirrors.ustc.edu.cn/mariadb/yum/10.2/ce ntos7-amd64 gpgkey=https:Վˌmirrors.ustc.edu.cn/mariadb/yum/ RPM-GPG-KEY-MariaDB gpgcheck=1 EOF 2.安装maraidb，并检查版本是否ՎҲ10.2 # yum install MariaDB-server MariaDB-client MariaDB-devel MariaDB-shared -y # mysql Վʔversion 3.启动mariadb，然后配置本地root登录密码 # systemctl enable mariadb # systemctl start mariadb # mysqladmin password oldxu.net123

MariaDB [(none)]> create database jumpserver default charset 'utf8'; MariaDB [(none)]> grant all privileges on jumpserver.* to jumpserver@'localhost' identified by 'oldxu.net123';
```
### 2.3 安装Redis

```
1.安装yum源，获取最新的redis版本 # yum install -y http:Վˌrpms.famillecollet.com/enterprise/remi- release-7.rpm # yum Վʔenablerepo=remi install redis -y # redis-server Վʔversion
```
2.配置并启动redis

```
[root@jumpserver ~]# systemctl start redis [root@jumpserver ~]# systemctl enable redis
```
### 2.4 安装Jumpserver

#### 2.4.1 安装依赖及Python

1.安装基础依赖环境

```
yum install gcc gcc-cՎҡ make epel-release openldap-devel MariaDB-devel MariaDB-shared libffi-devel sshpass -y
```
2.安装Python3环境

```
yum install python36 python36-devel -y
```
#### 2.4.1 下载Core源码包

```
cd /opt # wget -O /opt/jumpserver-v2.13.2.tar.gz https:Վˌgithub.com/jumpserver/jumpserver/archiv e/refs/tags/v2.13.2.tar.gz # tar xf jumpserver-v2.13.2.tar.gz # cd jumpserver-2.13.2/
```
#### 2.4.2 安装Python依赖

```
1.为 JumpServer 项目单独创建 python3 虚拟环境，每次运 行项目都需要先执行 source /opt/py3/bin/activate 载入 此环境。 [root@jumpserver jumpserver-2.13.2]# python3 -m venv /opt/py3 [root@jumpserver jumpserver-2.13.2]# source /opt/py3/bin/activate (py3) #
```
2.安装jumpserver相关依赖软件

```
(py3) # pip install -r requirements/requirements.txt -i https:Վˌmirrors.aliyun.com/pypi/simple/
```
#### 2.4.3 修改配置文件

1.创建相关秘钥

加密秘钥 生产环境中请修改为随机字符串，请勿外泄, 可使 用命令生成 # cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 49;echo # 预共享Token koko 和 lion 用来注册服务账号，不在使 用原来的注册接受机制 # cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 24;echo

```
(py3) # cp config_example.yml config.yml # 修改后的配置文件 (py3) # grep "^[a-Z]" config.yml SECRET_KEY: BpjpU68Z08SmUgr9HMHbdcGgTkSkDKmz8o6aLt5mJo2kgkN oH   # 必须有 BOOTSTRAP_TOKEN: 1gaKWv09fjmip6IISbHX0Ggl # 必须有 SESSION_EXPIRE_AT_BROWSER_CLOSE: true DB_ENGINE: mysql DB_HOST: 127.0.0.1 DB_PORT: 3306 DB_USER: jumpserver DB_PASSWORD: oldxu.net123 DB_NAME: jumpserver HTTP_BIND_HOST: 0.0.0.0 HTTP_LISTEN_PORT: 8080 WS_LISTEN_PORT: 8070 REDIS_HOST: 127.0.0.1

REDIS_PORT: 6379
```
#### 2.4.4 启动Core程序

## 1.前台运行测试

```
./jms start
```
## 2.测试没有问题后可以通过-d选项放入后台

```
./jms start -d
```
### 2.5 安装Lina

#### 2.5.1 下载源码

```
cd /opt # wget -O /opt/luna-v2.13.2.tar.gz https:Վˌgithub.com/jumpserver/luna/archive/refs /tags/v2.13.2.tar.gz # tar -xf lina-v2.13.2.tar.gz # cd lina-2.13.2
```
#### 2.5.1 安装Node

1.安装node 10版本

```
执行命令后会有提示，否则没安装成功 # curl Վʔsilent Վʔlocation https:Վˌrpm.nodesource.com/setup_10.x | sudo bash # curl -sL https:Վˌrpm.nodesource.com/setup_12.x | bash - # yum makecache # yum install nodejs -y 2.配置node下载站点为国内，加速下载 # npm config set sass_binary_site https:Վˌnpm.taobao.org/mirrors/node-sass # npm config set registry https:Վˌregistry.npm.taobao.org
```
#### 2.5.2 安装依赖

```
npm install -g yarn # yarn config set registry https:Վˌregistry.npm.taobao.org # yarn install
```
#### 2.5.3 修改配置

```
vim .env.development # 全局环境变量 请勿随意改动 ENV = 'development' # base api

VUE_APP_BASE_API = '' VUE_APP_PUBLIC_PATH = '/ui/' VUE_CLI_BABEL_TRANSPILE_MODULES = true # External auth VUE_APP_LOGIN_PATH = '/core/auth/login/' VUE_APP_LOGOUT_PATH = '/core/auth/logout/' # Dev server
for core proxy VUE_APP_CORE_HOST = 'http:Վˌlocalhost:8080' VUE_APP_CORE_WS = 'ws:Վˌlocalhost:8070' VUE_APP_ENV = 'development'
```
#### 2.5.4 运行Lina

yarn serve # 确保能正常访问

#### 2.5.5 构建Lina

yarn build:prod # 拷贝静态资源至该目录，后续可以使用nginx调用 # cp -rp lina/ /opt/

### 2.6 安装Luna

#### 2.6.1 下载源码

```
cd /opt # wget -O /opt/luna-v2.13.2.tar.gz https:Վˌgithub.com/jumpserver/luna/archive/refs /tags/v2.13.2.tar.gz # tar -xf luna-v2.13.2.tar.gz # cd luna-v2.13.2
```
#### 2.6.2 安装依赖

```
npm config set registry https:Վˌmirrors.huaweicloud.com/repository/npm/ # npm install # npm rebuild node-sass
```
#### 2.6.3 修改配置文件

```
vi proxy.conf.json { "/koko": { "target": "http:Վˌlocalhost:5000", # KoKo 地址 "secure": false, "ws": true }, "/media/": { "target": "http:Վˌlocalhost:8080", # Core 地址 "secure": false, "changeOrigin": true

}, "/api/": { "target": "http:Վˌlocalhost:8080",  # Core 地址 "secure": false,                    # https ssl 需要开启 "changeOrigin": true }, "/core": { "target": "http:Վˌlocalhost:8080",  # Core 地址 "secure": false, "changeOrigin": true }, "/static": { "target": "http:Վˌlocalhost:8080",  # Core 地址 "secure": false, "changeOrigin": true }, "/lion": { "target": "http:Վˌlocalhost:9529",  # Lion 地址 "secure": false, "pathRewrite": { "^/lion/monitor": "/monitor" }, "ws": true, "changeOrigin": true }, "/omnidb": { "target": "http:Վˌlocalhost:8082",

"secure": false, "ws": true, "changeOrigin": true } }
```
#### 2.6.5 构建luna

```
npm run-script build # cp -rp luna/ /opt/
```
### 2.7 安装KoKo

Koko 是 Go 版本的 coco，重构了 coco 的 SSH/SFTP 服务和 Web Terminal 服务。

#### 2.7.1 安装Go

```
yum install go # go version go version go 2.15.14 linux/amd64
```
#### 2.7.2 下载源码

```
cd /opt # wget -O /opt/koko-v2.13.2.tar.gz https:Վˌgithub.com/jumpserver/koko/archive/refs /tags/v2.13.2.tar.gz # tar -xf koko-v2.13.2.tar.gz # cd koko-2.13.2
```
#### 2.7.3 编译程序

```
go env -w GOPROXY=https:Վˌmirrors.aliyun.com/goproxy/ # make # 编译完成后在build目录 # cd build/ # tar xf kokoՎՎʕlinux-amd64.tar.gz # cd kokoՎՎʕlinux-amd64
```
#### 2.7.4 修改配置

```
cp config_example.yml config.yml # vim config.yml # 项目名称, 会用来向Jumpserver注册, 识别而已, 不能重 复 # NAME: {{ Hostname }} # Jumpserver项目的url, api请求注册会使用 CORE_HOST: http:Վˌ127.0.0.1:8080
```
Bootstrap Token, 预共享秘钥, 用来注册coco使用的 service account和terminal # 请和jumpserver 配置文件中保持一致，注册完成后可以删 除 BOOTSTRAP_TOKEN: 1gaKWv09fjmip6IISbHX0Ggl # 启动时绑定的ip, 默认 0.0.0.0 BIND_HOST: 0.0.0.0 # 监听的SSH端口号, 默认2222 SSHD_PORT: 2222 # 监听的HTTP/WS端口号，默认5000 HTTPD_PORT: 5000 # 设置日志级别 [DEBUG, INFO, WARN, ERROR, FATAL, CRITICAL] LOG_LEVEL: INFO # SSH连接超时时间 (default 15 seconds) SSH_TIMEOUT: 15 # 语言 [en,zh] LANGUAGE_CODE: zh # SFTP是否显示隐藏文件 # SFTP_SHOW_HIDDEN_FILE: false # 是否复用和用户后端资产已建立的连接(用户不会复用其他用 户的连接) # REUSE_CONNECTION: true

资产加载策略, 可根据资产规模自行调整. 默认异步加载资 产, 异步搜索分页; 如果为all, 则资产全部加载, 本地搜索 分页. # ASSET_LOAD_POLICY: # zip压缩的最大额度 (单位: M) # ZIP_MAX_SIZE: 1024M # zip压缩存放的临时目录 /tmp # ZIP_TMP_PATH: /tmp # 向 SSH Client 连接发送心跳的时间间隔 (单位: 秒)， 默认为30, 0则表示不发送 # CLIENT_ALIVE_INTERVAL: 30 # 向资产发送心跳包的重试次数，默认为3 # RETRY_ALIVE_COUNT_MAX: 3 # 会话共享使用的类型 [local, redis], 默认local # SHARE_ROOM_TYPE: local # Redis配置 # REDIS_HOST: 127.0.0.1 # REDIS_PORT: 6379 # REDIS_PASSWORD: # REDIS_CLUSTERS: # REDIS_DB_ROOM: # 是否开启本地转发 (目前仅对 vscode remote ssh 有效 果) # ENABLE_LOCAL_PORT_FORWARD: false

是否开启 针对 vscode 的 remote-ssh 远程开发支持 (前置条件: 必须开启 ENABLE_LOCAL_PORT_FORWARD ) # ENABLE_VSCODE_SUPPORT: false

#### 2.7.5 启动KoKo

```
./koko -d # netstat -lntp |grep koko tcp6 0 0 ՎՎʧ5000 ՎՎʧ* LISTEN 21470/./koko tcp6 0 0 ՎՎʧ2222 ՎՎʧ* LISTEN 21470/./koko
```
### 2.8 安装Lion

Lion 使用了 Apache 软件基金会的开源项目 Guacamole， JumpServer 使用 Golang 和 Vue 重构了 Guacamole 实现 RDP/VNC 协议跳板机功能。

#### 2.8.1 构建 Guacd

```
yum install libpng-devel libjpeg-devel cairo- devel uuid-devel -y # 安装插件，否则会提供不支持rdp等协议：guacd: Support
for protocol "rdp" is not installed # yum install libtelnet-devel libvncserver- devel pulseaudio-libs-devel openssl-devel freerdp-devel pango-devel libssh2-devel libtelnet-devel

下载 # cd /opt # wget http:Վˌdownload.jumpserver.org/public/guacamole -server-2.3.0.tar.gz # cd guacamole-server-2.3.0/ # 构建 # ./configure Վʔwith-init-dir=/etc/init.d # make ՎҐ make install # ldconfig
```
#### 2.8.2 启动 Guacd

```
/etc/init.d/guacd start
```
#### 2.8.3 下载Lion

```
cd /opt # wget https:Վˌgithub.com/jumpserver/lion- release/releases/download/v2.13.2/lion-v2.13.2- linux-amd64.tar.gz # tar -xf lion-v2.13.2-linux-amd64.tar.gz # cd lion-v2.13.2-linux-amd64
```
#### 2.8.4 修改配置

```
cp config_example.yml config.yml # vi config.yml

项目名称, 会用来向Jumpserver注册, 识别而已, 不能重 复 # NAME: {{ Hostname }} # Jumpserver项目的url, api请求注册会使用 CORE_HOST: http:Վˌ127.0.0.1:8080 # Bootstrap Token, 预共享秘钥, 用来注册使用的 service account和terminal # 请和jumpserver 配置文件中保持一致，注册完成后可以删 除 BOOTSTRAP_TOKEN: 1gaKWv09fjmip6IISbHX0Ggl # 启动时绑定的ip, 默认 0.0.0.0 BIND_HOST: 0.0.0.0 # 监听的HTTP/WS端口号，默认8081 HTTPD_PORT: 8081 # 设置日志级别 [DEBUG, INFO, WARN, ERROR, FATAL, CRITICAL] LOG_LEVEL: INFO # Guacamole Server ip， 默认127.0.0.1 # GUA_HOST: 127.0.0.1 # Guacamole Server 端口号，默认4822 # GUA_PORT: 4822 # 会话共享使用的类型 [local, redis], 默认local # SHARE_ROOM_TYPE: local

Redis配置 # REDIS_HOST: 127.0.0.1 # REDIS_PORT: 6379 # REDIS_PASSWORD: # REDIS_DB_ROOM:
```
#### 2.8.5 启动Lion

```
./lion
```
### 2.9 安装Nginx

#### 2.9.1 安装nginx

```
yum install nginx -y # systemctl enable nginx
```
#### 2.9.2 整合jumpserver

```
server { listen 80; server_name jumpserver.oldxu.net; client_max_body_size 5000m; # Luna 配置 location /luna/ { try_files $uri / /index.html; alias /opt/luna/; }

Core data 静态资源 location /media/replay/ { add_header Content-Encoding gzip; root /opt/jumpserver-2.13.2/data/; } location /media/ { root /opt/jumpserver-2.13.2/data/; } location /static/ { root /opt/jumpserver-2.13.2/data/; } # KoKo Lion 配置 location /koko/ { proxy_pass http:Վˌlocalhost:5000; proxy_set_header Host $host; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_http_version 2.1; proxy_buffering off; proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "upgrade"; } # lion 配置 location /lion/ { proxy_pass http:Վˌlocalhost:8081; proxy_buffering off; proxy_request_buffering off; proxy_http_version 2.1;

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection $http_connection; proxy_ignore_client_abort on; proxy_connect_timeout 600; proxy_send_timeout 600; proxy_read_timeout 600; send_timeout 6000; } # Core 配置 location /ws/ { proxy_pass http:Վˌlocalhost:8070; proxy_set_header Host $host; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_http_version 2.1; proxy_buffering off; proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "upgrade"; } location /api/ { proxy_pass http:Վˌlocalhost:8080; proxy_set_header Host $host; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; } location /core/ {

proxy_pass http:Վˌlocalhost:8080; proxy_set_header Host $host; proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; } # 前端 Lina location /ui/ { try_files $uri / /ui/index.html; alias /jumpserver/lina/; } location / { rewrite ^/(.*)$ /ui/$1 last; } }
```
#### 2.9.3 访问Jumpserver
