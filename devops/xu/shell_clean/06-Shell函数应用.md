# 06. Shell 函数应用

> 整理函数的定义、调用、传参、返回值、作用域,并订正原文档错误。

---

## 一、知识体系

```
函数
├── 1. 定义函数 (两种语法)
├── 2. 调用函数
├── 3. 函数传参与位置参数
├── 4. 返回值 return 与 $?
├── 5. 作用域 local
└── 6. 函数库 (多文件) source 加载
```

---

## 二、定义函数

```bash
# 方式 1 (POSIX,推荐)
fun_name() {
    命令序列
}

# 方式 2 (Bash 关键字,可省略 ())
function fun_name {
    命令序列
}

# 方式 3 (Bash 全写)
function fun_name() {
    命令序列
}
```

> 函数**必须先定义后调用**;若定义后不调用,函数体不会执行。

---

## 三、调用函数 & 传参

调用函数:**直接写函数名**(不要带 `()`)。

```bash
greet() {
    echo "hello, $1"      # $1 是函数的第一个参数,与脚本的 $1 互相独立
    echo "all args: $*"
    echo "count: $#"
}

greet alice bob
```

| 变量 | 作用域 |
|------|--------|
| `$0` | 始终是**脚本名**(不会被函数覆盖) |
| `$1`、`$2`…`${10}` | 当前函数 / 脚本的位置参数 |
| `$#` `$*` `$@` `$?` `$$` | 与脚本中含义相同 |

---

## 四、返回值

### 1. 使用 `return` (退出码,0–255)

```bash
fun_double() {
    read -p "enter num: " n
    return $(( 2 * n ))     # 必须是 0-255 的整数
}

fun_double
echo "$?"                   # 取出 return 值
```

> `return` 返回的不是数据,而是状态码,**取值范围 0-255**(超过会被 mod 256)。

### 2. 使用 `echo` + 命令替换(传"数据")

```bash
fun_sum() {
    echo $(( $1 + $2 ))
}

result=$(fun_sum 3 5)       # 8
echo "$result"
```

> 真正"返回数据"的标准做法:函数里 `echo` 数据,调用方用 `$()` 接收。

---

## 五、位置参数 & 数组传参

### 1. 强制要求 3 个参数

```bash
#!/usr/bin/bash
if [ $# -ne 3 ]; then
    echo "usage: $(basename "$0") par1 par2 par3"
    exit 1
fi

fun_mul() { echo "$(( $1 * $2 * $3 ))"; }

result=$(fun_mul "$1" "$2" "$3")
echo "result: $result"
```

### 2. 数组传参

```bash
#!/usr/bin/bash
arr=(1 2 3 4 5)

product() {
    local fac=1
    for x in "$@"; do
        fac=$(( fac * x ))
    done
    echo "$fac"
}

product "${arr[@]}"        # 推荐:每个元素独立传参,保留含空格的元素
```

> 注意:`${arr[*]}` 在不加引号时与 `${arr[@]}` 表现相同;加双引号时,`*` 合并为一个字符串,`@` 保持多个独立参数。**通常用 `"${arr[@]}"`**。

---

## 六、作用域:local

默认所有变量都是**全局**的。在函数内用 `local` 修饰才是局部变量:

```bash
v=outer
fun() {
    local v=inner    # 只在函数内有效
    echo "$v"        # inner
}
fun
echo "$v"            # outer
```

---

## 七、函数库(多文件调用)

把通用函数放在独立文件,**主脚本 source 加载**。

### 1. 调用流程图

```
┌───────────────────┐   source / .    ┌──────────────────┐
│  main.sh          │ ──────────────▶ │ lib_common.sh    │
│                   │                 │  fn_log()        │
│  source lib...    │                 │  fn_check_root() │
│  fn_log "start"   │ ◀─加载到当前──── │  fn_install()    │
│  fn_check_root    │     Shell        └──────────────────┘
│  fn_install nginx │
└───────────────────┘
```

### 2. 示例

`lib_common.sh`

```bash
#!/usr/bin/bash
fn_log()        { echo "[$(date +%F\ %T)] $*"; }
fn_check_root() { [ "$EUID" -eq 0 ] || { echo "Need root"; exit 1; } }
fn_install()    { yum install -y "$1" &>/dev/null && fn_log "$1 installed"; }
```

`main.sh`

```bash
#!/usr/bin/bash
source ./lib_common.sh        # 等价 `.  ./lib_common.sh`

fn_check_root
fn_log "begin install"
fn_install nginx
fn_log "done"
```

> 关键点:`source` / `.` **不会创建子 shell**,被加载文件中的函数和变量都进入当前 shell。

---

## 八、综合案例:菜单驱动的工具箱

```bash
#!/usr/bin/bash
menu() {
    cat <<-EOF
    ====================
    1) 查看内存使用
    2) 查看磁盘使用
    3) 查看系统负载
    q) 退出
    ====================
EOF
}

view_mem()   { free -h; }
view_disk()  { df -h; }
view_load()  { uptime; }

while true; do
    menu
    read -p "请选择: " op
    case "$op" in
        1) view_mem  ;;
        2) view_disk ;;
        3) view_load ;;
        q|Q) exit 0  ;;
        *) echo "无效输入" ;;
    esac
done
```

---

## 九、原文档错误订正

| # | 原文 | 问题 | 正确写法 |
|---|------|------|----------|
| 1 | `function 函数名 { ... }` 说明 | 没解释关键字 `function` 后**可以省略 `()`**,但纯 POSIX 必须 `name() { }` | 见本篇"定义函数"三种语法 |
| 2 | `return $[2*$num]` | `return` 只能返回 0-255 状态码,这里语义混淆为"数据返回" | 数据用 `echo` 返回,用 `$()` 接收 |
| 3 | `array ${num[*]}` | 不加引号易在含空格元素时被切割 | 使用 `"${num[@]}"` |
| 4 | 案例缺少 `local` 关键字说明 | 易污染全局变量 | 局部变量必须用 `local` 声明 |
| 5 | 未提及 `source` / `.` 加载函数库 | 缺关键知识点 | 见本篇"函数库"小节 |
