# python7 迭代器和生成器 知识点梳理

> 原文档:`python/Article/PythonBasis/python7/` 下 `1.md ~ 5.md`
> 整理对象:可迭代对象、迭代器、列表生成式、生成器(yield)、反向迭代、zip 等综合用法

---

## 一、可迭代(Iterable)、迭代器(Iterator)、生成器(Generator)的关系

```
┌─────────────────────────────────────────────────────┐
│ Iterable  实现了 __iter__,可以被 for 循环消费       │
│   ├── list / tuple / str / dict / set / range / file│
│   └── 自定义类(实现 __iter__)                       │
│                                                     │
│ Iterator  实现 __iter__ + __next__,有状态、能记位置 │
│   ├── iter(iterable) 返回的对象                     │
│   ├── enumerate(...) / zip(...) / map(...) / filter │
│   └── 生成器(generator)                             │
│                                                     │
│ Generator  特殊的迭代器,用 yield 自动生成 __next__   │
│   ├── 生成器表达式 (x*x for x in ...)               │
│   └── 含 yield 的函数                               │
└─────────────────────────────────────────────────────┘
```

判定:
- `for x in obj:` 要求 obj 是 **Iterable**;Python 会先调 `iter(obj)` 拿到迭代器。
- `next(it)` 要求 it 是 **Iterator**。
- 所有 Iterator 同时也是 Iterable(`iter(iterator)` 返回自身)。

---

## 二、迭代

### 1. for 循环本质

```python
for x in obj:
    do(x)

# 等价于:
_it = iter(obj)
while True:
    try:
        x = next(_it)
    except StopIteration:
        break
    do(x)
```

### 2. 常见迭代姿势

```python
# 字符串、列表、元组
for c in 'abc': ...
for x in [1, 2, 3]: ...

# 字典:默认迭代 key
for k in d: ...
for v in d.values(): ...
for k, v in d.items(): ...

# 同时拿索引和元素
for i, x in enumerate(lst, start=1): ...

# 并行迭代多序列
for n, a in zip(names, ages): ...

# Python 3.10+ 严格 zip(长度不等就报错)
for n, a in zip(names, ages, strict=True): ...
```

> ⚠️ 原文遗漏(`5.md` §2)
> 原文介绍 `zip(a, b)` 时说"以最短的为准,遍历完即止"——这是 **默认行为(strict=False)**,**Python 3.10 起新增了 `strict=True` 参数**,长度不等时直接抛 `ValueError`。在工程中处理"理论应当等长"的两份数据,推荐 `strict=True` 防止悄无声息丢数据。

---

## 三、迭代器

### 1. 手工创建

```python
it = iter([1, 2, 3])
next(it)        # 1
next(it)        # 2
next(it)        # 3
next(it)        # StopIteration
```

### 2. 关键特性

- 一次性:迭代到末尾就耗尽,不能重置(重置需要 `iter(原对象)` 再来一次)。
- 状态保存:迭代器内部记住当前位置。
- 惰性:只在 `next()` 时才计算下一个值,适合大数据/无限流。

### 3. 自定义迭代器

```python
class Counter:
    def __init__(self, n): self.n = n; self.i = 0
    def __iter__(self): return self
    def __next__(self):
        if self.i >= self.n: raise StopIteration
        self.i += 1
        return self.i
```

更现代的做法是用 **生成器函数**(下面),代码更短。

---

## 四、列表生成式 / 推导式

### 1. 四种推导式

| 写法 | 类型 |
|------|------|
| `[x*x for x in r]` | list |
| `{x*x for x in r}` | set |
| `{k: v for k, v in pairs}` | dict |
| `(x*x for x in r)` | **生成器**,不是 tuple |

> 注意:`(x*x for x in r)` 是 **生成器表达式**,要得到 tuple 需写 `tuple(x*x for x in r)`。

### 2. 带过滤、嵌套

```python
[x*x for x in range(10) if x % 2 == 0]            # 过滤
[(x, y) for x in range(3) for y in range(5)]      # 嵌套循环
[[r*c for c in range(3)] for r in range(3)]       # 嵌套生成二维
```

### 3. 何时不要用推导式

