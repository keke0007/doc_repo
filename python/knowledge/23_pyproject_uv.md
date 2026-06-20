# python23 pyproject.toml 与 uv · 知识点梳理

> 原文档:`python/Article/PythonBasis/python23/1.md`
> 整理对象:`pyproject.toml`(PEP 621)、uv 包管理器、项目初始化、依赖管理、PyPI 发布

---

## 一、旧工具链 vs 新工具链

```
旧世界(2018 前):
  setup.py          ← 安装逻辑(可执行代码，安全风险)
  setup.cfg         ← 配置
  requirements.txt  ← 依赖(无锁定)
  MANIFEST.in       ← 包含非代码文件
  Pipfile / Pipfile.lock  ← pipenv 尝试

新世界(2024+):
  pyproject.toml    ← 单一配置文件(项目元数据 + 依赖 + 工具配置)
  uv.lock           ← 确定性依赖锁定
```

## 二、`pyproject.toml` 结构

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-app"
version = "0.1.0"
description = "A sample project"
readme = "README.md"
requires-python = ">=3.11"
license = {text = "MIT"}
authors = [
    {name = "Alice", email = "alice@example.com"}
]

dependencies = [
    "httpx>=0.27",
    "python-json-logger>=3",
]

[project.optional-dependencies]
dev = [
    "pytest>=8",
    "ruff>=0.5",
]

[project.scripts]
my-cli = "my_app.cli:main"

[tool.ruff]
line-length = 100

[tool.pytest.ini_options]
testpaths = ["tests"]
```

| 字段 | 说明 |
|------|------|
| `[build-system]` | 构建后端和构建依赖 |
| `[project]` | 项目元数据(PEP 621) |
| `[project.dependencies]` | 运行时依赖 |
| `[project.optional-dependencies]` | 可选依赖组(dev, test, docs...) |
| `[project.scripts]` | CLI 入口点 |
| `[tool.*]` | 各工具的配置(ruff, mypy, pytest, coverage... ) |

## 三、uv — 包管理器

### 安装
```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# 或 pip
pip install uv
```

### 核心命令

```bash
# 创建新项目
uv init my-app                    # 生成 pyproject.toml + src/ + README.md
uv init --lib my-lib              # 库项目(src layout)

# 依赖管理
uv add httpx                      # 添加到 dependencies
uv add --dev pytest ruff          # 添加到 dev 依赖
uv remove httpx                   # 移除

# 同步/安装
uv sync                           # 安装所有依赖(按 uv.lock)
uv sync --frozen                  # CI 中使用，不更新 lock 文件

# 运行
uv run python main.py             # 在项目虚拟环境中运行
uv run pytest                     # 运行测试
uv run ruff check .               # 运行 lint

# 一次性工具(不装到项目)
uvx ruff check .                  # 等价于 npx
uvx --with httpx python -c "import httpx; ..."  # 3.11+ PEP 723 内联依赖
```

### uv.lock

- 由 `uv sync` / `uv add` 自动生成和更新。
- 锁定所有依赖的精确版本(包括间接依赖)。
- **必须提交到 Git**(保证团队/CI 环境一致)。
- `uv sync --frozen` 在 CI 中使用，确保依赖完全按 lock 文件安装。

### uv 工作流(ASCII 图)

```
项目生命周期与 uv:

    创建项目
    uv init my-app
        │
        ▼
    ┌──────────────────────┐
    │  my-app/             │
    │  ├── pyproject.toml  │  ← 项目描述 + 依赖声明
    │  ├── README.md       │
    │  └── src/            │
    │      └── my_app/     │
    │          └── __init__.py
    └──────────────────────┘
        │
        ▼
    添加依赖
    uv add httpx          ──→ pyproject.toml 中 dependencies 新增 httpx>=...
    uv add --dev pytest   ──→ [project.optional-dependencies].dev 新增
        │                    uv.lock 自动生成/更新
        ▼
    同步环境
    uv sync                ──→ 创建/更新 .venv/
        │                    安装 uv.lock 中所有包
        ▼
    日常开发
    uv run python main.py
    uv run pytest
    uv run ruff check .
        │
        ▼
    发布(配合 uv build / uv publish)
        │
        ▼
    他人克隆项目
    git clone ...
    uv sync                ──→ 按 uv.lock 精确复现环境
```

---

## 四、uv vs pip vs poetry vs pdm

| 特性 | uv | pip | poetry | pdm |
|------|-----|-----|--------|-----|
| 速度 | Rust，极快 | Python，慢 | Python，中 | Python，中 |
| 依赖解析 | 快(类 uv 解析器) | 慢(回溯) | 中 | 中 |
| lock 文件 | `uv.lock` | 无(需 pip-tools) | `poetry.lock` | `pdm.lock` |
| 项目管理 | ✅(add/remove/sync) | ❌ | ✅ | ✅ |
| 虚拟环境 | 自动管理 | 需手动 venv | 自动管理 | 自动管理 |
| 内联脚本( PEP 723) | ✅(`uv run` 自动) | ❌ | ❌ | ❌ |
| 工具运行 | ✅(`uvx`) | ❌(需 pipx) | ❌ | ❌ |

---

## 五、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 未强调 uv.lock 必须提交到 Git | 这是确定性构建的关键，应明确标注 |
| 2 | 1.md | `uv sync --frozen` 在 CI 中的角色未展开 | CI 中应使用 --frozen 确保依赖不漂移 |
| 3 | 1.md | PEP 723 内联依赖(`uv run --with`)未完整说明 | `uvx` 和 `uv run` 已整合此特性 |
| 4 | 1.md | 与 poetry/pdm 的对比不够详细 | 补充了速度、lock、项目管理等维度的对比 |
