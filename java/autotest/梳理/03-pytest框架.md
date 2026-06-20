# Pytest 框架(知识点梳理)

> 源文档:`pytest框架介绍.html` / `WEB自动化测试用例参数化处理.html` / `Pytest框架的钩子函数.html`

---

## 一、Pytest 简介与安装

### 1. 简介

pytest 是一个流行的 Python 测试框架,广泛用于单元测试、集成测试和功能测试。它具有简单、灵活、可扩展的特点,提供了丰富的功能和插件生态系统,通过简洁的语法让测试变得容易、灵活且易于理解。

### 2. 安装

```bash
pip install pytest -i https://pypi.tuna.tsinghua.edu.cn/simple/
```

### 3. 常用插件

| 插件 | 作用 | 安装命令 |
| --- | --- | --- |
| pytest-xdist | 多进程 / 多线程并发执行,`-n` 指定进程数 | `pip install pytest-xdist` |
| pytest-rerunfailures | 失败用例自动重跑(应对网络抖动等不稳定因素) | `pip install pytest-rerunfailures` |
| pytest-ordering | 自定义测试用例执行顺序 | `pip install pytest-ordering` |
| allure-pytest | 生成 Allure 测试报告 | `pip install allure-pytest` |

---

## 二、用例命名规则与运行方式

### 1. 编写规则(默认约定)

- 测试文件以 `test_` 开头(以 `_test` 结尾也可以)
- 测试类以 `Test` 开头,并且 **不能** 带有 `__init__` 方法
- 测试函数 / 方法以 `test_` 开头
- 断言直接使用 Python 基本的 `assert` 关键字

### 2. 运行方式

**主函数模式**

```python
import pytest

if __name__ == '__main__':
    pytest.main(['-s', '-v'])
```

**命令行模式**

```bash
# 运行某一个测试文件
pytest test_login.py

# 运行测试文件里的指定类
pytest test_login.py::TestLogin2

# 运行测试文件里指定类的指定方法
pytest test_login.py::TestLogin2::test_case_03
```

**配置文件模式**:在项目根目录下创建 `pytest.ini`,详见第九节。

### 3. 并发执行

```python
import pytest
from time import sleep


class TestPlugIn:
    def test_plug_case_01(self):
        sleep(3); print('用例 01')
    def test_plug_case_02(self):
        sleep(3); print('用例 02')
    def test_plug_case_03(self):
        sleep(3); print('用例 03')


if __name__ == '__main__':
    pytest.main(['-n', '3'])
```

### 4. 失败重跑

```python
import pytest, random

class TestReRun:
    @pytest.mark.flaky(reruns=3, reruns_delay=2)
    def test_rerun(self):
        assert random.choice([True, False])

if __name__ == '__main__':
    pytest.main(['--reruns=3'])
```

### 5. 自定义执行顺序

```python
import pytest

class TestOrdering:
    @pytest.mark.run(order=3)
    def test_case_01(self): print('第一个')

    @pytest.mark.run(order=1)
    def test_case_02(self): print('第二个')

    @pytest.mark.run(order=2)
    def test_case_03(self): print('第三个')
```

---

## 三、断言

pytest 直接使用 Python 内置的 `assert` 关键字进行断言。

```python
class TestAssert:
    def test_eq(self):       assert 1 == 1
    def test_ne(self):       assert 1 != 2
    def test_truth(self):    assert True is True
    def test_in(self):       assert 'py' in ['py', 'java']
    def test_set(self):      assert {1,2,3} == {3,2,1}

    def test_exception(self):
        import pytest
        with pytest.raises(ValueError):
            int("abc")
```

常用断言形式速查:

| 类型 | 示例 |
| --- | --- |
| 相等 / 不相等 | `assert a == b` / `assert a != b` |
| 真 / 假 | `assert x is True` / `assert x is False` |
| 成员关系 | `assert item in container` |
| 集合相等 | `assert set_a == set_b` |
| 异常断言 | `with pytest.raises(ValueError): ...` |

---

## 四、Fixtures(夹具)

### 1. 三种前后置方式

1. `setup` / `teardown` / `setup_class` / `teardown_class`
2. `@pytest.fixture` 装饰器
3. `conftest.py` + `@pytest.fixture` 实现跨文件共享

