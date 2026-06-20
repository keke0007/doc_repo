# python6 函数 知识点梳理

> 原文档:`python/Article/PythonBasis/python6/` 下 `1.md ~ 5.md`
> 整理对象:自定义函数、返回值、参数(默认/关键字/可变长/仅关键字)、传值机制、lambda、类型注解

---

## 一、自定义函数

### 1. 语法形态

```python
def func_name(param1, param2=default, *args, key, **kwargs) -> ReturnType:
    """函数文档(docstring)。"""
    function_body
    return value
```

要点:
- `def` 关键字定义,函数名遵循变量命名规则。
- 缩进决定函数体边界。
- 不带 `return` 或 `return` 不带值,**默认返回 `None`**。
- 函数也是对象,可以赋值、作参数、作返回值(一等公民)。

### 2. 命名建议

- 函数名小写下划线分隔:`get_user_info()`、`is_valid()`。
- 谓词函数返回 bool 用 `is_/has_/can_` 等开头。
- 避免与内建名冲突(如 `sum`、`list`、`type`、`id`),`def sum(...)` 会遮盖内建 `sum()`。

> ⚠️ 原文示例(`1.md` / `2.md`)反复使用 `def sum(num1, num2):` —— 这会 **遮蔽内建 `sum()`**,在工程中应当避免。教学示例最好改成 `def add(num1, num2)`。

---

## 二、返回值

- 任何表达式都能 `return`,**没有 `void` 概念**。
- **多返回值本质是返回 tuple**:`return a, b` 等价 `return (a, b)`,接收端可解包 `x, y = func()`。

```python
def divmod_(n, d):
    return n // d, n % d         # 实际是 tuple

q, r = divmod_(10, 3)            # 解包
pair = divmod_(10, 3)            # (3, 1)
```

- 多返回值更现代的写法是 `NamedTuple` 或 `dataclass`,可读性更好:
  ```python
  from typing import NamedTuple
  class DivResult(NamedTuple):
      quotient: int
      remainder: int
  def divmod_(n, d) -> DivResult:
      return DivResult(n // d, n % d)
  ```

---

## 三、参数

### 1. 五种参数(按声明顺序)

```python
def f(pos1, pos2, /, pos_or_kw, *, kw_only, **kwargs):
    ...
```

| 位置 | 形式 | 说明 |
|------|------|------|
| `pos1, pos2` | 位置参数(可选,Python 3.8+ `/` 之前的) | **只能** 按位置传 |
| `pos_or_kw` | 普通参数 | 既可按位置也可按关键字 |
| `*args` | 可变位置参数 | 收集成 tuple |
| `kw_only` | 强制关键字参数 | 必须 `f(kw_only=x)` |
| `**kwargs` | 可变关键字参数 | 收集成 dict |

### 2. 默认参数 —— 必避的坑

**默认值在函数定义时计算并保存,只算一次**,所以可变对象作默认值会被多次调用共享:

```python
def f(b=[]):     # ❌ 反模式
    b.append(1)
    return b

f()              # [1]
f()              # [1, 1]   不是预期的 [1]
```

正确写法:

```python
def f(b=None):
    if b is None:
        b = []
    b.append(1)
    return b
```

### 3. 关键字参数

调用时显式写 `name=value`,顺序无所谓,大幅提高可读性:

```python
print_user_info(name='小明', age=18, sex='男')
```

### 4. `*args` / `**kwargs`

```python
def f(*args, **kwargs):
    print(args)      # tuple
    print(kwargs)    # dict

f(1, 2, 3, name='x')
# args   = (1, 2, 3)
# kwargs = {'name': 'x'}
```

调用方也能解包:

```python
nums = [1, 2, 3]
f(*nums)            # 等价 f(1, 2, 3)

d = {'name': 'x'}
f(**d)              # 等价 f(name='x')
```

### 5. 强制关键字参数(`*` 隔板)

```python
def f(name, *, age, sex='男'):   # age/sex 必须用关键字传
    ...
f('小明', age=18)                # OK
f('小明', 18)                    # TypeError
```

### 6. 仅位置参数(Python 3.8+ PEP 570)

