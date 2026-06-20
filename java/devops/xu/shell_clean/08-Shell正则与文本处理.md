# 08. Shell 正则与文本处理(grep / sed / awk)

> 整理基础正则、sed 编辑、awk 编程,并订正原文档错误。

---

## 一、知识体系

```
正则与文本处理
├── 1. 正则表达式
│   ├── 基础正则 BRE  (grep / sed)
│   └── 扩展正则 ERE  (grep -E / sed -r / egrep / awk)
├── 2. sed 流编辑器
│   ├── 工作机制(模式空间 / 暂存空间)
│   ├── 选项 -n -i -r -e
│   ├── 命令 p a i d c s n h H g G w
│   └── 后向引用 \1
└── 3. awk 编程语言
    ├── 处理模型(BEGIN / 主体 / END)
    ├── 内部变量 $0 $n NR NF FS OFS RS ORS FNR
    ├── 模式(正则 / 比较 / 范围)
    ├── 流程控制(if / while / for)
    └── 数组(用作频次统计)
```

---

## 二、基础正则表达式 (BRE)

| 元字符 | 含义 |
|--------|------|
| `\` | 转义 |
| `^` | 行首(awk 中:字符串首) |
| `$` | 行尾(awk 中:字符串尾) |
| `^$` | 空行 |
| `.` | 任意单个字符(不含 `\n`) |
| `[abc]` | 任一字符 |
| `[^abc]` | 排除 |
| `[a-z]` | 范围 |
| `*` | 前面项 0 次或多次 |
| `\{n\}` | 前面项重复 n 次 (BRE) |
| `\{n,m\}` | 重复 n 到 m 次 |
| `\(...\)` | 分组,可用 `\1` 反向引用 |

**扩展正则 (ERE)** —— `grep -E` / `egrep` / `sed -r` / `awk`:

| 元字符 | 含义 |
|--------|------|
| `?` | 0 或 1 次 |
| `+` | 1 或多次 |
| `{n}` / `{n,m}` | 不再需要转义 |
| `()` | 分组 |
| `\|` | 或 |

**POSIX 字符类**:

| 写法 | 等价 |
|------|------|
| `[[:digit:]]` | `[0-9]` |
| `[[:lower:]]` | `[a-z]` |
| `[[:upper:]]` | `[A-Z]` |
| `[[:alpha:]]` | `[a-zA-Z]` |
| `[[:alnum:]]` | `[a-zA-Z0-9]` |
| `[[:space:]]` | 空白(空格、tab 等) |

---

## 三、grep 速查

```bash
grep "^m"      file       # 以 m 开头
grep "m$"      file       # 以 m 结尾
grep -v "^$"   file       # 排除空行
grep "\.$"     file       # 以 . 结尾(. 需转义)
grep "[0-9]"   file       # 含数字
grep "[^0-9]"  file       # 不含数字
grep -o "8\{3\}" file     # 只输出匹配的部分,3 个 8 (BRE)
grep -E "8{3,5}" file     # 3~5 个 8 (ERE)
grep -nE "pattern" file   # 显示行号
```

---

## 四、sed 流编辑器

### 1. 工作机制

```
┌────────┐  逐行读入  ┌────────────┐ 执行命令 ┌────────┐
│ 原文件 │ ─────────▶ │ 模式空间   │ ───────▶ │ 输出   │
└────────┘            │ (pattern)  │          └────────┘
                      └────┬───────┘
                           │ h/H/g/G
                      ┌────▼──────┐
                      │ 暂存空间   │
                      │ (hold)    │
                      └───────────┘
```

- **模式空间 (pattern space)**:sed 处理当前行的缓冲区
- **暂存空间 (hold space)**:用于跨行处理的临时存储
- 默认**不修改原文件**,只输出;`-i` 才会原地修改

### 2. 命令行格式

```
sed [options] 'address command' file
```

`address`(地址)可为:行号 `3`、范围 `1,9`、最后一行 `$`、正则 `/pat/`。

### 3. 常用选项

| 选项 | 含义 |
|------|------|
| `-n` | 取消默认输出(配合 `p` 只输出匹配) |
| `-i` | **原地修改**文件(`-i.bak` 同时备份) |
| `-r` 或 `-E` | 支持扩展正则 |
| `-e` | 允许多条编辑命令 |
| `-f script` | 从脚本文件读命令 |

### 4. 常用命令

| 命令 | 作用 |
|------|------|
| `p` | 打印 |
| `d` | 删除 |
| `a\text` | 行后追加 |
| `i\text` | 行前插入 |
| `c\text` | 整行替换 |
| `s/old/new/[g]` | 替换(g 全局) |
| `n` | 读取下一行覆盖模式空间 |
| `!` | 取反 |
| `h` / `H` | 把模式空间复制 / 追加到暂存空间 |
| `g` / `G` | 把暂存空间复制 / 追加到模式空间 |
| `w file` | 写到指定文件 |

### 5. 经典示例

```bash
# 多命令(等价 -e ;)
sed -e '1,9d' -e 's/root/alex/g' /etc/passwd

