# python9 模块与包 知识点梳理

> 原文档:`python/Article/PythonBasis/python9/` 下 `1.md ~ 5.md`
> 整理对象:模块(`.py` 文件)、import 各种姿势、`__name__`、包(目录+`__init__.py`)、命名空间包(3.3+)、模块搜索路径
> **本章是典型的多文件调用场景,会画 ASCII 执行流程图**

---

## 一、模块 Module

### 1. 什么是模块

- **一个 `.py` 文件就是一个模块**;模块名 = 不含 `.py` 后缀的文件名。
- 模块内部可以定义变量、函数、类、其他 import 的内容;模块也是一个对象(`types.ModuleType` 的实例)。
- 模块的作用:封装、复用、避免命名冲突。

### 2. 模块的三种来源

| 类型 | 例子 | 安装/管理方式 |
|------|------|---------------|
| 内建/标准库 | `os`、`sys`、`json`、`math`、`pathlib`、`re` | 随 Python 一起装,无需额外安装 |
| 第三方库 | `requests`、`numpy`、`pandas` | `pip install` / `uv add` |
| 自定义模块 | 自己写的 `myutil.py` | 放到搜索路径即可 |

---

## 二、import 的几种姿势

### 1. 基本形态

```python
import math                           # 整模块,使用 math.pi
import math as m                      # 别名

from math import pi                   # 仅导入名字,使用 pi
from math import pi, sin, cos         # 多个名字
from math import sin as sine          # 别名

from math import *                    # 导入所有 *公共* 名字 ❌不推荐
```

### 2. `from M import *` 的实际行为

- 导入模块 M 中 **不以下划线开头的所有名字**。
- 如果模块定义了 `__all__ = [...]`,则只导入列表中的名字。
- **强烈不推荐** 在工程代码中使用,会污染命名空间、让静态分析失效。

### 3. 包内的相对导入(3.3+ 隐式相对导入已废弃)

在包内部可以用 `.` / `..` 表示当前包/上一级包:

```python
from . import sibling          # 当前包的兄弟模块
from .submod import f          # 兄弟模块的成员
from ..utils import logger     # 上一级包的 utils 模块
```

注意:**相对导入只能在包内的模块中使用**,不能在被直接当脚本运行的文件里(`__name__ == '__main__'`)用。如果需要从命令行测试包内模块,用 `python -m package.module` 方式。

---

## 三、`__name__` 与主模块

### 1. 规则

- 当模块被 **直接执行**(`python xx.py`)时,`__name__ == '__main__'`。
- 当模块被 **别处 import** 时,`__name__ == 'xx'`(它的模块名)。

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

### 2. 为什么要写 `if __name__ == '__main__':`?

- 允许同一个文件既能 **作为脚本运行**(`python tool.py`),也能 **被 import 复用**(`from tool import f`)而不会自动执行脚本入口逻辑。
- import 一个模块会执行模块顶层代码 **一次**(并缓存到 `sys.modules`),所以无保护的脚本逻辑会在被 import 时意外触发。

---

## 四、包 Package

### 1. 普通包(Regular Package)

目录里有 **`__init__.py`** 文件就构成一个包。`__init__.py` 在包被首次 import 时执行,通常用于:
- 暴露/重组接口:`from .core import f` 提升到包顶层。
- 设置版本号、`__all__`、初始化日志等。

```
mypkg/
├── __init__.py        # 让 mypkg 成为一个包
├── core.py            # mypkg.core
└── utils/
    ├── __init__.py    # 让 mypkg.utils 成为子包
    └── log.py         # mypkg.utils.log
```

### 2. 命名空间包(Namespace Package,3.3+ PEP 420)

**没有 `__init__.py`** 也能形成包(称为命名空间包),允许将同一逻辑命名空间分散到多个物理目录。

> ⚠️ 原文错误一(`4.md` 第 30 行)
> 原文写"每一个包目录下面都会有一个 `__init__.py` 的文件,**因为这个文件是必须的,否则 Python 就把这个目录当成普通目录,而不是一个包**"。
> **正确说法**:这是 **Python 2 与 3.2 之前** 的规定。Python 3.3+(PEP 420)起,**没有 `__init__.py` 的目录也可以被识别为命名空间包**。但对绝大多数工程,推荐 **保留 `__init__.py`**:可以放初始化逻辑、暴露公共接口、IDE/静态分析友好。

---

## 五、模块搜索路径

Python 找模块的顺序:

```
1. 内建模块(sys.builtin_module_names)
2. 冻结模块(_frozen_importlib 等)
3. sys.path 列出的目录,从前到后:
     a) 当前脚本所在目录(或 -m 启动的当前工作目录)
     b) PYTHONPATH 环境变量声明的目录
     c) 安装相关的默认目录(site-packages 等)
4. 都找不到 → ModuleNotFoundError
```

查看自己机器上的搜索路径:

```python
import sys
print(sys.path)
```

修改搜索路径(临时,不推荐):`sys.path.insert(0, '/some/dir')`。

**推荐管理路径的现代方式**:用 `uv` / `pip` 把项目装成可编辑包(`uv pip install -e .` 或 `pip install -e .`),自动加入 site-packages。

---

## 六、import 的执行流程(ASCII)

下面用一个最小工程演示 import 跨文件调用的完整流程。

### 项目结构

```
project/
├── main.py                      # 入口
└── mypkg/
    ├── __init__.py
    ├── greet.py                 # 定义 say_hi
    └── utils/
        ├── __init__.py
        └── log.py               # 定义 log
```

