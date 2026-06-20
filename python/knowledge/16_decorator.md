# python16 装饰器 · 知识点梳理

> 原文档:`python/Article/PythonBasis/python16/1.md`
> 整理对象:装饰器原理、`@` 语法糖、参数化装饰器、多层装饰、`functools.wraps`

---

## 一、装饰器的本质

**装饰器就是一个接受函数、返回新函数的可调用对象。**

```python
def deco(func):
    def wrapper(*args, **kwargs):
        print(f"before {func.__name__}")
        result = func(*args, **kwargs)
        print(f"after {func.__name__}")
        return result
    return wrapper
```

### `@` 语法糖

```python
@deco
def greet():
    print("hello")

# 等价于:
# greet = deco(greet)
```

- `@deco` 发生在 **定义时**(模块加载时)，不是调用时。
- 返回值(`wrapper`)替换了原函数名绑定。

## 二、三层装饰器(带参数的装饰器)

```python
def repeat(times):              # 第1层：接收装饰参数
    def actual_decorator(func): # 第2层：接收被装饰函数
        @functools.wraps(func)
        def wrapper(*args, **kwargs):  # 第3层：替换原函数
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return actual_decorator

@repeat(3)
def say_hi():
    print("hi")

# 等价于: say_hi = repeat(3)(say_hi)
```

三层嵌套是参数化装饰器的标准写法。`repeat(3)` 返回 `actual_decorator`，`actual_decorator(say_hi)` 返回 `wrapper`。

## 三、`functools.wraps` — 必须使用

```python
import functools

def log(func):
    @functools.wraps(func)   # ← 保留元信息
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

`wraps` 做的事:
- 复制 `func.__name__`、`func.__doc__`、`func.__module__`、`func.__qualname__` 到 wrapper。
- 设置 `wrapper.__wrapped__ = func`，可通过此属性访问原函数。
- 复制 `func.__dict__` 中的内容到 `wrapper.__dict__`。

> ⚠️ 原文错误1(`1.md`)
> 原文装饰器示例未使用 `functools.wraps`，会丢失原函数文档和名称。
> **正确做法**:装饰器内部永远 `@functools.wraps(func)`。

## 四、多层装饰器的执行顺序

```python
@deco_a   # 第三步: greet = deco_a(deco_b(greet))
@deco_b   # 第二步: greet = deco_b(greet)
def greet():
    print("hi")

# 等价于:
# greet = deco_a(deco_b(greet))
```

**装饰顺序**:从下往上 → 最接近函数定义的装饰器先执行。
**调用顺序**:从上往下 → 最外层装饰器的 wrapper 先进、最后出(洋葱模型)。

```
调用 greet() 时的执行顺序(deco_a 在 deco_b 外层):
  deco_a.wrapper 进入
    → deco_b.wrapper 进入
      → 原始 greet 执行
    → deco_b.wrapper 退出
  → deco_a.wrapper 退出
```

## 五、装饰器执行流(ASCII 图)

```
模块加载阶段(@ 装饰发生在定义时，非调用时):

    import / 执行 .py 文件
        │
        ▼
    Python 解释器逐行执行
        │
        ▼
    遇到 @decorator(arg)         ← 如果装饰器带参数
        │
        ├─ step1: decorator(arg)  调用第一层，返回 actual_decorator
        └─ step2: actual_decorator(func)  调用第二层，返回 wrapper
              │                                    (此时 func 还未被调用!)
              ▼
        原函数名 = wrapper         ← 绑定替换
        │
        ▼
    遇到真正的函数调用 greet()
        │
        ▼
    实际执行的是 wrapper(*args, **kwargs)
        │
        ├─ wrapper 中可做前置处理
        ├─ 调用原函数 f(*args, **kwargs)  (通过闭包持有的 func)
        ├─ wrapper 中可做后置处理
        └─ return result



三层装饰器 @repeat(3) 的执行时间线:

定义时:
  @repeat(3)  →  repeat(3) 调用  →  返回 actual_decorator
  def say_hi  →  actual_decorator(say_hi)  →  返回 wrapper
  say_hi 这个名称现在指向 wrapper 函数对象

调用时:
  say_hi()  →  wrapper()  →  (循环3次)  →  原始 say_hi()
```

## 六、基于类的装饰器

```python
class CountCalls:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"call #{self.count}")
        return self.func(*args, **kwargs)

@CountCalls
def foo():
    pass
```

类装饰器的优势:可通过 `self` 保存状态，不需要 `nonlocal`。

## 七、内置常用装饰器

| 装饰器 | 作用 |
|--------|------|
| `@staticmethod` | 静态方法，无 `self`/`cls` |
| `@classmethod` | 类方法，首参为 `cls` |
| `@property` | 属性访问器(getter) |
| `@functools.lru_cache` | 函数结果缓存 |
| `@functools.singledispatch` | 单分派泛型函数 |
| `@dataclass` | 自动生成 `__init__`/`__repr__` 等 |
| `@contextmanager` | 生成器形式的上下文管理器 |

---

## 八、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 装饰器示例未用 `functools.wraps` | 生产代码必须使用，否则丢 `__doc__`、`__name__`，影响文档生成和调试 |
| 2 | 1.md | 未说明 `@` 发生在定义时而非调用时 | 这是理解装饰器最关键的时序概念 |
| 3 | 1.md | 缺失基于类的装饰器 | 维护状态的装饰器用类更清晰(避免 `nonlocal` 嵌套) |
| 4 | 1.md | 未讨论多层装饰器的洋葱调用顺序 | 实际项目中装饰器通常叠加使用(如 `@login_required` + `@rate_limit`)，顺序敏感 |
