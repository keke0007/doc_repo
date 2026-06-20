# python8 面向对象 知识点梳理

> 原文档:`python/Article/PythonBasis/python8/` 下 `1.md ~ 9.md`
> 整理对象:类与对象、实例方法/类方法/静态方法、init/del、继承、多态、访问控制(单/双下划线)

---

## 一、面向对象核心概念

### 1. 类与对象

- **类(class)**:对象的模板/蓝图,描述具有相同属性和行为的对象的集合。
- **对象(instance)**:类的具体实例,占据自己的内存,持有自己的属性值。
- **三大特性**:封装、继承、多态(顺序可换,本质是组织代码的三种工具)。

### 2. 一切皆对象

Python 中类本身也是对象(由元类 `type` 创造),所以类可以赋值给变量、做参数、做返回值。详见 python12 元类。

---

## 二、定义类

### 1. 基本写法

```python
class User:          # Python 3 无需写 (object),自动继承 object
    """类的 docstring。"""

    species = 'human'        # 类属性(所有实例共享)

    def __init__(self, name, age):
        self.name = name     # 实例属性(每个实例独立)
        self.age = age

    def greet(self):
        return f'Hi, I am {self.name}'
```

### 2. 三种方法

| 类型 | 装饰器 | 首参 | 调用方式 | 用途 |
|------|--------|------|----------|------|
| 实例方法 | (无) | `self` | `obj.m()` 或 `Cls.m(obj)` | 操作实例 |
| 类方法 | `@classmethod` | `cls` | `Cls.m()` 或 `obj.m()` | 操作类、备选构造器 |
| 静态方法 | `@staticmethod` | 无 | `Cls.m()` 或 `obj.m()` | 与类相关的纯函数 |

```python
class Date:
    def __init__(self, y, m, d): self.y, self.m, self.d = y, m, d

    @classmethod
    def from_string(cls, s):         # 备选构造器
        y, m, d = map(int, s.split('-'))
        return cls(y, m, d)

    @staticmethod
    def is_valid_year(y):            # 与类相关但不需要 self/cls
        return 1 <= y <= 9999
```

### 3. self / cls 不是关键字

只是 **约定俗成的参数名**,可以改成别的(强烈建议保持惯例,否则其他程序员会迷惑)。

> ⚠️ 原文错误一(`2.md` 第 63~123 行)
> 原文教学示例里直接写:
> ```python
> class ClassA():
>     def fun1():
>         print('我是 fun1')
> ClassA.fun1()       # 通过类直接调用
> ```
> **正确说法**:
> - 该写法之所以"能跑",是因为 Python 3 直接通过 **类** 调用未绑定函数时,不再自动传入 `self`(Python 2 会报 unbound method 错误)。
> - 它 **不是** "类方法",也不是"静态方法"。真要表达"不依赖实例的方法",**正确写法是加 `@staticmethod`**:
>   ```python
>   class ClassA:
>       @staticmethod
>       def fun1():
>           print('hi')
>   ```
> - 如果实例化后调用 `ClassA().fun1()`,**会因为多传了 `self` 而报错**。原文教学示例容易让初学者误解"方法可以不写 self"。

> ⚠️ 原文错误二(`3.md` 第 42~62 行)
> 原文写"如果没有声明 `@classmethod`,方法参数中就没有 `cls`,就没法通过 `cls` 获取到类属性"。然后给出失败例子 `def fun1(cls):` 通过 `ClassA.fun1()` 调用报缺参错误,把锅归给"没写 `@classmethod`"。
> **正确说法**:报错的真实原因是 —— 没有 `@classmethod` 时 `fun1` 就是普通函数(在类内即"实例方法"),`ClassA.fun1()` 通过类直接调用 **不会自动传入第一个参数**;`ClassA().fun1()`(先实例化)就能正常工作。把它当"实例方法"用是另一条合法路径,不必非要 `@classmethod`。

---

## 三、属性

### 1. 类属性 vs 实例属性

```python
class C:
    cls_attr = 'shared'           # 类属性

    def __init__(self, x):
        self.ins_attr = x         # 实例属性

a = C(1); b = C(2)

a.cls_attr        # 'shared' (从类查到)
b.cls_attr        # 'shared'
C.cls_attr = 'changed'
a.cls_attr        # 'changed' (类改了,所有实例都跟着改)

a.cls_attr = 'a only'   # 在实例上"赋值",其实是在 a 上 *新建* 同名实例属性
a.cls_attr        # 'a only'(实例属性遮盖类属性)
b.cls_attr        # 'changed'(b 仍然走类属性)
C.cls_attr        # 'changed'
```

