# python10 Magic Method 知识点梳理

> 原文档:`python/Article/PythonBasis/python10/` 下 `1.md ~ 6.md`
> 整理对象:魔术方法分类、`__new__/__init__/__del__`、属性访问钩子、描述器、容器协议、运算符重载

---

## 一、什么是魔术方法

- 形如 `__name__`(双下划线包裹)的特殊方法,Python 用它们 **响应特定语法/内建函数调用**。
- 你 **几乎从不直接调用** 它们,而是通过对应语法触发:
  - `len(x)` → `x.__len__()`
  - `x[k]` → `x.__getitem__(k)`
  - `x + y` → `x.__add__(y)`,失败再试 `y.__radd__(x)`
  - `with x:` → `x.__enter__()` / `x.__exit__()`
  - `for i in x:` → `iter(x).__next__()` 循环
- 全部魔术方法见官方文档 [Data model](https://docs.python.org/3/reference/datamodel.html);本章只列常用的。

---

## 二、对象生命周期:`__new__` / `__init__` / `__del__`

### 1. 创建顺序

```
Cls(args)
    │
    ▼
1. type.__call__(Cls, args, kwargs)
    │
    ▼
2. obj = Cls.__new__(Cls, args, kwargs)   ← 真正构造对象,返回实例
    │
    ▼
3. if isinstance(obj, Cls):                ← 如果不是 Cls 实例,不调用 __init__
       Cls.__init__(obj, args, kwargs)
    │
    ▼
4. 返回 obj
```

### 2. `__new__` 要点

- 是 **类方法**(隐式 `@staticmethod` 处理),第一个参数 `cls`。
- 必须 **返回一个对象**,通常 `super().__new__(cls)`。
- 真正需要重写的少见场景:单例、不可变类型继承(`int/str/tuple` 子类)、元类配合。

### 3. `__init__` 要点

- 接收 `__new__` 返回的实例 + 构造参数,初始化属性。
- **不能返回非 None 的值**,否则 `TypeError`。
- 反序列化 `pickle.load` 默认 **不调用** `__init__`(走 `__reduce__/__setstate__`)。

### 4. `__del__` 要点

- 实例引用计数归零或 gc 回收时被调用,**不可依赖**。
- 释放资源用 `with`(上下文管理器)或 `weakref.finalize`,不要堆在 `__del__`。

---

## 三、属性访问钩子

| 钩子 | 触发时机 | 关键注意 |
|------|----------|----------|
| `__getattribute__(self, name)` | **每次** 属性访问都先经过 | 必须用 `super().__getattribute__(name)` 避免无限递归 |
| `__getattr__(self, name)` | 仅当 **正常查找失败** 才调用 | 用来动态返回属性,优雅捕获缺失 |
| `__setattr__(self, name, value)` | 给属性赋值时 | 内部 **必须用 `self.__dict__[name]=v`** 或 `super().__setattr__`,否则无限递归 |
| `__delattr__(self, name)` | `del obj.x` 时 | 同样要走 super 或 `__dict__` |
| `__dir__(self)` | `dir(obj)` 时 | 可定制属性列表 |

#### ASCII:属性查找顺序

```
obj.attr (含 obj.attr = v / del obj.attr)
       │
       ▼
   __getattribute__   ←─ 所有访问都走这里(若类定义了)
       │
       ▼ 默认实现:
   1) 找 type(obj).__mro__ 中是否有 data descriptor
   2) 找 obj.__dict__[attr]
   3) 找 type(obj).__mro__ 中是否有 non-data descriptor / 普通属性
       │
       ▼ 都没找到
   __getattr__       ←─ 兜底,只在前面失败时触发
       │
       ▼ 仍未提供
   AttributeError
```

> ⚠️ 原文错误一(`3.md` 第 14~19 行 示例)
> 原文示例 `__setattr__` 直接写 `self.name = value` 引发无限递归。**正解** 是 `self.__dict__[name] = value` 或 `super().__setattr__(name, value)`。原文虽然下面又给出了正解,但教学顺序应该先给正解再讲反模式,避免初学者复制错的。

---

## 四、描述器(Descriptor)

### 1. 协议

| 方法 | 含义 |
|------|------|
| `__get__(self, instance, owner)` | 读属性 |
| `__set__(self, instance, value)` | 写属性 |
| `__delete__(self, instance)` | 删属性 |

- 同时实现 `__get__` + `__set__` 或 `__delete__` 的叫 **data descriptor**(优先级最高,排在实例字典之前)。
- 仅实现 `__get__` 的叫 **non-data descriptor**(实例字典优先级高于它)。

### 2. 最常见的描述器:`@property`

```python
class C:
    def __init__(self, x): self._x = x

    @property
    def x(self):                     # getter
        return self._x

    @x.setter
    def x(self, v):                  # setter
        if v < 0: raise ValueError
        self._x = v
```

工程实践:**优先用 `@property`,只有需要在多个类间复用属性逻辑时才手写描述器**。

### 3. 描述器的"必须挂在类上"约束

描述器对象 **要作为类属性而不是实例属性** 才能生效。原文 `Distance.meter = Meter()`、`Distance.foot = Foot()` 是把描述器挂在类上,这样才会触发 `__get__/__set__`。

> ⚠️ 原文错误二(`4.md` 第 15 行)
> 原文写"只有在新式类中时描述器才会起作用"。
> **正确说法**:Python 3 中 **所有类都是新式类**,此句已经没有意义,建议直接删除或改为"由于 Python 3 所有类都自动继承 object,所以描述器协议总是生效"。

---

## 五、容器协议(自定义类做 list/dict)

| 想让对象支持 | 实现 |
|--------------|------|
| `len(obj)` | `__len__` |
| `obj[k]` | `__getitem__(k)` |
| `obj[k] = v` | `__setitem__(k, v)` |
| `del obj[k]` | `__delitem__(k)` |
| `k in obj` | `__contains__(k)`(没有则回退到 `__iter__`) |
| `for x in obj:` | `__iter__()` 返回迭代器 |
| `reversed(obj)` | `__reversed__()` 或自动从有序 `__len__/__getitem__` 推 |
| `next(obj)` | `__next__()`(对迭代器自身) |
| `obj()` 调用 | `__call__` |
| `with obj:` | `__enter__` / `__exit__`(详见 python21) |

判定一类:**不可变容器** 实现 `__len__ + __getitem__`,**可变容器** 再加 `__setitem__ + __delitem__`。

---

## 六、运算符重载

### 1. 算术运算符(部分)

| 魔术方法 | 触发 |
|----------|------|
| `__add__` / `__radd__` / `__iadd__` | `+`、`y + x`、`x += y` |
| `__sub__` / `__rsub__` / `__isub__` | `-`、`y - x`、`x -= y` |
| `__mul__` 等 | `*` |
| `__truediv__` | `/` |
| `__floordiv__` | `//` |
| `__mod__` | `%` |
| `__pow__` | `**` |
| `__matmul__` | `@`(矩阵乘,3.5+) |
| `__neg__/__pos__/__abs__/__invert__` | 一元 `-x / +x / abs(x) / ~x` |

> ⚠️ 原文错误三(`6.md` 表第 98 行)
> 原文仍列 `__div__`(Python 2 的 `/`)。
> **正确说法**:Python 3 已 **完全删除 `__div__`**,`/` 统一调用 `__truediv__`(真除),`//` 调用 `__floordiv__`。表里不应再保留 `__div__`。原文虽然在备注里提了一句,但既然是 Python 3 文档,表格直接删掉该条更清楚。

### 2. 比较运算符

| 方法 | 触发 |
|------|------|
| `__eq__` | `==` |
| `__ne__` | `!=`(默认会取 `__eq__` 的相反值,通常不必单独实现) |
| `__lt__` / `__le__` / `__gt__` / `__ge__` | `<` `<=` `>` `>=` |

更新:`__cmp__` 在 Python 3 **已删除**。需要全套比较运算又懒得写六个方法的,用 `@functools.total_ordering` 装饰类,只要实现 `__eq__` + 任意一个有序比较就能自动补齐其余:

```python
from functools import total_ordering

@total_ordering
class N:
    def __init__(self, v): self.v = v
    def __eq__(self, o): return self.v == o.v
    def __lt__(self, o): return self.v < o.v
```

### 3. 重写 `__eq__` 后的一条铁律

**重写 `__eq__` 会让 `__hash__` 自动变为 None**,实例无法放进 set / dict key。
若该类的实例应当可哈希,需要显式实现 `__hash__`(通常基于不可变属性):

```python
class Point:
    def __init__(self, x, y): self.x, self.y = x, y
    def __eq__(self, o): return (self.x, self.y) == (o.x, o.y)
    def __hash__(self): return hash((self.x, self.y))
```

---

## 七、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 2.md | "析构函数 `__del__` 由 Python 自己对对象进行垃圾回收时调用" | 描述基本对,但应补:循环引用时 `__del__` 可能 **不被立即调用**;关键资源释放应放 `with` |
| 2 | 3.md | `__setattr__` 内 `self.name = value` 示例触发无限递归 | 应优先给出正解 `self.__dict__[name]=value` 或 `super().__setattr__(name, value)`,反模式放后面 |
| 3 | 4.md | "只有在新式类中描述器才生效" | Python 3 所有类都是新式类,此句无意义,应删除 |
| 4 | 6.md | 表里列 `__div__` | Python 3 已删除,应只列 `__truediv__/__floordiv__` |
| 5 | 全章 | 没有提"重写 `__eq__` 会让 `__hash__` 变 None" | 是工程实际中最容易踩的坑,应补充 |

> 本章魔术方法虽然不一定跨文件,但属性查找顺序、`__call__` / 容器协议都属于"关键执行流",已用 ASCII 流程图表示。