### 2. setup / teardown(方法级)与 setup_class / teardown_class(类级)

- `setup` / `teardown`:类中 **每个** 测试用例执行前后 **各执行一次**
- `setup_class`:类内所有用例**执行前**只执行一次
- `teardown_class`:类内所有用例**执行后**只执行一次

```python
class TestSetupTeardown:
    def setup(self):       print('用例前置')
    def teardown(self):    print('用例后置')
    def setup_class(self): print('类前置(只 1 次)')
    def teardown_class(self): print('类后置(只 1 次)')

    def test_case_01(self): print('用例 01')
    def test_case_02(self): print('用例 02')
```

### 3. @pytest.fixture + yield(推荐风格)

```python
import pytest

@pytest.fixture
def init_browser():
    print('前置:初始化浏览器')
    yield
    print('后置:关闭浏览器')

class TestFixture:
    def test_case_01(self, init_browser):
        print('用例 01')
```

### 4. @pytest.fixture 参数详解

```python
@pytest.fixture(scope='function', autouse=False, params=None, ids=None, name=None)
```

| 参数 | 说明 |
| --- | --- |
| **scope** | 作用域:`function`(默认)/ `class` / `module` / `session` |
| **autouse** | 是否自动应用,默认 `False`;为 `True` 时无需在用例形参中显式传入 |
| **params** | 为 fixture 参数化提供多组取值,通过 `request.param` 获取 |
| **ids** | 与 `params` 配合,为每组参数定义可读标识(在报告中展示) |
| **name** | 为 fixture 自定义名称(原函数名失效,需用新名引用) |

**scope 四种取值:**

- `function`:每个测试函数前后各运行一次(默认)
- `class`:每个测试类前后运行一次
- `module`:每个 `.py` 文件前后运行一次
- `session`:整个 pytest 会话前后运行一次

```python
@pytest.fixture(scope='class', autouse=True)
def init_browser():
    print('前置')
    yield
    print('后置')

@pytest.fixture(params=['apple','banana','orange'],
                ids=['苹果','香蕉','橘子'],
                name='setup_init')
def setup(request):
    return request.param

class TestFixture:
    def test_param(self, setup_init):   # 必须用 name 引用
        assert len(setup_init) > 0
```

---

## 五、参数化(parametrize / 数据驱动)

### 1. 基本用法

- 装饰器:`@pytest.mark.parametrize("名1,名2,...", iterable)`
- 装饰器中声明的参数名 **必须** 与用例形参名一致,数量也要对应
- 可迭代对象推荐用 `list` / `tuple` 保证顺序

### 2. 直接传值

```python
import pytest

class TestParams:
    @pytest.mark.parametrize("lang", ['python', 'java', 'C#'])
    def test_single(self, lang):
        print(lang)

    @pytest.mark.parametrize(
        "username,password,address",
        [("test01", "qwe123", 'BJ'),
         ("test02", "qwe456", 'CD'),
         ("test03", "qwe789", 'NC')]
    )
    def test_multi(self, username, password, address):
        print(f"{username}/{password}/{address}")
```

### 3. JSON 文件参数化

`data/login.json`(dict 列表):

```json
[
  {"username": "admin123",    "password": "123456"},
  {"username": "admin123567", "password": "123456"},
  {"username": "",            "password": ""}
]
```

用例(**正确写法**):

```python
@pytest.mark.parametrize('data', read_json('./data/login.json'))
def test_login(self, get_driver, data):
    username = data['username']         # ✅ 用 key 取 dict 的 value
    password = data['password']
    LoginPage(get_driver).login(username, password)
```

> ⚠ 原文写 `username, password = data`,**这是错的**。对 dict 解包会得到 keys 不是 values。

### 4. YAML 文件参数化

```bash
pip install pyyaml
```

`data/login.yaml`(**用 YAML list 语法,而不是逗号字符串**):

```yaml
- [admin123, "123456"]
- [admin123567, "123456"]
- [admin123, "000000"]
- [admin123, ""]
```

```python
@pytest.mark.parametrize('data', read_yaml('./data/login.yaml'))
def test_login(self, get_driver, data):
    username, password = data
    LoginPage(get_driver).login(username, password)
```