# 只打印匹配
sed -n '/halt/p' /etc/passwd
sed -n '$p'    /etc/passwd       # 最后一行
sed -n '2,5p'  /etc/passwd

# 追加 / 插入
sed -i '30a listen 80;'    nginx.conf
sed -i '30i listen 80;'    nginx.conf

# 修改某行
sed -i '7c SELINUX=disabled' /etc/selinux/config
sed -ri '/^SELINUX=/c SELINUX=disabled' /etc/selinux/config

# 删除注释行 + 空行
sed -ri '/^[[:space:]]*#/d; /^[[:space:]]*$/d' /etc/vsftpd/vsftpd.conf

# 给 2-6 行加 # 注释
sed -ri '2,6s/^/#/' /etc/passwd

# 替换(忽略大小写 + 全局)
sed -ri 's/root/alice/gi' /etc/passwd

# 后向引用 \1
sed -r 's/(Roo)/\1-alice/g' /etc/passwd

# 路径含 / 时,使用其它分隔符
sed -r 's#/etc/abc/456#/dev/null#g' a.txt

# h/H/g/G 用法:把第一行放到最后追加显示
sed '1h; $G' /etc/hosts
```

---

## 五、awk 编程

### 1. 处理模型

```
awk 'BEGIN{ ... } pattern{ action } END{ ... }'
       ↓             ↓                  ↓
   读文件前    每读一行执行匹配       读完文件后
```

执行步骤:

1. 读入一行 → 赋值给 `$0`
2. 按 `FS` 切分 → `$1`、`$2`…
3. `NR` +1、`FNR` +1、`NF` 设为字段数
4. 对每条 `pattern{action}` 规则:模式成立则执行 action
5. 处理下一行,直到 EOF;最后执行 `END`

### 2. 内部变量

| 变量 | 含义 |
|------|------|
| `$0` | 当前整行 |
| `$1`…`$NF` | 当前行的第 n 个字段;`$NF` 是最后一个 |
| `NF` | 当前行字段数 |
| `NR` | 已读总记录数 |
| `FNR` | 当前文件的记录数(多文件时与 NR 不同) |
| `FS` | 输入字段分隔符(默认空白) |
| `OFS` | 输出字段分隔符(默认空格) |
| `RS` | 输入记录分隔符(默认 `\n`) |
| `ORS` | 输出记录分隔符(默认 `\n`) |
| `FILENAME` | 当前文件名 |

### 3. 设置分隔符

```bash
awk -F:  '{print $1,$3}'              /etc/passwd
awk      'BEGIN{FS=":";OFS="--"} {print $1,$3}' /etc/passwd
awk -F'[: \t]' '{print $1,$2}'        /etc/passwd      # 多种分隔符
```

### 4. 模式

| 模式形式 | 含义 |
|----------|------|
| `/正则/` | 整行匹配正则 |
| `$n ~ /正则/` | 第 n 字段匹配 |
| `$n !~ /正则/` | 第 n 字段不匹配 |
| `$3 > 1000` | 比较表达式 |
| `NR==1, /end/` | 范围模式(第 1 行到匹配 /end/ 之间) |
| `BEGIN` / `END` | 起始 / 结束块 |

### 5. 条件 / 循环 / 数组

```bash
# 三元:UID==0 是管理员,统计数量
awk -F: '{ if ($3==0) i++; else j++ } END { print "admin="i,"normal="j }' /etc/passwd

# while
awk -F: '{ i=1; while (i<=NF) { print $i; i++ } }' /etc/passwd

# C 风格 for
awk -F: '{ for (i=1;i<=NF;i++) print $i }' /etc/passwd

