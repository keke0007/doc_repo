# Shell-markdown 知识点维度汇总

> 说明：已读取 `nginx-markdown/1.md` 到 `nginx-markdown/10.md`。该目录名虽然是 `nginx-markdown`，但文件实际内容是 Shell 脚本课程资料，涵盖 Shell 基础、变量、条件判断、流程控制、循环、函数、数组、正则、sed、awk、练习题和项目实战。以下按知识点维度重新梳理。

## 1. Shell 脚本基础

### 核心知识点

Shell 脚本是把系统命令、固定语法和业务逻辑写入文件，用来自动化完成重复操作。常见用途包括系统初始化、软件安装、配置调整、服务部署、备份恢复、日志分析、监控告警、扩容缩容等。

脚本基本结构：

```bash
#!/usr/bin/env bash

echo "Hello world"
```

常见执行方式：

```bash
bash script.sh
chmod +x script.sh
./script.sh
```

### 示例代码

```bash
#!/usr/bin/env bash

echo "当前用户: $(whoami)"
echo "当前目录: $(pwd)"
echo "当前时间: $(date '+%F %T')"
```

### 应用场景

- 封装常用运维命令，减少手工操作。
- 编写服务器初始化脚本。
- 配合 crontab 定时执行备份、巡检、清理任务。

## 2. Bash 常用特性

### 核心知识点

Bash 提供命令补全、历史命令、别名、重定向、管道、前后台任务、通配符等能力。

常见符号：

```bash
command1 ; command2     # 无论 command1 是否成功，都执行 command2
command1 && command2    # command1 成功才执行 command2
command1 || command2    # command1 失败才执行 command2
command > file          # 标准输出覆盖写入
command >> file         # 标准输出追加写入
command 2> err.log      # 标准错误写入
command &> all.log      # 标准输出和错误都写入
command1 | command2     # 管道
```

### 示例代码

```bash
#!/usr/bin/env bash

mkdir -p /tmp/shell-demo && echo "目录已准备"
df -h | awk 'NR==1 || /\/$/ {print}'
```

### 应用场景

- 用 `&&` 串联部署步骤，前一步失败则停止后续动作。
- 用管道组合 `grep`、`awk`、`sort`、`uniq` 做日志统计。
- 用重定向记录脚本执行日志。

## 3. 变量与参数

### 核心知识点

Shell 变量用于保存可变内容，常见变量包括自定义变量、环境变量、位置参数变量和预定义变量。

常见写法：

```bash
name="nginx"
echo "$name"
echo "${name}"

export APP_ENV="prod"

echo "$0"    # 脚本名
echo "$1"    # 第一个参数
echo "$#"    # 参数个数
echo "$*"    # 所有参数
echo "$@"    # 所有参数
echo "$$"    # 当前 shell PID
echo "$?"    # 上一条命令退出状态
```

变量赋值方式：

```bash
today=$(date +%F)
read -r -p "请输入服务名: " service_name
```

变量截取与替换：

```bash
url="www.sina.com.cn"
echo "${#url}"              # 长度
echo "${url#*.}"            # 从前删除最短匹配
echo "${url##*.}"           # 从前删除最长匹配
echo "${url%.*}"            # 从后删除最短匹配
echo "${url%%.*}"           # 从后删除最长匹配
echo "${url:0:3}"           # 截取
echo "${url/sina/baidu}"    # 替换一次
echo "${url//n/N}"          # 全部替换
```

### 示例代码

```bash
#!/usr/bin/env bash

if [ "$#" -lt 1 ]; then
  echo "用法: $0 <backup_dir>"
  exit 1
fi

backup_dir="$1"
today=$(date +%F)
mkdir -p "$backup_dir"
df -h > "${backup_dir}/${today}-disk.log"

echo "磁盘信息已写入: ${backup_dir}/${today}-disk.log"
```

### 应用场景

- 用参数控制脚本行为，例如备份目录、服务名、IP 地址。
- 用变量拼接文件名、日志名、配置路径。
- 用变量替换批量处理域名、路径、文件后缀。

## 4. 数值运算

### 核心知识点

Shell 支持整数运算，常用方式包括 `$(( ))`、`let`、`expr`。小数运算通常使用 `bc` 或 `awk`。

