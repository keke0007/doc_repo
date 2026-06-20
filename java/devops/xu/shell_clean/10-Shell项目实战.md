# 10. Shell 项目实战

> 整理主机存活检测、MySQL/LNMP 部署、服务器初始化、监控主控脚本等典型工程化场景,
> 并对涉及**多文件协作**的脚本绘制 ASCII 执行流程图,订正原文档错误。

---

## 一、典型项目结构

```
shell_project/
├── monitor_main.sh        # 主控,菜单调度各子脚本
├── system_info.sh         # 系统信息采集
├── nginx_status.sh        # Nginx 状态采集
├── mysql_slave_check.sh   # MySQL 主从检测
├── loginfo.sh             # 日志状态码分析
├── lib/
│   └── common.sh          # 公共函数库
└── config/
    └── env.sh             # 环境变量集中存放
```

---

## 二、主机存活检测(三次重试)

```bash
#!/usr/bin/bash
SUBNET="192.168.70"

check_host() {
    local ip=$1
    for try in 1 2 3; do
        if ping -c1 -W1 "$ip" &>/dev/null; then
            echo "$ip 第 $try 次 连接成功"
            return 0
        fi
        echo "$ip 第 $try 次 失败"
    done
    return 1
}

for i in {150..170}; do
    check_host "${SUBNET}.$i" &           # 并发探活
done
wait
```

**订正**:原文用了**两层嵌套循环且使用相同的循环变量 `$i`**,内层会覆盖外层:

```bash
# 错误示例(节选)
for i in {150..170}; do
    IP_UP=$IP.$i
    for i in {1..3}; do     # ← 覆盖了外层 i
        ...
    done
done
```

正确做法:**外层 / 内层使用不同变量名**,或封装成函数(如上)。原文也少了 `wait`。

---

## 三、MySQL 5.7 一键部署

```bash
#!/usr/bin/bash
set -e

install_mysql_yum() {
    if rpm -q mysql-community-server &>/dev/null; then
        echo "MySQL 已安装: $(rpm -q mysql-community-server)"
        return
    fi
    cat >/etc/yum.repos.d/mysql.repo <<-EOF
        [mysql57-community]
        name=MySQL 5.7 Community Server
        baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/x86_64
        enabled=1
        gpgcheck=0
EOF
    yum clean all && yum makecache fast
    yum install -y mysql-community-server
}

init_mysql() {
    systemctl enable --now mysqld
    local old_pwd
    old_pwd=$(awk -F': ' '/temporary password/{print $NF; exit}' /var/log/mysqld.log)
    mysqladmin -uroot -p"$old_pwd" password "Xuliangwei.com123"
}

install_mysql_yum
init_mysql
```

---

## 四、服务器初始化脚本(关键项一览)

| 步骤 | 关键命令 / 操作 |
|------|-----------------|
| 1. 配置 yum 源 | 替换 base.repo,使用阿里云 / 内网 |
| 2. 安装基础工具 | `yum install -y vim nc wget telnet net-tools lsof bash-completion` |
| 3. 调整文件描述符 | 编辑 `/etc/security/limits.conf`(`* soft/hard nofile 65535`) |
| 4. 调整时区 | `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime` |
| 5. 调整语言 | `sed -i 's#LANG=.*#LANG="en_US.UTF-8"#' /etc/locale.conf` |
| 6. 时间同步 | `chronyd` / `ntpdate` |
| 7. 防火墙 / SELinux | `systemctl disable --now firewalld`;`setenforce 0` + `sed -ri 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config` |
| 8. 关闭 IPv6 | `/etc/modprobe.d/ipv6.conf` 写 `alias net-pf-10 off / alias ipv6 off` |
| 9. 历史命令带时间戳 | `HISTTIMEFORMAT="%F %T "` 写入 `/etc/profile` |
| 10. 内核参数 | `/etc/sysctl.conf` 设 `net.ipv4.tcp_tw_reuse=1` 等 |

> **注意**:`tcp_tw_recycle` 在新内核已废弃;NAT 后客户端易被丢包,**生产不建议开启**。`net.ipv4.tcp_tw_reuse=1` 仅对客户端 TIME_WAIT 复用有效。

---

## 五、主控脚本(monitor_main.sh)

### 1. 设计思路

- 自动扫描当前目录中除自身以外的 `*.sh`
- 列出编号菜单,用户输入编号即调用对应脚本
- 用关联数组保存 编号 → 脚本文件名

### 2. 多文件调度执行流程图

