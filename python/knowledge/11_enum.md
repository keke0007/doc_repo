# python11 枚举类 知识点梳理

> 原文档:`python/Article/PythonBasis/python11/` 下 `1.md ~ 4.md`
> 整理对象:`enum.Enum`、`IntEnum/StrEnum/Flag`、`@unique`、自动值 `auto()`、3.11+ `StrEnum`、3.12+ `@enum.member/@enum.nonmember`

---

## 一、为什么要枚举

朴素的"常量集合"做法:

```python
JAN, FEB, MAR = 1, 2, 3   # 不限制取值,容易传错;打印时丢失语义
```

枚举的优势:
- 取值集合在编译期固定,**单例**,可以 `is` 比较。
- 名字和值绑定,打印/调试可读性强。
- 与类型注解、`match/case` 配合良好。
- 标准库 `enum` 模块支持。

---

## 二、`Enum` 基本用法

### 1. 类语法(推荐)

```python
from enum import Enum

class Month(Enum):
    JAN = 1
    FEB = 2
    MAR = 3
    # ...
```

- 成员 `Month.JAN` 既是 Month 的实例,又是单例。
- `Month.JAN.name`  → `'JAN'`;`Month.JAN.value` → `1`。
- 遍历:`for m in Month: print(m)`,顺序按定义顺序。
- 成员表:`Month.__members__`(只读视图)。

### 2. 函数式语法(原文用的)

```python
Month = Enum('Month', ('JAN', 'FEB', 'MAR', 'APR'))   # 值自动从 1 递增
Month = Enum('Month', 'JAN FEB MAR APR')              # 也行
```

### 3. 自动值

```python
from enum import Enum, auto

class Color(Enum):
    RED = auto()
    GREEN = auto()
    BLUE = auto()
```

`auto()` 默认按 1, 2, 3 递增;可重写 `_generate_next_value_` 改成 `'red'`、`'green'` 字符串等。

### 4. `@unique` 装饰器

强制不允许重复值(默认允许 **别名**:`A = 1; ALIAS = 1` 会让 `ALIAS` 成为 `A` 的别名):

```python
from enum import Enum, unique

@unique
class Color(Enum):
    R = 1
    G = 2
    B = 1     # ValueError: duplicate values
```

---

## 三、枚举的比较

| 操作 | 行为 |
|------|------|
| `Month.JAN is Month.JAN` | True(单例) |
| `Month.JAN == Month.FEB` | False |
| `Month.JAN < Month.FEB` | **`Enum` 默认报 `TypeError`** |
| `Month.JAN == 1` | False(普通 Enum 与 int 不相等) |

如果需要 **可排序、可与 int 比较**,继承 `IntEnum`:

```python
from enum import IntEnum

class Level(IntEnum):
    LOW = 1
    MID = 5
    HIGH = 9

Level.LOW < Level.HIGH      # True
Level.LOW == 1              # True
```

但 `IntEnum` 会让枚举退化为 int,会丢失"类型独立性"的部分好处。**仅在确实要参与算术/排序时使用**。

---

## 四、枚举家族(3.11+ 现代化)

| 类 | 引入版本 | 用途 |
|----|---------|------|
| `Enum` | 3.4 | 通用枚举,值任意,默认不可与原始值比较 |
| `IntEnum` | 3.4 | 同时是 int,可参与算术、排序、与 int 比较 |
| `Flag` | 3.6 | 位标志组合(类似 `O_RDONLY \| O_WRONLY`) |
| `IntFlag` | 3.6 | Flag + 与 int 互操作 |
| `StrEnum` | **3.11+** | 同时是 str,序列化/JSON 友好 |
| `ReprEnum` | 3.11+ | 内部基类,定制 repr,通常不直接用 |

### `StrEnum` 例子(3.11+)

```python
from enum import StrEnum

class Color(StrEnum):
    RED = 'red'
    GREEN = 'green'

Color.RED == 'red'             # True
json.dumps({'c': Color.RED})   # '{"c": "red"}'
```

### `Flag` 例子

```python
from enum import Flag, auto

class Perm(Flag):
    READ = auto()       # 1
    WRITE = auto()      # 2
    EXEC = auto()       # 4
    ALL = READ | WRITE | EXEC

p = Perm.READ | Perm.WRITE
Perm.READ in p          # True
```

---

## 五、与 `match/case` 配合(Python 3.10+)

```python
def react(c: Color):
    match c:
        case Color.RED:    return 'stop'
        case Color.GREEN:  return 'go'
        case _:            return 'unknown'
```

注意 case 用的是 **限定名** `Color.RED`,不能写裸名 `RED`(那会被当作捕获模式)。

---

## 六、原理速看:元类驱动

`Enum` 的"成员创建即固化、单例、可遍历"等行为,本质上由 **元类 `EnumMeta`**(详见 python12)实现。简要流程:

```
class Color(Enum):
    RED = 1
    GREEN = 2
        │
        ▼ 类体被收集为命名空间
EnumMeta.__new__:
   ├─ 把 RED=1 / GREEN=2 当成"待提升的成员"
   ├─ 为每个成员构造 Color 类型的单例对象
   ├─ 写入 _member_map_(底层 OrderedDict-like)
   └─ Color 类装好后返回
        │
        ▼
访问 Color.RED:
   ├─ EnumMeta 拦截属性访问
   ├─ 命中 _member_map_['RED']
   └─ 返回那个单例
```

---

## 七、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | "Enum 的成员均为单例,**不可实例化**" | 表述不准确。**枚举类本身不能像普通类那样实例化**(`Color()` 会触发"按值查找成员"语义),但 **成员就是 Color 类的实例**。说"枚举不能再添加新成员、不能改值"更准确 |
| 2 | 3.md | 函数式语法 `Enum('Month', ...)` 和类语法并列出现却没注释 | 应明确:**首选类语法**,函数式语法主要用于动态生成枚举 |
| 3 | 全章 | 完全没提 `StrEnum`(3.11+)、`auto()`、`Flag`、与 `match/case` 配合 | 这些是现代项目中真正常用的写法,应补充 |
| 4 | 3.md 示例 | 月份用英文全称作 value,但 `Sep = 'September '` **尾部多一个空格** | 是原文的笔误,工程中会导致 `Month('September') == Month.Sep` 失败 |

> 本章不涉及多文件调用;`EnumMeta` 驱动机制属"关键执行流",已用 ASCII 流程图说明。