# 数组(键值,任意键)
awk -F: '{shells[$NF]++} END {for (k in shells) print k, shells[k]}' /etc/passwd
```

### 6. printf 格式化

```bash
awk -F: '{printf "%-15s %-5s %s\n", $1, $3, $7}' /etc/passwd
# %s 字符串  %d 整数  %f 浮点  %-15s 左对齐宽度 15
```

---

## 六、Nginx 日志分析综合实战

日志格式(示例):

```
52.55.21.59 - - [25/Jan/2018:14:55:36 +0800] "GET /feed/ HTTP/1.1" 404 162 "https://www.google.com/" "Opera/..." "-"
```

字段对照(默认空格切分):

| 字段 | 含义 |
|------|------|
| `$1` | 客户端 IP |
| `$4` | `[25/Jan/2018:14:55:36` |
| `$7` | URL($request 的中段) |
| `$9` | HTTP 状态码 |
| `$10` | 响应字节数 |

```bash
LOG=/var/log/nginx/access.log

# 1) 当天 PV
grep "25/Jan/2018" $LOG | wc -l

# 2) 当天访问最多的 10 个 IP
awk '/25\/Jan\/2018/{ips[$1]++} END{for(i in ips) print ips[i],i}' $LOG \
    | sort -rn | head

# 3) 访问大于 100 次的 IP
awk '/25\/Jan\/2018/{ips[$1]++} END{for(i in ips) if(ips[i]>100) print i,ips[i]}' $LOG

# 4) Top10 URL
awk '/25\/Jan\/2018/{u[$7]++} END{for(i in u) print u[i],i}' $LOG \
    | sort -rn | head

# 5) 每个 URL 总下行字节
awk '/25\/Jan\/2018/{cnt[$7]++; size[$7]+=$10} END{for(i in cnt) print cnt[i],i,size[i]}' $LOG \
    | sort -rn | head

# 6) 每个 IP+状态码 组合
awk '{key=$1" "$9; ipcode[key]++} END{for(k in ipcode) print ipcode[k],k}' $LOG \
    | sort -rn | head

# 7) 404 出现的次数
awk '$9=="404"{c[$9]++} END{for(i in c) print i,c[i]}' $LOG

# 8) 时间段过滤(15:00-19:00)的 404
awk '$4>="[25/Jan/2018:15:00:00" && $4<="[25/Jan/2018:19:00:00" && $9=="404" {c[$9]++}
     END{for(i in c) print i,c[i]}' $LOG

# 9) 各状态码分类计数(1xx / 2xx / ... / 5xx)
awk '{
        if      ($9>=100 && $9<200) i++
        else if ($9>=200 && $9<300) j++
        else if ($9>=300 && $9<400) k++
        else if ($9>=400 && $9<500) n++
        else if ($9>=500)           p++
     } END{ print (i?i:0), (j?j:0), (k?k:0), (n?n:0), (p?p:0), i+j+k+n+p }' $LOG
```

---

## 七、原文档错误订正

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `[[:alpha:]] [a-Z]` | `a-Z` 不是合法字符类 | `[[:alpha:]] = [a-zA-Z]` |
| 2 | "sed 多重编辑选项 e" 配 `sed '1,9d' passwd \| sed 's#root#alex#g'` | 这是管道,不是多重编辑 | 多重编辑应写 `sed -e '1,9d' -e 's#root#alex#g' passwd` |
| 3 | `sed -r 's/[0-9][0-9]\$/& .5/'` | `\$` 不该转义 | `sed -r 's/[0-9][0-9]$/& .5/'` |
| 4 | `sed -r '\etc\abc/456/d' a.txt` | 反斜杠分隔符使用错误 | `sed -r '\#/etc/abc/456#d' a.txt`(反斜杠 + 自定义分隔符) |
| 5 | `awk '//^root/'` | 双斜杠后再 `/^root/` 是非法 | `awk '/^root/'` |
| 6 | `awk '$0 ~ !/^root/'` | 取反应写在中间 | `awk '$0 !~ /^root/'` |
| 7 | `awk -F: '{if($3>5555){print $3} else {print $1}'` | 缺右花括号 | `awk -F: '{if($3>5555){print $3} else {print $1}}'` |
| 8 | `awk -F: '{if($3>1000){i++}} END {print i}'` 标注"统计普通用户数量" | UID 1000 起为普通,应 `>=1000` 或 `>=500`(老版本) | 视发行版定;CentOS 7+ 通常 `>=1000` |
| 9 | `awk "/nameserver/{print $2}"` 用双引号 | shell 会先把 `$2` 展开为空 | 用单引号:`awk '/nameserver/{print $2}'` |
| 10 | `size[ $7$ ]+$10` | OCR 错位 | `size[$7] += $10` |
| 11 | `awk "/25\Jan\2018/"` | 反斜杠在正则中无效转义 | `awk '/25\/Jan\/2018/'` 或 `'25\\/Jan\\/2018'` |
| 12 | `awk '/25\/Jan\/2018/ {ips[$1]++} END {for(i in ips) {sum+=ips[i]} {print sum}}'` | END 中多余 `{}` | `END{ for(i in ips) sum+=ips[i]; print sum }` |
| 13 | `df \|awk '/\\/$/'` | 试图匹配根目录但反斜杠 stray | `df \| awk '/\/$/'` |
| 14 | `username e[i]` / `++x Advertising` | OCR 噪声 | `username[i]` / `++x` |
| 15 | `wc -1` | 数字 1 与小写 l 混淆 | `wc -l` |