```bash
a=10
b=3

echo $((a + b))
echo $((a - b))
echo $((a * b))
echo $((a / b))
echo $((a % b))

let a++
echo "$a"

awk 'BEGIN {print 10 / 3}'
```

### 示例代码

```bash
#!/usr/bin/env bash

mem_total=$(free -m | awk '/^Mem:/ {print $2}')
mem_used=$(free -m | awk '/^Mem:/ {print $3}')
mem_percent=$((mem_used * 100 / mem_total))

echo "内存使用率: ${mem_percent}%"
```

### 应用场景

- 计算 CPU、内存、磁盘使用率。
- 统计请求量、错误率、备份文件大小。
- 控制循环计数和阈值判断。

## 5. 条件测试

### 核心知识点

Shell 条件测试常用 `test`、`[ ]`、`[[ ]]`。

文件测试：

```bash
[ -e file ]    # 是否存在
[ -f file ]    # 是否为普通文件
[ -d dir ]     # 是否为目录
[ -r file ]    # 是否可读
[ -w file ]    # 是否可写
[ -x file ]    # 是否可执行
[ -L file ]    # 是否为软链接
```

数值比较：

```bash
[ "$a" -gt "$b" ]    # 大于
[ "$a" -lt "$b" ]    # 小于
[ "$a" -eq "$b" ]    # 等于
[ "$a" -ne "$b" ]    # 不等于
[ "$a" -ge "$b" ]    # 大于等于
[ "$a" -le "$b" ]    # 小于等于
```

字符串比较：

```bash
[ "$name" = "nginx" ]
[ "$name" != "httpd" ]
[ -z "$name" ]       # 空字符串
[ -n "$name" ]       # 非空字符串
[[ "$name" =~ ^ng ]] # 正则匹配
```

### 示例代码

```bash
#!/usr/bin/env bash

disk_used=$(df -h / | awk 'NR==2 {gsub("%", "", $5); print $5}')

if [ "$disk_used" -ge 80 ]; then
  echo "告警: 根分区使用率 ${disk_used}%"
else
  echo "正常: 根分区使用率 ${disk_used}%"
fi
```

### 应用场景

- 判断目录是否存在，不存在则创建。
- 判断用户、服务、进程、端口是否存在。
- 根据资源使用率触发告警或自动处理。

## 6. if 流程控制

### 核心知识点

`if` 适合处理一到多个条件分支。

```bash
if 条件; then
  命令
elif 条件; then
  命令
else
  命令
fi
```

### 示例代码

```bash
#!/usr/bin/env bash

service_name="${1:-nginx}"

if systemctl is-active --quiet "$service_name"; then
  echo "${service_name} 正在运行"
elif systemctl list-unit-files | grep -q "^${service_name}.service"; then
  echo "${service_name} 未运行，尝试启动"
  systemctl start "$service_name"
else
  echo "${service_name} 未安装或不存在"
  exit 1
fi
```

### 应用场景

- 安装脚本中判断网络、仓库、软件包状态。
- 服务管理中判断 start、stop、restart 是否成功。
- 监控脚本中按不同阈值输出不同级别告警。

## 7. case 流程控制

### 核心知识点

`case` 适合菜单、命令参数、服务动作等多分支匹配。

```bash
case "$变量" in
  模式1)
    命令
    ;;
  模式2)
    命令
    ;;
  *)
    默认命令
    ;;
esac
```

### 示例代码

```bash
#!/usr/bin/env bash

service_name="nginx"
action="$1"

case "$action" in
  start|stop|restart|reload|status)
    systemctl "$action" "$service_name"
    ;;
  *)
    echo "用法: $0 {start|stop|restart|reload|status}"
    exit 1
    ;;
esac
```

### 应用场景

- 编写服务控制脚本。
- 编写交互式管理菜单。
- 根据用户输入执行不同任务。

## 8. for 循环

### 核心知识点

`for` 适合遍历固定列表、文件列表、IP 段、命令输出。

```bash
for item in item1 item2 item3
do
  echo "$item"
done
```

### 示例代码

```bash
#!/usr/bin/env bash

network="192.168.70"
output="/tmp/ip-up.txt"
> "$output"

for i in {1..254}; do
  {
    ip="${network}.${i}"
    if ping -c1 -W1 "$ip" &>/dev/null; then
      echo "$ip" | tee -a "$output"
    fi
  } &
done

wait
echo "探测完成，存活主机已写入: $output"
```

