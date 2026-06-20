# python24 ruff · 知识点梳理

> 原文档:`python/Article/PythonBasis/python24/1.md`
> 整理对象:ruff linter、formatter、配置、pre-commit、GitHub Actions CI、从 flake8/black/isort 迁移

---

## 一、ruff 是什么

ruff 是 Rust 编写的 Python linter + formatter，替代:
- **Flake8**(lint) + 插件(flake8-bugbear、pyflakes、pep8-naming 等)
- **isort**(import 排序)
- **Black**(格式化)

**一个工具统一代码质量 + 格式化，速度快 10-100x。**

## 二、安装

```bash
uv add --dev ruff
# 或
pip install ruff
# 或使用 uvx 一次性运行
uvx ruff check .
```

## 三、核心命令

```bash
ruff check .                # lint 检查(当前目录)
ruff check --fix .          # 自动修复
ruff check --watch .        # 监听模式
ruff format .               # 格式化(类似 Black)
ruff format --check .       # 仅检查格式(CI 用)
ruff rule E501              # 查看规则说明
```

## 四、`pyproject.toml` 配置

```toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = [
    "E",      # pycodestyle errors
    "F",      # Pyflakes
    "I",      # isort
    "N",      # pep8-naming
    "B",      # flake8-bugbear
    "SIM",    # flake8-simplify
    "UP",     # pyupgrade (升级到现代语法)
]
ignore = ["E501"]           # 行长度交给 formatter 处理
fixable = ["ALL"]
unfixable = []

[tool.ruff.lint.isort]
known-first-party = ["my_app"]

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = ["S101"]  # assert 在测试中允许
"__init__.py" = ["F401"]    # 未使用的 import 在 __init__.py 中允许

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
line-ending = "auto"
```

## 五、pre-commit 集成

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.5.0
    hooks:
      - id: ruff-format      # 格式化
      - id: ruff              # lint(自动修复)
        args: [--fix, --exit-non-zero-on-fix]
```

```bash
pre-commit install              # 安装 git hook
pre-commit run --all-files      # 手动全量运行
```

## 六、GitHub Actions CI

```yaml
# .github/workflows/lint.yml
name: Lint
on: [push, pull_request]
jobs:
  ruff:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/ruff-action@v1
        with:
          args: "format --check"
      - uses: astral-sh/ruff-action@v1
        with:
          args: "check"
```

## 七、规则分类速查

| 前缀 | 来源 | 检查内容 |
|------|------|---------|
| `E` / `W` | pycodestyle | 代码风格(PEP 8) |
| `F` | Pyflakes | 未使用变量/import、语法错误 |
| `I` | isort | import 排序和分组 |
| `N` | pep8-naming | 命名规范 |
| `B` | flake8-bugbear | 常见 bug 模式 |
| `SIM` | flake8-simplify | 简化代码建议 |
| `UP` | pyupgrade | 升级到新版本 Python 语法 |
| `C4` | flake8-comprehensions | 推导式优化 |
| `T20` | flake8-print | 禁止 print 遗留 |
| `S` | flake8-bandit | 安全相关 |
| `PLE` / `PLW` / `PLC` | Pylint | Pylint 规则子集 |
| `RUF` | ruff 专有 | ruff 特有规则 |

## 八、从 flake8 + black + isort 迁移

```bash
# 1. 生成初始 ruff 配置(自动分析项目)
ruff check --select ALL --statistics .   # 查看哪些规则适用

# 2. 增量迁移
# - 启用 E, F, I 基础规则
# - 逐步添加 B, SIM, UP 等
# - 最后清理旧工具配置

# 3. 移除旧依赖
uv remove flake8 isort black
```

## 九、常见 lint 修复示例

```python
# SIM108: 用三元表达式
if x > 0:            # →  ruff --fix →
    y = 1            #     y = 1 if x > 0 else -1
else:
    y = -1

# UP006: typing 类型 → 内建类型
from typing import List    # →  ruff --fix →
names: List[str] = []      #     names: list[str] = []

# I001: import 排序
import os                 #     import os
import sys                # →   import sys
from pathlib import Path  # ruff --fix →  from pathlib import Path
```

---

## 十、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 配置分散在多处讲解 | 统一 `pyproject.toml` 下的 `[tool.ruff]` 是推荐方式 |
| 2 | 1.md | 未提 `ruff format` 与 `black` 的细微差异 | ruff format 99.9% 兼容 Black，极少数边界 case 有差异 |
| 3 | 1.md | 未区分 `select` 和 `extend-select` | `select` 会覆盖默认；推荐用 `extend-select` 追加 |
| 4 | 1.md | VS Code 集成部分不够详细 | `ruff.nativeServer: true` 可启用 Rust 原生 LSP 获得更好性能 |
