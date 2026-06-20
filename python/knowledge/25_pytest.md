# python25 pytest · 知识点梳理

> 原文档:`python/Article/PythonBasis/python25/1.md`
> 整理对象:pytest 基础、assert 重写、fixtures、参数化、异常测试、标记、coverage

---

## 一、pytest 基础

```bash
uv add --dev pytest
uv run pytest                          # 运行所有测试
uv run pytest tests/ -v                # 详细输出
uv run pytest tests/ -x                # 首次失败即停止
uv run pytest tests/ -k "test_user"    # 只运行匹配名称的测试
uv run pytest tests/ --lf             # 只运行上次失败的
uv run pytest tests/ --ff             # 先运行上次失败的
```

## 二、测试函数与 assert

```python
# test_calc.py
def add(a, b):
    return a + b

def test_add():
    assert add(1, 2) == 3
    assert add(-1, 1) == 0
    assert add(0.1, 0.2) == pytest.approx(0.3)  # 浮点比较
```

pytest 的 **assert 重写** 机制:
- 不等于自动显示两个值的 diff。
- `assert a == 1 and b == 2` 会分别显示 a 和 b 的值。
- `pytest.approx()` 处理浮点误差。

## 三、Fixtures(夹具)

### 基本 fixture

```python
import pytest

@pytest.fixture
def sample_user():
    return {"name": "Alice", "age": 30}

def test_user_name(sample_user):       # 直接作为参数注入
    assert sample_user["name"] == "Alice"
```

### scope(作用域)

```python
@pytest.fixture(scope="function")   # 默认:每个测试函数创建一次
@pytest.fixture(scope="class")      # 每个测试类创建一次
@pytest.fixture(scope="module")     # 每个测试模块(.py 文件)创建一次
@pytest.fixture(scope="session")    # 整个测试会话创建一次(如数据库连接)
```

### setup/teardown — yield

```python
@pytest.fixture
def db():
    conn = create_connection()       # setup
    yield conn                       # 测试执行
    conn.close()                     # teardown(一定执行)

@pytest.fixture
def temp_file(tmp_path):             # 使用内置 fixture
    f = tmp_path / "test.txt"
    f.write_text("hello")
    return f
```

## 四、conftest.py — 共享 fixtures

```
tests/
├── conftest.py          ← 所有子目录的测试都能用其中的 fixture
├── test_a.py
├── unit/
│   ├── conftest.py      ← 只对 unit/ 下的测试可见
│   └── test_b.py
└── integration/
    ├── conftest.py
    └── test_c.py
```

conftest.py 中的 fixture 自动被 pytest 发现，无需 import。

## 五、内置有用 fixtures

| fixture | 作用 |
|---------|------|
| `tmp_path` | 临时目录(Path 对象)，测试后自动删除 |
| `tmp_path_factory` | session 级临时目录工厂 |
| `monkeypatch` | 运行时修改属性/环境变量/字典 |
| `capsys` | 捕获 stdout/stderr |
| `caplog` | 捕获 logging 输出 |
| `tmpdir` | 临时目录(py.path，旧，推荐 tmp_path) |

```python
def test_env(monkeypatch):
    monkeypatch.setenv("API_KEY", "test-key")
    assert os.environ["API_KEY"] == "test-key"

def test_output(capsys):
    print("hello")
    captured = capsys.readouterr()
    assert captured.out == "hello\n"
```

## 六、参数化 `@pytest.mark.parametrize`

```python
@pytest.mark.parametrize("a,b,expected", [
    (1, 2, 3),
    (-1, 1, 0),
    (100, 200, 300),
])
def test_add(a, b, expected):
    assert add(a, b) == expected

# 组合参数化
@pytest.mark.parametrize("x", [1, 2])
@pytest.mark.parametrize("y", [10, 20])
def test_cross(x, y):
    ...   # 测试 4 个组合: (1,10) (1,20) (2,10) (2,20)
```

## 七、异常测试

```python
def test_raises():
    with pytest.raises(ValueError, match="不能为负"):
        withdraw(-100)

    with pytest.raises(ValueError) as exc_info:
        withdraw(-100)
    assert exc_info.value.args[0] == "金额不能为负"
```

## 八、标记(Markers)

```python
@pytest.mark.slow
def test_heavy():
    ...

@pytest.mark.skip(reason="暂未实现")
def test_future():
    ...

@pytest.mark.skipif(sys.version_info < (3, 11), reason="需要 3.11+")
def test_exception_group():
    ...

@pytest.mark.xfail(reason="已知 bug")
def test_known_bug():
    ...
```

运行指定标记:
```bash
uv run pytest -m "slow"
uv run pytest -m "not slow"
```

## 九、覆盖率(pytest-cov)

```bash
uv add --dev pytest-cov
uv run pytest --cov=my_app --cov-report=term --cov-report=html
```

## 十、pytest 发现与执行流(ASCII 图)

```
pytest 测试发现与执行流程:

    uv run pytest tests/ -v
        │
        ▼
    ┌──────────────────────────┐
    │ 1. 收集阶段(Collection)  │
    │                          │
    │ 遍历 tests/ 目录          │
    │   ├── 找 test_*.py       │
    │   ├── 找 *_test.py       │
    │   ├── 找 Test* 类        │
    │   └── 找 test_* 函数     │
    │                          │
    │ 配置文件发现顺序:          │
    │   pytest.ini → pyproject.toml → tox.ini → setup.cfg │
    │                          │
    │ 加载 conftest.py          │
    │   ├── 扫描目录层级        │
    │   ├── 收集 fixtures       │
    │   └── 收集 hooks         │
    └──────────┬───────────────┘
               │
               ▼
    ┌──────────────────────────┐
    │ 2. 执行阶段(Execution)   │
    │                          │
    │ 对每个测试:               │
    │   ├── 解析参数名          │
    │   ├── 按 scope 解析       │
    │   │   fixtures(依赖注入)  │
    │   ├── 运行 setup          │
    │   ├── 执行测试函数        │
    │   │   ├── assert 通过 → ✅│
    │   │   └── assert 失败 → ❌│
    │   └── 运行 teardown      │
    └──────────┬───────────────┘
               │
               ▼
    ┌──────────────────────────┐
    │ 3. 报告阶段(Reporting)   │
    │                          │
    │ 输出: . 通过  F 失败      │
    │       E 错误  s 跳过      │
    │       x xfail  X xpass   │
    │                          │
    │ 最后: 失败 summary        │
    │       覆盖率报告(可选)     │
    └──────────────────────────┘
```

```
Fixture 依赖解析图(scope 层级):

    session scope:
        db_engine ───────────── (整个会话共享)
            │
    module scope:
        db_session ─────────── (每个 .py 文件共享)
            │
    function scope (默认):
        test_a    test_b    test_c (各自独立)
```

---

## 十一、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | `tmpdir` fixture(旧) | 推荐 `tmp_path`(返回 Path 对象，现代 API) |
| 2 | 1.md | 未明确 `conftest.py` 的层级覆盖规则 | 子目录 conftest 覆盖父级，pytest 按目录层级自底向上合并 |
| 3 | 1.md | fixture scope 的 `package` scope 未提 | 3.8+ 支持 package scope |
| 4 | 1.md | 参数化与 fixture 组合用法不够清晰 | `@pytest.mark.parametrize` 可以参数化 fixture 值(通过 `indirect=True`) |