### 应用场景

- 批量探测主机存活状态。
- 批量创建用户、修改文件名、检查服务。
- 批量处理日志文件或备份文件。

## 9. while 循环

### 核心知识点

`while` 适合持续监控、按条件循环、逐行读取文件。

```bash
while 条件
do
  命令
done
```

### 示例代码

```bash
#!/usr/bin/env bash

log_file="/var/log/nginx/access.log"

while true; do
  count=$(tail -n 300 "$log_file" | awk '$9 == 502 {n++} END {print n+0}')
  if [ "$count" -ge 100 ]; then
    echo "最近 300 行出现 ${count} 次 502，重启 php-fpm"
    systemctl restart php-fpm
    sleep 30
  fi
  sleep 5
done
```

### 应用场景

- 持续监控 Nginx 状态码。
- 定时检测端口或进程。
- 读取文件逐行处理数据。

## 10. Shell 内置控制命令

### 核心知识点

常见内置控制命令：

```bash
break       # 跳出循环
continue    # 跳过本次循环
exit        # 退出脚本
return      # 从函数返回
```

### 示例代码

```bash
#!/usr/bin/env bash

for i in {1..3}; do
  if ping -c1 -W1 192.168.70.10 &>/dev/null; then
    echo "连接成功"
    break
  fi
  echo "第 ${i} 次失败"
done
```

### 应用场景

- 多次重试成功后提前退出。
- 参数非法时立即退出脚本。
- 循环处理中跳过不符合条件的数据。

## 11. 函数

### 核心知识点

函数用于封装重复逻辑，提高脚本复用性和可读性。

```bash
function_name() {
  命令
}

function_name
```

函数可以接收参数：

```bash
sum() {
  echo $(($1 + $2))
}

sum 10 20
```

### 示例代码

```bash
#!/usr/bin/env bash

check_command() {
  local cmd="$1"
  if ! command -v "$cmd" &>/dev/null; then
    echo "缺少命令: $cmd"
    return 1
  fi
}

check_service() {
  local service_name="$1"
  if systemctl is-active --quiet "$service_name"; then
    echo "${service_name}: running"
  else
    echo "${service_name}: stopped"
  fi
}

check_command systemctl || exit 1
check_service nginx
```

### 应用场景

- 安装脚本中封装安装、配置、启动、检查步骤。
- 服务器初始化脚本中封装 yum、时区、内核参数、limits 调整。
- 监控脚本中封装告警发送、状态采集、日志分析。

## 12. 数组

### 核心知识点

Shell 支持普通数组和关联数组。

普通数组：

```bash
books=(nginx mysql shell)
echo "${books[0]}"
echo "${books[@]}"
echo "${#books[@]}"
```

关联数组：

```bash
declare -A info
info[name]="bgx"
info[skill]="linux"
echo "${info[name]}"
```

遍历数组：

```bash
for item in "${books[@]}"; do
  echo "$item"
done
```

### 示例代码

```bash
#!/usr/bin/env bash

declare -A services=(
  [web]="nginx"
  [php]="php-fpm"
  [db]="mysqld"
)

for role in "${!services[@]}"; do
  service_name="${services[$role]}"
  if systemctl is-active --quiet "$service_name"; then
    echo "${role}/${service_name}: running"
  else
    echo "${role}/${service_name}: stopped"
  fi
done
```

### 应用场景

- 保存批量服务列表、主机列表、用户列表。
- 用关联数组统计 Nginx 日志中各 IP 访问次数。
- 编写菜单脚本时把编号映射到脚本路径。

## 13. 正则表达式

### 核心知识点

正则表达式用于匹配、过滤和提取文本，是 `grep`、`sed`、`awk` 的基础。

常见元字符：

```text
^        行首
$        行尾
^$       空行
.        任意单个字符
*        前一个字符出现 0 次或多次
[]       字符集合
[^]      排除字符集合
{n}      重复 n 次
{n,m}    重复 n 到 m 次
|        或
()       分组
```

常见字符类：

