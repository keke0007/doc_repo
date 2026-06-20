# python1 第一个 Python 程序 知识点梳理

> 原文档:`python/Article/PythonBasis/python1/` 下 `Preface / Introduction / Installation / The_first_procedure / IDE`
> 整理对象:Python 的来源、安装、运行第一个程序、IDE 选择

---

## 一、Python 简介

### 1. 起源

- **作者**:Guido van Rossum(吉多·范罗苏姆,昵称"龟叔"),荷兰人。
- **诞生时间**:1989 年圣诞节期间打草稿,**1991 年 2 月** 发布 0.9.0,1994 年 1.0 正式版。
- **名字来源**:取自英国喜剧团体 Monty Python(蒙提·派森),与"蟒蛇"无关。

### 2. 语言特点

| 特性 | 说明 |
|------|------|
| 解释型 | 由 CPython 等解释器逐行翻译成字节码 → 虚拟机执行,**不预编译为机器码** |
| 动态类型 | 变量绑定的对象类型在运行时确定,可重新绑定到不同类型 |
| 强类型 | `'1' + 1` 会直接 `TypeError`,不会隐式转型 |
| 跨平台 | Windows / macOS / Linux 等都有官方解释器 |
| 自带电池 | 标准库覆盖网络、文件、压缩、邮件、GUI、SQLite 等,开箱即用 |
| 一切皆对象 | 函数、类、模块、整数都是对象,都有属性和方法 |

### 3. 缺点

- 运行速度比 C/C++ 慢(纯计算密集场景常慢 10×~100×),需要时可用 C 扩展、Cython、Numba、PyPy 缓解。
- 源码即发布,无法天然加密。可选 `pyarmor`、`Cython` 编译成 `.so`,但都只是混淆,不是真正加密。

> ⚠️ 原文错误一(`Introduction.md` 第 42 行)
> 原文写"Python 是解释型语言,你的代码在执行时会一行一行地翻译成 CPU 能理解的机器码"。
> **正确说法**:CPython 实际是 **先把源码编译成字节码(`.pyc`),再由 Python 虚拟机(PVM)逐条解释执行字节码**,字节码并不是 CPU 机器码。"一行一行翻译成机器码"是不准确的简化说法。真正能直接生成本地机器码的是 PyPy(JIT)或者 Cython 那种工具。

---

## 二、Python 版本与安装

### 1. 版本说明

- Python 2.x 已于 **2020-01-01 EOL**,不要再学。
- Python 3.x 当前主线:**3.12 / 3.13**(本教程基于 3.10+)。
- 同一台机器可以同时装多个版本,Windows 用 `py -3.12` 选择,macOS/Linux 用 `python3.12` 选择,**强烈推荐用 `uv` 或 `pyenv` 管理多版本**(详见 python23)。

> ⚠️ 原文错误二(`Installation.md` 第 3、44 行)
> 原文写"目前 Python 有两个版本,2.x 和 3.x,这两个版本是不兼容的""安装 3.6.1 / 3.7"。
> **现状(2026)**:2.x 早已退役,直接装最新稳定版即可(写本文时为 3.13)。macOS 自带的 `python` 在 macOS 12.3+ 已经移除,系统不再预装 Python 2。

### 2. Windows 安装要点

