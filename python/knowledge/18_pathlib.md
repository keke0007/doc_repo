# python18 pathlib · 知识点梳理

> 原文档:`python/Article/PythonBasis/python18/1.md`
> 整理对象:`pathlib` 模块 — `Path` 对象、路径操作、文件读写、目录遍历、路径属性

---

## 一、Path 对象 vs os.path

```python
from pathlib import Path

# os.path 风格(不推荐新代码)
import os
base = os.path.join(os.path.dirname(__file__), "data", "config.json")

# pathlib 风格(✅ 推荐)
base = Path(__file__).parent / "data" / "config.json"
```

| os.path | pathlib |
|---------|---------|
| 返回字符串 | 返回 `Path` 对象 |
| 函数式调用 | 链式调用，`/` 运算符拼接 |
| 跨平台处理繁琐 | 自动处理 `/` 和 `\` |
| 需要 `open()` 包装 | 内置 `.read_text()` `.write_bytes()` |

## 二、路径创建与拼接

```python
from pathlib import Path

# 创建
p = Path(".")                    # 相对路径
p = Path("/home/user/data")      # 绝对路径
p = Path.home()                  # 用户主目录
p = Path.cwd()                   # 当前工作目录

# 拼接 — 用 / 运算符
config = Path("src") / "app" / "config.py"
# Windows: src\app\config.py
# Linux:   src/app/config.py

# 展开 ~
p = Path("~/docs").expanduser()
```

## 三、路径属性(只读)

```python
p = Path("/home/alice/docs/report.txt")

p.name          # "report.txt"
p.stem          # "report"
p.suffix        # ".txt"
p.suffixes      # [".tar", ".gz"]  对于 "archive.tar.gz"
p.parent        # Path("/home/alice/docs")
p.parents       # [Path("/home/alice/docs"), Path("/home/alice"), Path("/home"), Path("/")]
p.anchor        # "/" (Windows: "C:\\")
p.parts         # ("/", "home", "alice", "docs", "report.txt")
p.is_absolute() # True / False
p.drive         # Windows: "C:", Linux: ""
```

## 四、文件 I/O(直接读/写)

```python
p = Path("data.txt")

text = p.read_text(encoding="utf-8")      # 读取整个文件为 str
p.write_text("hello", encoding="utf-8")   # 写入 str

data = p.read_bytes()                     # 读取整个文件为 bytes
p.write_bytes(b"\x00\x01")                # 写入 bytes

# 逐行读取(流式，适合大文件)
with p.open("r", encoding="utf-8") as f:
    for line in f:
        ...
```

> ⚠️ 原文差异1
> 原文未提到 `newline` 参数在 `read_text` / `write_text` 中的默认行为。`read_text` 默认 `newline=None`(通用换行模式)，跨平台一致。

## 五、目录操作

```python
p = Path("some_dir")

# 创建
p.mkdir()                # 创建单级目录
p.mkdir(parents=True)    # mkdir -p(递归创建中间级)
p.mkdir(exist_ok=True)   # 已存在不报错

# 删除
p.rmdir()                # 删除空目录
p.unlink()               # 删除文件/符号链接
p.unlink(missing_ok=True)# 3.8+ 文件不存在不报错

# 遍历
for item in p.iterdir():
    print(item)

# glob 匹配
list(p.glob("*.py"))           # 当前目录下 .py 文件
list(p.rglob("**/*.py"))       # 递归匹配
list(p.glob("**/*.py"))        # 同 rglob("*.py")

# 文件类型判断
p.is_file()
p.is_dir()
p.is_symlink()
p.exists()

# 文件信息
stat = p.stat()                # os.stat_result(st_size, st_mtime, ...)
```

## 六、路径转换

```python
p = Path("foo/bar/../baz.txt")

p.resolve()            # 解析符号链接和 ../   →  /abs/path/foo/baz.txt
p.absolute()           # 拼接 cwd，但不解析 ../   →  /cwd/foo/bar/../baz.txt

# 相对路径(3.12+)
p.relative_to("/home")
# 3.12+ walk_up=True: 允许 ../ 来匹配不在子路径下时
p.relative_to("/other", walk_up=True)  # 3.12+ 新特性

# 拼接 cwd
p2 = p.with_name("new.txt")       # 替换文件名
p3 = p.with_suffix(".md")         # 替换后缀
p4 = p.with_stem("new")           # 3.9+ 替换 stem
```

## 七、PurePath — 不访问文件系统的路径操作

```python
from pathlib import PurePosixPath, PureWindowsPath

# 在 Linux 上处理 Windows 路径
p = PureWindowsPath("C:\\Users\\alice\\docs")
print(p.parts)   # ("C:\\", "Users", "alice", "docs")
print(p.as_posix())  # "C:/Users/alice/docs"
```

`PurePath` 及其子类不执行任何 I/O，适合在服务器处理客户端路径字符串。

## 八、`os.path` → `pathlib` 迁移对照

| `os.path` | `pathlib` |
|-----------|-----------|
| `os.path.join(a, b)` | `Path(a) / b` |
| `os.path.basename(p)` | `Path(p).name` |
| `os.path.splitext(p)` | `Path(p).stem`, `Path(p).suffix` |
| `os.path.dirname(p)` | `Path(p).parent` |
| `os.path.exists(p)` | `Path(p).exists()` |
| `os.path.isdir(p)` | `Path(p).is_dir()` |
| `os.path.abspath(p)` | `Path(p).resolve()` |
| `os.path.getsize(p)` | `Path(p).stat().st_size` |
| `os.listdir(p)` | `Path(p).iterdir()` |
| `glob.glob("*.py")` | `Path().glob("*.py")` |
| `os.makedirs(p, exist_ok=True)` | `Path(p).mkdir(parents=True, exist_ok=True)` |

## 九、常见 Path 操作模式

```python
# 1. 配置文件在脚本同级目录
CONFIG = Path(__file__).parent / "config.toml"

# 2. 项目根目录
ROOT = Path(__file__).parent.parent

# 3. 在用户目录下创建数据目录
DATA_DIR = Path.home() / ".myapp"
DATA_DIR.mkdir(exist_ok=True)

# 4. 批量重命名
for p in Path(".").glob("*.jpeg"):
    p.rename(p.with_suffix(".jpg"))

# 5. 统计目录下所有文件总大小
total = sum(p.stat().st_size for p in Path(".").rglob("*") if p.is_file())

# 6. 查找最新修改的文件
latest = max(Path(".").glob("*.py"), key=lambda p: p.stat().st_mtime)
```

---

## 十、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 未明确区分 `resolve()` 与 `absolute()` | `resolve()` 解析 `..` 和符号链接;`absolute()` 仅拼接 cwd |
| 2 | 1.md | 未提 `walk_up=True`(3.12+) | 这是 `relative_to` 的重要新特性 |
| 3 | 1.md | 缺少 `with_stem()`(3.9+) | 批量改文件名利器 |
| 4 | 1.md | I/O 操作未提 `encoding` 默认值 | `read_text` 默认 UTF-8，与 `open()` 的 locale 默认不同 |
| 5 | 1.md | 未提 `missing_ok=True`(3.8+) | `unlink(missing_ok=True)` 避免 race condition |