```text
[[:digit:]]  数字
[[:lower:]]  小写字母
[[:upper:]]  大写字母
[[:alpha:]]  字母
[[:space:]]  空白字符
```

### 示例代码

```bash
#!/usr/bin/env bash

log_file="/var/log/nginx/access.log"

grep -E '^[0-9]{1,3}(\.[0-9]{1,3}){3}' "$log_file" | head
grep -Ev '^$|^[[:space:]]*#' /etc/nginx/nginx.conf
```

### 应用场景

- 过滤配置文件中的空行和注释。
- 从日志中提取 IP、URL、状态码。
- 在批量替换前精确定位目标文本。

## 14. grep 文本过滤

### 核心知识点

`grep` 用于按模式筛选行。

```bash
grep "keyword" file
grep -n "keyword" file       # 显示行号
grep -v "^$" file            # 排除空行
grep -E "a|b" file           # 扩展正则
grep -o "[0-9]\+" file       # 只输出匹配内容
```

### 示例代码

```bash
#!/usr/bin/env bash

conf="/etc/nginx/nginx.conf"

grep -nE 'server_name|listen|root' "$conf"
```

### 应用场景

- 快速定位配置项。
- 从日志中过滤错误关键字。
- 配合管道做进一步统计。

## 15. sed 文本处理

### 核心知识点

`sed` 是流编辑器，适合非交互式修改文本。默认逐行读取到模式空间，执行命令后输出。

常见选项：

```bash
sed -n '1,10p' file              # 打印 1 到 10 行
sed -i 's/old/new/g' file        # 原地替换
sed -r 's/(old|OLD)/new/g' file  # 扩展正则
sed '/pattern/d' file            # 删除匹配行
```

常见命令：

```text
p  打印
d  删除
a  行后追加
i  行前插入
c  替换整行
s  字符串替换
```

### 示例代码

```bash
#!/usr/bin/env bash

conf="/etc/nginx/nginx.conf"

cp "$conf" "${conf}.$(date +%F-%H%M%S).bak"
sed -i -r 's/^[[:space:]]*worker_processes[[:space:]]+.*/worker_processes auto;/' "$conf"
nginx -t && systemctl reload nginx
```

### 应用场景

- 自动修改 Nginx、MySQL、SSH 等配置文件。
- 批量注释或取消注释配置项。
- 批量替换路径、域名、端口。

## 16. awk 文本分析

### 核心知识点

`awk` 适合按字段处理结构化文本，常用于统计、格式化输出、条件筛选。

基本格式：

```bash
awk 'pattern { action }' file
awk -F: '{print $1, $3}' /etc/passwd
```

常用内置变量：

```text
$0      当前整行
$1      第一个字段
NF      当前行字段数
NR      当前处理行号
FS      输入字段分隔符
OFS     输出字段分隔符
```

### 示例代码

统计 Nginx 访问 IP Top 10：

```bash
#!/usr/bin/env bash

log_file="/var/log/nginx/access.log"

awk '{ip[$1]++} END {for (i in ip) print ip[i], i}' "$log_file" |
  sort -rn |
  head -10
```

统计 HTTP 状态码：

```bash
#!/usr/bin/env bash

awk '{code[$9]++} END {for (i in code) print i, code[i]}' /var/log/nginx/access.log |
  sort -n
```

### 应用场景

- 统计日志访问量、状态码、接口请求量。
- 从系统命令输出中提取字段。
- 计算内存、磁盘、进程等资源数据。

## 17. expect 交互自动化

### 核心知识点

`expect` 用于自动处理需要交互输入的命令，例如 SSH 登录、密码输入、远程执行命令。

基本思路：

```text
spawn 启动交互命令
expect 等待指定输出
send 发送输入
interact 保持交互
```

### 示例代码

```tcl
#!/usr/bin/expect

set timeout 10
set host [lindex $argv 0]
set user [lindex $argv 1]
set pass [lindex $argv 2]

spawn ssh $user@$host
expect {
  "yes/no" { send "yes\r"; exp_continue }
  "password:" { send "$pass\r" }
}
interact
```

### 应用场景

- 自动登录远程主机。
- 自动分发 SSH key。
- 批量执行带交互确认的命令。

## 18. 日志分析

### 核心知识点

