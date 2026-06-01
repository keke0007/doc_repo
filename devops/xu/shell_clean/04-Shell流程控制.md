# 04. Shell 流程控制

> 整理 `if` / `case` / `expect` 三类流程控制结构,并订正原文档错误。
> 涉及多文件调用的场景(批量分发秘钥)给出 ASCII 执行流程图。

---

## 一、知识体系

```
流程控制
├── 1. if 语句
│   ├── 单分支
│   ├── 双分支 (if-else)
│   └── 多分支 (if-elif-else)
├── 2. case 语句
└── 3. expect 交互脚本
    ├── 基础交互
    ├── 变量化
    ├── 参数传递
    └── 配合 shell 批量分发秘钥
```

---

## 二、if 流程控制

### 1. 单分支

```bash
if 条件; then
    命令序列
fi
```

### 2. 双分支

```bash
if 条件; then
    成立时执行
else
    不成立执行
fi
```

### 3. 多分支

```bash
if 条件1; then
    ...
elif 条件2; then
    ...
elif 条件3; then
    ...
else
    ...
fi
```

### 4. 综合案例:根据 CentOS 版本配置 YUM 源

```bash
#!/usr/bin/bash
os_name=$(cat /etc/redhat-release)
os_ver=$(awk '{for(i=1;i<=NF;i++) if($i ~ /^[0-9]+\./) {split($i,a,"."); print a[1]; exit}}' /etc/redhat-release)

if [ "$os_ver" -eq 7 ]; then
    mkdir -p /etc/yum.repos.d/backup
    mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/backup/ 2>/dev/null
    cat >/etc/yum.repos.d/base.repo <<-EOF
        [base]
        name=Local Base Yum Source
        baseurl=ftp://192.168.56.1/base/7/x86_64
        enabled=1
        gpgcheck=0
        EOF
    echo "$os_name 已配置 yum 源"

elif [ "$os_ver" -eq 6 ]; then
    wget -O /etc/yum.repos.d/CentOS-Base.repo \
         http://mirrors.aliyun.com/repo/Centos-6.repo &>/dev/null

else
    echo "未知系统版本,请检查 /etc/redhat-release"
fi
```

---

## 三、case 流程控制

### 1. 语法

```bash
case 变量 in
    模式1)
        命令序列1
        ;;
    模式2|模式2b)
        命令序列2
        ;;
    *)
        默认命令序列
        ;;
esac
```

> 关键点:每个分支以 `;;` 结尾;模式支持 `|` 多匹配、`*` 通配。

### 2. 服务启停脚本骨架(典型 init.d 风格)

```bash
#!/usr/bin/bash
# Manager nginx start|stop|restart|reload|status

start() { /usr/sbin/nginx;            echo "Nginx started"; }
stop()  { /usr/sbin/nginx -s stop;    echo "Nginx stopped"; }
reload(){ /usr/sbin/nginx -s reload;  echo "Nginx reloaded"; }
status(){
    pid=$(pgrep -f 'nginx: master')
    [ -n "$pid" ] && echo "Running (pid=$pid)" || echo "Stopped"
}

case "$1" in
    start)   start ;;
    stop)    stop ;;
    restart) stop; sleep 1; start ;;
    reload)  reload ;;
    status)  status ;;
    *)       echo "Usage: $0 {start|stop|restart|reload|status}"; exit 1 ;;
esac
```

### 3. 简易跳板机 (JumpServer)

```bash
#!/usr/bin/bash
trap "" HUP INT TSTP        # 屏蔽 Ctrl+C / Ctrl+Z

declare -A HOSTS=(
    [1]=192.168.70.160   # mysql-master
    [2]=192.168.70.161   # mysql-slave1
    [3]=192.168.70.162   # mysql-slave2
)

menu(){
    cat <<-EOF
        ============================
        1) mysql-master
        2) mysql-slave1
        3) mysql-slave2
        h) help
        q) quit
        ============================
EOF
}

menu
while true; do
    read -p "请输入要连接的主机编号:" num
    case "$num" in
        1|2|3) ssh "root@${HOSTS[$num]}" ;;
        h|help) menu ;;
        q|quit) break ;;
        *)      echo "无效输入";;
    esac
done
```

---

## 四、expect 交互脚本

`expect` 用于在 SSH 登录、passwd 等**交互式命令**中实现自动应答。

### 1. 最简交互登录

```expect
#!/usr/bin/expect
spawn ssh root@192.168.70.161
expect {
    "yes/no"   { send "yes\r"; exp_continue }
    "password:"{ send "centos\r" }
}
interact
```

