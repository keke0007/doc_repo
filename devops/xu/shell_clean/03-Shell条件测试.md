# 03. Shell 条件测试

> 整理三类条件测试(文件 / 数值 / 字符串)、与/或组合、正则匹配,并订正原文档错误。

---

## 一、知识体系

```
条件测试
├── 三种语法形式
│   ├── test EXPR
│   ├── [ EXPR ]        ← POSIX,内部空格必须有
│   └── [[ EXPR ]]      ← Bash 扩展,支持 && || =~ 正则
├── 1. 文件测试
├── 2. 数值比较  -eq -ne -gt -lt -ge -le
├── 3. 字符串比较 = != -z -n =~
└── 4. 多条件组合 -a/-o (旧)、&&/|| (新)
```

---

## 二、三种语法

| 语法 | 适用 | 说明 |
|------|------|------|
| `test 表达式` | POSIX | 等价 `[ ]` |
| `[ 表达式 ]` | POSIX | **方括号内左右两侧必须有空格** |
| `[[ 表达式 ]]` | Bash | 支持 `&&`、`\|\|`、`=~`、可省略 `$` |

> Bash 中正则匹配 `=~`、`<`/`>` 字符串比较**只能用 `[[ ]]`**。

---

## 三、文件测试

| 选项 | 测试条件 |
|------|----------|
| `-e file` | 存在(任意类型) |
| `-f file` | 是普通文件 |
| `-d file` | 是目录 |
| `-r file` | 当前用户可读 |
| `-w file` | 当前用户可写 |
| `-x file` | 当前用户可执行 |
| `-s file` | 文件非空 |
| `-L file` | **是符号链接** |
| `f1 -nt f2` | f1 比 f2 新 |
| `f1 -ot f2` | f1 比 f2 旧 |
| `f1 -ef f2` | 同一 inode |

惯用法:

```bash
[ ! -d /data ] && mkdir -p /data        # 不存在则创建
[ -d /data ] || mkdir -p /data          # 等价写法

if [ ! -d "$back_dir" ]; then
    mkdir -p "$back_dir"
fi
```

---

## 四、数值比较

| 操作符 | 含义 |
|--------|------|
| `-eq` | 等于 |
| `-ne` | 不等于 |
| `-gt` | 大于 |
| `-lt` | 小于 |
| `-ge` | 大于等于 |
| `-le` | 小于等于 |

> **数值比较只用上述操作符,不要用 `=`、`>`、`<`**(后者在 `[ ]` 中是字符串比较或重定向)。
> 在 `(( ))` 中可以用 `>`、`<`、`==` 等 C 风格写法。

示例:磁盘使用率告警

```bash
#!/usr/bin/bash
Disk_Use=$(df -h | grep '/$' | awk '{print $5}' | tr -d '%')
if [ "$Disk_Use" -ge 80 ]; then
    echo "Disk Used: ${Disk_Use}%"  > /tmp/disk_use.txt
fi
```

---

## 五、字符串比较

| 操作符 | 含义 |
|--------|------|
| `=` 或 `==` | 相等 |
| `!=` | 不等 |
| `-z STR` | 字符串长度为 0(空) |
| `-n STR` | 字符串长度非 0 |
| `STR1 < STR2` | 字典序小(只能 `[[ ]]`) |
| `STR =~ REGEX` | 正则匹配(只能 `[[ ]]`) |

```bash
[ "$USER" = "root" ];   echo $?    # 注意变量两侧加双引号,避免空值导致语法错
[[ "$USER" =~ ^r ]];    echo $?

# 判断输入是否数字
num=123
[[ "$num" =~ ^[0-9]+$ ]] && echo "is number" || echo "not number"
```

陷阱:

- `[ "$USER " = "root" ]` —— 等号前后保留了空格会被当作字符串内容,导致永远不等。
- `[-z "$VAR"]` —— 中括号内必须有空格,否则报错。

---

## 六、多条件组合

| 旧式(`[ ]` 内) | 新式(`[[ ]]` 或外部) | 含义 |
|------------------|----------------------|------|
| `-a` | `&&` | 与 |
| `-o` | `\|\|` | 或 |
| `!` | `!` | 非 |

```bash
[ 1 -lt 2 -a 5 -gt 10 ];     echo $?    # 旧式
[[ 1 -lt 2 && 5 -gt 10 ]];    echo $?    # 新式(推荐)
```

---

## 七、综合案例

### 1. 创建用户(若不存在)

```bash
#!/usr/bin/bash
read -p "Please input a username: " user
if id "$user" &>/dev/null; then
    echo "User '$user' already exists"
else
    useradd "$user" && echo "User '$user' created."
fi
```

### 2. 内存使用率监控

```bash
#!/usr/bin/bash
Mem_Total=$(free -m | awk '/^Mem:/{print $2}')
Mem_Use=$(free -m   | awk '/^Mem:/{print $3}')
Mem_Pct=$(( Mem_Use * 100 / Mem_Total ))

if [ "$Mem_Pct" -ge 80 ]; then
    echo -e "\033[31m Memory High: ${Mem_Pct}% \033[0m"
else
    echo -e "\033[32m Memory OK:   ${Mem_Pct}% \033[0m"
fi
```

### 3. 批量创建用户(完整健壮版)

```bash
#!/usr/bin/bash
read -p "Please input number: " num
if [[ ! "$num" =~ ^[0-9]+$ ]]; then
    echo "error: not a number" && exit 1
fi

read -p "Please input prefix: " prefix
if [ -z "$prefix" ]; then
    echo "error: empty prefix" && exit 1
fi

for i in $(seq "$num"); do
    user="${prefix}${i}"
    if id "$user" &>/dev/null; then
        echo "$user already exists"
    else
        useradd "$user" && \
        echo "123456" | passwd --stdin "$user" &>/dev/null && \
        echo "$user is created."
    fi
done
```

---

## 八、原文档错误订正

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `Mem_B=$(($Mem_Use*100)/$Mem_Total))` | 括号不匹配 | `Mem_B=$(( Mem_Use*100 / Mem_Total ))` |
| 2 | `[ "$USER " = "root" ]` | 变量后有多余空格 | `[ "$USER" = "root" ]` |
| 3 | `[-z "$BBB"]` | 方括号内缺空格 | `[ -z "$BBB" ]` |
| 4 | `[[ "$num10" =~^[0-9]+$)]];echo $?` | `=~` 后缺空格;右括号错误 | `[[ "$num" =~ ^[0-9]+$ ]]` |
| 5 | `if [-z "$prefix"]` | 方括号内缺空格 | `if [ -z "$prefix" ]` |
| 6 | `if [[ ! "$num" =~^[0-9]+$ ]]` | `=~` 后缺空格 | `if [[ ! "$num" =~ ^[0-9]+$ ]]` |
| 7 | "字符长度为 0" 注释紧挨命令 | 排版混乱 | 见上文字符串比较表 |
| 8 | "[ -L file ]" 没解释 | 缺含义 | `-L`:判断是否为**符号链接** |
| 9 | 数值比较示例未给清单 | 缺整理 | 见数值比较表 |
| 10 | `[ "$USER" =~ ^r ]` | `=~` 不能用于 `[ ]` 必须 `[[ ]]` | `[[ "$USER" =~ ^r ]]` |