日志分析常见流程是提取字段、排序、去重、计数、排序输出。

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head
```

### 示例代码

```bash
#!/usr/bin/env bash

log_file="${1:-/var/log/nginx/access.log}"

echo "访问 IP Top 10:"
awk '{ip[$1]++} END {for (i in ip) print ip[i], i}' "$log_file" |
  sort -rn |
  head -10

echo "状态码统计:"
awk '{code[$9]++} END {for (i in code) print i, code[i]}' "$log_file" |
  sort -n
```

### 应用场景

- 统计 Nginx 每个 IP 的访问量。
- 发现异常状态码，例如 404、500、502。
- 为限流、防刷、故障定位提供数据依据。

## 19. 服务与端口监控

### 核心知识点

监控脚本通常检查进程、端口或服务状态，异常时执行恢复动作。

常用命令：

```bash
ps aux | grep '[n]ginx'
ss -lntp | grep ':80'
systemctl is-active --quiet nginx
```

### 示例代码

```bash
#!/usr/bin/env bash

service_name="nginx"
port="80"

if ss -lnt | awk '{print $4}' | grep -Eq "(:|\\.)${port}$"; then
  exit 0
fi

if systemctl is-active --quiet "$service_name"; then
  systemctl reload "$service_name"
else
  systemctl start "$service_name"
fi

echo "$(date '+%F %T') ${service_name} ${port} 端口异常，已尝试恢复" >> /var/log/service-watch.log
```

### 应用场景

- 检测 Nginx 80/443 端口是否监听。
- 配合 crontab 每分钟执行服务自愈。
- 服务异常时重启并记录日志或发送告警。

## 20. 系统巡检

### 核心知识点

系统巡检脚本通常采集系统版本、内核、主机名、IP、DNS、登录用户、磁盘、内存、负载等信息。

### 示例代码

```bash
#!/usr/bin/env bash

echo "系统版本: $(hostnamectl | awk -F: '/Operating System/ {print $2}')"
echo "内核版本: $(hostnamectl | awk -F: '/Kernel/ {print $2}')"
echo "主机名: $(hostname)"
echo "内网 IP: $(hostname -I)"
echo "DNS: $(awk '/nameserver/ {print $2}' /etc/resolv.conf | xargs)"
echo "登录用户:"
who
echo "负载:"
uptime
echo "磁盘:"
df -h
echo "内存:"
free -m
```

### 应用场景

- 服务器上线前检查。
- 故障排查时快速收集环境信息。
- 定时生成巡检报告。

## 21. 服务器初始化

### 核心知识点

服务器初始化通常包括配置 yum 源、安装基础软件、调整文件描述符、设置时区、设置语言、时间同步、关闭防火墙或 SELinux、关闭 IPv6、调整历史记录和内核参数。

### 示例代码

```bash
#!/usr/bin/env bash
set -e

yum install -y vim wget curl telnet nc lsof iotop iftop dstat

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
sed -i 's#^LANG=.*#LANG="en_US.UTF-8"#' /etc/locale.conf

cat >/etc/security/limits.d/99-custom.conf <<'EOF'
* soft nproc 65535
* hard nproc 65535
* soft nofile 65535
* hard nofile 65535
EOF

grep -q '^HISTTIMEFORMAT=' /etc/profile &&
  sed -i 's/^HISTTIMEFORMAT=.*/HISTTIMEFORMAT="%F %T "/' /etc/profile ||
  echo 'HISTTIMEFORMAT="%F %T "' >> /etc/profile

cat >/etc/sysctl.d/99-custom.conf <<'EOF'
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 1024 65535
EOF

sysctl --system
```

### 应用场景

- 新服务器批量交付。
- 云主机自动扩容后的基础环境配置。
- 统一企业服务器基线。

## 22. 软件自动化部署

### 核心知识点

自动化部署脚本一般包含检查环境、配置仓库、安装软件、生成配置、启动服务、健康检查等步骤。

### 示例代码

```bash
#!/usr/bin/env bash
set -e

install_nginx() {
  if rpm -q nginx &>/dev/null; then
    echo "nginx 已安装"
    return
  fi

  cat >/etc/yum.repos.d/nginx.repo <<'EOF'
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/x86_64/
gpgcheck=0
enabled=1
EOF

  yum clean all
  yum install -y nginx
}