### 5. Excel 文件参数化

```bash
pip install openpyxl
```

```python
@pytest.mark.parametrize(
    'data',
    ExcelDataReader('./data/login_testdata.xlsx').read_multiple_rows()
)
def test_login(self, get_driver, data):
    username, password = data
    LoginPage(get_driver).login(username, password)
```

> ⚠ 原文 Excel 工具类在每个 read 方法的 `finally` 中都执行了 `self.close()`,**调用任一方法后实例就报废了**。应当移除 `finally` 中的 close,改由调用方在用完后显式关闭,或用上下文管理器。

---

## 六、标记与跳过(skip / xfail / marker)

### 1. 分组标记

```python
import pytest

class TestAddUser:
    @pytest.mark.P1
    def test_add_01(self): print('add 01')

    @pytest.mark.P2
    def test_add_02(self): print('add 02')
```

命令行按标记执行:

```bash
pytest -vs -m "P1"            # 只跑 P1
pytest -vs -m "P1 or P2"      # P1 或 P2
pytest -vs -m "P1 and not slow"
```

> ⚠ 自定义 marker 需在 `pytest.ini` 的 `markers` 段注册,否则触发 `PytestUnknownMarkWarning`,严格模式下会报错。

### 2. 跳过执行

- `@pytest.mark.skip(reason)`:无条件跳过
- `@pytest.mark.skipif(condition, reason)`:满足条件时跳过
- `@pytest.mark.xfail`:预期失败,失败不计入 failed,而是 xfail

```python
class TestSkip:
    @pytest.mark.skip(reason='功能未实现')
    def test_skip_01(self): pass

    sum = 5

    @pytest.mark.skipif(condition=sum == 5, reason='不符合条件')
    def test_skip_02(self): pass

    @pytest.mark.xfail
    def test_xfail(self): assert 1 == 2
```

---

## 七、conftest.py

`conftest.py` 是 pytest 的 **本地插件 / 夹具配置文件**,实现跨模块共享前后置逻辑。

### 1. 使用规则

- 文件名 **必须** 是 `conftest.py`,固定不能改
- 项目可在不同目录下创建多个 `conftest.py`,每个文件只对其所在目录及 **子目录** 下的测试模块生效
- 使用 conftest.py 中的 fixture **无需 import**,pytest 自动发现
- 多层 conftest 中同名 fixture,**就近覆盖**

### 2. 示例

```python
# conftest.py
import pytest

@pytest.fixture(scope='session', autouse=True)
def setup():
    print('全局前置')
    yield
    print('全局后置')
```

---

## 八、常用钩子函数

钩子函数(hook)是 pytest 提供的扩展机制,以 `pytest_` 为前缀,放在 `conftest.py` 或插件中,pytest 在特定时机自动调用。

| 钩子 | 调用时机 | 典型用途 |
| --- | --- | --- |
| `pytest_configure(config)` | 配置阶段 | 注册插件、自定义命令行选项、注册 markers |
| `pytest_collection_modifyitems(config, items)` | 收集用例后 | 修改、过滤、重新排序 items |
| `pytest_runtest_protocol(item, nextitem)` | 单个用例执行协议入口 | 在用例前后插入操作 |
| `pytest_fixture_setup(fixturedef, request)` | 每个 fixture setup 前 | 自定义 fixture 行为 |
| `pytest_fixture_post_finalizer(fixturedef, request)` | 每个 fixture 终结阶段 | 自定义 fixture 清理 |
| `pytest_runtest_makereport(item, call)` | 每个用例运行结束后 | 失败截图、自定义报告、收集失败信息 |
| `pytest_terminal_summary(terminalreporter, exitstatus, config)` | 全部用例执行完毕、输出摘要时 | 自定义总结输出、推送通知 |

### 1. 失败截图(常用,**修正后**)

```python
import allure
import pytest

@pytest.hookimpl(hookwrapper=True)
def pytest_runtest_makereport(item, call):
    outcome = yield                       # ✅ 拼写:outcome,不是 coucome
    result = outcome.get_result()
    if result.when == 'call':
        xfail = hasattr(result, 'wasxfail')
        if (result.skipped and xfail) or (result.failed and not xfail):
            with allure.step('测试用例失败截图'):
                allure.attach(
                    driver.get_screenshot_as_png(),
                    '失败截图',
                    attachment_type=allure.attachment_type.PNG,
                )
```

