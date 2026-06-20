# POM 设计模式(知识点梳理)

> 源文档:`POM设计模式.html`

---

## 一、POM 是什么 / 解决了什么问题

**Page Object Model(POM)** 是 UI 自动化测试中常用的一种设计模式,尤其在 Selenium 测试中被广泛采用。

**核心思想**:把「被测页面」抽象成一个类(Page Class),类里维护该页面上的元素定位与对元素的操作,测试脚本只通过调用 Page 类暴露出来的业务方法来执行测试,而不再直接和 Selenium API 打交道。

### 解决的问题

- **页面变化导致用例大面积返工**:把元素定位写在用例里,UI 一改就要改一堆 case。POM 把定位收敛到 Page 类,UI 变更只改 Page 即可
- **重复代码多**:登录、点击、输入这类动作在不同用例中反复出现,缺乏复用
- **可读性差**:用例里充斥 `find_element`、`send_keys`、`By.ID`,业务意图被淹没
- **行为与状态混杂**:点击按钮、填写表单(行为)与页面标题、元素可见性(状态)耦合在一起

### 带来的收益

1. **页面类(Page Class)** —— 每个被测页面对应一个类,持有元素和操作
2. **行为与状态分离** —— Page 类内部封装「做什么」和「页面当前是什么样」,用例只关心业务流
3. **可重用性** —— Page 方法可被多个用例共享,页面变了只改 Page
4. **可读性** —— 用例像在描述业务流程(`login_page.login(u, p)`),不再充满 Selenium 调用细节

---

## 二、典型目录结构

```
auto_project/
├── base/                  # 基础层:所有 Page 的公共能力
│   └── base_page.py       # BasePage 基类:封装通用的元素查找/操作/等待
│
├── pages/                 # 页面对象层:每个页面一个类,继承 BasePage
│   ├── __init__.py
│   ├── login_page.py      # LoginPage:登录页元素 + 业务方法
│   ├── home_page.py       # HomePage
│   └── search_page.py     # SearchPage
│
├── testcases/             # 用例层:只调 Page 方法,不直接用 selenium
│   ├── __init__.py
│   ├── conftest.py        # pytest fixture(driver、登录态等)
│   ├── test_login.py
│   └── test_search.py
│
├── utils/                 # 工具层
│   ├── driver_factory.py  # 浏览器/驱动初始化
│   ├── logger.py          # 日志
│   ├── yaml_reader.py     # 配置/数据读取
│   └── screenshot.py      # 失败截图
│
├── data/                  # 测试数据 / 配置(与代码解耦)
│   ├── config.yaml        # 环境 URL、超时时间、浏览器类型等
│   ├── locators.yaml      # (可选)元素定位也可外置
│   └── test_data.yaml     # 用例数据:账号、关键字等
│
├── report/                # 测试报告产物
│   ├── allure-results/    # allure 原始结果
│   └── allure-report/     # 生成的 HTML 报告
│
├── pytest.ini             # pytest 配置
├── requirements.txt
└── README.md
```

---

## 三、BasePage 基类

所有 Page 的公共父类,把「和 Selenium 打交道」的通用动作收敛在这里。Page 子类不再直接调 `self.driver.find_element`,而是调 `self.find(locator)`、`self.click(locator)` 等。

```python
# base/base_page.py
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


class BasePage:
    def __init__(self, driver, timeout: int = 10):
        self.driver = driver
        self.timeout = timeout
        self.wait = WebDriverWait(driver, timeout)

    # ---------- 元素查找(带显式等待) ----------
    def find(self, locator):
        """locator 是 (By.XX, 'xxx') 元组"""
        return self.wait.until(EC.presence_of_element_located(locator))

    def find_clickable(self, locator):
        return self.wait.until(EC.element_to_be_clickable(locator))

    def finds(self, locator):
        return self.wait.until(EC.presence_of_all_elements_located(locator))

    # ---------- 基础动作 ----------
    def click(self, locator):
        self.find_clickable(locator).click()

    def input(self, locator, text):
        el = self.find(locator)
        el.clear()
        el.send_keys(text)

    def text_of(self, locator) -> str:
        return self.find(locator).text

    def is_visible(self, locator) -> bool:
        try:
            self.wait.until(EC.visibility_of_element_located(locator))
            return True
        except Exception:
            return False

    # ---------- 页面级 ----------
    def open(self, url):
        self.driver.get(url)

    def title(self) -> str:
        return self.driver.title
```

