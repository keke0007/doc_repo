# Shell 学习笔记归纳版

> 本文由 `Shell` 目录下 6 份 Markdown 文档归纳整理而成，按学习路径重新编排。每个知识点都包含：核心概念、代码示例、应用场景。

## 目录

- [1. Shell 脚本基础](#1-shell-脚本基础)
- [2. 变量与参数](#2-变量与参数)
- [3. 条件判断：if 与 case](#3-条件判断if-与-case)
- [4. 循环控制：for 与 while](#4-循环控制for-与-while)
- [5. 函数](#5-函数)
- [6. 数组](#6-数组)
- [7. 正则表达式与 grep](#7-正则表达式与-grep)
- [8. sed 文本处理](#8-sed-文本处理)
- [9. awk 文本处理](#9-awk-文本处理)
- [10. 综合实践案例](#10-综合实践案例)
- [11. 学习建议与排错清单](#11-学习建议与排错清单)

## 1. Shell 脚本基础

### 1.1 什么是 Shell

Shell 是用户和 Linux 内核之间的命令解释器。常见 Shell 有 `bash`、`sh`、`zsh`、`ksh`。日常 Linux 运维脚本通常使用 `bash`。

应用场景：

- 批量执行 Linux 命令。
- 自动化部署、备份、巡检。
- 处理日志、配置文件、用户数据。

查看当前 Shell：

```bash
echo "$SHELL"
```

### 1.2 Shell 脚本基本格式

脚本通常以 `#!/bin/bash` 开头，称为 shebang，用来指定解释器。

```bash
#!/bin/bash

echo "Hello Shell"
date
```

运行方式：

```bash
chmod +x hello.sh
./hello.sh

# 或者直接指定解释器
bash hello.sh
```

应用场景：

- 固定脚本解释器，避免不同环境默认 Shell 不一致。
- 将多条命令保存为可重复执行的自动化任务。

### 1.3 脚本命名与书写规范

建议：

- 文件名使用英文、数字、下划线，例如 `backup_mysql.sh`。
- 变量名见名知意，例如 `backup_dir`、`log_file`。
- 命令、变量引用尽量加双引号，减少空格和特殊字符问题。
- 脚本开头写清用途、作者、日期、依赖。

模板：

```bash
#!/bin/bash
# Description: Backup application logs
# Usage: bash backup_logs.sh

set -o errexit
set -o nounset
set -o pipefail

log_dir="/var/log/nginx"
backup_dir="/backup/nginx"
mkdir -p "$backup_dir"
tar -czf "$backup_dir/nginx-$(date +%F).tar.gz" "$log_dir"
```

应用场景：

- 生产环境脚本需要可读、可维护、可排错。
- `set -o errexit` 让命令失败时退出，避免错误继续扩大。

## 2. 变量与参数

### 2.1 自定义变量

变量用于保存可复用的数据。定义变量时等号两边不能有空格。

```bash
name="linux"
version="8"

echo "$name"
echo "CentOS $version"
```

应用场景：

- 保存路径、用户名、端口、服务名。
- 避免脚本中大量重复硬编码。

### 2.2 环境变量

环境变量由系统或当前 Shell 提供，例如 `PATH`、`HOME`、`USER`。

```bash
echo "$PATH"
echo "$HOME"
echo "$USER"
```

临时导出变量给子进程：

```bash
export APP_ENV="prod"
bash deploy.sh
```

应用场景：

- 部署脚本根据 `APP_ENV` 区分测试、预发、生产环境。
- 调整 `PATH` 让系统能找到自定义命令。

### 2.3 位置参数

位置参数用于接收脚本执行时传入的参数。

| 参数 | 含义 |
| --- | --- |
| `$0` | 脚本名称 |
| `$1` | 第 1 个参数 |
| `$2` | 第 2 个参数 |
| `$#` | 参数个数 |
| `$*` | 所有参数，整体看待 |
| `$@` | 所有参数，逐个看待 |
| `$?` | 上一条命令的退出状态 |
| `$$` | 当前脚本进程 PID |

示例：

```bash
#!/bin/bash

echo "脚本名: $0"
echo "第一个参数: $1"
echo "第二个参数: $2"
echo "参数个数: $#"
```

执行：

```bash
bash args.sh nginx start
```

应用场景：

- 写通用脚本，例如 `service_control.sh nginx start`。
- 根据参数选择备份目录、服务名称、操作模式。

### 2.4 read 交互输入

`read` 用于接收用户输入。

```bash
#!/bin/bash

read -r -p "请输入用户名: " username
read -r -s -p "请输入密码: " password
echo

echo "用户 $username 已输入密码，长度为 ${#password}"
```

应用场景：

- 交互式创建用户。
- 执行高风险操作前二次确认。
- 输入主机名、IP、备份路径。

### 2.5 变量删除与截取

Bash 提供字符串删除能力，常用于处理路径、文件名、百分比。

```bash
file="/data/logs/nginx/access.log"

echo "${file#*/}"      # 删除从左开始最短匹配
echo "${file##*/}"     # 删除从左开始最长匹配，得到 access.log
echo "${file%/*}"      # 删除从右开始最短匹配，得到目录
echo "${file%%/*}"     # 删除从右开始最长匹配
```

应用场景：

- 从完整路径提取文件名。
- 从 URL 提取域名或路径。
- 从 `80%` 中提取数字。

示例：提取内存使用率数字。

```bash
mem_used="72%"
mem_num="${mem_used%\%}"

if [ "$mem_num" -ge 80 ]; then
  echo "内存使用率过高: $mem_used"
else
  echo "内存使用率正常: $mem_used"
fi
```

### 2.6 变量替换

```bash
text="hello shell shell"

echo "${text/shell/bash}"    # 替换第一个 shell
echo "${text//shell/bash}"   # 替换所有 shell
```

应用场景：

- 替换配置模板中的占位符。
- 批量处理字符串中的旧域名、旧路径。

示例：生成配置文件。

```bash
template="server_name __DOMAIN__;"
domain="www.example.com"

echo "${template/__DOMAIN__/$domain}" > nginx.conf
```

### 2.7 变量运算

常用整数运算方式：

```bash
a=10
b=3

echo $((a + b))
echo $((a - b))
echo $((a * b))
echo $((a / b))
echo $((a % b))
```

应用场景：

- 计算磁盘、内存、日志数量。
- 循环计数。
- 根据日期计算保留周期。

示例：计算今年还剩多少天。

```bash
today=$(date +%j)
year=$(date +%Y)

if date -d "$year-02-29" >/dev/null 2>&1; then
  total=366
else
  total=365
fi

echo "今年还剩 $((total - today)) 天"
```

## 3. 条件判断：if 与 case

### 3.1 if 基础语法

单分支：

```bash
if [ -f /etc/passwd ]; then
  echo "/etc/passwd 存在"
fi
```

双分支：

```bash
if systemctl is-active nginx >/dev/null 2>&1; then
  echo "nginx 正在运行"
else
  echo "nginx 未运行"
fi
```

多分支：

```bash
score=85

if [ "$score" -ge 90 ]; then
  echo "优秀"
elif [ "$score" -ge 60 ]; then
  echo "及格"
else
  echo "不及格"
fi
```

应用场景：

- 判断文件是否存在。
- 判断服务是否运行。
- 判断用户输入是否合法。

### 3.2 文件判断

| 表达式 | 含义 |
| --- | --- |
| `-f file` | 是否为普通文件 |
| `-d dir` | 是否为目录 |
| `-e path` | 路径是否存在 |
| `-r file` | 是否可读 |
| `-w file` | 是否可写 |
| `-x file` | 是否可执行 |

示例：备份前确认目录存在。

```bash
#!/bin/bash

src="/etc"
dst="/backup"

if [ ! -d "$dst" ]; then
  mkdir -p "$dst"
fi

tar -czf "$dst/etc-$(date +%F).tar.gz" "$src"
```

应用场景：

- 备份、发布、日志归档前检查路径。
- 防止脚本对不存在的文件执行操作。

### 3.3 整数比较

| 表达式 | 含义 |
| --- | --- |
| `-eq` | 等于 |
| `-ne` | 不等于 |
| `-gt` | 大于 |
| `-ge` | 大于等于 |
| `-lt` | 小于 |
| `-le` | 小于等于 |

示例：磁盘使用率告警。

```bash
#!/bin/bash

usage=$(df / | awk 'NR==2 {gsub("%","",$5); print $5}')

if [ "$usage" -ge 80 ]; then
  echo "磁盘使用率过高: ${usage}%"
else
  echo "磁盘使用率正常: ${usage}%"
fi
```

应用场景：

- 资源巡检。
- 判断端口数量、进程数量、日志大小。

### 3.4 字符串比较

| 表达式 | 含义 |
| --- | --- |
| `=` | 字符串相等 |
| `!=` | 字符串不等 |
| `-z str` | 字符串为空 |
| `-n str` | 字符串非空 |

示例：检查用户输入。

```bash
read -r -p "请输入服务名: " service

if [ -z "$service" ]; then
  echo "服务名不能为空"
  exit 1
fi

echo "准备检查服务: $service"
```

应用场景：

- 防止空变量导致误操作。
- 判断用户选择的环境、操作类型。

### 3.5 正则判断

在 Bash 中使用 `[[ string =~ regex ]]` 做正则匹配。

```bash
read -r -p "请输入端口号: " port

if [[ "$port" =~ ^[0-9]+$ ]] && [ "$port" -ge 1 ] && [ "$port" -le 65535 ]; then
  echo "端口合法: $port"
else
  echo "端口不合法"
fi
```

应用场景：

- 校验手机号、邮箱、IP、端口。
- 限制用户输入必须为数字。

### 3.6 case 基础语法

`case` 适合处理固定菜单或固定操作类型。

```bash
#!/bin/bash

case "${1:-}" in
  start)
    systemctl start nginx
    ;;
  stop)
    systemctl stop nginx
    ;;
  restart)
    systemctl restart nginx
    ;;
  status)
    systemctl status nginx
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
    exit 1
    ;;
esac
```

应用场景：

- 服务启停脚本。
- 菜单式工具箱。
- 根据参数选择不同任务。

## 4. 循环控制：for 与 while

### 4.1 for 循环

`for` 适合处理已知列表。

```bash
for user in alice bob carol; do
  echo "创建用户: $user"
  useradd "$user"
done
```

应用场景：

- 批量创建用户。
- 批量检查主机。
- 批量处理文件。

数字循环：

```bash
for i in {1..10}; do
  echo "$i"
done
```

### 4.2 从文件读取并循环

```bash
#!/bin/bash

while IFS= read -r username; do
  [ -z "$username" ] && continue
  id "$username" >/dev/null 2>&1 && {
    echo "$username 已存在"
    continue
  }
  useradd "$username"
  echo "已创建 $username"
done < users.txt
```

应用场景：

- 从清单文件批量创建用户。
- 从 IP 列表批量检查主机。
- 从域名列表批量检测解析。

### 4.3 while 循环

`while` 适合处理条件未知、需要持续判断的任务。

```bash
count=1

while [ "$count" -le 5 ]; do
  echo "第 $count 次执行"
  count=$((count + 1))
done
```

应用场景：

- 轮询服务状态。
- 等待端口启动。
- 交互式菜单持续运行。

示例：等待 nginx 启动。

```bash
#!/bin/bash

systemctl start nginx

retry=0
while [ "$retry" -lt 10 ]; do
  if systemctl is-active nginx >/dev/null 2>&1; then
    echo "nginx 已启动"
    exit 0
  fi
  retry=$((retry + 1))
  sleep 1
done

echo "nginx 启动超时"
exit 1
```

### 4.4 break、continue、exit

| 语句 | 作用 |
| --- | --- |
| `break` | 跳出当前循环 |
| `continue` | 跳过本次循环，进入下一次 |
| `exit` | 退出整个脚本 |

示例：

```bash
for i in {1..10}; do
  if [ "$i" -eq 3 ]; then
    continue
  fi

  if [ "$i" -eq 8 ]; then
    break
  fi

  echo "$i"
done
```

应用场景：

- 遇到非法数据跳过。
- 找到目标后停止搜索。
- 遇到严重错误直接退出脚本。

### 4.5 嵌套循环

示例：九九乘法表。

```bash
for i in {1..9}; do
  for j in $(seq 1 "$i"); do
    printf "%s*%s=%-2s " "$j" "$i" "$((i * j))"
  done
  echo
done
```

应用场景：

- IP 段和端口组合扫描。
- 多数据库、多表批量备份。
- 多目录、多文件批量处理。

## 5. 函数

### 5.1 函数基础语法

函数用于封装重复逻辑，提高脚本复用性。

```bash
log_info() {
  echo "[INFO] $(date '+%F %T') $*"
}

log_info "开始部署"
log_info "部署完成"
```

应用场景：

- 统一日志输出。
- 封装服务启停、备份、检查逻辑。
- 减少脚本重复代码。

### 5.2 函数参数

函数内部也可以使用 `$1`、`$2`、`$#` 接收参数。

```bash
check_service() {
  local service="$1"

  if systemctl is-active "$service" >/dev/null 2>&1; then
    echo "$service running"
  else
    echo "$service stopped"
  fi
}

check_service nginx
check_service sshd
```

应用场景：

- 写一个函数检查多个服务。
- 写一个函数备份多个目录。

### 5.3 函数返回值

函数返回值通常有两种：

- `echo` 返回文本数据。
- `return` 返回状态码，范围通常是 0-255。

示例：用 `return` 表示成功失败。

```bash
is_root() {
  [ "$(id -u)" -eq 0 ]
}

if is_root; then
  echo "当前是 root 用户"
else
  echo "请使用 root 执行"
  exit 1
fi
```

示例：用 `echo` 返回计算结果。

```bash
add() {
  echo $(("$1" + "$2"))
}

result=$(add 10 20)
echo "结果: $result"
```

应用场景：

- `return` 用于判断函数是否执行成功。
- `echo` 用于向主流程返回字符串、数字、路径。

### 5.4 菜单工具箱

```bash
#!/bin/bash

show_menu() {
  echo "1) 查看磁盘"
  echo "2) 查看内存"
  echo "3) 查看负载"
  echo "q) 退出"
}

while true; do
  show_menu
  read -r -p "请选择: " choice

  case "$choice" in
    1) df -h ;;
    2) free -h ;;
    3) uptime ;;
    q) exit 0 ;;
    *) echo "无效选择" ;;
  esac
done
```

应用场景：

- 运维巡检工具。
- 跳板机菜单。
- 常用任务入口。

## 6. 数组

### 6.1 普通数组

普通数组使用数字索引。

```bash
services=(nginx sshd crond)

echo "${services[0]}"
echo "${services[@]}"
echo "${#services[@]}"
```

应用场景：

- 保存服务列表。
- 保存主机列表。
- 保存待处理文件列表。

遍历数组：

```bash
for service in "${services[@]}"; do
  systemctl is-active "$service" >/dev/null 2>&1 \
    && echo "$service running" \
    || echo "$service stopped"
done
```

### 6.2 关联数组

关联数组使用字符串作为索引，需要先声明。

```bash
declare -A ports
ports[nginx]=80
ports[ssh]=22
ports[mysql]=3306

for name in "${!ports[@]}"; do
  echo "$name 使用端口 ${ports[$name]}"
done
```

应用场景：

- 服务名和端口映射。
- 用户名和角色映射。
- 统计日志中 IP 出现次数。

### 6.3 数组统计

示例：统计 `/etc/passwd` 中不同 Shell 的使用次数。

```bash
#!/bin/bash

declare -A shell_count

while IFS=: read -r user _ uid _ _ _ shell; do
  shell_count["$shell"]=$((shell_count["$shell"] + 1))
done < /etc/passwd

for shell in "${!shell_count[@]}"; do
  echo "$shell: ${shell_count[$shell]}"
done
```

应用场景：

- 统计日志 IP 次数。
- 统计状态码次数。
- 统计用户 Shell、进程类型、访问路径。

## 7. 正则表达式与 grep

### 7.1 正则表达式基础

常用规则：

| 符号 | 含义 |
| --- | --- |
| `^` | 行首 |
| `$` | 行尾 |
| `.` | 任意单个字符 |
| `*` | 前一个字符出现 0 次或多次 |
| `+` | 前一个字符出现 1 次或多次，通常配合 `grep -E` |
| `?` | 前一个字符出现 0 次或 1 次，通常配合 `grep -E` |
| `[]` | 字符集合 |
| `[^]` | 取反字符集合 |
| `{n,m}` | 重复次数，通常配合 `grep -E` |

### 7.2 grep 常用示例

过滤空行和注释：

```bash
grep -Ev '^\s*$|^\s*#' nginx.conf
```

提取 IP：

```bash
grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}' access.log
```

匹配邮箱：

```bash
grep -E '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' users.txt
```

应用场景：

- 查看配置文件有效行。
- 从日志中提取 IP、URL、状态码。
- 校验输入数据格式。

### 7.3 grep 常用选项

| 选项 | 含义 |
| --- | --- |
| `-n` | 显示行号 |
| `-i` | 忽略大小写 |
| `-v` | 反向匹配 |
| `-E` | 使用扩展正则 |
| `-o` | 只输出匹配内容 |
| `-r` | 递归搜索目录 |

示例：递归查找脚本中的 IP。

```bash
grep -RniE '([0-9]{1,3}\.){3}[0-9]{1,3}' /opt/scripts
```

应用场景：

- 排查代码或脚本中的硬编码地址。
- 快速定位配置项。

## 8. sed 文本处理

### 8.1 sed 基础

`sed` 是流式文本编辑器，适合按行过滤、替换、删除、追加内容。

基础格式：

```bash
sed [选项] '地址 动作' 文件
```

常用选项：

| 选项 | 含义 |
| --- | --- |
| `-n` | 取消默认输出，通常配合 `p` |
| `-i` | 直接修改文件 |
| `-E` | 使用扩展正则 |
| `-e` | 执行多个 sed 命令 |

### 8.2 sed 打印

```bash
# 打印第 10 行
sed -n '10p' /etc/passwd

# 打印第 10 到 20 行
sed -n '10,20p' /etc/passwd

# 打印 root 开头的行
sed -n '/^root/p' /etc/passwd
```

应用场景：

- 查看大文件指定范围。
- 从配置文件中提取指定段落。

### 8.3 sed 删除

```bash
# 删除空行
sed '/^\s*$/d' file.txt

# 删除注释行
sed '/^\s*#/d' nginx.conf

# 删除第 1 到 5 行
sed '1,5d' file.txt
```

应用场景：

- 清理配置文件。
- 删除无效数据。

### 8.4 sed 替换

```bash
# 替换每行第一个 old
sed 's/old/new/' file.txt

# 替换每行所有 old
sed 's/old/new/g' file.txt

# 原地修改配置
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```

应用场景：

- 修改配置项。
- 批量替换域名、IP、路径。
- 部署时渲染配置模板。

### 8.5 sed 追加与插入

```bash
# 第 3 行后追加
sed '3a append line' file.txt

# 第 3 行前插入
sed '3i insert line' file.txt
```

应用场景：

- 给配置文件增加一行配置。
- 在指定位置插入标记或说明。

### 8.6 sed 实践：统计 Ansible 主机组

假设 inventory：

```ini
[web]
10.0.0.11
10.0.0.12

[db]
10.0.0.21
```

提取组名：

```bash
sed -n 's/^\[\(.*\)\]$/\1/p' hosts
```

统计某个组的主机：

```bash
sed -n '/^\[web\]/,/^\[/p' hosts | grep -Ev '^\[|^$'
```

应用场景：

- 分析主机清单。
- 从配置文件提取某个配置块。

## 9. awk 文本处理

### 9.1 awk 基础

`awk` 适合按列处理文本。

基础格式：

```bash
awk '模式 {动作}' 文件
```

示例：

```bash
awk -F: '{print $1,$3}' /etc/passwd
```

应用场景：

- 提取列。
- 统计日志。
- 做条件筛选和格式化输出。

### 9.2 awk 常用内置变量

| 变量 | 含义 |
| --- | --- |
| `FS` | 输入字段分隔符 |
| `OFS` | 输出字段分隔符 |
| `RS` | 输入记录分隔符 |
| `ORS` | 输出记录分隔符 |
| `NR` | 当前总行号 |
| `NF` | 当前行字段数量 |
| `$0` | 当前整行 |
| `$1` | 第 1 列 |
| `$NF` | 最后一列 |

示例：

```bash
awk -F: '{print NR, $1, $NF}' /etc/passwd
```

应用场景：

- 查看用户及登录 Shell。
- 给输出内容添加行号。
- 获取最后一列数据。

### 9.3 awk 格式化输出

```bash
awk -F: 'BEGIN {printf "%-20s %-10s\n", "USER", "UID"} {printf "%-20s %-10s\n", $1, $3}' /etc/passwd
```

应用场景：

- 输出对齐的报表。
- 巡检脚本生成易读结果。

### 9.4 awk 条件判断

筛选 UID 大于等于 1000 的普通用户：

```bash
awk -F: '$3 >= 1000 {print $1, $3, $NF}' /etc/passwd
```

if 判断：

```bash
awk -F: '{
  if ($3 == 0) {
    print $1, "管理员"
  } else if ($3 >= 1000) {
    print $1, "普通用户"
  }
}' /etc/passwd
```

应用场景：

- 根据状态码、UID、金额、次数筛选数据。
- 给数据打标签。

### 9.5 awk 循环

```bash
awk 'BEGIN {
  for (i = 1; i <= 9; i++) {
    for (j = 1; j <= i; j++) {
      printf "%d*%d=%-2d ", j, i, i*j
    }
    print ""
  }
}'
```

应用场景：

- 处理一行中的多个字段。
- 生成简单报表。

### 9.6 awk 数组统计

统计 Nginx 访问日志 IP Top 10：

```bash
awk '{count[$1]++} END {for (ip in count) print count[ip], ip}' access.log \
  | sort -rn \
  | head -10
```

统计状态码：

```bash
awk '{code[$9]++} END {for (c in code) print c, code[c]}' access.log \
  | sort -k2 -rn
```

统计访问 URL 总流量：

```bash
awk '{size[$7]+=$10} END {for (url in size) print size[url], url}' access.log \
  | sort -rn \
  | head
```

应用场景：

- 日志访问排名。
- 统计异常状态码。
- 分析流量最大的接口或页面。

## 10. 综合实践案例

### 10.1 服务启停脚本

知识点：位置参数、case、函数、状态码。

```bash
#!/bin/bash

service_name="nginx"

usage() {
  echo "Usage: $0 {start|stop|restart|status}"
}

check_root() {
  if [ "$(id -u)" -ne 0 ]; then
    echo "请使用 root 用户执行"
    exit 1
  fi
}

check_root

case "${1:-}" in
  start)
    systemctl start "$service_name"
    ;;
  stop)
    systemctl stop "$service_name"
    ;;
  restart)
    systemctl restart "$service_name"
    ;;
  status)
    systemctl status "$service_name"
    ;;
  *)
    usage
    exit 1
    ;;
esac
```

应用场景：

- 为 Nginx、Rsync、LVS 等服务写统一控制脚本。
- 作为生产脚本模板，便于扩展检查和日志。

### 10.2 自动备份脚本

知识点：变量、文件判断、日期、函数。

```bash
#!/bin/bash

src_dir="/etc"
backup_dir="/backup"
keep_days=7
backup_file="$backup_dir/etc-$(date +%F_%H%M%S).tar.gz"

log() {
  echo "[$(date '+%F %T')] $*"
}

if [ ! -d "$src_dir" ]; then
  log "源目录不存在: $src_dir"
  exit 1
fi

mkdir -p "$backup_dir"
tar -czf "$backup_file" "$src_dir"

if [ $? -eq 0 ]; then
  log "备份成功: $backup_file"
else
  log "备份失败"
  exit 1
fi

find "$backup_dir" -type f -name 'etc-*.tar.gz' -mtime +"$keep_days" -delete
log "已清理 $keep_days 天前的旧备份"
```

应用场景：

- 定时备份配置文件。
- 配合 crontab 做周期任务。

### 10.3 主机存活探测

知识点：for 循环、命令状态码、文件读取。

```bash
#!/bin/bash

ip_file="hosts.txt"

if [ ! -f "$ip_file" ]; then
  echo "主机文件不存在: $ip_file"
  exit 1
fi

while IFS= read -r ip; do
  [ -z "$ip" ] && continue

  if ping -c 1 -W 1 "$ip" >/dev/null 2>&1; then
    echo "$ip up"
  else
    echo "$ip down"
  fi
done < "$ip_file"
```

应用场景：

- 巡检主机连通性。
- 批量确认服务器是否在线。

### 10.4 批量端口检测

知识点：数组、嵌套循环、超时控制。

```bash
#!/bin/bash

hosts=(10.0.0.11 10.0.0.12)
ports=(22 80 443)

for host in "${hosts[@]}"; do
  for port in "${ports[@]}"; do
    if timeout 1 bash -c ">/dev/tcp/$host/$port" 2>/dev/null; then
      echo "$host:$port open"
    else
      echo "$host:$port closed"
    fi
  done
done
```

应用场景：

- 发布前确认服务端口。
- 快速排查网络访问问题。

### 10.5 日志分析脚本

知识点：参数、文件判断、awk 数组、排序。

```bash
#!/bin/bash

log_file="${1:-/var/log/nginx/access.log}"

if [ ! -f "$log_file" ]; then
  echo "日志文件不存在: $log_file"
  exit 1
fi

echo "访问 IP Top 10:"
awk '{ip[$1]++} END {for (i in ip) print ip[i], i}' "$log_file" | sort -rn | head -10

echo
echo "状态码统计:"
awk '{code[$9]++} END {for (c in code) print c, code[c]}' "$log_file" | sort -k2 -rn

echo
echo "404 URL Top 10:"
awk '$9 == 404 {url[$7]++} END {for (u in url) print url[u], u}' "$log_file" | sort -rn | head -10
```

应用场景：

- 分析 Nginx/Apache 访问日志。
- 定位异常 IP、404 页面、高频访问接口。

## 11. 学习建议与排错清单

### 11.1 推荐学习顺序

1. 先掌握脚本格式、变量、参数、退出状态。
2. 再学习 `if`、`case`、`for`、`while`。
3. 用函数把重复逻辑封装起来。
4. 用数组处理列表和统计。
5. 学会 `grep`、`sed`、`awk` 后再做日志分析和配置处理。
6. 最后组合成备份、巡检、部署、分析类脚本。

### 11.2 常见错误

| 问题 | 原因 | 修复 |
| --- | --- | --- |
| `command not found` | 命令不存在或 PATH 不正确 | 用 `which command` 检查 |
| 变量为空 | 变量名写错或未赋值 | 用 `set -u` 或 `echo "$var"` 调试 |
| 判断表达式报错 | `[ ]` 两边缺少空格 | 写成 `[ "$a" = "$b" ]` |
| 文件路径有空格导致失败 | 变量没加引号 | 使用 `"$file"` |
| 脚本无权限 | 没有执行权限 | `chmod +x script.sh` |
| Windows 换行导致报错 | 文件是 CRLF | `dos2unix script.sh` |

### 11.3 调试方法

```bash
# 检查语法
bash -n script.sh

# 显示执行过程
bash -x script.sh

# 在脚本中开启调试
set -x
command1
command2
set +x
```

### 11.4 写生产脚本的最低要求

- 关键变量集中放在脚本开头。
- 所有路径变量都加双引号。
- 高风险操作前判断变量是否为空。
- 删除、覆盖、重启服务前加确认或明确条件。
- 重要步骤输出日志。
- 对命令执行结果做判断。
- 能用函数封装的重复逻辑不要复制多份。


