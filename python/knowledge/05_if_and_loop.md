# python5 条件语句和循环语句 知识点梳理

> 原文档:`python/Article/PythonBasis/python5/` 下 `If.md / Cycle.md / Example.md`
> 整理对象:if/elif/else、match/case(3.10+)、for/while、循环控制、循环 else 分支

---

## 一、条件语句

### 1. 真值规则

Python 把以下值视为假(falsy):
```
False, None, 0, 0.0, 0j, '', "", '''  ''', [], (), {}, set(), range(0)
```
其余视为真(truthy)。可以用 `bool(x)` 强转查看。

### 2. if / elif / else 基本形态

```python
if cond1:
    ...
elif cond2:
    ...
else:
    ...
```

要点:
- 缩进 **强制 4 空格**;冒号不能漏。
- `elif` 是 `else if` 的合并形式,**没有 `else if` 写法**。
- 条件不需要括号(写了也合法,但不必要)。

### 3. 多条件:布尔运算

| 运算 | 含义 | 短路 |
|------|------|------|
| `and` | 与 | 左假 → 直接返回左,不算右 |
| `or` | 或 | 左真 → 直接返回左,不算右 |
| `not` | 非 | — |

优先级(从高到低):比较运算 `< <= == != > >=` > `not` > `and` > `or`。

### 4. 三目表达式

```python
result = 'pass' if score >= 60 else 'fail'
```

### 5. 链式比较(Pythonic)

```python
if 80 <= score < 90:     # 等价 score>=80 and score<90
    print('良好')
```

> 原文反复使用 `(java >= 80 and java < 90)`,链式比较写法更清晰。

### 6. match / case (Python 3.10+ PEP 634)

仅 **结构化分发** 才适合,简单值判断仍用 `if/elif`。

```python
match value:
    case 0:                       # 字面值
        ...
    case [x, 0]:                  # 序列模式 + 捕获
        ...
    case [first, *rest]:          # 收集剩余
        ...
    case {'status': 'ok', 'data': data}:   # 映射模式
        ...
    case Point(x=0, y=y):         # 类模式(需要 dataclass / __match_args__)
        ...
    case x if x > 0:              # 守卫
        ...
    case _:                       # 兜底
        ...
```

关键点:
- **没有 fallthrough**,匹配到一个 case 后自动结束(与 C/Java 的 switch 不同,不写 break)。
- `case x:` 这种 **裸名字** 会被当作 **捕获模式**(匹配任何值并把它绑到 x),不是匹配名为 x 的变量。要匹配某个常量值,用 `case Color.RED:` 这种 **限定名**,或自定义 guard。

---

## 二、循环语句

### 1. for 循环

```python
for item in iterable:
    ...
```

- `iterable` 可以是 list/tuple/str/dict/set/file/generator 等任何 **可迭代对象**。
- 对 `dict` 迭代默认是 **键**,要值用 `d.values()`,要键值对用 `d.items()`。
- **不能迭代** `int` / `float` 等非可迭代对象,会 `TypeError: 'int' object is not iterable`。

### 2. range 函数

```python
range(stop)              # 0, 1, ..., stop-1
range(start, stop)       # start, start+1, ..., stop-1
range(start, stop, step) # 含 step,可负
```

- `range` 返回 **range 对象**(惰性),不是 list,**O(1) 内存**。
- `range(0, 10, 2)` → 0,2,4,6,8。
- `range(10, 0, -1)` → 10,9,...,1。

### 3. while 循环

```python
while cond:
    ...
```

- 适合"满足某条件就一直执行"。
- 死循环写 `while True:` + `break` 退出。

### 4. 循环控制

| 关键字 | 行为 |
|--------|------|
| `break` | 立即跳出最内层循环 |
| `continue` | 跳过本次,进入下一次 |
| `pass` | 占位空语句,不做任何事(用于语法上需要语句但暂无逻辑时) |

### 5. for/while else 子句

```python
for x in items:
    if found(x):
        break
else:
    print('没找到')      # 仅当 for 正常跑完(没 break)才执行
```

- `else` 在 **没有 break 的情况下** 才执行。
- 常用于"找东西没找到给出提示"。
- 注意:**异常退出也不会执行 else**(只看 break)。

---

## 三、综合实例点评

### 1. 九九乘法表

```python
for i in range(1, 10):
    for j in range(1, i + 1):
        print(f'{i}x{j}={i*j}\t', end='')
    print()
```

要点:
- `print(..., end='')` 抑制换行。
- 单独的 `print()` 输出一个换行,结束当前行。
- f-string 比 `%s/%d` / `.format()` 更直观。

### 2. 闰年判断

```python
year = int(input('请输入一个年份: '))
if (year % 4 == 0 and year % 100 != 0) or year % 400 == 0:
    print(f'{year} 是闰年')
else:
    print(f'{year} 不是闰年')
```

提醒:`input()` 始终返回 `str`,数值计算前需要 `int()` / `float()` 强转,并 **应包在 try/except 里** 防止用户输入非数字字符:

```python
try:
    year = int(input('请输入一个年份: '))
except ValueError:
    print('请输入有效数字')
    raise SystemExit
```

---

## 四、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | If.md §1 | "非 0 和非空值为 True,0 或 null 为 False" | 描述方向对,但 Python 中是 `None` 不是 `null`;还有 0.0、空字符串、空容器都为假 |
| 2 | Cycle.md §6 例 `else: print(...)` | 文中没强调 for-else 的触发条件 | for-else 中 else 仅在 **没 break** 时才执行,这是核心要点 |

> 本章无多文件调用,流程图保留原文档已有的 if/for 单文件流程图即可。