关键点:
- **统一显式等待**,不再依赖 `time.sleep`
- **locator 用元组传入**,解包用 `*locator`
- **方法语义化**:`click / input / text_of / is_visible`,Page 层调用直观

---

## 四、Page 类继承 BasePage,提供业务方法

Page 类做两件事:**声明本页元素定位** + **暴露业务级方法**。它不直接操作 driver,而是调用从 BasePage 继承来的 `click / input` 等。

```python
# pages/login_page.py
from selenium.webdriver.common.by import By
from base.base_page import BasePage


class LoginPage(BasePage):
    # 元素定位:类属性,集中管理
    USERNAME = (By.ID, 'username')
    PASSWORD = (By.ID, 'password')
    LOGIN_BTN = (By.ID, 'login-button')
    ERROR_MSG = (By.CSS_SELECTOR, '.error-tip')

    # 业务方法:组合基础动作
    def set_username(self, username):
        self.input(self.USERNAME, username)

    def set_password(self, password):
        self.input(self.PASSWORD, password)

    def click_login(self):
        self.click(self.LOGIN_BTN)

    def login(self, username, password):
        """对外暴露的业务方法:用例只调用这一个"""
        self.set_username(username)
        self.set_password(password)
        self.click_login()

    def get_error_msg(self) -> str:
        return self.text_of(self.ERROR_MSG)
```

对比原文示例:原文 `LoginPage` 是独立类,**没有继承 BasePage**,所有 `find_element` 都自己写。改成继承 BasePage 后,定位元组 + 通用方法的写法更干净,显式等待也内建其中。

---

## 五、TestCase:只调 Page 提供的方法

用例层的纪律:**禁止 `import selenium`,禁止出现 `By`、`find_element`、`send_keys`**。用例读起来应该像一段业务描述。

```python
# testcases/test_login.py
import allure
import pytest
from pages.login_page import LoginPage


@allure.feature("登录")
class TestLogin:

    @allure.story("正常登录")
    def test_login_success(self, driver, config):
        login_page = LoginPage(driver)
        login_page.open(config["base_url"] + "/login")
        login_page.login("username123", "password456")
        assert "首页" in login_page.title()

    @allure.story("密码错误")
    @pytest.mark.parametrize("user,pwd", [
        ("u1", "wrong"),
        ("u2", ""),
    ])
    def test_login_fail(self, driver, config, user, pwd):
        login_page = LoginPage(driver)
        login_page.open(config["base_url"] + "/login")
        login_page.login(user, pwd)
        assert login_page.get_error_msg() != ""
```

`driver`、`config` 来自 `conftest.py` 的 fixture。

---

## 六、数据 / 配置 / 工具分离

把「会变的东西」从代码里挪出去,这是 POM 工程化的关键一步。

- **配置(`data/config.yaml`)**:环境 URL、浏览器类型、超时秒数、headless 与否。换环境(dev/test/prod)只改 YAML
- **测试数据(`data/test_data.yaml`)**:账号、查询关键字、预期结果。用 `pytest.mark.parametrize` 注入
- **工具(`utils/`)**:
  - `driver_factory.py` —— 根据 config 创建 Chrome/Firefox/Edge driver
  - `yaml_reader.py` —— 统一读取 YAML/JSON
  - `logger.py` —— logging 封装
  - `screenshot.py` —— `pytest_runtest_makereport` 钩子里调用,失败自动截图

`conftest.py` 把这些拼起来:

```python
# testcases/conftest.py
import pytest
from utils.driver_factory import create_driver
from utils.yaml_reader import load_yaml


@pytest.fixture(scope="session")
def config():
    return load_yaml("data/config.yaml")


@pytest.fixture
def driver(config):
    drv = create_driver(config["browser"], headless=config.get("headless", False))
    drv.implicitly_wait(config.get("implicit_wait", 5))
    yield drv
    drv.quit()
```

---

## 七、与 pytest + allure + Jenkins 整合

**pytest 负责「组织和跑」**
- 用例发现:`test_*.py` / `Test*` / `test_*`
- 数据驱动:`@pytest.mark.parametrize`
- fixture:driver、config、登录态、数据库连接等都用 fixture 注入
- 失败重跑:`pytest-rerunfailures`

**allure 负责「报告」**
- 装饰器组织报告结构:`@allure.feature` / `@allure.story` / `@allure.step` / `@allure.title`
- 在 BasePage 的 `click / input` 里加 `@allure.step`,自动记录每一步
- 失败钩子里 `allure.attach` 截图、页面源码、日志

