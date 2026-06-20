# python27 打包发布与 typer CLI · 知识点梳理

> 原文档:`python/Article/PythonBasis/python27/1.md`
> 整理对象:PyPI 发布、src layout、`uv build`/`uv publish`、Trusted Publishers(OIDC)、typer CLI

---

## 一、src layout(推荐项目结构)

```
my_project/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── my_project/           ← Python 包(这是真正的包代码)
│       ├── __init__.py
│       ├── core.py
│       └── cli.py
├── tests/
│   ├── __init__.py
│   └── test_core.py
└── .github/
    └── workflows/
        └── publish.yml
```

**为什么用 src layout**:
- 避免无意中 import 本地开发目录(而非已安装的包)。
- 测试时安装的是真正发布出去的内容(`uv run pytest` 会以 editable 模式安装 src 下的包)。
- 打包工具自动识别。

## 二、`pyproject.toml` 打包完整配置

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "todo-cli"
version = "0.1.0"
description = "A simple todo CLI tool"
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.11"
authors = [
    {name = "Alice", email = "alice@example.com"}
]
keywords = ["todo", "cli"]
classifiers = [
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
]

dependencies = [
    "typer>=0.12",
    "rich>=13",
]

[project.optional-dependencies]
dev = ["pytest>=8", "ruff>=0.5"]

[project.scripts]
todo = "todo_cli.cli:app"        # 安装后命令行直接 `todo`

[project.urls]
Homepage = "https://github.com/alice/todo-cli"
Repository = "https://github.com/alice/todo-cli.git"

[tool.hatch.build.targets.wheel]
packages = ["src/todo_cli"]       # 从 src/ 下打包
```

## 三、构建与发布

```bash
# 1. 构建
uv build
# 生成 dist/todo_cli-0.1.0-py3-none-any.whl
#       dist/todo_cli-0.1.0.tar.gz

# 2. 本地测试安装
uv run pip install dist/todo_cli-0.1.0-py3-none-any.whl --force-reinstall

# 3. 发布到 TestPyPI(先测试)
uv publish --publish-url https://test.pypi.org/legacy/

# 4. 从 TestPyPI 安装验证
uv run pip install -i https://test.pypi.org/simple/ todo-cli

# 5. 发布到正式 PyPI
uv publish
```

### Trusted Publishers(OIDC) — 不配 token 的方案

GitHub Actions 中通过 OIDC 认证 PyPI，无需手动配置 token:

```yaml
# .github/workflows/publish.yml
name: Publish
on:
  push:
    tags: ["v*"]
jobs:
  pypi-publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write       # OIDC 需要
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v1
      - run: uv build
      - run: uv publish        # 自动通过 OIDC 认证
```

在 PyPI 项目设置中配置 "Trusted Publisher" → 绑定 GitHub 仓库。

## 四、typer CLI 基础

```python
import typer

app = typer.Typer()

@app.command()
def greet(name: str, count: int = 1):
    """Say hello"""
    for _ in range(count):
        typer.echo(f"Hello {name}")

@app.command()
def goodbye(name: str, formal: bool = False):
    """Say goodbye"""
    if formal:
        typer.echo(f"Goodbye {name}.")
    else:
        typer.echo(f"Bye {name}!")

if __name__ == "__main__":
    app()
```

```bash
$ todo greet Alice --count 3
Hello Alice
Hello Alice
Hello Alice

$ todo goodbye Alice --formal
Goodbye Alice.
```

类型注解自动转换为 CLI 参数:
- `str` → 字符串参数
- `int` → 整数(`--count 3`)
- `bool` → 标志(`--formal` / `--no-formal`)
- `float` → 浮点

## 五、typer 进阶功能

```python
# 确认/提示
if typer.confirm("确认删除?"):
    delete()

name = typer.prompt("请输入名称")

# 彩色输出(使用底层 rich)
typer.secho("错误!", fg=typer.colors.RED, bold=True)
typer.secho("成功", fg=typer.colors.GREEN)

# 错误处理
raise typer.Exit(code=1)         # 优雅退出
raise typer.Abort()              # Exit(1) 简写

# 进度条(rich 集成)
from rich.progress import track
for item in track(items, description="处理中..."):
    process(item)

# 子命令
items_app = typer.Typer()
app.add_typer(items_app, name="items")    # todo items ...

@items_app.command()
def list():
    """列出所有项目"""
```

## 六、`[project.scripts]` 入口点

```toml
[project.scripts]
todo = "todo_cli.cli:app"        # 命令名 = "模块:对象"
```

安装后，`todo` 命令直接可用(虚拟环境中的 bin/ 或 Scripts/ 目录)。

## 七、版本管理

```bash
# 手动方式
# 在 pyproject.toml 中维护 version = "0.1.2"

# 自动化方式 — hatch-vcs(从 git tag 读取)
# pyproject.toml:
# [build-system]
# requires = ["hatchling", "hatch-vcs"]
# [tool.hatch.version]
# source = "vcs"
```

## 八、构建→发布流水线(ASCII 图)

```
┌──────────────────────────────────────────────────────────┐
│                    开发 → 发布完整流水线                    │
│                                                          │
│   1. 开发阶段                                              │
│   ┌──────────┐                                           │
│   │ 写代码    │ → uv run pytest → uv run ruff check .    │
│   └──────────┘                                           │
│        │                                                 │
│        ▼                                                 │
│   2. 构建阶段                                              │
│   ┌──────────┐                                           │
│   │ uv build │                                           │
│   └────┬─────┘                                           │
│        │                                                 │
│        ├── dist/xxx.whl (wheel — 二进制分发包)            │
│        └── dist/xxx.tar.gz (sdist — 源码分发包)           │
│        │                                                 │
│        ▼                                                 │
│   3. 测试发布阶段                                          │
│   ┌──────────────────────────────────────┐               │
│   │ uv publish --publish-url TestPyPI    │               │
│   │ uv run pip install -i TestPyPI xxx   │               │
│   │ → 全新虚拟环境验证安装和运行           │               │
│   └──────────────────┬───────────────────┘               │
│                      │                                   │
│                      ▼                                   │
│   4. 正式发布阶段                                          │
│   ┌──────────────┐                                       │
│   │ git tag v0.1.0│                                      │
│   │ git push --tags│                                     │
│   │ uv publish    │  ← 发布到 PyPI                        │
│   └──────┬───────┘                                       │
│          │                                               │
│          ▼                                               │
│   5. 用户安装                                              │
│   ┌──────────────────────┐                               │
│   │ uv tool install xxx  │  或  pip install xxx          │
│   │ 或 uvx xxx           │  或  uv run --with xxx ...    │
│   └──────────────────────┘                               │
│                                                          │
│   CI/CD 自动发布(打 tag 触发):                             │
│   git tag v0.1.0 → push → GitHub Actions →              │
│   → uv build → uv publish (OIDC Trusted Publisher)     │
│   → PyPI(无需手动 token)                                 │
└──────────────────────────────────────────────────────────┘
```

---

## 九、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 使用了 `setup.py` / `setup.cfg` 打包示例 | 新项目应全部用 `pyproject.toml` + `hatchling`(PEP 621) |
| 2 | 1.md | PyPI token 管理未提 OIDC | Trusted Publishers(OIDC)是更安全的免 token 方案 |
| 3 | 1.md | 版本号手动管理 | `hatch-vcs` 可从 git tag 自动读取版本，减少人为错误 |
| 4 | 1.md | typer 示例较基础 | 补充了 `secho`、`prompt`、`Exit/Abort`、子命令等常用功能 |