`mypkg/__init__.py`:
```python
print('[init] mypkg loaded')
from .greet import say_hi        # 提升到包顶层
```

`mypkg/greet.py`:
```python
from .utils.log import log

def say_hi(name):
    log(f'say hi to {name}')
    print(f'Hello, {name}!')
```

`mypkg/utils/log.py`:
```python
def log(msg):
    print(f'[log] {msg}')
```

`main.py`:
```python
import mypkg
mypkg.say_hi('小明')
```

### 执行流程图

```
python main.py
      │
      ▼
┌──────────────────────────────────────────┐
│ 解释器启动                                 │
│   - 把 main.py 所在目录加入 sys.path[0]   │
└──────────────────────────────────────────┘
      │
      ▼ 执行 main.py
┌──────────────────────────────────────────┐
│ import mypkg                              │
│   1. 在 sys.modules 找 'mypkg' ──→ 没有   │
│   2. 沿 sys.path 找到 mypkg/__init__.py   │
│   3. 创建 mypkg 模块对象,登记 sys.modules │
│   4. 执行 mypkg/__init__.py 顶层代码:     │
│        ┌─────────────────────────────┐   │
│        │ print('[init] mypkg loaded')│   │
│        │ from .greet import say_hi   │   │ ←─ 触发链式 import
│        └─────────────────────────────┘   │
└──────────────────────────────────────────┘
      │
      │  链式导入
      ▼
┌──────────────────────────────────────────┐
│ from .greet import say_hi                 │
│   1. import mypkg.greet                   │
│   2. 创建 mypkg.greet 模块对象            │
│   3. 执行 greet.py 顶层:                  │
│        ┌────────────────────────────────┐│
│        │ from .utils.log import log     ││ ←─ 再链一次
│        │ def say_hi(name): ...          ││
│        └────────────────────────────────┘│
└──────────────────────────────────────────┘
      │
      ▼
┌──────────────────────────────────────────┐
│ from .utils.log import log                │
│   1. import mypkg.utils (执行其 __init__) │
│   2. import mypkg.utils.log               │
│   3. 执行 log.py 顶层 (定义 log 函数)     │
│   4. 把 log 名字绑定到 greet 模块命名空间 │
└──────────────────────────────────────────┘
      │
      ▼ 回到 mypkg/__init__.py
┌──────────────────────────────────────────┐
│ say_hi 名字也提升到 mypkg 命名空间        │
└──────────────────────────────────────────┘
      │
      ▼ 回到 main.py
┌──────────────────────────────────────────┐
│ mypkg.say_hi('小明')                      │
│   ① 查找 mypkg.say_hi (在 mypkg 上)       │
│   ② 调用 → 进入 greet.say_hi              │
│   ③ greet 内调用 log('say hi to 小明')    │
│        ↓                                 │
│        log.py 中的 log 打印               │
│   ④ 打印 'Hello, 小明!'                   │
└──────────────────────────────────────────┘
```

### 模块缓存(sys.modules)

```
sys.modules 在程序进程内全局:
  ┌─────────────────────┬─────────────────────────┐
  │ 'sys'               │ <module 'sys'>          │
  │ 'mypkg'             │ <module 'mypkg'>        │
  │ 'mypkg.greet'       │ <module 'mypkg.greet'>  │
  │ 'mypkg.utils'       │ <module 'mypkg.utils'>  │
  │ 'mypkg.utils.log'   │ <module 'mypkg.utils.log'> │
  └─────────────────────┴─────────────────────────┘

第二次 import 同一模块时:
  - 不会重新执行顶层代码
  - 直接从 sys.modules 取出已有对象
  - 这也是为什么模块顶层逻辑只会运行一次
```

需要"强制重载"用:
```python
import importlib, mypkg
importlib.reload(mypkg)        # 慎用,常用于热更新或交互调试
```

---

## 七、作用域与公开/私有约定

| 命名 | 含义 |
|------|------|
| `name` | 公开,模块外可用 |
| `_name` | 约定的私有,**不应** 被外部使用 |
| `__name` | 类中触发 name mangling(参考 python8) |
| `__name__` | 特殊属性/方法,Python 保留 |

模块层级的"私有"完全靠约定;`from M import *` 默认不会导入 `_xxx`,但显式 `from M import _xxx` 仍然可以。

**`__all__`** 控制 `from M import *` 的导出列表:

```python
# mymod.py
__all__ = ['public_api']

def public_api(): ...
def _helper(): ...
```

---

## 八、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 4.md | 包必须有 `__init__.py`,否则当普通目录 | Python 3.3+ 起没有 `__init__.py` 也能识别为 **命名空间包**(PEP 420),工程中仍建议保留 |
| 2 | 1.md 项目结构注释 | 反复使用 `#!/usr/bin/env python` + `# -*- coding: UTF-8 -*-` | shebang 写 `python3` 更稳;coding 声明 Python 3 不需要 |
| 3 | 3.md §1 | "如果一个函数没有调用其他函数,我们称这种函数为非主函数" | 这不是 Python 的官方定义。主模块/非主模块的区分 **完全是看是否被直接执行**,与"是否调用其他函数"无关。原文用类比函数过度引申了 |

> 本章是典型多文件调用场景,已绘制详细 ASCII 执行流程图(import 链式触发、sys.modules 缓存、跨模块调用)。