1. 官网 [python.org](https://www.python.org/) 下载安装器。
2. **务必勾选 `Add Python to PATH`**,否则需要手工配置环境变量。
3. 建议同时勾选 `Install launcher for all users`,这样可用 `py -3.13` 选择多版本。
4. 验证安装:`python --version` 应输出版本号。

### 3. macOS 安装要点

- 官网 pkg 安装器装入 `/Library/Frameworks/Python.framework/Versions/<X.Y>/bin/`。
- 或者 `brew install python@3.13`(更推荐,便于升级)。
- 配置环境变量:
  - **新版 macOS(10.15+)默认 zsh**,要写 `~/.zshrc` 而 **不是** `~/.bash_profile`。
  - 如果使用 bash,才写 `~/.bash_profile`。

```bash
# zsh(macOS 默认)
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

> ⚠️ 原文错误三(`Installation.md` 第 54 行)
> 原文一律让用户写 `~/.bash_profile`。这在 macOS Catalina(10.15)之后已经不适用,因为系统默认 shell 改为 **zsh**,正确文件是 `~/.zshrc`。

> ⚠️ 原文错误四(`Installation.md` 第 60、63 行)
> 原文写路径 `/Library/Frameworks/Python. Framework/Versions/3.7/bin`,**`Python. Framework` 中间多了一个空格**,正确写法是 `Python.framework`(且 framework 是小写 f)。

### 4. Linux 安装要点

- 多数发行版自带 Python 3。
- Debian / Ubuntu:`sudo apt install python3 python3-pip python3-venv`。
- 仍然推荐用 `pyenv` 或 `uv` 管理多版本,避免和系统 Python 冲突。

---

## 三、第一个 Python 程序

### 1. 三种运行方式

| 方式 | 命令 | 适用场景 |
|------|------|----------|
| 交互式 REPL | `python` 进入 `>>>` | 临时调试、试一两行代码 |
| 脚本运行 | `python hello.py` | 完整脚本 |
| 模块运行 | `python -m mymod` | 运行包内模块,支持相对导入 |

### 2. Hello World

新建 `hello.py`:

```python
print('Hello Python')
```

执行:

```bash
python hello.py
```

### 3. 多文件执行视角(ASCII 流程图)

虽然 Hello World 只有一个文件,但即使最简单的运行也涉及多个组件协作。流程如下:

```
┌─────────────┐    1. 启动     ┌──────────────────┐
│  你的终端    │ ─────────────▶ │ python 解释器   │
└─────────────┘                 │  (CPython 可执行)│
                                └──────────────────┘
                                         │
                              2. 读取并解析 hello.py
                                         ▼
                                ┌──────────────────┐
                                │  词法/语法分析   │
                                │  AST(抽象语法树)│
                                └──────────────────┘
                                         │
                                3. 编译为字节码
                                         ▼
                                ┌──────────────────┐
                                │  hello.pyc 字节码│
                                │  (可能缓存到     │
                                │  __pycache__/)   │
                                └──────────────────┘
                                         │
                                4. 字节码送入虚拟机
                                         ▼
                                ┌──────────────────┐
                                │ Python Virtual   │
                                │ Machine (PVM)    │
                                │ 逐条执行字节码    │
                                └──────────────────┘
                                         │
                          5. 调用内建函数 print()
                                         ▼
                                ┌──────────────────┐
                                │  builtins.print  │
                                │  写到 sys.stdout │
                                └──────────────────┘
                                         │
                                6. 输出到终端
                                         ▼
                                  Hello Python
```

> 说明:`print` 是内建函数,在解释器启动时已加载到 `builtins` 命名空间;`sys.stdout` 是 `io.TextIOWrapper`,默认指向终端。

---

## 四、IDE 选择

| IDE | 特点 | 适用人群 |
|-----|------|----------|
| PyCharm | JetBrains 全家桶,重型,免费社区版 | 中大型工程、Django/Flask |
| VS Code + Python 插件 | 轻量、生态丰富、远程开发强 | 大多数人(2026 年主流) |
| Cursor / Trae / Windsurf | 内置 AI 编程,可调本地或云端模型 | 偏向 AI 辅助开发的新趋势 |
| Sublime Text | 极轻量纯文本 | 临时小脚本 |
| Jupyter Notebook | 单元格执行 + 可视化 | 数据分析、机器学习 |
| IDLE | Python 自带,极简 | 初学纯尝鲜 |

> ⚠️ 原文遗漏
> `IDE.md` 只推荐了 PyCharm,但 2026 年 **VS Code + Python 扩展** 已经是更主流的选择,带 AI 助手的编辑器(Cursor / Trae)更接近当下趋势。对入门者直接推荐 VS Code 更稳妥。

---

## 五、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | Introduction §最后 | Python 一行一行翻译成 CPU 机器码 | CPython 先编成字节码,再由 PVM 解释执行 |
| 2 | Installation §1/§2 | 装 3.6.1 / 3.7 | 装最新稳定版(本文写作时 3.13) |
| 3 | Installation §2 | 配置 `~/.bash_profile` | macOS 10.15+ 默认 zsh,应写 `~/.zshrc` |
| 4 | Installation §2 | 路径 `Python. Framework` | 应为 `Python.framework`,无多余空格、小写 f |
| 5 | IDE | 只推荐 PyCharm | 2026 主流是 VS Code + Python 插件,可补充 AI 增强编辑器 |
