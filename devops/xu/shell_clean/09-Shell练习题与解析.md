# 09. Shell 练习题与解析

> 整理 20 道经典 Shell 练习题,给出正确解法,并订正原文档中代码的错误。

---

## 一、按时间生成日志文件

```bash
#!/usr/bin/bash
df -h > "$(date +%F).log"
```

> 原文档无错。

---

## 二、统计 Nginx 日志每个 IP 的访问量

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head
```

套路:**取值 → 排序 → 去重计数 → 再排序**(`uniq -c` 前必须先 `sort`)。

---

## 三、计算所有进程占用内存之和(单位 MB)

```bash
#!/usr/bin/bash
sum=0
while read -r rss; do
    sum=$(( sum + rss ))
done < <(ps -eo rss=)

echo "$(( sum / 1024 )) MB"
```

**订正**:原文使用 `num=$(($i+$num)]` 和 `num=$($num/1024]` 都是括号错配,正确为 `$(( ... ))`。

---

## 四、批量改名 + 打包 + 还原

```bash
#!/usr/bin/bash
# 1. 找到所有 .txt 改名 .txt.bak
find /backup/ -type f -iname "*.txt" -print0 | \
    while IFS= read -r -d '' f; do mv "$f" "$f.bak"; done

# 2. 打包
cd /backup/ && tar czf 123.tar.gz ./*.bak

# 3. 还原(把 .bak 去掉)
find /backup/ -type f -iname "*.bak" -print0 | \
    while IFS= read -r -d '' f; do mv "$f" "${f%.*}"; done
```

**订正**:原文 `find ... | xargs >/backup/txt.tt` 中 `xargs` 没有跟命令,等价于"什么也不做",应去掉 `xargs`。

---

## 五、监控 80 端口,异常则重启 Nginx

```bash
#!/usr/bin/bash
while true; do
    if ! ss -lnt | awk '{print $4}' | grep -q ':80$'; then
        if pgrep -x nginx &>/dev/null; then
            /usr/sbin/nginx -s reload
        else
            /usr/sbin/nginx
        fi
        # 发邮件
        echo "nginx restarted at $(date)" | mail -s "Nginx Alert" me@x.com
    fi
    sleep 60
done
```

---

## 六、监控 502,触发重启 php-fpm

```bash
#!/usr/bin/bash
while true; do
    sleep 3
    cnt=$(tail -n 300 /var/log/nginx/access.log | awk '$9=="502"' | wc -l)
    if [ "$cnt" -ge 100 ]; then
        systemctl restart php-fpm
        sleep 5
    fi
done
```

**订正**:原文 `wc -1` 是错的,应为 `wc -l`。

---

## 七、打印一句话中字母数 < 6 的单词

```bash
#!/usr/bin/bash
for w in Bash also interprets a number of multi-user options; do
    # 注意:echo 末尾会带 \n,所以 wc -c 比真实长度多 1
    if [ "$(echo -n "$w" | wc -c)" -lt 6 ]; then
        echo "$w"
    fi
done
```

要点:`echo -n` 去掉末尾换行,使 `wc -c` 即为字符数;或使用 `${#w}` 直接取长度:

```bash
[ "${#w}" -lt 6 ] && echo "$w"
```

---

## 八、批量创建 user_00 ~ user_09,带随机密码并记录

```bash
#!/usr/bin/bash
[ "$EUID" -ne 0 ] && { echo "请使用 root 执行"; exit 1; }

for i in $(seq -w 0 9); do
    user="user_${i}"
    pass=$(mkpasswd -l 10 -C 3 -c 3 -d 2 -s 2)   # yum install expect
    if id "$user" &>/dev/null; then
        echo "$user already exists" >&2
        continue
    fi
    useradd "$user" && echo "$pass" | passwd --stdin "$user" &>/dev/null
    echo -e "${user}\t${pass}" >> /tmp/user.txt
done
```

---

## 九、统计普通用户数量(UID ≥ 1000)

```bash
#!/usr/bin/bash
total=0
while IFS=: read -r name _ uid _; do
    if [ "$uid" -ge 1000 ] && [ "$uid" -ne 65534 ]; then    # 排除 nobody
        echo "${name}:${uid}"
        total=$(( total + 1 ))
    fi
done < /etc/passwd
echo "总共普通用户: $total"
```

**订正**:

- `awk -F ":" '{print "$3}'` 引号写错,应为 `awk -F: '{print $3}'`。
- nobody 通常 UID 是 65534,严格统计可排除。

---

## 十、磁盘 / inode 使用率监控告警

```bash
#!/usr/bin/bash
date_tag=$(date +%F)
log=/tmp/${date_tag}_disk.log
: > "$log"

df -h     | awk 'NR>1{gsub("%","",$5); print "USE",   $6, $5}' >> "$log"
df -i     | awk 'NR>1{gsub("%","",$5); print "INODE", $6, $5}' >> "$log"

awk '$3+0 >= 85 {print}' "$log" | mail -s "Disk Alert" me@x.com
```

**订正**:原文 `if [ $Disk_Inode -ge 0 ]` 永远成立,应为 `-ge 85`。

---

## 十一、最常用命令 Top10

```bash
history | awk '{print $4}' | sort | uniq -c | sort -rn | head
# 或:
awk '{print $1}' ~/.bash_history | sort | uniq -c | sort -rn | head
```

> 注:`history` 的列号取决于是否启用了 `HISTTIMEFORMAT`,启用时命令在第 4 列,未启用在第 2 列。

---

## 十二、判断 80 端口跑的是 Nginx 还是 Httpd

```bash
#!/usr/bin/bash
n=$(ss -lntp | awk '$4 ~ /:80$/ {print $0}' | grep -c nginx)
h=$(ss -lntp | awk '$4 ~ /:80$/ {print $0}' | grep -c httpd)

if   [ "$n" -gt 0 ];          then echo "Nginx"
elif [ "$h" -gt 0 ];          then echo "Httpd"
elif [ "$n$h" = "00" ];       then echo "无 80 端口服务"; exit 1
else                                echo "未知服务";        exit 2
fi
```

---

## 十三、检测 MySQL 服务与主从状态

```bash
#!/usr/bin/bash
USER=remote
PASS=Bgx123.com
HOSTS=(192.168.70.160 192.168.70.161)

for ip in "${HOSTS[@]}"; do
    echo "== checking $ip =="
    out=$(mysql -h"$ip" -u"$USER" -p"$PASS" -e "show slave status\G" 2>/dev/null \
          | grep -E 'Slave_IO_Running|Slave_SQL_Running')
    if [ -z "$out" ]; then
        echo "$ip 是主库"; continue
    fi
    io=$(echo "$out"  | awk '/Slave_IO_Running:/{print $2}')
    sql=$(echo "$out" | awk '/Slave_SQL_Running:/{print $2}')
    if [[ "$io" == "Yes" && "$sql" == "Yes" ]]; then
        echo "$ip 主从 OK"
    else
        echo "$ip 主从异常 IO=$io SQL=$sql"
        echo "$out" | mail -s "$(date +%F) $ip slave fail" me@x.com
    fi
done
```

`/etc/mail.rc` 配置(发件邮箱示例):

```
set from=foo@qq.com
set smtp=smtps://smtp.qq.com:465
set smtp-auth-user=foo@qq.com
set smtp-auth-password=授权码
set smtp-auth=login
set ssl-verify=ignore
```

---

## 十四、1 ~ 100 中被 3 整除的整数之和

```bash
#!/usr/bin/bash
sum=0
for i in {1..100}; do
    (( i % 3 == 0 )) && sum=$(( sum + i ))
done
echo "$sum"      # 1683
```

---

## 十五、输入网卡名,输出 IP

```bash
#!/usr/bin/bash
read -p "请输入网卡名称: " nic
if ! ip link show "$nic" &>/dev/null; then
    echo "网卡 $nic 不存在"; exit 1
fi
ip=$(ip -4 addr show "$nic" | awk '/inet /{print $2; exit}')
[ -z "$ip" ] && echo "$nic 未分配 IP" || echo "$nic IP: $ip"
```

**订正**:原文 `echo $$` 输出当前 PID,与判断逻辑无关,应为 `echo $?`;`$Network` 与 `NetWork` 大小写不一致。

---

## 十六、猜数字游戏(0-100)

```bash
#!/usr/bin/bash
target=$(( RANDOM % 101 ))      # 真正的 0-100 随机
while true; do
    read -p "猜数字 (0-100): " n
    if [[ ! "$n" =~ ^[0-9]+$ ]]; then
        echo "请输入数字"; continue
    fi
    if   [ "$n" -lt "$target" ]; then echo "小了"
    elif [ "$n" -gt "$target" ]; then echo "大了"
    else                              echo "猜中,正确数字 $target"; exit 0
    fi
done
```

**订正**:原文 `n=$(echo $RANDOM | cut -c 1)` 只取首字符,无法生成 0-100;应用 `$(( RANDOM % 101 ))`。

---

## 十七、判断用户是否登录

```bash
#!/usr/bin/bash
read -p "请输入要查询的用户: " user
if who | awk '{print $1}' | grep -qw "$user"; then
    who | awk -v u="$user" '$1==u {print $1, $2, $5}'
else
    echo "用户未登录"
fi
```

---

## 十八、判断 httpd / mysql 是否安装并启动

```bash
#!/usr/bin/bash
for svc in httpd mysql-community-server; do
    if ! rpm -q "$svc" &>/dev/null; then
        echo "安装 $svc ..."
        yum install -y "$svc"
    fi
    if ! systemctl is-active --quiet "${svc%%-*}"; then
        systemctl start "${svc%%-*}" && echo "$svc 启动成功" || echo "$svc 启动失败"
    fi
done
```

**订正**:原文 `if [ $MySQL -eq 0$ ]` 多了 `$`,应为 `if [ "$MySQL" -eq 0 ]`。

---

## 十九、curl 状态码判断网站可用

```bash
#!/usr/bin/bash
[ $# -ne 1 ] && { echo "Usage: $0 url"; exit 1; }

code=$(curl -o /dev/null -s -w "%{http_code}" "$1")
case "$code" in
    200|301|302) echo "$1 OK ($code)" ;;
    *)           echo "$1 ERR ($code)" ;;
esac
```

> 用 `-w "%{http_code}"` 直接拿状态码比解析头更稳。

---

## 二十、Nginx 日志中 10-12 点访问最多的 IP

```bash
grep "22/May/2018:1[0-2]:[0-5][0-9]" access.log \
    | awk '{print $1}' \
    | sort | uniq -c | sort -rn | head -1
```

---

## 二十一、总错误清单(原文档 vs 正确)

| # | 原文 | 问题 | 正确 |
|---|------|------|------|
| 1 | `num=$(($i+$num)]` | 括号错配 | `num=$(( i + num ))` |
| 2 | `num=$($num/1024]` | 括号错配 | `num=$(( num / 1024 ))` |
| 3 | `find ... \| xargs >/backup/txt.tt` | xargs 缺命令 | 去掉 xargs;`find ... > txt.tt` |
| 4 | `wc -1` | 数字 1 与字母 l 混淆 | `wc -l` |
| 5 | `awk -F ":" '{print "$3}'` | 引号错位 | `awk -F: '{print $3}'` |
| 6 | `if [ $Disk_Inode -ge 0 ]` | 永远成立 | 应 `-ge 85` |
| 7 | `ErrNet=$(ifconfig $net &>/dev/null;echo $$)` | `$$` 是 PID | 应 `echo $?` |
| 8 | `echo "对应的IP地址是: $Network"` | 大小写不一致 | `$NetWork` |
| 9 | `n=$(echo $RANDOM \| cut -c 1)` | 只取一位,实际是 0/1/2…9 | `$(( RANDOM % 101 ))` |
| 10 | `if [[ ! $num =~^[0-9]+$ ]]` | `=~` 后缺空格 | `[[ ! "$num" =~ ^[0-9]+$ ]]` |
| 11 | `if [ $MySQL -eq 0$ ]` | 多余 `$` | `[ "$MySQL" -eq 0 ]` |
| 12 | `optios` | 拼写 | `options` |
| 13 | `cat /etc/passwd\|awk -F ":" '$3>1000'\|wc -l` 注释 "大于 1000 的都是普通" | 应为 `>=1000` | `'$3>=1000'` |