### 2. 关键命令

| 命令 | 含义 |
|------|------|
| `spawn cmd` | 启动一个新进程并接管其输入输出 |
| `expect "pat"` | 等待匹配字符串出现 |
| `send "..."` | 发送内容(`\r` 表示回车) |
| `exp_continue` | 继续匹配剩余 expect 分支 |
| `interact` | 把控制权交还给用户(可继续手敲) |
| `expect eof` | 等待子进程结束 |
| `set var val` | 定义变量 |
| `[lindex $argv N]` | 取第 N 个位置参数 |

### 3. 参数化版本

```expect
#!/usr/bin/expect
set ip       [lindex $argv 0]
set user     root
set password centos
set timeout  5

spawn ssh $user@$ip
expect {
    "yes/no"   { send "yes\r"; exp_continue }
    "password:"{ send "$password\r" }
}
expect "#"
send "useradd bgx\r"
send "exit\r"
expect eof
```

---

## 五、Shell + expect 批量分发秘钥(多文件调用)

**场景**:用 shell 探测在线主机列表,把列表交给 expect 完成 `ssh-copy-id`。

### 1. 涉及的文件

| 文件 | 作用 |
|------|------|
| `for_ip.sh` | 探测在线主机 → 写入 `ip.txt` |
| `ip.txt` | 中转的主机清单 |
| `ssh_copy.sh` | 读 `ip.txt`,内嵌 expect 完成秘钥分发 |
| `~/.ssh/id_rsa(.pub)` | 本机秘钥对 |

### 2. 执行流程图

```
┌─────────────┐   并发 ping       ┌────────────┐
│ for_ip.sh   │ ───────────────▶  │   ip.txt   │  在线主机清单
└──────┬──────┘                   └─────┬──────┘
       │ wait                           │
       ▼                                ▼
┌────────────────────────────────────────────────────────────┐
│ ssh_copy.sh                                                │
│   ├─ 若 ~/.ssh/id_rsa 不存在 → ssh-keygen -P "" 生成秘钥对  │
│   └─ while read line < ip.txt                              │
│        └─ 内嵌 /usr/bin/expect <<EOF                       │
│             ├─ spawn ssh-copy-id $line                     │
│             ├─ expect "yes/no"   → send "yes\r"            │
│             └─ expect "password:" → send "$password\r"     │
└────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  目标主机的 ~/.ssh/authorized_keys
                  追加本机公钥 → 完成免密
```

### 3. 完整脚本

**for_ip.sh**

```bash
#!/usr/bin/bash
> ip.txt
for i in {160..162}; do
    ip=192.168.70.$i
    {
        if ping -c1 -W1 "$ip" &>/dev/null; then
            echo "$ip" >> ip.txt
        fi
    } &
done
wait
```

**ssh_copy.sh**

```bash
#!/usr/bin/bash
if [ ! -f ~/.ssh/id_rsa ]; then
    ssh-keygen -P "" -f ~/.ssh/id_rsa
fi

password=centos
while read line; do
    /usr/bin/expect <<-EOF
        set timeout 5
        spawn ssh-copy-id $line
        expect {
            "yes/no"   { send "yes\r"; exp_continue }
            "password:"{ send "$password\r" }
        }
        expect eof
    EOF
done < ip.txt
```

---

## 六、原文档错误订正

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | yum 仓库示例 `enable=1` | 选项名错误 | `enabled=1` |
| 2 | `echo "###########` | 引号未闭合 | `echo "###########"` |
| 3 | `q) break` 后直接 `*` 而无 `;;` | case 缺分隔符 | 每个分支以 `;;` 结束 |
| 4 | `if [ $? -ne 0 ]; then echo "..." exit 1` | 缺 `fi` | 补上 `fi` |
| 5 | jumpserver 中用 `exec)` 作为退出关键字 | 与内建 `exec` 命令同名易混淆 | 改为 `q)` 或 `quit)` |
| 6 | `[root@Shell day03]# cat /home/alex/.bashrc sh /home/alex/jumpserver.sh` | 命令拼接错乱 | 应在 `~/.bashrc` 末尾追加 `sh /home/alex/jumpserver.sh` |
| 7 | expect 中 `interact` 与脚本退出后命令同写 | 语法位置不正确 | `interact` 必须在 expect 块最后,且不与 `expect eof` 混用 |
| 8 | `os_version=$(cat /etc/redhat-release \|awk '{print $4}'...)` 写两次 | 重复且 logic 混乱 | 用一次性 awk 提取主版本号即可 |
