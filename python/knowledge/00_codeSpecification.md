# 代码规范 知识点梳理

> 原文档:`python/Article/codeSpecification/codeSpecification_Preface.md` 及 `_first/_second/_third.md`
> 整理对象:PEP 8 与 PEP 257 的核心规则

---

## 一、简明概述

### 1. 文件编码

- 统一使用 **UTF-8** 编码。
- 一个 `.py` 文件中只放一个模块的内容,避免多个不相关模块挤在一起。

> ⚠️ 原文错误一(`codeSpecification_first.md` 第 6 行)
> 原文写"文件头部必须加入 `#-*-coding:utf-8-*-` 标识"。
> **正确说法**:Python 3 默认源码编码就是 UTF-8(PEP 3120),**完全不需要**这条魔法声明。这是 Python 2 遗留写法,加上反而显得过时。仅当确需使用非 UTF-8 编码时才写 `# -*- coding: <encoding> -*-`。

### 2. 代码格式

| 项目 | 规范 |
|------|------|
| 缩进 | 4 个空格,**禁止 tab 与空格混用** |
| 行宽 | 每行 ≤ 79(注释/docstring ≤ 72);PEP 8 允许团队约定放宽到 99/120 |
| 引号 | 单/双引号选一种保持一致即可,字符串中含双引号则用单引号包外层 |
| 空行 | 顶层函数/类之间空 2 行,类内方法之间空 1 行 |

> ⚠️ 原文错误二(`codeSpecification_first.md` 第 28~33 行)
> 原文区分 "自然语言用双引号、机器标识用单引号",并写 `u"你好世界"`。
> **正确说法**:
> 1. PEP 8 没有规定哪种语义用什么引号,只要求 **保持一致**。Black/Ruff 默认双引号。
> 2. Python 3 字符串本身就是 Unicode,`u"..."` 前缀虽然在 Python 3.3+ 重新允许,但 **没有任何作用**,不要再写。

### 3. import 语句

- 一个 import 一行:`import os` / `import sys` 分两行。
- 顺序分三组,组间一个空行:
  1. 标准库
  2. 第三方库
  3. 本项目模块
- 放在文件头部(模块 docstring 之后、全局变量之前)。
- **优先使用绝对导入**(absolute import)。包内部可使用 **显式相对导入**(`from . import x`),但 **禁止隐式相对导入**(Python 3 早已废除)。

> ⚠️ 原文错误三(`codeSpecification_first.md` 第 73~100 行前后矛盾)
> 原文先说"应该使用 absolute import,不推荐 `from ..bar import Bar`",紧接着又说"导入其他模块的类定义时,可以使用相对导入,如 `from myclass import MyClass`"。
> **正确解释**:
> - `from myclass import MyClass` —— 这其实是 **绝对导入**(顶层模块名),不是相对导入,原文混淆了概念。
> - `from . import x`、`from .sub import y` 才是 **显式相对导入**,在 **包内部** 允许使用,PEP 8 仅说"复杂包布局可优先使用显式相对导入"。
> - `from ..bar import Bar` 这种跨多层的相对导入是合法的,但应尽量减少。

### 4. 空格

- 二元运算符两侧各空一格:`a = b + c`,`x == y`,`x is not None`。
- 函数参数列表逗号后空一格:`def f(a, b, c):`。
- 函数 **默认值的 =** 两侧 **不加** 空格:`def f(a=1):`。
- 括号/方括号/花括号内侧不留空格:`spam(ham[1], {eggs: 2})`。
- 不要为了对齐而加额外空格。

### 5. 换行

- 长行优先使用 **括号隐式换行**(Python 在括号内自然续行)。
- 二元运算符换行时,**运算符放在新行行首**(Knuth 风格,PEP 8 现行推荐),例如:

```python
total = (first_long_variable
         + second_long_variable
         + third_long_variable)
```

