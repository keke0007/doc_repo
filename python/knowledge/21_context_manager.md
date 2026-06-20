# python21 上下文管理器 · 知识点梳理

> 原文档:`python/Article/PythonBasis/python21/1.md`
> 整理对象:`with` 语句、`__enter__`/`__exit__`、`@contextmanager`、`ExitStack`、`async with`

---

## 一、`with` 语句的本质

```python
with expression as target:
    body
```

**等价于:**

```python
manager = expression
target = manager.__enter__()
try:
    body
except:
    if not manager.__exit__(*sys.exc_info()):
        raise      # __exit__ 返回 False → 异常继续传播
finally:
    # 注意: __exit__ 在无异常时也会被调用
    pass
```

> 实际上无异常时 `__exit__(None, None, None)` 也被调用，上面是简化表示。

## 二、类方式实现 `__enter__` / `__exit__`

```python
class FileManager:
    def __init__(self, filename, mode="r"):
        self.filename = filename
        self.mode = mode

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()
        return False   # False = 不抑制异常

with FileManager("data.txt", "w") as f:
    f.write("hello")
# f 自动关闭
```

`__exit__` 三参数:
- `exc_type`: 异常类(无异常为 `None`)
- `exc_val`: 异常实例
- `exc_tb`: traceback 对象
- 返回 `True` 抑制异常，`False` 继续传播

## 三、`@contextmanager` 装饰器(生成器方式)

```python
from contextlib import contextmanager

@contextmanager
def timer(name: str):
    start = time.perf_counter()
    yield          # ← 进入 with 体; 之前的代码 = __enter__
    elapsed = time.perf_counter() - start   # ← 之后的代码 = __exit__
    print(f"{name}: {elapsed:.3f}s")

with timer("work"):
    time.sleep(1)       # 输出: work: 1.001s
```

**必须用 `try/finally` 包裹 `yield`**，否则 `with` 体中抛异常时，`yield` 之后的清理代码不执行:

```python
@contextmanager
def managed():
    resource = acquire()
    try:
        yield resource
    finally:
        resource.release()   # 无论有没有异常都执行
```

## 四、3.10+ 多上下文(括号语法)

```python
# 3.9 及之前
with open("a.txt") as fa:
    with open("b.txt") as fb:
        ...

# 3.10+: 括号多上下文
with (
    open("a.txt") as fa,
    open("b.txt") as fb,
    open("c.txt") as fc,
):
    ...
```

## 五、contextlib 常用工具

| 工具 | 功能 |
|------|------|
| `closing(obj)` | 为有 `.close()` 的对象提供上下文(如 `urllib` 连接) |
| `suppress(*excs)` | 忽略指定异常 |
| `redirect_stdout(f)` | 重定向 stdout |
| `redirect_stderr(f)` | 重定向 stderr |
| `nullcontext()` | 无操作上下文占位(常用于条件性 with) |
| `chdir(path)` | 3.11+ 临时切换工作目录 |
| `ExitStack()` | 动态管理多个上下文(运行时数量不确定时) |

### 常用示例

```python
# suppress — 比 try/except pass 更语义化
from contextlib import suppress
with suppress(FileNotFoundError):
    os.remove("temp.txt")

# nullcontext — 条件性上下文
cm = open(fname) if fname else nullcontext()
with cm as f:
    ...

# ExitStack — 动态上下文
with ExitStack() as stack:
    files = [stack.enter_context(open(f)) for f in file_list]
    # 退出时按 LIFO 顺序关闭所有文件

# chdir (3.11+)
with chdir("/tmp"):
    print(Path.cwd())  # /tmp
```

## 六、`async with`(异步上下文)

```python
class AsyncSession:
    async def __aenter__(self):
        self.session = await create_session()
        return self.session

    async def __aexit__(self, *args):
        await self.session.close()

async with AsyncSession() as s:
    await s.get("https://...")
```

同样有 `@asynccontextmanager`:

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def session_scope():
    session = await create_session()
    try:
        yield session
    finally:
        await session.close()
```

## 七、`with` 语句执行流(ASCII 图)

```
with 语句完整执行时间线:

    with expression as target:
        │
        ▼
    manager = expression        ← 1. 对表达式求值，获取上下文管理器对象
        │
        ▼
    target = manager.__enter__() ← 2. 进入上下文，获取返回值(可选的 as 绑定)
        │
        ▼
    ┌─ body 执行 ─┐             ← 3. 执行 with 代码块
    │              │
    └──────────────┘
        │
        ├── body 无异常:
        │       ▼
        │   manager.__exit__(None, None, None)
        │       │
        │       ▼
        │   继续执行(with 块之后)
        │
        ├── body 有异常:
        │       ▼
        │   manager.__exit__(exc_type, exc_val, exc_tb)
        │       │
        │       ├── 返回 True: 异常被抑制，继续执行(with 块之后)
        │       │
        │       └── 返回 False/None: 异常继续向上传播
        │
        └── body 中有 return/break/continue:
                ▼
            __exit__ 仍会被调用! (类似 finally 的语义)


@contextmanager 生成器方式的 yield 边界:

    @contextmanager
    def my_cm():
        # = __enter__ 阶段
        setup()
        try:
            yield resource      ← 分界线: resource 绑定到 as 变量
        finally:                # finally 确保清理
            # = __exit__ 阶段
            teardown()
```

---

## 八、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 未强调 `@contextmanager` 必须用 try/finally 包裹 yield | 否则 with 体内异常会导致清理代码不执行 |
| 2 | 1.md | 未提 3.10+ 括号多上下文语法 | `with (a, b, c):` 比嵌套 with 更可读 |
| 3 | 1.md | 未提 `chdir()`(3.11+) | contextlib 新增的实用工具 |
| 4 | 1.md | `ExitStack` 示例不够突出 | 动态数量上下文(如批量打开文件)的最佳方案 |
| 5 | 1.md | 未提 `__exit__` 返回 True 会抑制异常 | 这是一个常见误区 |
