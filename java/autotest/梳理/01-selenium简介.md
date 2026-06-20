# Selenium 简介(知识点梳理)

> 源文档:`selenium简介.html`(印象笔记 YXBJ 导出)

---

## 一、Selenium 是什么

Selenium 是一个流行的 Web 应用自动化测试工具,用于测试 Web 应用程序的功能与用户界面。它能够模拟用户在浏览器中的操作(点击、输入、下拉选择等),并验证页面元素的状态与属性,从而帮助测试人员自动执行重复测试任务,提高效率、减少人工错误。

### 核心特点

| 特点 | 说明 |
| --- | --- |
| 开源免费 | Apache 2.0 协议 |
| 多语言 | Java、Python、C#、JavaScript、Ruby 等 |
| 多平台 | Windows / Linux / macOS |
| 多浏览器 | Chrome、Edge、Firefox、Safari |
| 分布式 | 借助 Selenium Grid 把用例分发到不同机器执行 |
| 社区成熟 | 文档、生态、案例丰富 |

---

## 二、安装(pip + Python 版本要求)

| 项 | 要求 |
| --- | --- |
| Python | **3.7 及以上**(Selenium 4.x 强制) |
| 安装命令 | `pip install selenium` |
| 国内源(清华) | `pip install selenium -i https://pypi.tuna.tsinghua.edu.cn/simple/` |
| 验证 | `python -c "import selenium; print(selenium.__version__)"` |

---

## 三、浏览器驱动(Chrome / Edge)

### 1. 版本匹配原则

**驱动版本必须与浏览器版本匹配**,否则启动报错。若驱动暂未跟进最新浏览器版本,则需把浏览器降级到驱动支持的版本。

### 2. 下载地址

| 浏览器 | 驱动下载地址 |
| --- | --- |
| Chrome(旧版,≤114) | <http://chromedriver.storage.googleapis.com/index.html> |
| Chrome(新版,≥115,Chrome for Testing) | <https://googlechromelabs.github.io/chrome-for-testing/> |
| Chrome 历史版本(浏览器本体) | <https://www.slimjet.com/chrome/google-chrome-old-version.php> |
| Microsoft Edge | <https://developer.microsoft.com/zh-cn/microsoft-edge/tools/webdriver/> |

### 3. 安装方式

1. 在浏览器地址栏访问 `chrome://version` 或 `edge://version` 查看浏览器版本号
2. 下载对应版本的驱动可执行文件(`chromedriver.exe` / `msedgedriver.exe`)
3. 放到 **Python 安装目录** 或任何已加入 `PATH` 的目录;**Selenium 4.6+ 内置 Selenium Manager**,通常无需手动下载

### 4. Selenium 4.x 推荐的驱动创建方式(原文未提及)

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options

service = Service(executable_path="chromedriver.exe")   # 4.x 推荐
options = Options()
driver = webdriver.Chrome(service=service, options=options)
```

> Selenium 3.x 的 `webdriver.Chrome(executable_path=...)` 写法在 4.x 已经 **DeprecationWarning**,4.x 起统一通过 `Service` 对象注入。

---

## 四、Selenium 工作原理(三层架构)

### 文字描述

1. 自动化脚本中导入 `selenium` 库的 `WebDriver` 类并创建对象
2. 调用 WebDriver 提供的 API(如 `find_element`、`click`),客户端库把调用 **转化为 HTTP 请求**(JSON Wire / W3C WebDriver 协议)
3. 请求发送给 **浏览器驱动**(ChromeDriver、msedgedriver 等),驱动是 Selenium 与浏览器之间的桥梁
4. 驱动把 HTTP 请求 **翻译成浏览器原生指令**(Chrome 使用 DevTools Protocol),发送给浏览器执行
5. 浏览器执行完毕后把结果返回给驱动 → 驱动封装成 HTTP 响应 → 客户端库解析为 Python 对象返回给脚本

### 通信流程图(ASCII)

```
+-------------------------------+
|  Python 自动化脚本             |
|  driver.find_element(By.ID,…) |
+---------------+---------------+
                |
                | 函数调用
                v
+-------------------------------+
|  Selenium 客户端库             |
|  (selenium-python bindings)   |
+---------------+---------------+
                |
                | HTTP 请求 (W3C WebDriver 协议 / JSON Wire)
                | POST /session/{id}/element  { "using":"id","value":"kw" }
                v