```
              ┌────────────────────────────┐
              │ monitor_main.sh (主控)     │
              │  ┌──────────────────────┐  │
              │  │ ls *.sh 扫描目录     │  │
              │  └──────────┬───────────┘  │
              │             ▼              │
              │  ┌──────────────────────┐  │
              │  │ ssharray[i]=file     │  │
              │  │ 拼接菜单 numbers     │  │
              │  └──────────┬───────────┘  │
              │             ▼              │
              │  ┌──────────────────────┐  │
              │  │ while true           │  │
              │  │   read execshell     │  │
              │  │   bash ${ssharray[execshell]}
              │  └──────────┬───────────┘  │
              └─────────────┼──────────────┘
                            │ fork & exec
        ┌───────────────────┼───────────────────┬───────────────┐
        ▼                   ▼                   ▼               ▼
 system_info.sh    nginx_status.sh    mysql_slave_check.sh   loginfo.sh
 (采集主机信息)    (Nginx 状态)       (主从复制状态)         (HTTP 状态码)
```

### 3. 正确版主控

```bash
#!/usr/bin/bash
SELF=$(basename "$0")
declare -A SCRIPTS
RESET=$(tput sgr0)

i=0
nums=""
for f in $(ls *.sh 2>/dev/null); do
    [ "$f" = "$SELF" ] && continue
    echo -e "\e[1;35m  [$i]${RESET} $f"
    SCRIPTS[$i]="$f"
    nums="${nums}${nums:+|}$i"
    i=$(( i + 1 ))
done

while true; do
    read -p "请选择 [$nums] (q 退出): " sel
    [ "$sel" = "q" ] && break
    if [[ ! "$sel" =~ ^[0-9]+$ ]] || [ -z "${SCRIPTS[$sel]}" ]; then
        echo "无效选项"; continue
    fi
    bash "./${SCRIPTS[$sel]}"
done
```

**原文档错误**:

| 原文 | 问题 | 修正 |
|------|------|------|
| `numbers="` 引号未闭合 | 语法错 | `numbers=""` |
| `ls -I "montor_main.sh"` | 拼写 | `monitor_main.sh` |
| `i=$(i+1))` | 括号错配 | `i=$(( i + 1 ))` |
| `if [[ ! ${execshell} =~ ^[0-9]+ ]];then exit` | 输入字符就退出,体验差 | 应 `continue` 让用户重输 |

---

## 六、系统信息采集(system_info.sh)

```bash
#!/usr/bin/bash
RESET=$(tput sgr0)
line() { echo -e "\e[1;35m $1${RESET} $2"; }

line "OS 版本-->"   "$(hostnamectl | awk -F': ' '/Operating System/ {print $2}')"
line "OS 内核-->"   "$(uname -r)"
line "OS 平台-->"   "$(uname -m)"
line "主机名-->"    "$(hostname)"
line "内网 IP-->"   "$(hostname -I)"
line "外网 IP-->"   "$(curl -s --max-time 3 ifconfig.me || echo 'N/A')"
line "DNS-->"       "$(awk '/^nameserver/{print $2}' /etc/resolv.conf | xargs)"
line "网络-->"      "$(ping -c1 -W2 223.5.5.5 &>/dev/null && echo Online || echo Offline)"
line "登录用户-->"   "$(who | wc -l)"
line "空闲内存-->"   "$(free -h | awk '/^Mem:/{print $7}')"
line "系统负载-->"   "$(uptime | awk -F'load average:' '{print $2}')"
line "磁盘使用-->"
df -hT | awk '$2!~/tmpfs|devtmpfs/'
```

**原文档错误**:`awk "/nameserver/{print $2}"` 用了**双引号**,shell 会先把 `$2` 展开为空,结果是错的;必须用单引号。

---

## 七、Nginx + MySQL 主从监控

### 1. MySQL 主从检测要点

```
┌─────────────────────────────────────────┐
│ mysql_slave_check.sh                    │
│   1) nc -z -w2 IP 3306 探活             │
│   2) mysql -h -u -p -e "SHOW SLAVE..."  │
│   3) 解析 Slave_IO_Running              │
│       Slave_SQL_Running                 │
│       Seconds_Behind_Master             │
│   4) 异常 → mail 告警                   │
└─────────────────────────────────────────┘
```

