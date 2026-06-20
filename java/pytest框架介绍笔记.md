# pytest 框架介绍笔记

> 本文档基于 `pytest框架介绍笔记.xmind` 整理而成，系统梳理了 pytest 单元测试框架的核心知识点。

---

## 一、pytest 框架概述

pytest 是一个非常成熟的 Python 单元测试框架，相比 `unittest` 更加灵活强大，主要特点包括：

1. **成熟稳定**：pytest 是一个非常成熟的单元测试框架（类似 unittest）。
2. **应用广泛**：可以与 `requests`、`selenium` 等结合，实现：
   - Web 自动化测试
   - 接口自动化测试
   - App 自动化测试
3. **跳过与重跑**：可以实现测试用例的跳过以及测试用例失败重跑机制。
4. **测试报告美观**：可以与 `allure` 结合生成美观的测试报告。
5. **持续集成**：可以与 `Jenkins` 进行持续集成。
6. **插件丰富**：拥有非常丰富的插件功能。

---

## 二、常用插件

| 插件名称 | 作用 |
| --- | --- |
| `pytest` | 框架本体 |
| `pytest-html` | 生成 HTML 格式的测试报告 |
| `pytest-xdist` | 测试用例分布式执行 |
| `pytest-ordering` | 改变测试用例的执行顺序 |
| `pytest-rerunfailures` | 测试用例失败后重跑 |
| `allure-pytest` | 用于生成美观的 allure 测试报告 |

---

## 三、测试用例的使用规则

pytest 在识别测试用例时遵循固定的命名规则：

1. **模块名称**必须以 `test_` 开头，或者以 `_test` 结尾。
2. **测试类**必须以 `Test` 开头，并且**不能有 `__init__` 方法**。
3. **测试方法**必须以 `test` 开头。

---

## 四、pytest 测试用例的运行方式

### 1. 主函数运行方式

通过在 Python 文件中调用 `pytest.main()` 方法来运行。

```python
# 默认运行
pytest.main()

# 指定模块运行
pytest.main(['-vs', './testcase/Login/test_login.py'])

# 指定文件夹下的测试用例
pytest.main(['-vs', './testcase'])
```

#### 参数详解

| 参数 | 说明 |
| --- | --- |
| `-s` | 输出调试信息，包括代码中的 `print` 打印信息 |
| `-v` | 显示更加详细的信息 |
| `-vs` | 上面两个参数结合使用 |
| `-n` | 支持多进程或分布式运行测试用例（需 pytest-xdist） |
| `--reruns` | 设置测试用例失败重跑次数，例如 `--reruns=2` |

#### 改变测试用例执行顺序

通过装饰器指定执行顺序：

```python
@pytest.mark.run(order=1)
def test_xxx():
    pass
```

---

### 2. 命令行运行

直接在终端中执行 `pytest` 命令：

```bash
# 默认运行当前目录下所有用例
pytest

# 指定模块运行
pytest -vs ./testcase/Login/test_login.py

# 指定运行某个文件夹下的所有测试用例
pytest -vs ./testcase

# 测试用例分布式执行
pytest -vs ./testcase -n 3
```

---

### 3. 通过读取 pytest.ini 配置文件运行

`pytest.ini` 是 pytest 框架的核心配置文件：

- **文件名固定**：文件名及后缀为固定写法，不可变更。
- **存放位置**：一般存放在项目的根目录下。
- **编码格式**：必须为 **ANSI** 格式，文件内不可以加上中文（容易引起编码问题）。
- **作用**：约定或者改变 pytest 默认的行为。
- **运行规则**：不管是通过主函数运行还是通过命令行运行，pytest 都会自动读取 `pytest.ini` 配置文件。

---

## 五、pytest 测试用例分组分模块执行

实现"按需执行某一类用例"的能力。

### 步骤 1：在测试用例方法上加上分组模块标记

模块名称可以自定义，但必须与 `pytest.ini` 的分组名一致：

```python
@pytest.mark.smock
def test_login():
    pass
```

### 步骤 2：在 pytest.ini 加上 markers 参数

```ini
[pytest]
markers =
    smock:登录模块冒烟测试
```

### 步骤 3：在命令行带上 `-m` 参数执行

```bash
pytest -vs ./testcase -m "maoyan or usermanager"
```

---

## 六、pytest 测试用例跳过执行

在需要跳过的测试用例方法名上加上 `@pytest.mark.skip` 标记即可：