start_nginx() {
  nginx -t
  systemctl enable --now nginx
  systemctl status nginx --no-pager
}

install_nginx
start_nginx
```

### 应用场景

- 自动安装 Nginx、MySQL、PHP、Redis。
- 搭建 LNMP/LAMP 环境。
- 在多台服务器保持部署步骤一致。

## 23. 备份与文件批处理

### 核心知识点

备份和文件批处理常用 `find`、`tar`、变量截取、循环、时间命名。

### 示例代码

```bash
#!/usr/bin/env bash

src_dir="/backup"
today=$(date +%F)
archive="/tmp/backup-${today}.tar.gz"

find "$src_dir" -type f -name "*.txt" -print0 |
while IFS= read -r -d '' file; do
  mv "$file" "${file}.bak"
done

tar czf "$archive" -C "$src_dir" .

find "$src_dir" -type f -name "*.txt.bak" -print0 |
while IFS= read -r -d '' file; do
  mv "$file" "${file%.bak}"
done

echo "备份完成: $archive"
```

### 应用场景

- 按日期保存磁盘状态、日志、配置。
- 批量修改文件后缀并打包。
- 数据库、配置文件、应用日志定期备份。

## 24. 主机存活探测与重试

### 核心知识点

探测主机时可以结合循环、并发、重试机制，提高准确性和效率。

### 示例代码

```bash
#!/usr/bin/env bash

network="192.168.70"

for host in {150..170}; do
  {
    ip="${network}.${host}"
    ok=0

    for retry in {1..3}; do
      if ping -c1 -W1 "$ip" &>/dev/null; then
        echo "$ip 连接成功"
        ok=1
        break
      fi
    done

    [ "$ok" -eq 0 ] && echo "$ip 三次探测均失败"
  } &
done

wait
```

### 应用场景

- 上线前批量检查主机连通性。
- 巡检内网资产。
- 生成可用主机清单。

## 25. 综合脚本设计建议

### 核心知识点

写 Shell 脚本时建议遵守以下原则：

```bash
#!/usr/bin/env bash
set -euo pipefail
```

- 参数、路径、服务名尽量变量化。
- 对外部输入加引号，避免空格和特殊字符导致异常。
- 重要操作前做检查，重要修改前做备份。
- 函数化封装重复逻辑。
- 输出明确的成功、失败和错误原因。
- 需要长期运行的脚本要考虑日志、退出条件和重试间隔。

### 示例代码

```bash
#!/usr/bin/env bash
set -euo pipefail

log() {
  echo "$(date '+%F %T') $*"
}

require_root() {
  if [ "$(id -u)" -ne 0 ]; then
    log "请使用 root 执行"
    exit 1
  fi
}

backup_file() {
  local file="$1"
  if [ -f "$file" ]; then
    cp "$file" "${file}.$(date +%F-%H%M%S).bak"
  fi
}

require_root
backup_file /etc/nginx/nginx.conf
log "脚本执行完成"
```

### 应用场景

- 编写生产环境可维护脚本。
- 降低误操作风险。
- 让脚本适合多人协作和长期使用。

## 26. 知识点与典型任务映射

| 任务 | 主要知识点 |
| --- | --- |
| 自动安装 Nginx/MySQL/PHP | 变量、if、函数、yum、systemctl |
| 批量探测主机 | for、并发、ping、重试、wait |
| 服务自愈 | 条件测试、systemctl、端口检查、日志记录 |
| Nginx 日志 Top IP | awk 数组、sort、head |
| 统计 HTTP 状态码 | awk 字段处理、关联数组 |
| 修改配置文件 | sed、正则、备份、服务 reload |
| 服务器初始化 | 函数、文件写入、sysctl、limits、时区 |
| 定时备份 | date、tar、find、crontab |
| 交互自动化 | expect、spawn、send、expect |
| 系统巡检 | hostnamectl、df、free、uptime、who、awk |

## 27. 推荐学习顺序

1. Shell 脚本基础与 Bash 常用特性。
2. 变量、参数、数值运算。
3. 条件测试、if、case。
4. for、while、break、continue、exit。
5. 函数与数组。
6. 正则表达式。
7. grep、sed、awk。
8. 日志分析、服务监控、系统初始化、软件部署等项目实战。