**Jenkins 负责「调度」**
- 拉代码 → `pip install -r requirements.txt` → `pytest --alluredir=report/allure-results` → `allure generate` → 发布报告 → 邮件 / 钉钉通知
- 关键 Jenkinsfile 阶段:`Checkout → Install → Test → Report → Notify`
- 配合 `Allure Jenkins Plugin` 在构建页直接展示趋势图

---

## 八、调用链(ASCII 流程图):TestCase → Page → BasePage → Selenium → 浏览器

```
+-------------------+
|     TestCase      |   只写业务断言,如 login_page.login(u, p)
| (test_login.py)   |   不出现 selenium / By / find_element
+---------+---------+
          | 调用业务方法 login(u, p)
          v
+-------------------+
|    Page  Class    |   LoginPage:声明 USERNAME/PASSWORD/LOGIN_BTN
|  (login_page.py)  |   组合基础动作: self.input / self.click
+---------+---------+
          | 调用 self.click(locator) / self.input(locator, text)
          v
+-------------------+
|     BasePage      |   显式等待 + 通用动作封装
|  (base_page.py)   |   self.wait.until(EC.xxx).click()
+---------+---------+
          | 调用 driver.find_element / element.click / send_keys
          v
+-------------------+
|   Selenium  API   |   WebDriver / WebElement / By / EC / WebDriverWait
+---------+---------+
          | 通过 W3C WebDriver 协议(HTTP/JSON)
          v
+-------------------+
|   浏览器驱动      |   chromedriver / geckodriver / msedgedriver
+---------+---------+
          | 进程间通信
          v
+-------------------+
|     浏览器        |   Chrome / Firefox / Edge
+-------------------+
```

每往下一层,抽象层级降低、与「具体怎么做」靠近;每往上一层,语义更贴近业务。

---

## 九、易错点 / 原文错误订正

| # | 问题 / 易错点 | 原文写法 / 常见错误 | 推荐写法 / 订正 |
| --- | --- | --- | --- |
| 1 | Page 类未继承 BasePage | 原文 `class LoginPage:` 直接吃 driver,所有 `find_element` 自己写 | `class LoginPage(BasePage):`,通用动作交给基类 |
| 2 | 没有显式等待 | 直接 `driver.find_element(...).click()`,元素未渲染就操作会抛异常 | 用 `WebDriverWait + expected_conditions`,封装在 BasePage |
| 3 | 用 `time.sleep` 凑等待 | `time.sleep(3)` 散落各处 | 一律改为显式等待;只在极个别场景(动画)用 sleep |
| 4 | locator 写死在方法里 | `driver.find_element(By.ID, 'username')` 直接写在业务方法中 | 元素定位声明为类属性元组 `USERNAME = (By.ID, 'username')`,业务方法引用属性 |
| 5 | 元组解包写法 | 忘记 `*` 解包,写成 `find_element(self.username_locator)` 会报参数错误 | `find_element(*self.username_locator)` |
| 6 | 用例直接 import selenium | `from selenium import webdriver` 出现在 test_xxx.py | 用例只 import Page;driver 由 fixture 提供 |
| 7 | driver 没有清理 | 用完不 `quit()`,导致进程残留、端口占用 | fixture 用 `yield ... driver.quit()` 保证清理 |
| 8 | Page 方法返回值不规范 | 业务方法什么都不返回,断言只能在用例里再查元素 | 跳转类方法返回新 Page 实例(`return HomePage(self.driver)`),链式调用更顺 |
| 9 | 配置硬编码 | URL、账号写死在用例 | 抽到 `data/config.yaml` + `data/test_data.yaml`,用 fixture / parametrize 注入 |
| 10 | 失败无截图、无日志 | 报告里只有 `AssertionError`,排查靠猜 | `conftest.py` 的 `pytest_runtest_makereport` 钩子里截图并 `allure.attach` |
| 11 | 隐式等待和显式等待混用 | `implicitly_wait(10)` + `WebDriverWait(10)` 叠加,实际等待时间不可控 | 二选一,推荐统一用显式等待;隐式等待设很小值或不设 |
| 12 | 把断言写进 Page | Page 方法里 `assert xxx`,失败堆栈难看,Page 复用性变差 | 断言留在用例;Page 只返回状态(文本、可见性) |
| 13 | URL 拼接 | 原文 `driver.get("https://example.com/login")` 写死 | `self.open(config["base_url"] + "/login")` |
