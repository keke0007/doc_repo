# python20 dataclass / Pydantic · 知识点梳理

> 原文档:`python/Article/PythonBasis/python20/1.md`
> 整理对象:`dataclass`、`field()`、`NamedTuple`、`TypedDict`、Pydantic v2、内外部边界模式

---

## 一、dataclass 基础

```python
from dataclasses import dataclass, field

@dataclass
class User:
    name: str
    age: int
    email: str = ""                  # 有默认值的字段放后面
    tags: list[str] = field(default_factory=list)  # 可变默认值必须用 default_factory
```

自动生成: `__init__`、`__repr__`、`__eq__`(按字段值比较，非按 id)。

### field() 常用参数

| 参数 | 说明 |
|------|------|
| `default` | 默认值(不可变类型) |
| `default_factory` | 默认值工厂(`list`, `dict`, `set` 等可变类型必须用这个) |
| `init=True` | 是否作为 `__init__` 参数 |
| `repr=True` | 是否出现在 `__repr__` |
| `compare=True` | 是否参与 `__eq__` 比较 |
| `hash=False` | 是否参与 `__hash__` |
| `metadata` | 自定义元数据(如 Pydantic `Field` 使用) |

### dataclass 装饰器参数

```python
@dataclass(frozen=True)           # 不可变实例(类似 namedtuple)
@dataclass(order=True)            # 自动生成 __lt__ __le__ __gt__ __ge__
@dataclass(kw_only=True)          # 3.10+: 强制关键字参数
@dataclass(slots=True)            # 3.10+: 使用 __slots__ 节省内存
```

## 二、`__post_init__` 与继承

```python
@dataclass
class Rectangle:
    width: float
    height: float
    area: float = field(init=False)   # 不出现在 __init__ 参数中

    def __post_init__(self):
        self.area = self.width * self.height
```

继承注意事项:
- 子类字段的默认值字段必须在父类默认值字段之后。
- 如果父类有默认值字段，子类所有字段都必须有默认值(对 `__init__` 签名顺序的约束)。

```python
@dataclass
class Animal:
    name: str = ""

@dataclass
class Dog(Animal):
    breed: str = ""   # ✅ 可以
    # breed: str       # ❌ 不行，因为 name 有默认值
```

## 三、NamedTuple 与 TypedDict

```python
from typing import NamedTuple, TypedDict

# NamedTuple — 不可变，支持索引
class Point(NamedTuple):
    x: float
    y: float

p = Point(1.0, 2.0)
print(p.x, p[0])   # 1.0  (两种访问方式)

# TypedDict — 为 dict 定义 key 类型
class UserDict(TypedDict):
    name: str
    age: int

d: UserDict = {"name": "Alice", "age": 30}
```

| 工具 | 可变 | 可迭代/索引 | 适用场景 |
|------|------|-----------|---------|
| `dataclass` | ✅(默认) | ❌ | 通用数据容器 |
| `frozen dataclass` | ❌ | ❌ | 不可变数据+方法 |
| `NamedTuple` | ❌ | ✅(tuple 兼容) | 小型不可变数据 |
| `TypedDict` | ✅ | ❌ | JSON/dict 类型标注 |

## 四、Pydantic v2 基础

```python
from pydantic import BaseModel, Field, EmailStr

class UserCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)   # ... = 必填
    age: int = Field(ge=0, le=150)
    email: EmailStr
    tags: list[str] = []

# 构造
u = UserCreate(name="Alice", age=25, email="alice@example.com")
u = UserCreate.model_validate({"name": "Alice", "age": 25, "email": "a@e.com"})
u = UserCreate.model_validate_json('{"name":"Alice","age":25,"email":"a@e.com"}')

# 导出
u.model_dump()                         # → dict
u.model_dump_json()                    # → JSON str
u.model_dump(exclude={"age"})          # 排除字段
```

### Field() 校验参数

```python
age: int = Field(ge=0, le=150)                       # 范围
email: EmailStr                                      # 格式(Pydantic 内置)
name: str = Field(min_length=1, max_length=100)      # 长度
pattern: str = Field(pattern=r"^\d{3}-\d{4}$")       # 正则
items: list[str] = Field(min_length=1, max_length=10)# 容器长度
```

### Pydantic v1 → v2 迁移要点

| v1 API | v2 API |
|--------|--------|
| `.dict()` | `.model_dump()` |
| `.json()` | `.model_dump_json()` |
| `.schema()` | `.model_json_schema()` |
| `.parse_obj(d)` | `.model_validate(d)` |
| `.parse_raw(s)` | `.model_validate_json(s)` |
| `@validator` | `@field_validator` |
| `@root_validator` | `@model_validator` |

## 五、dataclass vs Pydantic 边界(内外部模式)

```
┌──────────────────────────────────────────────────────────────┐
│                    内外部数据边界                               │
│                                                              │
│   外部世界(API/DB/文件)                                       │
│   ────────────────────  ─ ─ ─ ─ 边界 ─ ─ ─ ─ ─────────────  │
│   内部世界(业务逻辑)                                          │
│                                                              │
│   HTTP Request JSON                                           │
│       │                                                      │
│       ▼                                                      │
│   ┌──────────────────────┐                                   │
│   │ Pydantic BaseModel    │  ← 入口校验层                      │
│   │ - 类型校验 + 强制转换  │     (model_validate / Field)       │
│   │ - 自定义 validator    │                                    │
│   │ - JSON Schema 生成    │                                    │
│   └──────────┬───────────┘                                   │
│              │ model_dump() 或直接传字段                      │
│              ▼                                               │
│   ┌──────────────────────┐                                   │
│   │ dataclass / NamedTuple│  ← 内部业务数据                    │
│   │ - 轻量、快速          │     (无校验开销)                    │
│   │ - frozen=True(可选)   │                                    │
│   │ - 纯业务逻辑          │                                    │
│   └──────────┬───────────┘                                   │
│              │                                               │
│              ▼                                               │
│   业务处理 / 数据库操作 / 返回                                  │
│                                                              │
│   返回路径: dataclass → Pydantic model → JSON response        │
│                                                              │
│   原则:                                                       │
│   - Pydantic 在边界做"门卫":验证、清洗、转换外部数据            │
│   - dataclass 在内部做"信使":轻量传递已校验的可信数据           │
│   - 不要用 Pydantic 穿透所有内部层级                          │
└──────────────────────────────────────────────────────────────┘
```

---

## 六、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | `field(default=[])` 可变默认值 | 应用 `field(default_factory=list)`，否则所有实例共享同一个列表 |
| 2 | 1.md | 未提 `kw_only=True` `slots=True`(3.10+) | 这两个参数在生产代码中很实用 |
| 3 | 1.md | Pydantic v1 API(`.dict()` `.parse_raw()`) | 应强调 v2 API(`.model_dump()` `.model_validate_json()`) |
| 4 | 1.md | 未区分 dataclass/Pydantic 的角色 | 内外部边界模式是架构级最佳实践 |
| 5 | 1.md | `NamedTuple` 和 `TypedDict` 对比较松散 | 应明确场景区分: tuple 兼容 vs dict 类型标注 |