> ⚠ 原文该钩子函数有两处 Bug:
> - 函数签名缺少 `(item, call)` 参数
> - 变量名拼成了 `coucome`(应为 `outcome`)

### 2. 自动统计执行结果

```python
import time

def pytest_terminal_summary(terminalreporter, exitstatus, config):
    total    = terminalreporter._numcollected
    passed   = len(terminalreporter.stats.get('passed', []))
    failed   = len(terminalreporter.stats.get('failed', []))
    error    = len(terminalreporter.stats.get('error', []))
    skipped  = len(terminalreporter.stats.get('skipped', []))
    duration = round(time.time() - terminalreporter._sessionstarttime, 2)

    summary = f"""
    自动化测试结果:
    用例总数:{total}
    通过:{passed}
    失败:{failed}
    错误:{error}
    跳过:{skipped}
    总时长:{duration}s
    """
    print(summary)
```

---

## 九、配置文件 pytest.ini

`pytest.ini` 放在项目 **根目录**,文件名固定。

```ini
[pytest]
# -v 详细输出;-s 不捕获 print;-n NUM 并发数;--reruns NUM 失败重跑次数
addopts = -vs -n 3 --reruns 3

# 用例搜索路径
testpaths = ./other

# 改写默认测试文件命名规则
python_files = test_*.py

# 改写默认测试类命名规则
python_classes = Test*

# 改写默认测试函数命名规则
# ⚠ 原文写成 python_functions = test,会匹配 testing/tester 等,导致误收集
# ✅ 推荐写 test_*
python_functions = test_*

# 注册自定义标记(避免 PytestUnknownMarkWarning)
markers =
    first: 优先执行
    second: 第二执行
    last:  最后执行
    P1: 冒烟用例
    P2: 回归用例
```

---

## 十、钩子函数执行顺序 + conftest 查找链 ASCII 流程图

### 1. 钩子函数执行顺序

```
+-----------------------------------------------------------+
|                    pytest 启动                            |
+-----------------------------------------------------------+
              |
              v
+-----------------------------------------------------------+
|  pytest_configure(config)                                 |
|    - 加载 pytest.ini / 注册插件 / 注册 markers            |
+-----------------------------------------------------------+
              |
              v
+-----------------------------------------------------------+
|  收集用例 collection                                      |
|    -> pytest_collection_modifyitems(config, items)        |
|       (修改 / 过滤 / 排序 items)                          |
+-----------------------------------------------------------+
              |
              v
   for each item (测试用例)
              |
              v
+-----------------------------------------------------------+
|  pytest_runtest_protocol(item, nextitem)                  |
|     |                                                     |
|     |-- setup 阶段                                        |
|     |     pytest_fixture_setup(fixturedef, request)       |
|     |     -> 执行 fixture 前置 (yield 之前)               |
|     |                                                     |
|     |-- call  阶段                                        |
|     |     执行测试函数体 + assert                         |
|     |                                                     |
|     |-- teardown 阶段                                     |
|     |     pytest_fixture_post_finalizer(...)              |
|     |     -> 执行 fixture 后置 (yield 之后 / finalizer)   |
|     |                                                     |
|     +-- pytest_runtest_makereport(item, call)             |
|           (when = 'setup' | 'call' | 'teardown')          |
|           -> 失败截图 / 收集报告                          |
+-----------------------------------------------------------+
              |
              v
   所有 item 执行完毕
              |
              v
+-----------------------------------------------------------+
|  pytest_terminal_summary(terminalreporter, ...)           |
|    - 输出汇总信息 / 发送通知 / 统计覆盖率                 |
+-----------------------------------------------------------+
              |
              v
                       pytest 退出
```

### 2. conftest.py 查找链(就近覆盖)

