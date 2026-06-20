# 05. Shell 循环语句

> 整理 `for` / `while` / `until` 循环、并发执行、循环控制内置命令,并订正原文档错误。

---

## 一、知识体系

```
循环结构
├── 1. for 循环
│   ├── 取值列表风格   for i in 1 2 3
│   ├── 序列展开       for i in {1..10}
│   ├── C 风格         for ((i=0;i<10;i++))
│   └── 并发执行       { ...; }&  +  wait
├── 2. while 循环
│   ├── while [ 条件 ]; do ... done
│   ├── while read line; do ... done < file
│   └── 死循环         while true / while :
├── 3. until 循环 (与 while 相反)
└── 4. 循环控制
    ├── break    跳出当前循环
    ├── continue 跳过本次进入下一次
    └── exit     退出整个脚本
```

---

## 二、for 循环

### 1. 三种形式

```bash
# 形式 1:取值列表
for i in 1 2 3 4 5; do echo "$i"; done

# 形式 2:序列展开
for i in {1..10}; do echo "$i"; done
for i in $(seq 1 10); do echo "$i"; done    # 等价

# 形式 3:C 风格
for ((i=1; i<=10; i++)); do echo "$i"; done
```

### 2. 并发探活主机

```bash
#!/usr/bin/bash
> ip.txt
for i in {1..254}; do
    {
        ip=192.168.70.$i
        if ping -c1 -W1 "$ip" &>/dev/null; then
            echo "$ip" | tee -a ip.txt
        fi
    } &
done
wait                        # 等待所有后台任务完成
echo "Get IP Is Finish!"
```

> 关键点:`{ ... } &` 让花括号内命令在子 shell 后台运行;`wait` 等待所有后台任务结束才继续。

### 3. 批量创建用户(交互式)

```bash
#!/usr/bin/bash
read -p "用户名前缀|密码|数量: " prefix pass num

if [[ ! "$num" =~ ^[0-9]+$ ]]; then
    echo "数量必须是数字" && exit 1
fi

cat <<-EOF
    前缀: $prefix
    密码: $pass
    数量: $num
EOF

read -p "确认创建? [y|n] " ans
case "$ans" in
    y|Y|yes|YES)
        for i in $(seq "$num"); do
            user="${prefix}${i}"
            if ! id "$user" &>/dev/null; then
                useradd "$user" && \
                echo "$pass" | passwd --stdin "$user" &>/dev/null && \
                echo "$user is created."
            else
                echo "$user already exists"
            fi
        done
        ;;
    n|N|no|NO) exit 0 ;;
    *)         echo "未知输入" ;;
esac
```

---

## 三、while 循环

### 1. 基础语法

```bash
while 条件成立; do
    循环体
done
```

### 2. 逐行读取文件(典型用法)

```bash
while read user; do
    if id "$user" &>/dev/null; then
        echo "$user already exists"
    else
        useradd "$user" &>/dev/null && echo "$user created"
    fi
done < user.txt
```

> 比 `for i in $(cat file)` 更安全:`while read` 按行读取,不会因为空格被切割。

### 3. 读取用户名+密码文件

```bash
# user.txt 每行格式:user password
while read user pass; do
    if ! id "$user" &>/dev/null; then
        useradd "$user" && echo "$pass" | passwd --stdin "$user" &>/dev/null
    fi
done < user.txt
```

### 4. 死循环

```bash
while true; do
    echo "tick"
    sleep 1
done
# 等价
while :; do ... ; done
```

---

## 四、until 循环

```bash
# 条件为假时执行循环体,条件成立则退出 —— 与 while 相反
i=1
until [ "$i" -gt 5 ]; do
    echo "$i"
    let i++
done
```

---

## 五、循环控制内置命令

| 命令 | 作用 |
|------|------|
| `break [n]` | 跳出 n 层循环(默认 1 层) |
| `continue [n]` | 跳过本次,直接下一轮(可跨层) |
| `exit [code]` | 退出整个脚本,返回码 code |

```bash
for i in {1..10}; do
    [ "$i" -eq 5 ] && continue      # 跳过 5
    [ "$i" -eq 8 ] && break         # 到 8 就停
    echo "$i"
done
```

---

## 六、原文档错误订正

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `if [ ! $num =~^[0-9]+$ ];then` | `=~` 必须用 `[[ ]]`;且 `=~` 后需空格 | `if [[ ! "$num" =~ ^[0-9]+$ ]]; then` |
| 2 | `id $username &>/dev/null; if [ $? -ne 1 ]; then useradd ...` | 逻辑反了:id 成功返回 0(用户已存在),不应再创建 | 应判断 `if [ $? -ne 0 ]` —— 不存在时才 useradd |
| 3 | `useradd $user &/dev/null` | 符号错误 | `useradd $user &>/dev/null` |
| 4 | `done[user.tt` | 重定向符错误 | `done < user.tt` |
| 5 | `Finsh` | 拼写 | `Finish`(不影响执行,但要规范) |
| 6 | for 并发示例缺 `wait` | 主进程会先于子进程结束 | 循环 done 后必须 `wait` |
| 7 | "while 循环基础语法"代码块未闭合 | 排版断裂 | 完整 `while ...; do ... done` |
