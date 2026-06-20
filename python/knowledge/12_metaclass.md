# python12 元类 知识点梳理

> 原文档:`python/Article/PythonBasis/python12/` 下 `1.md ~ 5.md`
> 整理对象:类即对象、`type()` 既是函数又是元类、自定义元类、`metaclass=` 语法、应用场景

---

## 一、Python 里"类也是对象"

### 1. 一切皆对象

`int`、`str`、`function`、自定义类、模块……所有东西都是某个类的实例。
甚至 **类本身** 也是对象 —— 它是 **元类(metaclass)** 的实例。

```python
class Foo: pass

type(Foo)            # <class 'type'>   ← Foo 是 type 的实例
type(Foo())          # <class '__main__.Foo'>
```

### 2. 类作为对象,可以:

- 赋值给变量:`Alias = Foo`
- 作参数传递:`echo(Foo)`
- 在运行时动态创建:`type(name, bases, dict)`

---

## 二、`type()` 的双重身份

### 1. 一参形态:查看类型

```python
type(1)           # <class 'int'>
type('a')         # <class 'str'>
type(Foo)         # <class 'type'>
```

### 2. 三参形态:动态创建类

```python
Hello = type('Hello', (object,), {'hello': lambda self, name='Py': print('Hello,', name)})
Hello().hello()    # Hello, Py
```

参数:
- 类名(字符串)
- 父类元组(单父类要写成 `(object,)`,**末尾的逗号不能漏**)
- 命名空间字典(属性/方法)

`class C(B): ...` 这种语法,**本质上就是 Python 在背后调用 `type('C', (B,), {...})`**。

---

## 三、什么是元类

> **元类 = 创建类的类。**(Class of a class.)

- 普通对象的"class" = 它的类。
- 类对象的"class" = 它的元类。
- Python 中所有不显式指定元类的类,默认元类是 `type`。

### `__class__` 链

```python
n = 1
n.__class__           # <class 'int'>      n 是 int 的实例
n.__class__.__class__ # <class 'type'>     int 是 type 的实例
type.__class__        # <class 'type'>     type 是 type 自己的实例(自循环)
```

#### ASCII:对象-类-元类

```
     实例层                    类层                  元类层
   ┌───────────┐         ┌───────────┐         ┌───────────┐
   │  obj      │ ─实例-→ │   Cls     │ ─实例-→ │   type    │ ←─┐
   └───────────┘         └───────────┘         └───────────┘   │
                              ▲                                │
                              │                                │
                              └────  type 的"类"还是 type  ────┘
```

---

## 四、Python 3 自定义元类的正确语法

> ⚠️ 原文错误一(`4.md` 第 29~34 行)
> 原文写:
> ```python
> class MyObject(object):
>     __metaclass__ = something…
> ```
> **正确说法**:这是 **Python 2** 的语法。**Python 3 的正确写法** 是在 `class` 头部用关键字参数 `metaclass=`:
>
> ```python
> class MyObject(metaclass=MyMeta):
>     ...
> ```
>
> 在 Python 3 中,类体里写 `__metaclass__ = X` 完全 **不会生效**,Python 不会用它。

### 1. 写一个元类

```python
class UpperAttrMeta(type):
    def __new__(mcls, name, bases, namespace, **kwargs):
        # 把所有非 __xxx__ 的属性名改成大写
        upper_ns = {
            (k.upper() if not k.startswith('__') else k): v
            for k, v in namespace.items()
        }
        return super().__new__(mcls, name, bases, upper_ns)

class Foo(metaclass=UpperAttrMeta):
    bar = 'bip'

hasattr(Foo, 'bar')   # False
hasattr(Foo, 'BAR')   # True
Foo().BAR             # 'bip'
```

### 2. 类创建的"三阶段"

```
class Foo(Base, metaclass=UpperAttrMeta, **kw):
    body
        │
        ▼  (1) 准备命名空间
   UpperAttrMeta.__prepare__(name, bases, **kw)
        │ 返回一个 dict-like(可选,默认 {})
        ▼  (2) 执行 class 体,把名字写入命名空间
   exec(body, globals(), namespace)
        │
        ▼  (3) 真正构造类对象
   Foo = UpperAttrMeta(name, bases, namespace, **kw)
              │
              ├─ __new__:创建并填充类对象
              └─ __init__:对类对象做最后的初始化
        │
        ▼  (4) 把 Foo 绑定到当前作用域
```

### 3. `__metaclass__` 在 Python 3 里彻底废止

> ⚠️ 原文错误二(`4.md` 中的整段流程图)
> 原文的查找流程图描述"Python 在类里找 `__metaclass__`,找不到再去父类找,再去模块层找,最后用 type" —— 这是 Python 2 的语义。
> **Python 3 实际流程**:
> 1. **优先使用 class 头部的 `metaclass=` 关键字参数**。
> 2. 没指定的话,**从所有父类中取它们的元类的最派生类型(most derived metaclass)**;若父类有冲突会 `TypeError`。
> 3. 都没有,默认 `type`。
> **不再扫描类体里的 `__metaclass__` 也不再扫描模块层 `__metaclass__`**。

---

## 五、典型用法

### 1. 不需要时,不要用

Tim Peters 名言:"如果你需要问'我是不是该用元类',那你不该。"

> ⚠️ 原文错误三(`4.md` 例子与 `5.md` Django 例子)
> 原文代码片段中混用了 Python 2 `print 'x'`(无括号)、`__metaclass__ = ...`,在 Python 3 都不可运行。Django 现行版本(>=2.x)已经用 `models.Model` 的元类 `ModelBase`,但元类语法是 `class Model(metaclass=ModelBase):`,**不是** `__metaclass__ = ...`。

### 2. 现代替代:`__init_subclass__`

Python 3.6+(PEP 487)新增 `__init_subclass__`,**可以在不写元类的情况下** 拦截子类创建:

```python
class Plugin:
    registry = []

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        Plugin.registry.append(cls)

class A(Plugin): pass
class B(Plugin): pass

Plugin.registry           # [A, B]
```

90% 用元类做的"自动注册 / 自动校验"场景,都能用 `__init_subclass__` 优雅替代,**强烈优先考虑它**。

### 3. 仍然必须用元类的少数场景

- 控制 **类被调用** 时的行为(`type.__call__`)。
- 控制类的属性查找(`__getattribute__` on the metaclass)。
- 实现枚举(`EnumMeta`)、ABC(`ABCMeta`)、Django ORM 这种深度框架。

---

## 六、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 4.md 类体内 `__metaclass__ = ...` | 把它当作 Python 3 写法 | Python 3 必须用 `class X(metaclass=Meta):`;类体里的 `__metaclass__` 已 **完全失效** |
| 2 | 4.md 元类查找流程图 | 描述 Python 2 多级回退查找 | Python 3 只看 `metaclass=`,再看所有父类元类合并,否则默认 `type` |
| 3 | 4.md `print hasattr(...)` 示例 | `print` 不带括号 | Python 2 语法,Python 3 必须 `print(hasattr(...))` |
| 4 | 5.md Django 例子 | 描述用 `__metaclass__` 实现 ORM 钩子 | Django 现代版用 `class Model(metaclass=ModelBase):` |
| 5 | 全章 | 完全没提 `__init_subclass__` | 现代项目中 90% 的"伪元类需求"都该用这个,应作为首选推荐 |

> 元类涉及类创建过程跨多文件、跨多步骤,本章 ASCII 流程图覆盖了"类创建三阶段"这一关键执行流。元类驱动 Enum 详见 python11,元类与类继承的 ABC 抽象类详见标准库 `abc.ABCMeta`。