+-------------------------------+
|  浏览器驱动                    |
|  ChromeDriver / msedgedriver  |
|  (本地起 HTTP server)          |
+---------------+---------------+
                |
                | 原生协议 (Chrome DevTools Protocol 等)
                v
+-------------------------------+
|  浏览器 (Chrome / Edge)        |
|  渲染、执行 JS、操作 DOM        |
+-------------------------------+

      <==== HTTP 响应原路返回 ====
```

---

## 五、Selenium 4.0 新特性

| # | 变化点 | 说明 |
| --- | --- | --- |
| 1 | Python 版本要求 | 必须 **Python 3.7+** |
| 2 | 元素定位 API 重构 | **废弃** `find_element_by_*` 系列,统一为 `find_element(by, value)` / `find_elements(by, value)`,配合 `By` 枚举使用 |
| 3 | 相对定位(Relative Locators) | `above` / `below` / `to_left_of` / `to_right_of` / `near` |
| 4 | 新窗口 / 标签页 API | `driver.switch_to.new_window('tab')` / `new_window('window')` |
| 5 | Service 对象 | 推荐通过 `Service(executable_path=...)` 创建驱动,旧的 `executable_path` 参数已弃用 |
| 6 | W3C 协议 | 完全切换到 W3C WebDriver 标准,JSON Wire 协议被移除 |
| 7 | Selenium Manager(4.6+) | 自动下载/管理浏览器驱动,可省去手动放 chromedriver 的步骤 |

### 新版定位 API 用法示例

```python
from selenium.webdriver.common.by import By

driver.find_element(By.ID, "kw")
driver.find_element(By.CSS_SELECTOR, "input.s_btn")
driver.find_elements(By.XPATH, "//a")
```

### 相对定位示例

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.relative_locator import locate_with

password_field = driver.find_element(By.ID, "password")
# 找到 password 输入框 "上方" 的 input
username = driver.find_element(
    locate_with(By.TAG_NAME, "input").above(password_field)
)
```

---

## 六、原文错误与勘误

| # | 原文写法 | 问题 | 正确写法 |
| --- | --- | --- | --- |
| 1 | "selenium4.0 以上已经弃用 `find_element_by_*`" | 表述正确,但应明确:`find_element_by_id` / `find_element_by_xpath` / `find_element_by_css_selector` 等 **全部** snake_case 方法在 4.x 全部移除(3.x 末期是 deprecation warning,4.x 已直接删除) | 统一使用 `find_element(By.XXX, value)` |
| 2 | 相对定位方法名写作 `toLeftOf` / `ToRightOf` | **错误**:这是 Java 绑定的驼峰命名。**Python 绑定为 snake_case** | `to_left_of` / `to_right_of` |
| 3 | `above` / `below` 大小写 | 写法正确,但应注意全部小写 | `above` / `below` / `near` |
| 4 | 未提到 `Service` 对象 | 原文未指出 4.x 推荐使用 `Service` 创建驱动 | 见本文第三节示例 |
| 5 | 未提到 Selenium Manager | 4.6+ 已能自动管理驱动,文中仍要求手动下载放到 Python 目录,信息略陈旧 | 4.6+ 可省略驱动手动下载 |
| 6 | "WebDriver 类" | 严格来说调用方是 `webdriver.Chrome` 等具体子类,而非抽象的 `WebDriver` 类本身 | `from selenium import webdriver; driver = webdriver.Chrome(...)` |
| 7 | "selenium 客户端库相应的函数发送请求给浏览器驱动" | 正确,但可补充协议名:**W3C WebDriver Protocol**(4.x 默认),取代旧的 JSON Wire Protocol | 见第四节 |

### 相对定位方法名对照表(语言绑定)

| 含义 | Java(原文写法) | **Python(正确)** |
| --- | --- | --- |
| 在……之上 | `above` | `above` |
| 在……之下 | `below` | `below` |
| 在……之左 | `toLeftOf` | **`to_left_of`** |
| 在……之右 | `toRightOf` | **`to_right_of`** |
| 在……附近 | `near` | `near` |

---

## 七、快速上手最小示例

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By

service = Service()              # 4.6+ 留空即可自动管理驱动
driver = webdriver.Chrome(service=service)

driver.get("https://www.baidu.com")
driver.find_element(By.ID, "kw").send_keys("selenium")
driver.find_element(By.ID, "su").click()

driver.quit()
```