- 反斜杠 `\` 续行能不用就不用,仅在 `with` 语句多个上下文管理器等无法用括号时使用。
- 禁止一行多语句(用 `;` 串联)。
- `if/for/while` 一定换行写,不要 `if x: do()` 写一行。

> ⚠️ 原文错误四(`codeSpecification_first.md` 第 217~219 行)
> 原文示例:
> ```python
> print 'Hello, ' \
>       '%s %s!' % \
>       ('Harry', 'Potter')
> ```
> 这是 **Python 2 语法**,Python 3 必须写成:
> ```python
> print('Hello, '
>       '%s %s!' % ('Harry', 'Potter'))
> ```
> 现代写法直接用 f-string:`print(f'Hello, {first} {last}!')`。

### 6. docstring

- 公共模块/函数/类/方法都写 docstring(PEP 257)。
- 单行 docstring:`"""一句话总结。"""` 三引号在同一行。
- 多行 docstring:首行不空,正文换行,结束 `"""` 独占一行。

---

## 二、注释

### 1. 行注释 / 块注释

- 块注释:`#` 后空一格,与所注释代码 **同缩进**。
- 行注释:与代码间至少 **2 个空格**,`#` 后再 1 空格。
- 不写 `x = x + 1  # x 加 1` 这种"复读机注释",注释要说明 **why**,不是 **what**。

### 2. 文档注释(Docstring)

- Google / Numpy / reStructuredText 三种主流风格,选一种,**全项目统一**。
- 不要把函数签名复制进 docstring(IDE 已经能显示签名)。
- 不要中英文混杂。
- 短能说清楚就别写长。

> ⚠️ 原文错误五(`codeSpecification_second.md` 第 115~116 行)
> docstring 示例里写 `print [x + 3 for x in a]`,这是 **Python 2 print 语句**。
> **正确写法**:`print([x + 3 for x in a])`。

---

## 三、命名规范

| 对象 | 规范 | 示例 |
|------|------|------|
| 模块名 | 全小写,可加下划线 | `html_parser`、`utils` |
| 包名 | 全小写,**尽量不带下划线** | `mypackage` |
| 类名 | 大驼峰(PascalCase / CamelCase) | `AnimalFarm`、`_PrivateFarm` |
| 函数/方法 | 全小写下划线分隔 | `run_with_env()` |
| 私有函数/属性 | 单下划线前缀 | `_internal` |
| name mangling | 双下划线前缀(无后缀) | `__secret` |
| 魔术方法 | 双下划线前后包裹 | `__init__` |
| 变量 | 全小写下划线分隔 | `school_name` |
| 常量 | 全大写下划线分隔 | `MAX_CLIENT = 100` |
| 类型变量 | 大驼峰 + 后缀 T 可选 | `T = TypeVar('T')` |

> ⚠️ 原文错误六(`codeSpecification_third.md` 第 77 行)
> 原文写 `Class FooBar:`,**关键字 `class` 必须小写**,这里是笔误。`Class` 写大写会报 `NameError`。

---

## 四、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | first §1 | 必须加 `#-*-coding:utf-8-*-` | Python 3 默认 UTF-8,无需该声明 |
| 2 | first §2.3 | 例子用 `u"你好世界"` | Python 3 字符串本身即 Unicode,`u` 前缀无作用 |
| 3 | first §3 | 把 `from myclass import MyClass` 说成相对导入 | 这是绝对导入;显式相对导入形如 `from . import x` |
| 4 | first §5 | `print 'Hello, ' \ ...` | Python 2 语法,Python 3 必须 `print(...)` |
| 5 | second §2 | docstring 内 `print [x+3 for x in a]` | Python 2 print 语句,3 必须括号 |
| 6 | third §5 | `Class FooBar:` | 关键字应小写 `class` |

---

## 五、补充:本章未提及但常用的现代约定

1. 用 **`ruff format`** 或 **`black`** 自动格式化,不再手工对齐空格。
2. 用 **`ruff`** / `flake8` + `isort` 自动管理 import 顺序。
3. **类型注解** 应作为函数签名标配:`def f(name: str) -> int:`(详见 python17 类型注解章节)。
4. **f-string** 是 Python 3.6+ 的格式化首选,而不是 `%` 或 `.format()`。

> 本章不涉及多文件调用,后续真正出现多文件调用的章节(python9 模块与包、python12 元类、python13 进程线程)会绘制 ASCII 执行流程图。