```bash
check_slave() {
    local host=$1
    if ! nc -z -w2 "$host" 3306; then
        echo "$host:3306 不可达"; return 1
    fi
    out=$(mysql -h"$host" -u"$REP_USER" -p"$REP_PASS" -e "show slave status\G" 2>/dev/null)
    [ -z "$out" ] && { echo "$host 是主库"; return; }
    io=$(echo "$out"  | awk '/Slave_IO_Running:/{print $2}')
    sql=$(echo "$out" | awk '/Slave_SQL_Running:/{print $2}')
    lag=$(echo "$out" | awk '/Seconds_Behind_Master:/{print $2}')
    if [[ "$io" == "Yes" && "$sql" == "Yes" && "$lag" -lt 60 ]]; then
        echo "$host OK lag=${lag}s"
    else
        echo "$host FAIL io=$io sql=$sql lag=$lag" | mail -s "MySQL Alert" me@x.com
    fi
}
```

---

## 八、Nginx 日志状态码分析(loginfo.sh)

```bash
#!/usr/bin/bash
LOG=/var/log/nginx/access.log
RESET=$(tput sgr0)

read -r i1 i2 i3 i4 i5 total < <(
    awk '{
        c=$9
        if      (c>=100 && c<200) i++
        else if (c>=200 && c<300) j++
        else if (c>=300 && c<400) k++
        else if (c>=400 && c<500) n++
        else if (c>=500)           p++
    } END { print (i?i:0), (j?j:0), (k?k:0), (n?n:0), (p?p:0), i+j+k+n+p }' "$LOG"
)

printf "\e[1;35m HTTP 1xx-->\e[0m %d\n" "$i1"
printf "\e[1;35m HTTP 2xx-->\e[0m %d\n" "$i2"
printf "\e[1;35m HTTP 3xx-->\e[0m %d\n" "$i3"
printf "\e[1;35m HTTP 4xx-->\e[0m %d\n" "$i4"
printf "\e[1;35m HTTP 5xx-->\e[0m %d\n" "$i5"
printf "\e[1;35m TOTAL  -->\e[0m %d\n" "$total"
```

**HTTP 状态码大类**:

| 区段 | 含义 |
|------|------|
| 1xx | 信息,请求被接收,继续处理 |
| 2xx | 成功 |
| 3xx | 重定向 |
| 4xx | 客户端错误 |
| 5xx | 服务器错误 |

---

## 九、整体执行流程图

```
                           ┌──────────────────────┐
                           │ monitor_main.sh      │
                           │ (用户菜单交互)         │
                           └──────────┬───────────┘
                                      │
        ┌──────────────┬──────────────┼──────────────┬────────────────┐
        ▼              ▼              ▼              ▼                ▼
 system_info.sh   nginx_status.sh  mysql_slave   loginfo.sh    server_init.sh
 收集主机/网络/   端口/进程/        IO 线程/      HTTP 状态码    初始化基础配置
 磁盘/负载       连接数             SQL 线程/     PV/UV 统计
                                    主从延迟

                                      │
                                      ▼
                              邮件 / 钉钉告警
                              (mail / curl webhook)
```

---

## 十、整体错误订正汇总(原文档)

| # | 原文 | 问题 | 正确 |
|---|------|------|------|
| 1 | 嵌套 for 都用 `$i` | 内层覆盖外层 | 改名或封装函数 |
| 2 | 并发探活缺 `wait` | 主进程提前退出 | 循环 done 后加 `wait` |
| 3 | `numbers="` 引号未闭合 | 语法错 | `numbers=""` |
| 4 | `ls -I "montor_main.sh"` | 拼写错 | `monitor_main.sh` |
| 5 | `i=$(i+1))` | 括号错配 | `i=$(( i + 1 ))` |
| 6 | `awk "/nameserver/{print $2}"` | 双引号导致 `$2` 被 shell 展开 | 用单引号 |
| 7 | `ipv6.conf` 写法 `cd ... && touch ...; cat>>...` | 顺序冗余 | 直接 `cat >/etc/modprobe.d/ipv6.conf <<EOF ... EOF` |
| 8 | `net.ipv4.tcp_tw_recycle=0` 同时配 `reuse=1` | 新内核已删 recycle | 仅留 `tcp_tw_reuse=1`,删除 recycle 行 |
| 9 | loginfo 数组解析 `$(cat $LogFile \| egrep ... awk ...)` 写法被 OCR 破坏 | 多处 `\(...\)` 噪声 | 见本篇 loginfo.sh 重写版 |
| 10 | `HISTTIMEFORMAT` 注释"修改为 100000" 但代码改的是时间格式 | 注释与代码不符 | 历史**条数**应改 `HISTSIZE=100000` |
