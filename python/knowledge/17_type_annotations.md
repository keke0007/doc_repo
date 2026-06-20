# python17 类型注解(深度) · 知识点梳理

> 原文档:`python/Article/PythonBasis/python17/1.md`
> 整理对象:类型注解基础、`Union`/`Optional`、容器类型、`Protocol`、type alias、`NewType`、`mypy`/`pyright`

---

## 一、基本类型注解

```python
name: str = "Alice"
age: int = 30
ratio: float = 0.5
active: bool = True
data: bytes = b"hello"
```

函数注解:

```python
def greet(name: str) -> str:
    return f"Hello, {name}"

def process(items: list[str], config: dict[str, int] | None = None) -> bool:
    ...
```

- Python **运行时不做类型检查**，注解只用于静态检查器(mypy/pyright)和 IDE 提示。
- `-> None` 表示无返回值(函数返回 `None`)。
- 注解可通过 `__annotations__` 属性访问:`greet.__annotations__`。

## 二、联合类型与 Optional

### Python 3.10+ 推荐写法

```python
# 旧写法(3.9 及之前)
from typing import Union, Optional
x: Union[int, str]
y: Optional[str]        # Union[str, None]

# 新写法(3.10+)
x: int | str
y: str | None
```

> ⚠️ 原文差异1
> 原文大量使用 `Optional[X]` / `Union[X, Y]`。3.10+ `X | None` 和 `X | Y` 更简洁,是 PEP 604 标准写法。

### 类型收窄(Type Narrowing)

```python
def f(x: int | str) -> None:
    if isinstance(x, int):
        reveal_type(x)     # int — mypy 自动推断
    else:
        reveal_type(x)     # str
```

## 三、容器类型(3.9+ 内建泛型)

### Python 3.9+

```python
names: list[str] = ["a", "b"]
scores: dict[str, int] = {"a": 1}
point: tuple[float, float] = (1.0, 2.0)
ids: set[int] = {1, 2, 3}
```

### Python 3.8 及之前(需从 typing 导入)

```python
from typing import List, Dict, Tuple, Set
names: List[str] = ["a"]
```

> ⚠️ 原文差异2
> 原文使用 `List[str]` 等 typing 导入版本。3.9+ 内置 `list[str]` 是标准，typing 版本仅用于兼容老代码。

## 四、Callable 与 Any

```python
from typing import Callable, Any

# 接受 str 返回 int 的函数
handler: Callable[[str], int]

# 任意参数返回 None
callback: Callable[..., None]

# 任意类型 — 逃逸舱口，谨慎使用
x: Any = something()
```

`Any` 会关闭类型检查，是特殊类型(兼容所有类型，所有类型也兼容它)。生产代码应尽量用更精确的类型替代。

## 五、Protocol(结构化子类型 / 鸭子类型)

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()

class Circle:
    def draw(self) -> None:
        print("○")

render(Circle())   # 静态检查通过！Circle 实现了 Drawable 协议
```

- Protocol 是 PEP 544，无需显式继承即可满足类型要求(结构化子类型)。
- 运行时 `Protocol` 就是普通类，但静态检查器会按协议成员检查。

## 六、Type Alias 与 NewType

### Type Alias(类型别名)

```python
# 3.10+: TypeAlias(显式标记)
from typing import TypeAlias
Vector: TypeAlias = list[float]

# 3.12+: type 语句
type Vector = list[float]
type Point = tuple[float, float]
```

`type` 语句(PEP 695)是 3.12 起最推荐的别名方式。

### NewType(新类型，运行时为恒等函数)

```python
from typing import NewType

UserId = NewType("UserId", int)

def get_user(uid: UserId) -> str:
    ...

uid = UserId(42)
get_user(uid)       # ✅
get_user(42)        # ❌ mypy 报错：int 不是 UserId
```

`NewType` 在运行时就是 `lambda x: x`，零开销；但静态检查器将其视为独立类型。

## 七、其他 typing 常用工具

| 工具 | 说明 |
|------|------|
| `Final` | 禁止重定义/重赋值 |
| `Literal["a", "b"]` | 限定为指定字面值 |
| `TypedDict` | 为 dict 的 key 定义类型 |
| `TypeVar` | 泛型类型变量 |
| `Generic` | 泛型类基类 |
| `overload` | 函数重载签名 |
| `cast(T, x)` | 强制类型断言(运行时为空函数) |
| `assert_never(x)` | 穷举性检查(配合 `match`) |

### TypeVar 基本用法

```python
from typing import TypeVar

T = TypeVar("T")

def first(items: list[T]) -> T:
    return items[0]

reveal_type(first([1, 2, 3]))    # int
reveal_type(first(["a", "b"]))   # str
```

## 八、类型检查工具

| 工具 | 特点 |
|------|------|
| **mypy** | 经典、成熟；配置丰富 |
| **pyright** | 微软开发，速度快(用 Rust/node)；VS Code Pylance 底层 |
| **ruff** | 3.9+ 起也提供部分类型检查规则 |

```bash
mypy src/
pyright src/
```

```toml
# pyproject.toml (mypy)
[tool.mypy]
strict = true
```

## 九、类型检查流水线(ASCII 图)

```
┌──────────────────────────────────────────────────────────────┐
│                      开发时 / CI 阶段                          │
│                                                              │
│   .py 源文件                                                  │
│       │                                                      │
│       ▼                                                      │
│   ┌──────────┐     ┌──────────┐                              │
│   │  IDE 提示 │     │  mypy /  │                              │
│   │ (pylance)│     │  pyright │                              │
│   └────┬─────┘     └────┬─────┘                              │
│        │                │                                    │
│        │                ▼                                    │
│        │         ┌──────────────┐                            │
│        │         │ 类型检查报告  │                            │
│        │         │ ✅ 通过/❌ 错误│                            │
│        │         └──────────────┘                            │
│        │                                                     │
│        ▼                                                     │
│   ┌──────────┐                                               │
│   │ 代码补全  │                                               │
│   │ 跳转定义  │                                               │
│   │ 重构建议  │                                               │
│   └──────────┘                                               │
│                                                              │
│   Python 运行时: 类型注解完全不执行，零性能影响                  │
│   (可通过 typing.get_type_hints() 读取注解用于运行时逻辑)      │
└──────────────────────────────────────────────────────────────┘
```

```
Protocol 结构化子类型的检查流程:

    检查调用 render(Circle())
        │
        ▼
    检查 Circle 是否继承 Drawable?
        │
        ├─ 否 (显式继承检查失败)
        │
        ▼
    检查 Circle 是否在结构上满足 Drawable 协议?
        │
        ├─ Circle 是否有 draw(self) -> None 方法?  ✅
        │
        ▼
    通过! render(Circle()) 静态检查通过
```

---

## 十、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 大量 `Union[X, Y]` `Optional[X]` | 3.10+ 推荐 `X | Y` `X | None`(PEP 604) |
| 2 | 1.md | 容器类型用 `List[str]` `Dict[str, int]` | 3.9+ 内置 `list[str]` `dict[str, int]` |
| 3 | 1.md | Type Alias 用简单赋值 | 3.12+ 推荐 `type Vector = list[float]`(PEP 695) |
| 4 | 1.md | 未提及 `pyright` | pyright 已成为 VS Code 生态首选，速度快 |
| 5 | 1.md | Protocol 讲解不够突出 | 结构化子类型是 Python 类型系统的核心概念 |
| 6 | 1.md | 未强调 `cast()` 运行时为空函数 | 容易产生误导以为 cast 有类型转换能力 |