#### ASCII:属性查找路径

```
访问 obj.attr 时:
   obj.__dict__  ──找到? ──Yes──> 返回
        │ No
        ▼
   type(obj).__dict__  ──找到? ──Yes──> 看是否描述符 / 普通值
        │ No
        ▼
   沿 MRO 向上找 base class.__dict__
        │ 全部没找到
        ▼
   触发 __getattr__(若定义),否则 AttributeError
```

### 2. 类属性的"踩雷点"

```python
class Box:
    items = []                     # ❌ 可变类属性

a = Box(); b = Box()
a.items.append(1)
b.items                            # [1] —— 因为 a/b 共享同一个 list
```

正确写法:把可变默认放进 `__init__`:

```python
class Box:
    def __init__(self):
        self.items = []
```

> ⚠️ 原文错误三(`5.md` §3)
> 原文说"修改实例属性不会影响类属性",结论对,但例子中 `a.var1 = '三点水'` 实际是 **在 a 上新建一个同名实例属性**,而不是"修改实例属性"。原文没区分"实例上新建属性"与"修改已有实例属性",对可变类属性的踩雷点没提醒。

---

## 四、方法的"重写/猴子补丁"

### 1. 修改类上的方法

```python
def new_greet(self):
    return 'patched'

C.greet = new_greet                # 直接替换类上的方法
a.greet()                          # 'patched'(所有实例都跟着变)
```

### 2. 想在 **单个实例** 上替换方法

直接 `a.greet = new_greet` 不能像类那样自动绑定 `self`,因为普通函数赋值到实例上 **不会触发描述符协议**。正确做法:

```python
import types
a.greet = types.MethodType(new_greet, a)
a.greet()           # 正确
```

> ⚠️ 原文错误四(`5.md` §4)
> 原文写 `a.fun1 = newFun1` 失败后下结论"不能重写实例方法"。
> **正确说法**:**可以重写**,但必须用 `types.MethodType` 或 `__get__` 进行绑定。原文的失败原因是 `newFun1(self)` 被当作普通函数赋给 `a.fun1`,调用 `a.fun1()` 时 Python **不会自动传 `a` 作为 `self`**,所以缺一个参数。结论应该改为"不能像类方法替换那样自动绑定 self,要手工绑定"。

---

## 五、初始化与销毁

### 1. `__init__`(初始化)

- 在实例 **已经创建之后** 被调用,负责初始化实例属性。
- 不能返回任何值(必须返回 `None`,否则报 `TypeError`)。
- 第一个参数固定为 `self`(名字约定)。

### 2. `__new__`(构造)

- **真正创建对象的方法**,先于 `__init__`。
- 通常不用重写,除非要做单例、不可变对象的子类化等。
- 返回的对象会被作为实例传给 `__init__`。详见 python10 Magic Method。

### 3. `__del__`(析构)

- 当对象的引用计数归零时被调用。
- **不是** "类销毁"时调用,而是"实例对象将被回收"时;并且循环引用时可能 **永远不被调用**(由 gc 兜底)。
- 一般 **不要在 `__del__` 里写关键资源释放**,改用 `with`(上下文管理器)。

> ⚠️ 原文错误五(`6.md` §2)
> 原文写"当一个类销毁的时候,就会调用析构函数"。
> **正确说法**:`__del__` 是当 **实例对象** 引用计数归零或被 gc 回收时调用,而不是"类销毁"。表述应改为"对象被回收时"。

### 4. Python 3 没有"新式类/旧式类"

`class C:` 与 `class C(object):` 完全等价(都是新式类),写不写 `(object)` 都行。

> ⚠️ 原文错误六(`6.md` §3)
> 原文说"Python 3 中所有类都是新式类",这是对的。但 **示例代码用 Python 2.7 演示旧式类的差异,对 Python 3 学习者无意义**,建议直接删掉这一段或仅保留一句结论。

---

## 六、继承

### 1. 单继承与多继承

```python
class Animal: ...
class Dog(Animal): ...                  # 单继承
class HybridDog(Animal, Loggable): ...  # 多继承
```

### 2. 父类方法调用 —— `super()`

```python
class UserVIP(User):
    def __init__(self, name, age, level):
        super().__init__(name, age)     # ✅ Python 3 推荐写法
        self.level = level
```

- Python 3 中 `super()` **无需参数**,自动识别当前类与实例。
- 比 `User.__init__(self, ...)` 显式调用更安全,适配多继承的 MRO(方法解析顺序)。

