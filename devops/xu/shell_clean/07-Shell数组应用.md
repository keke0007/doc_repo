# 07. Shell 数组应用

> 整理普通数组、关联数组的定义、访问、遍历,并订正原文档错误。

---

## 一、知识体系

```
Shell 数组
├── 1. 分类
│   ├── 普通(索引)数组 —— 整数下标,declare -a
│   └── 关联数组       —— 字符串下标,declare -A
├── 2. 定义与赋值
├── 3. 访问(单个 / 切片 / 全部 / 长度 / 索引列表)
├── 4. 遍历(按值 / 按索引)
└── 5. 数组实战
    ├── 统计 /etc/passwd 中各类型 shell 数量
    ├── 统计 nginx 日志 IP 频次
    └── 统计 TCP 连接状态
```

---

## 二、普通(索引)数组

### 1. 定义

```bash
# 单个赋值
arr[0]=pear
arr[1]=apple
arr[2]=orange

# 一次性赋值
arr=(pear apple orange peach)

# 含空格元素
arr=(tom jack alice "bash shell")

# 跳跃索引
arr=(1 2 3 "linux shell" [20]=puppet)

# 把文件每行变成一个元素(注意:默认按空白拆分单词,不一定是行)
mapfile -t arr < /etc/hostname        # 推荐
# 或
arr=( $(cat /etc/hostname) )
```

### 2. 访问

设 `arr=(pear apple orange peach)`:

| 表达式 | 含义 | 结果 |
|--------|------|------|
| `${arr[0]}` | 第一个元素 | pear |
| `${arr[@]}` 或 `${arr[*]}` | 全部元素 | pear apple orange peach |
| `${#arr[@]}` | 元素个数 | 4 |
| `${#arr[0]}` | 第一个元素长度 | 4 (pear 长度) |
| `${!arr[@]}` | 所有索引列表 | 0 1 2 3 |
| `${arr[@]:1}` | 从下标 1 起到末尾 | apple orange peach |
| `${arr[@]:1:2}` | 从下标 1 起取 2 个 | apple orange |

### 3. 查看 / 声明

```bash
declare -a               # 列出所有普通数组
declare -a arr           # 显式声明
```

---

## 三、关联(字典)数组

### 1. 定义与赋值

```bash
declare -A info                                  # 必须先声明
info[name]=bgx
info[age]=18
info[skill]=linux

# 一次性
declare -A info=([name]=bgx [age]=18 [skill]=linux)
```

### 2. 访问

```bash
echo "${info[name]}"        # bgx
echo "${info[@]}"           # 所有值(顺序不保证)
echo "${!info[@]}"          # 所有键(name age skill)
echo "${#info[@]}"          # 元素个数
```

### 3. 查看

```bash
declare -A                  # 列出所有关联数组
```

---

## 四、遍历数组

### 1. 按索引遍历(推荐,关联数组必须用此法)

```bash
for k in "${!info[@]}"; do
    echo "$k = ${info[$k]}"
done
```

### 2. 按值遍历(只能用于普通数组)

```bash
for v in "${arr[@]}"; do
    echo "$v"
done
```

---

## 五、实战:用关联数组做"频次统计"

**核心套路**:

> 把想要统计的字段当作**关联数组的键**,每出现一次就 `array[key]++`,最后遍历输出。

### 1. 统计 /etc/passwd 中各类型 shell 数量

```bash
#!/usr/bin/bash
declare -A shells

while IFS=: read -r _ _ _ _ _ _ shell; do
    (( shells[$shell]++ ))
done < /etc/passwd

for k in "${!shells[@]}"; do
    echo "$k: ${shells[$k]}"
done
```

### 2. Nginx 日志 IP 访问次数 Top10

```bash
#!/usr/bin/bash
declare -A ips
while read -r ip _; do
    (( ips[$ip]++ ))
done < /var/log/nginx/access.log

for k in "${!ips[@]}"; do
    printf "%-5d %s\n" "${ips[$k]}" "$k"
done | sort -rn | head
```

### 3. 统计 TCP 状态

```bash
#!/usr/bin/bash
declare -A state
while read s; do
    (( state[$s]++ ))
done < <(ss -an | awk '/:80/{print $2}')

for k in "${!state[@]}"; do
    echo "$k: ${state[$k]}"
done
```

---

## 六、原文档错误订正

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `declare -a` 用于"查看赋值结果" | `-a` 列普通数组;**关联数组要用 `-A`** | 普通:`declare -a`;关联:`declare -A` |
| 2 | `array5=(\`cat /etc/passwd\`)` 注释说"赋给 array3" | 变量名前后不一致 | 注释里也写 `array5` |
| 3 | `echo "hosts first: ${hosts[1]""` | 引号不闭合 | `echo "hosts first: ${hosts[1]}"` |
| 4 | `IFS=$_'\n'` | 多了 `_` | `IFS=$'\n'`(注意 `$'...'` 是 ANSI-C 引用,可解析转义) |
| 5 | 关联数组定义示例没先 `declare -A` | 直接 `arr[key]=val` 在某些 bash 版本会被当作普通数组 | 必须先 `declare -A name` 再赋值 |
| 6 | "关联数组 …… 仅针对关联数据" 句法不通 | 应为"仅针对关联数组" | 改正用语 |
| 7 | `while read line; do hosts[++i]=$line; done < /etc/hosts` 配合 `${hosts[1]}` | 第 1 个元素索引是 1 不是 0,容易踩坑 | 用 `mapfile -t hosts < /etc/hosts` 更直观 |
| 8 | 表格用 HTML `<table>` 嵌入 | 渲染混乱 | 改用 Markdown 表格(本篇已重写) |
