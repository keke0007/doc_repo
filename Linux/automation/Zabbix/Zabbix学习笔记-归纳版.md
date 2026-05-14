# Zabbix 学习笔记归纳版

> 本文由 `Zabbix` 目录下 7 份 Markdown 文档归纳整理而成，按学习路径重新编排。每个知识点都包含：核心概念、代码示例、应用场景。

## 目录

- [1. 监控基础与 Zabbix 架构](#1-监控基础与-zabbix-架构)
- [2. Zabbix Server 安装部署](#2-zabbix-server-安装部署)
- [3. 添加 Linux 与 Windows 主机](#3-添加-linux-与-windows-主机)
- [4. 监控项 Item](#4-监控项-item)
- [5. 触发器 Trigger](#5-触发器-trigger)
- [6. 告警通知与升级](#6-告警通知与升级)
- [7. 故障自愈](#7-故障自愈)
- [8. 图形、聚合图形与模板](#8-图形聚合图形与模板)
- [9. 应用服务监控](#9-应用服务监控)
- [10. SNMP 与 Web 监测](#10-snmp-与-web-监测)
- [11. 自动化监控](#11-自动化监控)
- [12. Proxy 分布式监控](#12-proxy-分布式监控)
- [13. JVM 监控](#13-jvm-监控)
- [14. Zabbix API](#14-zabbix-api)
- [15. Zabbix 调优](#15-zabbix-调优)
- [16. Grafana 展示](#16-grafana-展示)
- [17. 学习建议与排错清单](#17-学习建议与排错清单)

## 1. 监控基础与 Zabbix 架构

### 1.1 什么是监控

监控是持续采集系统、服务、网络、业务指标，并在异常时告警的过程。

常见监控对象：

- 主机：CPU、内存、磁盘、网络、进程。
- 服务：Nginx、PHP、MySQL、Redis、Tomcat。
- 网络设备：交换机、路由器、防火墙。
- 业务接口：HTTP 状态码、响应时间、登录流程。

应用场景：

- 发现故障。
- 定位性能瓶颈。
- 形成容量规划依据。
- 为故障复盘提供数据。

### 1.2 单机监控常用命令

CPU：

```bash
uptime
top -bn1 | head
mpstat 1 3
```

内存：

```bash
free -m
```

磁盘：

```bash
df -h
df -i
```

网络：

```bash
ip addr
ss -lntup
sar -n DEV 1 3
```

TCP 状态：

```bash
ss -ant | awk 'NR>1 {state[$1]++} END {for (s in state) print s,state[s]}'
```

应用场景：

- 写自定义监控脚本前先确认指标能否通过命令取到。
- 故障时快速手工排查。

### 1.3 Zabbix 是什么

Zabbix 是企业级开源监控系统，支持主机、网络、应用、日志、Web 场景、自动发现、分布式 Proxy、告警通知和图形展示。

常见组件：

| 组件 | 作用 |
| --- | --- |
| Zabbix Server | 核心服务，负责采集调度、触发器计算、告警 |
| Zabbix Agent/Agent2 | 被控端采集主机和应用指标 |
| Database | 存储配置、历史数据、趋势数据 |
| Zabbix Web | Web 管理界面 |
| Zabbix Proxy | 分布式采集代理 |
| Java Gateway | 采集 JVM/JMX 指标 |

应用场景：

- 中小型环境可以 Server + DB + Web 单机部署。
- 大型或跨机房环境使用 Proxy 分布式采集。

### 1.4 Zabbix 数据流程

```text
Agent/SNMP/Web/JMX/API -> Server/Proxy -> Database -> Trigger -> Action -> Media
```

应用场景：

- 理解数据从采集到告警的完整链路。
- 排错时判断问题出在采集、存储、触发器还是通知。

## 2. Zabbix Server 安装部署

### 2.1 安装仓库与服务

示例以 RHEL/CentOS 系为主：

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all

yum install -y zabbix-server-mysql zabbix-agent2
yum install -y centos-release-scl
yum install -y zabbix-web-mysql-scl zabbix-apache-conf-scl
```

应用场景：

- 快速搭建实验环境。
- 按官方仓库安装标准版本。

### 2.2 初始化数据库

```bash
yum install -y mariadb-server
systemctl enable --now mariadb

mysql -uroot <<'SQL'
create database zabbix character set utf8 collate utf8_bin;
create user zabbix@localhost identified by 'zabbix_password';
grant all privileges on zabbix.* to zabbix@localhost;
flush privileges;
SQL

zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -pzabbix_password zabbix
```

应用场景：

- 初始化 Zabbix Server 所需库表。
- 实验环境快速部署 MySQL/MariaDB。

### 2.3 配置 Zabbix Server

```bash
sed -i 's/^# DBPassword=.*/DBPassword=zabbix_password/' /etc/zabbix/zabbix_server.conf
grep -E '^(DBHost|DBName|DBUser|DBPassword)=' /etc/zabbix/zabbix_server.conf
```

启动服务：

```bash
systemctl enable --now zabbix-server zabbix-agent2 httpd rh-php72-php-fpm
ss -lntup | grep -E '10051|10050|80'
```

应用场景：

- Zabbix Server 默认监听 `10051`。
- Agent 默认监听 `10050`。

### 2.4 配置 PHP 时区与前端

```bash
sed -i 's#; php_value\\[date.timezone\\] =.*#php_value[date.timezone] = Asia/Shanghai#' /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
systemctl restart rh-php72-php-fpm httpd
```

访问：

```text
http://zabbix-server/zabbix
默认用户: Admin
默认密码: zabbix
```

应用场景：

- 初始化 Web 控制台。
- 避免前端安装检查时报时区错误。

### 2.5 中文字体乱码处理

```bash
cd /usr/share/zabbix/assets/fonts
mv graphfont.ttf graphfont.ttf.bak
cp /path/to/simkai.ttf graphfont.ttf
chown apache:apache graphfont.ttf
```

应用场景：

- 解决图形中中文显示方块或乱码。
- 中文监控项、触发器、图形名称需要正常展示。

### 2.6 数据库拆分

```bash
# 旧库备份
mysqldump -uroot -p zabbix > zabbix.sql

# 新库恢复
mysql -uroot -p -e "create database zabbix character set utf8 collate utf8_bin;"
mysql -uroot -p zabbix < zabbix.sql
```

修改 Server：

```ini
DBHost=10.0.0.20
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix_password
```

应用场景：

- 监控规模变大后，将数据库从 Zabbix Server 拆出去。
- 减少单机 CPU、内存、磁盘 IO 压力。

## 3. 添加 Linux 与 Windows 主机

### 3.1 Linux 安装 Agent2

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all
yum install -y zabbix-agent2
```

配置：

```ini
Server=10.0.0.10
ServerActive=10.0.0.10
Hostname=web01
```

启动：

```bash
systemctl enable --now zabbix-agent2
ss -lntup | grep 10050
```

服务端测试：

```bash
yum install -y zabbix-get
zabbix_get -s 10.0.0.11 -k agent.ping
zabbix_get -s 10.0.0.11 -k system.hostname
```

应用场景：

- 添加 Linux 主机基础监控。
- 确认 Server 到 Agent 被动取值是否正常。

### 3.2 Windows 安装 Agent2

配置文件关键项：

```ini
Server=10.0.0.10
ServerActive=10.0.0.10
Hostname=win01
```

注册服务：

```powershell
zabbix_agent2.exe --config "C:\zabbix\zabbix_agent2.conf" --install
zabbix_agent2.exe --start
netstat -ano | findstr 10050
```

应用场景：

- 监控 Windows CPU、内存、磁盘、服务状态。
- 统一纳入 Zabbix 主机管理。

### 3.3 Web 页面添加主机

基本步骤：

1. 创建主机。
2. 设置主机名和可见名。
3. 添加 Agent 接口 IP。
4. 关联模板，例如 `Linux by Zabbix agent`。
5. 添加到主机组。

应用场景：

- 少量主机手工录入。
- 初学阶段理解主机、接口、模板、监控项的关系。

## 4. 监控项 Item

### 4.1 什么是监控项

监控项是 Zabbix 采集数据的最小单位。每个监控项都有 key、类型、更新间隔、数据类型、单位等属性。

示例 key：

```text
agent.ping
system.hostname
vfs.fs.size[/,pused]
net.tcp.listen[80]
```

应用场景：

- 采集主机资源指标。
- 采集应用状态和业务指标。

### 4.2 自定义监控项 UserParameter

Agent 配置：

```ini
UnsafeUserParameters=1
UserParameter=login.user.count,who | wc -l
```

重启 Agent：

```bash
systemctl restart zabbix-agent2
```

服务端测试：

```bash
zabbix_get -s 10.0.0.11 -k login.user.count
```

应用场景：

- 官方模板没有覆盖的业务指标。
- 通过 Shell 命令快速采集自定义数据。

### 4.3 带参数的 UserParameter

Agent 配置：

```ini
UserParameter=tcp.status[*],ss -ant | awk 'NR>1 {s[$1]++} END {print s["$1"]+0}'
```

取值：

```bash
zabbix_get -s 10.0.0.11 -k tcp.status[ESTAB]
zabbix_get -s 10.0.0.11 -k tcp.status[TIME-WAIT]
```

更稳妥的脚本方式：

```bash
cat >/etc/zabbix/scripts/tcp_status.sh <<'EOF'
#!/bin/bash
state="$1"
ss -ant | awk -v state="$state" 'NR>1 && $1==state {count++} END {print count+0}'
EOF
chmod +x /etc/zabbix/scripts/tcp_status.sh
```

Agent 配置：

```ini
UserParameter=tcp.status[*],/etc/zabbix/scripts/tcp_status.sh "$1"
```

应用场景：

- 一个 key 复用多个参数。
- 采集 TCP 不同状态、服务不同端口、Redis 不同实例。

### 4.4 值映射

值映射把数值转成可读状态。

示例：

```text
0 => Down
1 => Up
```

监控脚本：

```bash
pgrep nginx >/dev/null && echo 1 || echo 0
```

应用场景：

- 进程存活状态。
- 主从状态。
- 接口是否可用。

### 4.5 历史与趋势

| 数据 | 含义 |
| --- | --- |
| History | 原始历史数据 |
| Trends | 按小时聚合后的趋势数据 |

应用场景：

- 短期排障看历史数据。
- 长期容量规划看趋势数据。

## 5. 触发器 Trigger

### 5.1 什么是触发器

触发器根据监控项数据计算是否发生故障。

常见表达式：

```text
last(/web01/agent.ping)=0
avg(/web01/system.cpu.load[all,avg1],5m)>5
min(/web01/vfs.fs.size[/,pused],5m)>80
nodata(/web01/agent.ping,5m)=1
```

应用场景：

- 主机不可达。
- CPU 负载过高。
- 磁盘使用率过高。
- 一段时间没有数据上报。

### 5.2 严重性分级

常见级别：

```text
Information
Warning
Average
High
Disaster
```

应用场景：

- 不同级别对应不同告警渠道。
- 严重故障升级给值班负责人。

### 5.3 单条件触发器

```text
last(/web01/net.tcp.listen[80])=0
```

应用场景：

- 80 端口未监听立即告警。
- 进程不存在立即告警。

### 5.4 多条件触发器

```text
min(/web01/vfs.fs.size[/,pused],5m)>80 and last(/web01/agent.ping)=1
```

应用场景：

- 主机在线且磁盘高才告警。
- 减少主机宕机时引发的无意义关联告警。

### 5.5 恢复表达式与滞后

问题表达式：

```text
avg(/web01/system.cpu.load[all,avg1],5m)>5
```

恢复表达式：

```text
avg(/web01/system.cpu.load[all,avg1],5m)<3
```

应用场景：

- 避免指标在阈值附近反复抖动导致告警恢复频繁切换。
- CPU、连接数、响应时间等波动型指标适合使用恢复阈值。

### 5.6 触发器依赖

示例：

```text
路由器不可达
  -> 依赖它的 web01 不可达
  -> 依赖它的 web02 不可达
```

应用场景：

- 核心交换机或路由器故障时，不再重复发送大量下游主机不可达告警。
- 减少告警风暴。

## 6. 告警通知与升级

### 6.1 告警链路

```text
Trigger 产生事件 -> Action 匹配条件 -> Media 发送通知 -> User 接收
```

应用场景：

- 不同业务组接收不同主机组告警。
- 不同级别走邮件、微信、短信、电话等不同渠道。

### 6.2 邮件通知

邮件媒介参数示例：

```text
SMTP server: smtp.example.com
SMTP helo: example.com
SMTP email: zabbix@example.com
Authentication: username/password
```

动作消息模板：

```text
故障: {TRIGGER.NAME}
主机: {HOST.NAME}
级别: {TRIGGER.SEVERITY}
时间: {EVENT.DATE} {EVENT.TIME}
详情: {ITEM.NAME}: {ITEM.VALUE}
```

应用场景：

- 标准告警通知。
- 低到中等严重级别的异步通知。

### 6.3 微信/企业微信脚本通知

脚本示例：

```bash
#!/bin/bash
to="$1"
subject="$2"
message="$3"

curl -s -X POST "https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=YOUR_KEY" \
  -H 'Content-Type: application/json' \
  -d "{
    \"msgtype\": \"text\",
    \"text\": {
      \"content\": \"${subject}\n${message}\",
      \"mentioned_mobile_list\": [\"${to}\"]
    }
  }"
```

放置路径：

```bash
chmod +x /usr/lib/zabbix/alertscripts/wechat.sh
```

应用场景：

- 值班群实时通知。
- 需要比邮件更及时的告警。

### 6.4 短信通知

脚本入参一般是：

```text
$1 收件人手机号
$2 标题
$3 内容
```

简化脚本：

```bash
#!/bin/bash
phone="$1"
subject="$2"
message="$3"

python3 /usr/lib/zabbix/alertscripts/send_sms.py "$phone" "$subject $message"
```

应用场景：

- 高严重级别故障。
- 夜间值班通知。

### 6.5 通知升级

升级策略示例：

```text
0-5 分钟: 通知一线值班
5-15 分钟: 通知业务负责人
15 分钟以上: 通知运维负责人
```

应用场景：

- 故障长时间未恢复时扩大通知范围。
- 建立值班响应制度。

## 7. 故障自愈

### 7.1 什么是故障自愈

故障自愈是 Zabbix 在触发器异常时自动执行远程命令或脚本，尝试恢复服务。

应用场景：

- Nginx 进程停止后自动启动。
- 磁盘临时目录满后自动清理。
- 应用健康检查失败后自动重启。

### 7.2 开启远程命令

Agent 配置：

```ini
EnableRemoteCommands=1
LogRemoteCommands=1
```

Agent2 新版本通常使用允许列表方式，建议把自愈逻辑封装成明确脚本，再通过动作调用。

自愈脚本：

```bash
cat >/usr/local/bin/restart_nginx.sh <<'EOF'
#!/bin/bash
systemctl restart nginx
systemctl is-active nginx
EOF
chmod +x /usr/local/bin/restart_nginx.sh
```

应用场景：

- 只开放有限脚本，不开放任意命令。
- 保留执行日志，便于审计。

### 7.3 自愈动作示例

动作远程命令：

```bash
/usr/local/bin/restart_nginx.sh
```

触发条件：

```text
last(/web01/net.tcp.listen[80])=0
```

应用场景：

- Web 端口异常后自动尝试恢复。
- 低风险、恢复动作明确的故障。

注意：

- 不要对未知原因的复杂故障盲目自愈。
- 自愈失败仍要告警。
- 自愈命令必须可重复执行。

## 8. 图形、聚合图形与模板

### 8.1 图形

图形用于把监控项趋势可视化。

应用场景：

- 查看 CPU、内存、磁盘、流量趋势。
- 分析故障前后的指标变化。

### 8.2 聚合图形

聚合图形把多个主机或多个指标放在一个页面中。

应用场景：

- 大屏展示。
- 同时观察 Web 集群多台机器的 CPU、流量、连接数。

### 8.3 模板

模板把监控项、触发器、图形、发现规则等打包复用。

应用场景：

- 同类主机统一监控。
- Nginx、PHP、MySQL、Redis 分别维护模板。
- 新主机只需关联模板即可获得完整监控能力。

自定义模板设计建议：

```text
Template App Nginx Custom
  Items:
    ngx.status[active]
    ngx.status[reading]
  Triggers:
    Nginx port down
  Graphs:
    Nginx connections
```

## 9. 应用服务监控

### 9.1 Nginx 状态监控

启用 Nginx stub_status：

```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

测试：

```bash
curl -s http://127.0.0.1/nginx_status
```

采集脚本：

```bash
#!/bin/bash
metric="$1"
status=$(curl -s http://127.0.0.1/nginx_status)

case "$metric" in
  active) echo "$status" | awk '/Active/ {print $3}' ;;
  accepts) echo "$status" | awk 'NR==3 {print $1}' ;;
  handled) echo "$status" | awk 'NR==3 {print $2}' ;;
  requests) echo "$status" | awk 'NR==3 {print $3}' ;;
  reading) echo "$status" | awk '/Reading/ {print $2}' ;;
  writing) echo "$status" | awk '/Writing/ {print $4}' ;;
  waiting) echo "$status" | awk '/Waiting/ {print $6}' ;;
  *) echo 0 ;;
esac
```

Agent 配置：

```ini
UserParameter=ngx.status[*],/etc/zabbix/scripts/ngx_status.sh "$1"
```

应用场景：

- 监控 Nginx 活跃连接数、请求量、读写等待状态。
- 判断连接数异常、流量突增。

### 9.2 Nginx 错误日志监控

主动式日志监控项 key：

```text
log[/var/log/nginx/error.log,error|crit|alert|emerg,,,skip]
```

触发器：

```text
count(/web01/log[/var/log/nginx/error.log,error|crit|alert|emerg,,,skip],5m)>0
```

应用场景：

- 捕获 Nginx upstream 失败、权限错误、配置异常。
- 日志类指标通常适合主动模式。

### 9.3 PHP-FPM 状态监控

PHP-FPM 配置：

```ini
pm.status_path = /php_status
```

Nginx 转发：

```nginx
location /php_status {
    fastcgi_pass 127.0.0.1:9000;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
}
```

采集：

```bash
curl -s "http://127.0.0.1/php_status?json"
```

示例脚本：

```bash
#!/bin/bash
metric="$1"
curl -s http://127.0.0.1/php_status | awk -F: -v key="$metric" '$1 ~ key {gsub(/ /,"",$2); print $2}'
```

Agent 配置：

```ini
UserParameter=php.status[*],/etc/zabbix/scripts/php_status.sh "$1"
```

应用场景：

- 监控 PHP-FPM 活跃进程、空闲进程、慢请求、最大子进程。
- 判断 PHP 进程池是否需要扩容。

### 9.4 MySQL 状态监控

创建监控用户：

```sql
create user 'zbx_monitor'@'localhost' identified by 'monitor_password';
grant process, replication client, select on *.* to 'zbx_monitor'@'localhost';
flush privileges;
```

采集脚本：

```bash
#!/bin/bash
metric="$1"
mysql -uzbx_monitor -pmonitor_password -Nse "show global status like '$metric';" | awk '{print $2}'
```

Agent 配置：

```ini
UserParameter=mysql.status[*],/etc/zabbix/scripts/mysql_status.sh "$1"
```

测试：

```bash
zabbix_get -s 10.0.0.21 -k mysql.status[Threads_connected]
```

应用场景：

- 监控连接数、查询数、慢查询、缓存命中。
- 分析数据库压力。

### 9.5 MySQL 主从监控

脚本：

```bash
#!/bin/bash
metric="$1"
mysql -uzbx_monitor -pmonitor_password -e "show slave status\G" | awk -F: -v key="$metric" '$1 ~ key {gsub(/ /,"",$2); print $2}'
```

Agent 配置：

```ini
UserParameter=mysql.slave[*],/etc/zabbix/scripts/mysql_slave.sh "$1"
```

触发器：

```text
last(/db01/mysql.slave[Slave_IO_Running])<>"Yes" or last(/db01/mysql.slave[Slave_SQL_Running])<>"Yes"
```

应用场景：

- 发现主从复制中断。
- 监控复制延迟。

### 9.6 Redis 监控

采集脚本：

```bash
#!/bin/bash
metric="$1"
redis-cli info | awk -F: -v key="$metric" '$1==key {gsub(/\r/,"",$2); print $2}'
```

Agent 配置：

```ini
UserParameter=redis.info[*],/etc/zabbix/scripts/redis_info.sh "$1"
UserParameter=redis.config.maxclients,redis-cli config get maxclients | awk 'NR==2'
```

应用场景：

- 监控 Redis 连接数、内存使用、命中率、持久化状态。
- 判断缓存压力和异常重启。

## 10. SNMP 与 Web 监测

### 10.1 SNMP 基础

SNMP 常用于网络设备监控。

核心概念：

| 概念 | 含义 |
| --- | --- |
| OID | 指标对象编号 |
| MIB | OID 的说明库 |
| Community | SNMP v1/v2c 认证字符串 |
| SNMP v3 | 支持认证和加密 |

应用场景：

- 交换机接口流量。
- 路由器 CPU、内存。
- 防火墙会话数。

### 10.2 Linux SNMP 实践

安装：

```bash
yum install -y net-snmp net-snmp-utils
```

配置：

```ini
rocommunity public 10.0.0.10
syslocation IDC
syscontact ops@example.com
```

启动：

```bash
systemctl enable --now snmpd
```

取值：

```bash
snmpwalk -v2c -c public 10.0.0.11 1.3.6.1.2.1.1.1.0
```

应用场景：

- 无法安装 Agent 的设备使用 SNMP。
- 统一监控网络设备和部分系统设备。

### 10.3 Web 场景监测

curl 检查：

```bash
curl -o /dev/null -s -w "code=%{http_code} time=%{time_total}\n" https://www.example.com
```

Web 场景关键指标：

```text
响应状态码
响应时间
下载速度
页面内容匹配
登录流程是否成功
```

触发器示例：

```text
last(/web01/web.test.fail[官网可用性])<>0
avg(/web01/web.test.time[官网可用性,首页,resp],5m)>3
last(/web01/web.test.rspcode[官网可用性,首页])<>200
```

应用场景：

- 监控网站首页是否可用。
- 监控登录、下单等多步骤业务流程。
- 发现响应变慢。

## 11. 自动化监控

### 11.1 网络发现

网络发现由 Zabbix Server 扫描指定网段，发现符合条件的主机后执行动作。

发现规则示例：

```text
IP range: 10.0.0.1-254
Checks: Zabbix agent "system.uname"
Update interval: 1h
```

动作示例：

```text
条件: 发现服务 = Zabbix agent
操作: 添加主机、加入主机组、关联模板
```

应用场景：

- 发现固定网段内的新服务器。
- 适合基础设施相对稳定的内网环境。

### 11.2 自动注册

Agent 主动向 Server 注册，Server 根据元数据执行动作。

Agent 配置：

```ini
Server=10.0.0.10
ServerActive=10.0.0.10
Hostname=web01
HostMetadata=linux web nginx
```

动态元数据：

```ini
HostMetadataItem=system.uname
```

应用场景：

- 云主机弹性扩容后自动纳入监控。
- 结合 Ansible 初始化 Agent 配置。

### 11.3 主动模式与被动模式

| 模式 | 特点 |
| --- | --- |
| 被动模式 | Server 主动连接 Agent 取值 |
| 主动模式 | Agent 主动向 Server 拉取任务并上报数据 |

主动模式配置：

```ini
ServerActive=10.0.0.10
Hostname=web01
```

应用场景：

- Agent 在 NAT 后面，Server 无法直连。
- 大规模主机场景降低 Server 连接压力。
- 日志监控通常使用主动模式。

### 11.4 LLD 低级自动发现

LLD 用于自动发现同类对象，并生成监控项、触发器、图形。

网卡发现脚本：

```bash
#!/bin/bash
printf '{"data":['
first=1
for nic in $(ls /sys/class/net | grep -v lo); do
  [ "$first" -eq 0 ] && printf ','
  printf '{"{#IFNAME}":"%s"}' "$nic"
  first=0
done
printf ']}'
```

Agent 配置：

```ini
UserParameter=net.if.discovery.custom,/etc/zabbix/scripts/net_if_discovery.sh
```

监控项原型：

```text
net.if.in[{#IFNAME}]
net.if.out[{#IFNAME}]
```

应用场景：

- 自动发现网卡、磁盘、端口、Redis 实例。
- 避免手工为每个对象创建监控项。

### 11.5 LLD 发现端口

发现脚本：

```bash
#!/bin/bash
ports=$(ss -lnt | awk 'NR>1 {split($4,a,":"); print a[length(a)]}' | sort -nu)
printf '{"data":['
first=1
for port in $ports; do
  [ "$first" -eq 0 ] && printf ','
  printf '{"{#PORT}":"%s"}' "$port"
  first=0
done
printf ']}'
```

监控项原型：

```text
net.tcp.listen[{#PORT}]
```

触发器原型：

```text
last(/host/net.tcp.listen[{#PORT}])=0
```

应用场景：

- 自动监控本机监听端口。
- 服务端口新增后自动纳入监控。

### 11.6 LLD 发现 Redis 多实例

发现脚本：

```bash
#!/bin/bash
ports=$(ss -lnt | awk '$4 ~ /:637/ {split($4,a,":"); print a[length(a)]}' | sort -nu)
printf '{"data":['
first=1
for port in $ports; do
  [ "$first" -eq 0 ] && printf ','
  printf '{"{#REDIS_PORT}":"%s"}' "$port"
  first=0
done
printf ']}'
```

采集脚本：

```bash
#!/bin/bash
port="$1"
metric="$2"
redis-cli -p "$port" info | awk -F: -v key="$metric" '$1==key {gsub(/\r/,"",$2); print $2}'
```

应用场景：

- 一台机器部署多个 Redis 实例。
- 自动生成每个实例的连接数、内存、QPS 监控。

## 12. Proxy 分布式监控

### 12.1 什么是 Proxy

Zabbix Proxy 是分布式采集节点，负责采集一批被控端数据，再转发给 Server。

应用场景：

- 跨机房、跨地域监控。
- 网络不稳定环境缓存数据。
- 降低 Server 直接连接压力。

### 12.2 Proxy 安装与数据库

```bash
yum install -y zabbix-proxy-mysql mariadb-server
systemctl enable --now mariadb

mysql -uroot <<'SQL'
create database zabbix_proxy character set utf8 collate utf8_bin;
create user zabbix@localhost identified by 'proxy_password';
grant all privileges on zabbix_proxy.* to zabbix@localhost;
flush privileges;
SQL

zcat /usr/share/doc/zabbix-proxy-mysql*/schema.sql.gz | mysql -uzabbix -pproxy_password zabbix_proxy
```

应用场景：

- Proxy 需要自己的本地数据库缓存采集数据。
- Proxy 数据库只导入 schema，不导入 Server 的完整初始化数据。

### 12.3 Proxy 配置

`/etc/zabbix/zabbix_proxy.conf`：

```ini
Server=10.0.0.10
Hostname=proxy-idc01
DBHost=localhost
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=proxy_password
ConfigFrequency=60
DataSenderFrequency=5
```

启动：

```bash
systemctl enable --now zabbix-proxy
```

Agent 指向 Proxy：

```ini
Server=10.0.1.10
ServerActive=10.0.1.10
Hostname=web-idc01-01
```

应用场景：

- IDC 内 Agent 只连接本地 Proxy。
- Server 统一接收多个 Proxy 数据。

## 13. JVM 监控

### 13.1 Zabbix 如何监控 JVM

Zabbix 通过 Java Gateway 采集 JMX 指标。

链路：

```text
Zabbix Server -> Java Gateway -> JMX Port -> JVM/Tomcat
```

应用场景：

- 监控 Tomcat 堆内存、线程数、GC、类加载。
- 监控 Java 应用运行状态。

### 13.2 Tomcat 开启 JMX

Tomcat 启动参数示例：

```bash
export CATALINA_OPTS="$CATALINA_OPTS \
-Dcom.sun.management.jmxremote \
-Dcom.sun.management.jmxremote.port=12345 \
-Dcom.sun.management.jmxremote.rmi.port=12345 \
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.ssl=false \
-Djava.rmi.server.hostname=10.0.0.31"
```

应用场景：

- 实验环境可关闭认证。
- 生产环境建议启用认证和访问控制。

### 13.3 Java Gateway 配置

安装：

```bash
yum install -y zabbix-java-gateway
systemctl enable --now zabbix-java-gateway
```

Server 配置：

```ini
JavaGateway=127.0.0.1
JavaGatewayPort=10052
StartJavaPollers=5
```

重启：

```bash
systemctl restart zabbix-server
```

应用场景：

- 让 Zabbix Server 具备 JMX 采集能力。
- 给 Java 主机添加 JMX 接口并关联 JVM 模板。

## 14. Zabbix API

### 14.1 API 基础

Zabbix API 使用 JSON-RPC。

获取 Token：

```bash
curl -s -X POST http://zabbix.example.com/api_jsonrpc.php \
  -H 'Content-Type: application/json-rpc' \
  -d '{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
      "user": "Admin",
      "password": "zabbix"
    },
    "id": 1
  }'
```

应用场景：

- 自动化创建主机。
- 批量维护模板、主机组、接口。
- 与 CMDB 或发布系统集成。

### 14.2 禁用主机

```bash
curl -s -X POST http://zabbix.example.com/api_jsonrpc.php \
  -H 'Content-Type: application/json-rpc' \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.update",
    "params": {
      "hostid": "10105",
      "status": 1
    },
    "auth": "TOKEN",
    "id": 2
  }'
```

应用场景：

- 服务器下线前批量禁用监控。
- 维护窗口内暂停主机。

### 14.3 删除主机

```bash
curl -s -X POST http://zabbix.example.com/api_jsonrpc.php \
  -H 'Content-Type: application/json-rpc' \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.delete",
    "params": ["10105"],
    "auth": "TOKEN",
    "id": 3
  }'
```

应用场景：

- 资源销毁后同步删除监控对象。
- CMDB 驱动监控生命周期。

### 14.4 创建主机

```bash
curl -s -X POST http://zabbix.example.com/api_jsonrpc.php \
  -H 'Content-Type: application/json-rpc' \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.create",
    "params": {
      "host": "web03",
      "interfaces": [
        {
          "type": 1,
          "main": 1,
          "useip": 1,
          "ip": "10.0.0.13",
          "dns": "",
          "port": "10050"
        }
      ],
      "groups": [
        {"groupid": "2"}
      ],
      "templates": [
        {"templateid": "10001"}
      ]
    },
    "auth": "TOKEN",
    "id": 4
  }'
```

应用场景：

- Ansible 或云平台创建机器后自动加入 Zabbix。
- 批量导入主机。

## 15. Zabbix 调优

### 15.1 架构层调优

常见方向：

- 拆分数据库。
- 使用 Proxy 分布式采集。
- 主动模式替代大量被动采集。
- 删除无用监控项。
- 调整历史和趋势保留周期。

应用场景：

- 队列延迟高。
- 数据库写入压力大。
- Server 采集进程繁忙。

### 15.2 进程参数调优

`zabbix_server.conf` 示例：

```ini
StartPollers=20
StartPollersUnreachable=10
StartTrappers=10
StartPingers=5
StartDiscoverers=5
StartHTTPPollers=5
StartTimers=2
```

应用场景：

- 普通 Agent 监控多，增加 `StartPollers`。
- Web 场景多，增加 `StartHTTPPollers`。
- ICMP 检测多，增加 `StartPingers`。

### 15.3 缓存参数调优

```ini
CacheSize=512M
HistoryCacheSize=256M
HistoryIndexCacheSize=128M
TrendCacheSize=128M
ValueCacheSize=512M
```

应用场景：

- 日志提示 cache is full。
- 监控项规模增长后 Server 内存缓存不足。

### 15.4 检查延迟队列

Web 页面：

```text
Administration -> Queue
```

命令检查日志：

```bash
tail -f /var/log/zabbix/zabbix_server.log
```

应用场景：

- 判断哪些监控项延迟。
- 识别不可达主机、慢脚本、采集进程不足。

## 16. Grafana 展示

### 16.1 Grafana 是什么

Grafana 是可视化平台，可以通过 Zabbix 插件读取 Zabbix 数据并展示仪表盘。

应用场景：

- 大屏展示。
- 面向业务或管理层提供更美观的监控视图。
- 多数据源统一展示。

### 16.2 安装 Grafana

```bash
yum install -y grafana
systemctl enable --now grafana-server
```

访问：

```text
http://grafana-server:3000
默认用户: admin
默认密码: admin
```

### 16.3 安装 Zabbix 插件

```bash
grafana-cli plugins install alexanderzobnin-zabbix-app
systemctl restart grafana-server
```

应用场景：

- Grafana 直接读取 Zabbix API。
- 基于 Zabbix 主机组、主机、监控项创建仪表盘。

### 16.4 配置数据源

关键配置：

```text
Type: Zabbix
URL: http://zabbix.example.com/api_jsonrpc.php
Username: Admin
Password: zabbix
Trends: enabled
```

应用场景：

- 历史数据较多时启用 Trends 提升查询效率。
- 用变量筛选主机组和主机。

### 16.5 Grafana 变量

变量示例：

```text
Group: query host groups
Host: query hosts by selected group
Item: query items by selected host
```

应用场景：

- 一个仪表盘复用多个主机。
- 切换不同业务组查看 CPU、内存、流量。

## 17. 学习建议与排错清单

### 17.1 推荐学习顺序

1. 先掌握监控基础和 Zabbix 架构。
2. 完成 Server、DB、Web、Agent 的基础部署。
3. 学会添加主机、关联模板、查看最新数据。
4. 学监控项、触发器、告警动作。
5. 学 UserParameter 自定义监控。
6. 学 Nginx、PHP、MySQL、Redis 等应用监控。
7. 学自动注册、网络发现、LLD。
8. 学 Proxy、JVM、API、Grafana 和调优。

### 17.2 常用排错命令

```bash
# Server 状态
systemctl status zabbix-server
tail -f /var/log/zabbix/zabbix_server.log

# Agent 状态
systemctl status zabbix-agent2
tail -f /var/log/zabbix/zabbix_agent2.log

# 端口检查
ss -lntup | grep -E '10050|10051|10052'

# 服务端主动取值
zabbix_get -s 10.0.0.11 -k agent.ping

# 检查自定义 key
zabbix_get -s 10.0.0.11 -k 'ngx.status[active]'

# 数据库连接
mysql -uzabbix -p -h 127.0.0.1 zabbix
```

### 17.3 常见问题

| 问题 | 常见原因 | 处理方式 |
| --- | --- | --- |
| Agent 不可用 | 防火墙、Server 配置错误、Hostname 不一致 | 检查 `Server`、端口、防火墙 |
| `zabbix_get` 无结果 | key 写错、脚本无权限、路径错误 | 在 Agent 本机执行脚本并看日志 |
| Web 无数据 | 主机未关联模板、监控项不支持 | 看 Latest data 和 Unsupported item |
| 触发器不告警 | 表达式错误、动作条件不匹配 | 查看 Problems 和 Actions |
| 邮件不发送 | 媒介、用户媒介、动作缺失 | 分别检查 Media types、Users、Actions |
| LLD 不生成项目 | JSON 格式错误、宏名不匹配 | 用脚本输出验证 `{#MACRO}` |
| 队列延迟高 | 主机不可达、采集进程不足、脚本慢 | 看 Queue、日志和进程配置 |

### 17.4 生产建议

- 优先使用官方模板，缺失指标再自定义。
- 自定义脚本要设置超时，避免阻塞采集。
- 日志监控、大规模 Agent 建议使用主动模式。
- 高风险自愈动作必须最小权限和可审计。
- 告警要设置依赖和升级，避免告警风暴。
- 大规模环境尽早拆分数据库并使用 Proxy。
- Grafana 用于展示，Zabbix 仍负责采集、触发器和告警。