> ⚠️ 原文错误七(`7.md` §3)
> 原文写 `super(UserInfo2, self).__init__(...)`。
> **正确说法**:这是 Python 2 写法,Python 3 简化为 `super().__init__(...)`。在 Python 3 中两者结果相同,但官方推荐无参形式。

### 3. MRO 与多继承

- 多继承的属性/方法查找顺序由 **C3 线性化算法** 决定:`Cls.__mro__`。
- 必要时阅读 `Cls.mro()` 或 `print(Cls.__mro__)`。

### 4. isinstance / issubclass

```python
isinstance(obj, Cls)         # obj 是 Cls 或其子类的实例?
isinstance(obj, (A, B))      # 多种之一即可
issubclass(SubCls, Cls)      # SubCls 是 Cls 的(自身或)子类?
```

判断"恰好是 Cls"(不考虑子类)用 `type(obj) is Cls`,但绝大多数情况下应使用 `isinstance`。

---

## 七、多态

```python
class User:           def printUser(self): print('User')
class UserVIP(User):  def printUser(self): print('VIP')
class UserNorm(User): def printUser(self): print('Norm')

def show(u): u.printUser()   # 不关心具体类型,只关心是否有 printUser

show(UserVIP())     # VIP
show(UserNorm())    # Norm
```

Python 是 **鸭子类型**(duck typing):"如果它走路像鸭子、叫起来像鸭子,那它就是鸭子"。
正式接口约束推荐用 `typing.Protocol`(结构化子类型)。

---

## 八、访问控制

### 1. 三档命名约定

| 命名 | 工程含义 | Python 行为 |
|------|----------|-------------|
| `name` | 公共属性,可任意访问 | 真公共 |
| `_name` | 约定的私有,**不建议** 在外部使用 | 完全可访问,仅约定 |
| `__name` | name mangling(改名) | 类外访问需写 `_ClassName__name` |
| `__name__` | 魔术方法/属性,**特殊用途** | 不属于私有 |

### 2. name mangling 演示

```python
class U:
    def __init__(self):
        self.__secret = 1

u = U()
u.__secret              # AttributeError
u._U__secret            # 1  名字被改写为 _类名__属性
```

**不是真的私有**,只是给名字加了类名前缀,避免父子类同名冲突。

---

## 九、类的专有(魔术)方法速览

更详细见 python10 Magic Method 章节。本章原表有需要修正之处:

| 原文方法 | 修正 |
|----------|------|
| `__cmp__` | **Python 3 已删除**。改用 `__lt__/__le__/__eq__/__gt__/__ge__/__ne__`,或 `@functools.total_ordering` |
| `__div__` | **Python 3 已删除**。改用 `__truediv__`(`/`)与 `__floordiv__`(`//`) |
| `__repr__` | "打印、转换" → 描述应为:返回 **对开发者友好的、可重建对象的字符串表示**,`repr(obj)` 与 REPL 直接显示对象时调用 |

> ⚠️ 原文错误八(`9.md` §2 表)
> 原文表里 `__cmp__` / `__div__` 均为 Python 2 时代的写法,Python 3 已删除,新文档应直接列 Python 3 的实际魔术方法。

---

## 十、本章核心代码组织(单文件)

本章所有示例都在单文件内运行。真正进入"多文件协作"是 python9 模块与包章节,届时会画类 + 继承的跨文件 ASCII 流程图。

---

## 十一、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 2.md | 方法不写 `self`,通过类直接调用 | 行为依赖 Python 3 实现细节,正确教学应加 `@staticmethod` |
| 2 | 3.md | 没写 `@classmethod` 就无法访问类属性 | 实际是没实例化时调用,应通过 `obj.method()` 或加 `@classmethod` |
| 3 | 5.md | "修改实例属性不影响类属性"未指出"实例上新建同名属性"才是根因 | 应说明是"在实例上新建同名实例属性遮盖类属性" |
| 4 | 5.md | "不能重写实例方法" | 可以,但要用 `types.MethodType` 绑定 |
| 5 | 6.md | "类销毁时调用 __del__" | 是"实例对象引用计数归零/被 gc 回收时" |
| 6 | 6.md | 用 Python 2.7 演示新旧式类差异 | Python 3 无差异,该段对学习无用 |
| 7 | 7.md | `super(UserInfo2, self).__init__(...)` | Python 3 推荐 `super().__init__(...)` |
| 8 | 9.md 魔术方法表 | 列出 `__cmp__`、`__div__` | Python 3 已删除,应改为 `__lt__/__eq__/__truediv__/__floordiv__` 等 |