```python
@pytest.mark.skip
def test_case02(self):
    print('我第三个执行')
```

---

## 七、pytest 框架前后置处理 ⭐

pytest 提供了三种实现前后置（setup/teardown）的方式。

### 方式一：setup / teardown

传统的 unittest 风格写法：

- `setup` / `teardown`：方法级别前后置
- `setup_class` / `teardown_class`：类级别前后置

---

### 方式二：通过装饰器 `@pytest.fixture` 实现（pytest 核心）

#### 1. 格式和结构

在方法前加上装饰器：

```python
@pytest.fixture(scope="", params="", autouse=False, ids="", name="")
def my_fixture():
    ...
```

#### 2. 参数详解

##### ⭐ scope — 作用域

被 `@pytest.fixture` 标记的方法作用域，主要有 4 个值：

| 取值 | 含义 |
| --- | --- |
| `function`（默认） | 作用域是方法，每个测试用例执行之前都会先执行前置操作，类似 `setup/teardown` |
| `class` | 每个类执行之前会执行一次前置操作 |
| `module` | 作用域是模块（文件）。一个文件有多个类时，整个文件只执行一次前置和后置 |
| `package` / `session` | 多个测试用例文件只执行一次前后置操作 |

##### params — 参数化

支持的格式：list、tuple、字典。

```python
@pytest.fixture(scope='function', autouse=True, params=['北京', '广州', '深圳'])
def fixture_test(request):
    """前后置处理"""
    # print('------------接口测试开始-------------')
    # yield
    # print('------------接口测试结束-------------')
    return request.param
```

##### ⭐ autouse — 是否自动使用

是否自动使用，默认 `False`。设置为 `True` 后，所有用例无需显式声明就会自动应用该 fixture。

##### ids — 参数化的 ID

当使用 `params` 参数化时，可以给每一个值设置一个变量名，意义不大。

##### name — 别名

给被 `@pytest.fixture` 标记的方法修改别名。

---

### 方式三：conftest.py + @pytest.fixture（全局前后置）

`conftest.py` 和 `@pytest.fixture` 结合使用来实现**全局的前后置应用**。

- **文件名固定**：`conftest.py` 是固定写法，不可更改。
- **作用范围**：会被同目录及子目录下的所有测试用例自动识别，无需显式 import。

#### 典型使用场景

1. **登录操作**：所有用例执行前先登录获取 token
2. **文件清除操作**：用例执行后清理生成的临时文件

---

## 八、pytest 框架参数化

通过 `@pytest.mark.parametrize` 实现数据驱动测试。

### 1. 基本语法

```python
@pytest.mark.parametrize(args_name, args_value)
```

- **`args_name`**：参数名称
- **`args_value`**：参数值，支持多种类型

### 2. args_value 支持的类型

| 类型 | 示例 |
| --- | --- |
| list（列表） | `[1, 2, 3]` |
| tuple（元组） | `(1, 2, 3)` |
| 字典列表 `[{}, {}, {}]` | `[{'name': '小张'}, {'name': '小明'}]` |
| 字典元组 `({}, {}, {})` | `({'name': '小张'}, {'name': '小明'})` |

### 3. 重要规则 ⭐

> **参数里面有多少个值，这个用例就会执行多少次。**

例如传入 3 个值，该测试用例会被执行 3 次，每次使用不同的参数。

---

## 九、知识点速查表

| 主题 | 关键字 / 命令 |
| --- | --- |
| 用例命名 | `test_*.py`、`Test*` 类、`test_*` 方法 |
| 运行用例 | `pytest`、`pytest.main()`、`pytest.ini` |
| 常用参数 | `-s`、`-v`、`-vs`、`-n`、`--reruns` |
| 分组执行 | `@pytest.mark.xxx` + `pytest.ini` 中 `markers` + `-m` |
| 跳过用例 | `@pytest.mark.skip` |
| 改变顺序 | `@pytest.mark.run(order=N)` |
| 前后置 | `setup/teardown`、`@pytest.fixture`、`conftest.py` |
| 参数化 | `@pytest.mark.parametrize` |
| 失败重跑 | `pytest-rerunfailures` + `--reruns=N` |
| 分布式执行 | `pytest-xdist` + `-n N` |
| 测试报告 | `pytest-html`、`allure-pytest` |

---

*文档整理自 pytest框架介绍笔记.xmind*