```python
def f(a, b, /, c):     # a,b 仅位置;c 位置/关键字均可
    ...
```

常见于内建函数 API,锁住参数名让未来重命名不破坏调用方。

---

## 四、参数传递机制("传值"还是"传引用"?)

### 1. Python 的真相

**Python 是"传对象引用"(call by object reference,也叫 call by sharing)。** 不是 C 的传值,也不是严格意义的传引用。

- 实参与形参 **绑定同一个对象**。
- 如果函数内 **重新赋值** 形参 → 只切换形参的指向,实参不变。
- 如果函数内对 **可变对象做原地修改**(append、设元素、+=) → 实参也跟着改。

### 2. ASCII 模型

```
b = 1                  ┌────┐
b ──────────────────▶  │ 1  │
                       └────┘
change(b):
    b = 1000           ┌────┐    ┌─────┐
    ↑形参 b 切换指向    │ 1  │    │1000 │
                       └────┘    └─────┘
                          ↑          ↑
                       外部 b      函数内的 b
退出函数后,外部 b 仍然是 1。

------------------------------------------------

lst = [1,2,3]          ┌──────────┐
lst ──────────────▶    │ [1,2,3]  │
                       └──────────┘
change(lst):
    lst.append(4)         ↑
                          原地修改这个对象 → [1,2,3,4]

退出函数后,外部 lst 看到 [1,2,3,4](是同一个对象)。
```

### 3. 实践要点

- 想避免被函数修改 → 传入副本:`func(lst.copy())`、`func(dict(d))`。
- 想明确返回新对象 → 函数内 **不要原地改**,返回新对象给调用方。

---

## 五、Lambda 匿名函数

### 1. 形式

```python
f = lambda x, y: x + y
f(1, 2)         # 3
```

- 表达式式,**只能写一个表达式**,不能含语句、`if/else` 完整块、`try`、`for`(可写表达式三元 `a if c else b`)。
- 通常配合 `sorted/filter/map/key=` 一类高阶函数使用。

### 2. 闭包陷阱(经典面试题)

```python
fs = [lambda: i for i in range(3)]
[f() for f in fs]    # [2, 2, 2] 而不是 [0, 1, 2]
```

原因:`i` 是 **自由变量**,运行时去外层作用域查值,循环结束 `i=2`。

修正:用默认参数把当时的 `i` 锁住:

```python
fs = [lambda i=i: i for i in range(3)]
[f() for f in fs]    # [0, 1, 2]
```

---

## 六、类型注解(3.10+ 现代写法)

```python
def find_user(name: str) -> str | None:    # 3.10+ 用 | 代替 Optional
    ...

def get_users() -> list[str]:              # 3.9+ 内建容器作泛型
    ...

type UserId = int | str                    # 3.12+ PEP 695 type 关键字
```

- **运行时不强制检查类型**,靠 `mypy` / `pyright` / `ruff` 静态校验。
- 给函数加签名后,IDE 自动补全、重构、可读性都会大幅提升,**强烈建议任何新代码都写**。

详见 `python17 类型注解` 章节。

---

## 七、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md / 2.md | 示例 `def sum(num1, num2)` | 遮蔽内建 `sum`,工程上应避免,改名 `add` |
| 2 | 3.md §2 | "默认参数赋的值是根据位置而赋值的"…(其余表述基本正确) | 表述无误,但应强调"默认值在函数 **定义时** 求一次值"才是引发可变默认值陷阱的根因 |
| 3 | 4.md | 把传递机制类比为"C++ 的值传递 / 引用传递" | 不严谨,Python 是 **传对象引用(call by sharing)**,与 C++ 引用传递不同(C++ 引用传递可以让赋值 b=1000 影响外部) |
| 4 | 5.md | "lambda 拥有自己的命名空间,且不能访问自由参数列表之外或全局命名空间里的参数" | 错误。Lambda **可以** 访问外层作用域(它是闭包)。原文自己后面的"自由变量"例子就证明能访问。这条结论应删除 |

> 本章不涉及多文件调用。可变 vs 不可变在传参时的差异已用 ASCII 模型说明。