- 单元素生成式可读性更差:`[x for x in items]` 直接写 `list(items)`。
- 副作用代码 **不要** 写进推导式(`[print(x) for x in items]` 反模式)。
- 复杂逻辑(超过 1 行 if/else 嵌套)用普通 for 循环更易读。

---

## 五、生成器(Generator)

### 1. 两种创建方式

```python
# (a) 生成器表达式
gen = (x*x for x in range(10))

# (b) 含 yield 的函数(更常用)
def fib(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b
```

### 2. 执行模型(关键)

普通函数被调用 → 立即执行 → 返回结果。
生成器函数被调用 → **不会立刻执行**,而是返回一个生成器对象。每次 `next()` 才前进到下一个 `yield`,挂起在那里,记住所有局部变量与执行位置。

#### ASCII:生成器 `yield` 暂停与恢复

```
def fib(n):
    a, b = 0, 1                     ← 第 1 次 next() 进入这里
    for _ in range(n):              ← 循环条件判断
        yield a                     ← 暂停点:把 a 抛出
                                    ← 下一次 next() 从这里恢复
        a, b = b, a + b
    # 函数自然结束 → 自动 raise StopIteration

时间线:
┌─────────────────────────────────────────────────────────────┐
│ caller            generator                       state     │
├─────────────────────────────────────────────────────────────┤
│ g = fib(5)        创建生成器,函数体未执行          a,b=未定 │
│ next(g)           执行到 yield a, 抛出 a=0          a=0,b=1 │
│ next(g)           恢复; a,b=b,a+b → 1,1; yield 1   a=1,b=1 │
│ next(g)           恢复; → 1,2; yield 1             a=1,b=2 │
│ next(g)           恢复; → 2,3; yield 2             a=2,b=3 │
│ next(g)           恢复; → 3,5; yield 3             a=3,b=5 │
│ next(g)           循环结束,函数 return → StopIter           │
└─────────────────────────────────────────────────────────────┘
```

### 3. 生成器的优势

- **节省内存**:`sum(x*x for x in range(10**8))` 不会创建一亿元素的 list。
- **惰性求值**:可表示无限序列(比如自然数流)。
- **解耦生产者与消费者**:把数据 **产生** 与 **使用** 分到两段代码。

### 4. 进阶:`yield from`、双向通信

```python
def chain(*iters):
    for it in iters:
        yield from it          # 把另一个可迭代对象"展平"产出
```

```python
def echo():
    while True:
        x = yield              # yield 也可以接收 send() 进来的值
        print('got', x)

g = echo(); next(g)            # 必须先 next 启动
g.send('hi')                   # 把 'hi' 当作 yield 表达式的值
```

### 5. 普通迭代器 vs 生成器

| 维度 | 迭代器类(__iter__+__next__) | 生成器函数(yield) |
|------|----------------------------|-------------------|
| 写法 | 10~20 行 | 3~5 行 |
| 状态保存 | 实例属性 | 局部变量(自动) |
| 异常处理 | 自己抛 StopIteration | 函数自然返回即可 |
| 实现 `send/throw` | 手工 | 内建支持 |

> 工程实践:**优先用生成器函数,除非你需要在迭代外部暴露大量状态/方法**。

---

## 六、反向迭代

```python
for x in reversed([1, 2, 3]): ...    # 3 2 1
```

- 仅当对象 **大小确定**(序列)或 **实现了 `__reversed__`** 才能 reverse。
- 生成器一般不支持反向(无法回放)。
- 自定义类支持反向迭代:

```python
class Countdown:
    def __init__(self, start): self.start = start
    def __iter__(self):
        n = self.start
        while n > 0: yield n; n -= 1
    def __reversed__(self):
        n = 1
        while n <= self.start: yield n; n += 1
```

---

## 七、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 5.md §2 | 介绍 `zip` 只提到默认行为 | Python 3.10+ 提供 `strict=True`,长度不等时报错 |
| 2 | 4.md §4 | 把"生成器函数的 print 改为 yield"作演示 | 演示没错,但应强调"生成器函数被调用时 **不会立即执行函数体**",这是最常被忽略的核心特性 |
| 3 | 4.md §4 | "继续 next 会报错" | 更准确表述:**自动抛出 `StopIteration` 异常**;for 循环会自动捕获并退出,手工 next 才看得见这个异常 |

> 本章无多文件调用,生成器的 yield 执行流已用 ASCII 时间线图说明。 