```
项目目录示例:

project_root/
|-- pytest.ini
|-- conftest.py              <-- (A) 顶层 conftest
|-- tests/
|   |-- conftest.py          <-- (B) tests 目录 conftest
|   |-- module_a/
|   |   |-- conftest.py      <-- (C) module_a 目录 conftest
|   |   `-- test_a.py        <-- 测试文件
|   `-- module_b/
|       `-- test_b.py

fixture 查找方向 (执行 test_a.py 时):

      test_a.py
          |
          v   向上查找,直到 rootdir
   +-- module_a/conftest.py  (C)  <-- 最近,优先级最高
          |
          v
   +-- tests/conftest.py     (B)
          |
          v
   +-- project_root/conftest.py (A)  <-- 全局,优先级最低


规则要点:
  1) conftest.py 文件名固定,不能修改
  2) 只对其所在目录及子目录生效:
       (C) 只对 module_a 下的用例生效
       (B) 对 tests 下所有子目录生效
       (A) 对整个项目生效
  3) 使用其中的 fixture 无需 import
  4) 同名 fixture 就近覆盖:(C) > (B) > (A)
  5) scope='session' 的 fixture 在整个会话中只执行一次
```

---

## 十一、易错点 / 原文错误订正

| # | 原文表述 | 问题 | 正确做法 |
| --- | --- | --- | --- |
| 1 | `pytest.main(['-n 3'])` | 选项与值要拆成两个元素 | `pytest.main(['-n', '3'])` |
| 2 | `python_functions = test`(pytest.ini) | 这条规则会匹配 `testing_xxx` / `tester_xxx`,误收集 | 写成 `python_functions = test_*` |
| 3 | 描述「setup_class、teardown_class 在测试用例执行前只执行一次前后置操作」 | 表述只提了「执行前」 | `setup_class` 类内所有用例 **执行前** 只执行一次;`teardown_class` 类内所有用例 **执行后** 只执行一次 |
| 4 | `@pytest.fixture` 示例既 `request.addfinalizer(teardown)` 又 `return teardown` | 二者同时存在语义混乱 | 二选一:`yield` 风格或 `addfinalizer + return 真正需要的资源` |
| 5 | `pytest_runtest_makereport` 函数签名缺 `(item, call)`,且 `coucome` 拼写错 | 签名错 + 拼写错,等价于钩子失效 | 应为 `def pytest_runtest_makereport(item, call):`,变量名 `outcome = yield` |
| 6 | JSON 参数化 `username, password = data`(data 是 dict) | 对 dict 解包得到 **keys**,不是 values | `username = data['username']; password = data['password']` |
| 7 | YAML `- admin123,123456` 整行当字符串 | 读出来是字符串,解包成单字符 | YAML 用列表语法:`- [admin123, "123456"]` |
| 8 | YAML `- admin123,` 缺值 | 不是合法的「两元素 list」 | 写成 `- [admin123, ""]` |
| 9 | Excel 工具类每个 read 方法的 `finally` 中 `self.close()` | 第一次调用就关闭,实例无法复用 | 移除 `finally` 中的 close,改由调用方显式关闭,或用 context manager |
| 10 | `@pytest.mark.parametrize("...", {(...), (...)})`(set) | `set` 无序,运行顺序与报告编号不固定 | 用 `list` / `tuple` |
| 11 | `@pytest.mark.parametrize("user_name", {"test01":"qwe123",...})`(dict) | 直接传 dict 只迭代 keys,密码完全用不到 | 改为 `[("test01","qwe123"),...]`,装饰器声明两个参数名 |
| 12 | `@pytest.mark.P1` 等自定义 marker 未在 pytest.ini 注册 | 6.0 后触发 `PytestUnknownMarkWarning`,严格模式失败 | 在 `markers =` 段注册 |
| 13 | pytest-ordering 示例同名方法重复定义 | 后定义覆盖前定义,前者永远不执行 | 改名 `test_ordering_case_04/05/06` 等 |
| 14 | `terminalreporter.stats.get('error', [])` | pytest 实际归类不一定有 `'error'` key | 兼容写法:同时统计 `'failed'` 与 `'errors'`,先打印 `stats.keys()` 确认 |
| 15 | 第七节标题「pytest 执行顺序」实际讲分组 + 跳过 | 标题与内容不符 | 拆成「标记与分组执行」+「跳过与预期失败」 |
