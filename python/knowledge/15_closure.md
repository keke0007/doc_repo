# python15 闭包 · 知识点梳理

> 原文档:`python/Article/PythonBasis/python15/1.md`
> 整理对象:闭包定义、`nonlocal`、`__closure__`、应用场景

---

## 一、闭包的定义

**闭包 = 函数 + 捕获的自由变量(外部作用域中被引用但非全局的变量)**

```python
def outer():
    msg = "hello"          # 自由变量(对 inner 而言)
    def inner():
        print(msg)         # 捕获了 outer 的 msg
    return inner           # 返回 inner 函数本身

f = outer()
f()   # "hello" — outer 已返回，但 msg 仍"活着"
```

核心要点:内层函数引用了外层函数的局部变量,且外层函数已返回、其栈帧已销毁,但变量被闭包"留住"。

## 二、`global` vs `nonlocal` vs 默认

```python
x = 1       # 全局

def outer():
    x = 2   # outer 局部
    def inner():
        x = 3              # 创建 inner 的局部变量，不影响 outer 的 x
    inner()
    print(x)               # 2

def outer2():
    x = 2
    def inner():
        nonlocal x         # 声明 x 来自外层(非全局)作用域
        x = 3              # 修改的是 outer2 的 x
    inner()
    print(x)               # 3
```

| 关键字 | 绑定目标 |
|--------|---------|
| 无声明 + 赋值 | 创建当前函数局部变量 |
| `global x` | 绑定到模块级全局变量 |
| `nonlocal x` | 绑定到最近的外层(非全局)作用域变量 |

## 三、闭包的内部结构 `__closure__`

```python
def outer():
    a = 10
    b = 20
    def inner():
        return a + b
    return inner

f = outer()
print(f.__closure__)          # (<cell at 0x...: int ...>, <cell at 0x...: int ...>)
print(f.__closure__[0].cell_contents)  # 10
print(f.__closure__[1].cell_contents)  # 20
print(f.__code__.co_freevars)          # ('a', 'b')
```

- `__closure__`:由 `cell` 对象组成的元组，每个 cell 保存一个自由变量的值。
- `__code__.co_freevars`:自由变量名的元组。
- 被闭包捕获的变量存储在堆上(cell 对象)，不会随栈帧销毁而消失。

### 经典陷阱:循环中的闭包

```python
# ❌ 错误 — 所有 func 都引用同一个 i(cell 内容为最终值)
funcs = []
for i in range(3):
    funcs.append(lambda: i)

[f() for f in funcs]   # [2, 2, 2]  而非 [0, 1, 2]

# ✅ 修正 — 用默认参数"快照"当前值
funcs = []
for i in range(3):
    funcs.append(lambda i=i: i)

[f() for f in funcs]   # [0, 1, 2]

# ✅ 或者用外层函数包装(利用闭包每次创建新 cell)
def make_func(n):
    return lambda: n
funcs = [make_func(i) for i in range(3)]
```

> ⚠️ 原文错误1(`1.md`)
> 原文用 `global` 示例演示闭包计数器,但闭包本身不需要 `global`——用 `nonlocal` 才是正确的闭包模式。
> ✅ 计数器闭包应写成:
> ```python
> def counter():
>     count = 0
>     def inc():
>         nonlocal count
>         count += 1
>         return count
>     return inc
> ```

## 四、闭包的典型应用

| 场景 | 示例 |
|------|------|
| 工厂函数/配置器 | `def power(exp): return lambda base: base ** exp` |
| 装饰器 | 装饰器本身就是一个闭包(捕获被装饰函数) |
| 回调/事件处理器 | GUI 按钮的回调携带上下文数据 |
| 惰性求值/延迟计算 | 预填部分参数,稍后再调用 |
| 状态保持(替代类) | 少量状态无需定义完整类 |

---

## 五、闭包变量绑定执行流(ASCII 图)

```
调用 outer()
    │
    ▼
┌──────────────────────┐
│ outer 栈帧            │
│   msg = "hello"      │  ← 自由变量
│   def inner(): ...   │  ← inner 是 outer 内部定义的函数对象
│   return inner       │
└──────┬───────────────┘
       │ 返回 inner 函数对象
       ▼
┌──────────────────────┐
│ f = outer()          │  ← f 现在指向 inner 函数
│                      │     同时 inner.__closure__ 持有 cell(msg="hello")
│ outer 栈帧销毁        │     msg 的值已迁移到堆上的 cell 对象
└──────┬───────────────┘
       │
       ▼  调用 f()
┌──────────────────────┐
│ inner 执行            │
│   查找变量 msg:       │
│   → 本地(local) 找不到 │
│   → 闭包(closure) 找到  │  cell(msg) = "hello"
│   → 使用闭包中的值     │
│   print("hello")     │
└──────────────────────┘
```

```
自由变量生命周期:
  outer 调用 → msg 在栈上
  inner 定义 → Python 检测到 inner 引用 msg
              → 创建 cell 对象保存 msg 的引用
  outer 返回 → 栈帧销毁，但 cell 存活(被 inner.__closure__ 持有)
  f() 调用   → inner 从 __closure__[i].cell_contents 读取
  f 被删除   → cell 引用计数归零，回收
```

---

## 六、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 用 `global` 演示闭包计数器 | 闭包核心是 `nonlocal`，`global` 污染模块命名空间 |
| 2 | 1.md | 缺少 `__code__.co_freevars` 说明 | 配合 `__closure__` 可完整检查闭包捕获了哪些变量 |
| 3 | 1.md | 未提及循环中的闭包陷阱 `lambda: i` | 这是初学者最高频踩坑点，必须指出并给出修正 |
| 4 | 1.md | 闭包应用场景描写较少 | 补充了工厂函数、装饰器、回调、惰性求值四大场景 |
