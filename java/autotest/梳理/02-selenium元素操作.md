# Selenium 元素操作全集(知识点梳理)

> 适用版本:Selenium 4.x(Python)
> 来源:`selenium4.0元素定位 / 常用函数 / 元素等待方式 / iframe 切换 / 弹框 / 验证码 / 文件上传 / 常见异常说明`

---

## 一、元素定位

Selenium 4 推荐的统一写法:`driver.find_element(By.XXX, '表达式')`,旧版的 `find_element_by_id()` 系列已被废弃。

### 1.1 八种基础定位

| 序号 | 定位方式 | By 常量 | 示例 |
| --- | --- | --- | --- |
| 1 | id | `By.ID` | `driver.find_element(By.ID, 'username')` |
| 2 | name | `By.NAME` | `driver.find_element(By.NAME, 'username')` |
| 3 | class_name | `By.CLASS_NAME` | `driver.find_element(By.CLASS_NAME, 'inputBg')` |
| 4 | tag_name | `By.TAG_NAME` | `driver.find_element(By.TAG_NAME, 'input')` |
| 5 | css_selector | `By.CSS_SELECTOR` | `driver.find_element(By.CSS_SELECTOR, '#username')` |
| 6 | link_text(**完整**链接文本) | `By.LINK_TEXT` | `driver.find_element(By.LINK_TEXT, '我已有账号,我要登录')` |
| 7 | partial_link_text(**部分**链接文本) | `By.PARTIAL_LINK_TEXT` | `driver.find_element(By.PARTIAL_LINK_TEXT, '我已有账号').click()` |
| 8 | XPath(★最常用) | `By.XPATH` | `driver.find_element(By.XPATH, '//input[@name="username"]')` |

> 提示:
> - `id` 应为页面唯一,优先使用;`tag_name` 默认返回第一个匹配项,通常配合 `find_elements` 取列表
> - `link_text` / `partial_link_text` **仅适用于 `<a>` 标签**

### 1.2 CSS 选择器语法表

| 类型 | 语法 | 示例 |
| --- | --- | --- |
| ID 选择器 | `#id` | `#username` |
| 类选择器 | `.class` | `.inputBg` |
| 标签选择器 | `tag` | `input` |
| 单属性 | `[attr="val"]` | `[type="text"]` |
| 多属性 | `[a="x"][b="y"]` | `[type="text"][id="username"]` |
| 标签+属性 | `tag[attr="val"]` | `input[class="inputBg"]` |
| 标签+ID | `tag#id` | `input#username` |
| 标签+类 | `tag.class` | `input.inputBg` |
| 层级(直接子) | `A>B` | `table>tbody>tr` |
| 表格定位 | 伪类组合 | `table>tbody>tr:first-child td:nth-child(2) input[type="text"]` |

### 1.3 XPath 语法表

| # | 用法 | 语法 | 示例 |
| --- | --- | --- | --- |
| 8.1 | 绝对路径(不推荐) | `/html/body/...` | `/html/body//input[@name="username"]` |
| 8.2 | 相对路径(推荐) | `//tag[@attr="val"]` | `//input[@name="username"]` |
| 8.3 | 文本精确匹配 | `//*[text()="文本"]` | `//*[text()="我已有账号,我要登录"]` |
| 8.4 | 文本模糊匹配 | `//*[contains(text(),"片段")]` | `//*[contains(text(),"我已有账号")]` |
| 8.5 | 属性定位 | `//*[@attr='val']` | `//*[@name="username"]` |
| 8.6 | 逻辑运算 | `and / or` | `//*[@name="username" and @id="username"]` |
| 8.7 | 函数匹配属性 | `contains(@attr,'val')` | `//input[contains(@class,"inputBg")]` |

> `/` 表示根节点,`//` 从任意位置开始查找

### 1.4 Selenium 4 新增:相对定位(Relative Locators)

```python
from selenium.webdriver.support.relative_locator import locate_with

# 上下左右、附近
locate_with(By.TAG_NAME, "input").above({By.ID: "pwd"})
locate_with(By.TAG_NAME, "input").below({By.ID: "user"})
locate_with(By.TAG_NAME, "input").to_left_of({By.ID: "btn"})
locate_with(By.TAG_NAME, "input").to_right_of({By.ID: "btn"})
locate_with(By.TAG_NAME, "input").near({By.ID: "label"})
```

---

## 二、常用 API:WebDriver / WebElement / Select / ActionChains / JS

### 2.1 WebDriver 初始化与导航

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get('http://localhost/ecshop/user.php?act=register')
driver.maximize_window()
driver.save_screenshot('scr.png')        # 整页截图
```

### 2.2 WebElement 常用方法

| 方法 | 含义 |
| --- | --- |
| `send_keys('xxx')` | 在输入框中输入内容 |
| `clear()` | 清空输入框 |
| `click()` | 点击元素 |
| `text` | 获取标签文本 |
| `get_attribute('attr')` | 获取属性 |
| `size` / `location` | 元素尺寸 / 坐标 |
| `screenshot('xx.png')` | 元素级截图(验证码常用) |

### 2.3 下拉框 Select

```python
from selenium.webdriver.support.ui import Select

sel = Select(driver.find_element(By.NAME, 'sel_question'))
sel.select_by_index(3)
sel.select_by_value('favorite_movie')
sel.select_by_visible_text('我最大的爱好?')

# 多选下取消选中
sel.deselect_by_index(2)
sel.deselect_all()
```

### 2.4 ActionChains 鼠标 / 键盘动作链

```python
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys

ActionChains(driver).context_click(ele).perform()        # 右键
ActionChains(driver).double_click(ele).perform()         # 双击
ActionChains(driver).move_to_element(ele).perform()      # 悬停
ActionChains(driver).drag_and_drop(src, dst).perform()   # 拖拽

# 滑动条
slider = driver.find_element(By.ID, 'j_idt106:j_idt120')
offset = slider.size['width'] / 2                        # 想滑一半就 /2
ActionChains(driver).click_and_hold(slider) \
    .move_by_offset(offset, 0).release().perform()

# 数值自增 / 自减
num = driver.find_element(By.ID, 'j_idt106:j_idt118_input')
num.clear(); num.send_keys("10")
num.send_keys(Keys.ARROW_UP)
num.send_keys(Keys.ARROW_DOWN)

ActionChains(driver).send_keys(Keys.ENTER).perform()
```

### 2.5 JS 执行 / 滚动

```python
# 滚动到页面底部
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
# 滚动到指定元素
driver.execute_script("arguments[0].scrollIntoView();", ele)
```

### 2.6 日期控件常见处理

```python
date_input = driver.find_element(By.ID, 'j_idt106:j_idt116_input')
date_input.click()
driver.find_element(
    By.XPATH,
    '//*[@id="j_idt106:j_idt116_panel"]/div/div[2]/table/tbody/tr[5]/td[7]/a'
).click()
```

---

## 三、元素等待

### 3.1 三种等待方式

```python
# 1) 强制等待
from time import sleep
sleep(5)

# 2) 显式等待
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
element = WebDriverWait(driver, 10).until(
    EC.visibility_of_element_located((By.ID, "username"))
)

# 3) 隐式等待(全局)
driver.implicitly_wait(10)
```

### 3.2 三种等待对比

| 维度 | 强制 `sleep` | 隐式 `implicitly_wait` | 显式 `WebDriverWait + EC` |
| --- | --- | --- | --- |
| 作用范围 | 当前代码行 | **全局**(driver 生命周期) | 仅指定元素 / 条件 |
| 触发方式 | 必等满 N 秒 | 找不到时轮询,找到立即继续 | 条件不满足时轮询,成立立即继续 |
| 灵活度 | 最低 | 中等(只能控制时间) | 高(条件 + 超时 + 轮询频率) |
| 异常 | 无 | 超时抛 `NoSuchElement` | 抛 `TimeoutException`,可 try/except |
| 推荐场景 | 调试 / 演示 | 通用兜底 | 关键节点、Ajax 异步元素 |

> 原文 “显**示**等待” 是错别字,正确为 “显**式**等待”(详见末尾订正表)

### 3.3 EC 常用条件速查

| EC 条件 | 含义 |
| --- | --- |
| `presence_of_element_located` | 元素已存在 DOM(不一定可见) |
| `visibility_of_element_located` | 元素可见(宽高 > 0) |
| `element_to_be_clickable` | 元素可见且可点击 |
| `text_to_be_present_in_element` | 元素含指定文本 |
| `title_is` / `title_contains` | 页面标题判断 |
| `alert_is_present` | 浏览器原生 alert 出现 |
| `frame_to_be_available_and_switch_to_it` | iframe 出现并自动切入 |
| `staleness_of` | 元素从 DOM 消失(过期) |
| `invisibility_of_element_located` | 元素不可见 / 不存在 |

### 3.4 显式等待回调机制(ASCII)

```
┌─────────────────────────────────────────────────────┐
│  WebDriverWait(driver, timeout=10, poll=0.5).until( │
│        EC.visibility_of_element_located(loc))       │
└─────────────────────────────────────────────────────┘
              │
              ▼
   ┌──────────────────────┐
   │ t = 0:  call EC(driver)
   │   ├─ True  ─► 返回元素,跳出循环 ✔
   │   └─ False / Exception ─► sleep(poll)
   ▼
   ┌──────────────────────┐
   │ t = 0.5,1.0,...      │
   │ 反复调用 EC(driver)  │
   └──────────────────────┘
              │
        ┌─────┴─────┐
        ▼           ▼
   条件成立      t >= timeout
   返回元素     抛 TimeoutException
```

---

## 四、iframe / frame 框架切换

```python
driver.get('https://www.leafground.com/frame.xhtml')

# 1) 三种切入方式
driver.switch_to.frame(0)                          # 索引
driver.switch_to.frame('frameName')                # name / id
frame_el = driver.find_element(By.TAG_NAME, 'iframe')
driver.switch_to.frame(frame_el)                   # WebElement

# 2) 操作内嵌元素
driver.find_element(By.XPATH, '//*[@id="Click"]').click()

# 3) 回到上一层
driver.switch_to.parent_frame()                    # 回父级
driver.switch_to.default_content()                 # 回主文档
```

> `<frame>` 在 HTML5 中已废弃,`<iframe>` 是当前推荐用法。

### iframe 多层切换流程(ASCII)

```
default_content (主文档)
   │  switch_to.frame(A)
   ▼
 frame A
   │  switch_to.frame(B)
   ▼
 frame B            ◄── 当前焦点;此处定位 B 内元素 OK
   │ parent_frame() │  default_content()
   │ ▼              │  ▼
 frame A         default_content
                     │
                     └─ 又能 switch 到任意其它 frame
```

要点:
- **跨层级直接定位会抛 `NoSuchElementException`**——必须先切换到对应 iframe
- 切回主文档统一用 `default_content()`,避免层级混乱

---

## 五、各种弹框

### 5.1 浏览器原生弹框(JS 触发)

| 类型 | 触发函数 | 处理方法 |
| --- | --- | --- |
| Alert(警告) | `alert("..")` | `switch_to.alert.accept()` |
| Confirm(确认) | `confirm("..")` | `.accept()` 确定 / `.dismiss()` 取消 |
| Prompt(输入) | `prompt("..")` | `.send_keys("xx")` → `.accept()` |

通用 API:`alert = driver.switch_to.alert` → `alert.text` 取文本 / `alert.accept()` / `alert.dismiss()` / `alert.send_keys()`

```python
# Alert 警告框
driver.find_element(By.XPATH, '//*[@id="j_idt88:j_idt91"]/span[2]').click()
alert = driver.switch_to.alert
sleep(2)
alert.accept()

# Confirm 弹框
driver.find_element(By.XPATH, '//*[@id="j_idt88:j_idt93"]/span[2]').click()
driver.switch_to.alert.dismiss()
```

### 5.2 HTML 模态框 / 自定义弹框

> 这类弹框是页面里的 `<div>`(并非浏览器原生),**不能**用 `switch_to.alert`,直接定位 DOM 即可。

```python
# Sweet Alert / Modal Dialog
driver.find_element(By.XPATH, '//*[@id="j_idt88:j_idt95"]/span[2]').click()
driver.find_element(By.XPATH, '//*[@id="j_idt88:j_idt98"]/span[2]').click()
```

### 5.3 对比表

| 维度 | 原生 Alert/Confirm/Prompt | HTML 模态框(Modal) |
| --- | --- | --- |
| 本质 | 浏览器层弹窗,不在 DOM 中 | 普通 `<div>`,属 DOM |
| 切换 | 必须 `switch_to.alert` | 不用切换,直接定位 |
| 文本 | `alert.text` | 元素 `.text` |
| 输入 | `alert.send_keys()`(仅 prompt) | 元素 `send_keys()` |
| 关闭 | `accept()` / `dismiss()` | 点击对应按钮元素 |

---

## 六、验证码处理思路

实战中**没有“绝对识别”**的方案,通常组合使用:

| # | 思路 | 适用 | 优劣 |
| --- | --- | --- | --- |
| 1 | 让开发提供**万能验证码**(如固定 `8888`) | 测试环境 | 简单稳定,**生产不可用** |
| 2 | 测试环境**屏蔽验证码** | 测试环境 | 与方案 1 类似 |
| 3 | **OCR 识别**(tesseract / pytesseract) | 简单图形验证码 | 配置复杂、复杂图形识别率低 |
| 4 | 后端开**测试接口**直接拿验证码 | 全栈联调 | 需要研发支持 |
| 5 | **Cookie 绕过登录**(先手工登录复制 cookie 注入) | 登录验证码 | 简单有效,需手动维护 |

### 6.1 tesseract / pytesseract 安装步骤

1. 下载安装包:`https://digi.bib.uni-mannheim.de/tesseract/`
2. 安装,记录安装目录
3. 把 tesseract 安装目录加入 `Path`
4. 新增环境变量 `TESSDATA_PREFIX`,指向 `tessdata` 目录
5. CMD 执行 `tesseract -v` 验证
6. `pip install pytesseract -i https://pypi.tuna.tsinghua.edu.cn/simple/`
7. 重启 PyCharm

### 6.2 OCR 识别管道(ASCII)

```
┌────────────────┐   截图   ┌────────────────┐
│ WebElement     │ ───────► │ captcha.png    │
│(验证码 <img>)  │ screenshot│ (本地文件)     │
└────────────────┘          └────────────────┘
                                    │ Image.open
                                    ▼
                          ┌──────────────────────┐
                          │ PIL.Image 对象       │
                          └──────────────────────┘
                                    │ pytesseract
                                    │ image_to_string
                                    ▼
                          ┌──────────────────────┐
                          │ captcha_text (str)   │ ──► 输入框 send_keys
                          └──────────────────────┘
                       异常:TesseractNotFoundError(未装 / 未配 PATH)
```

### 6.3 代码模板

```python
def ocr_captcha(self, locator: tuple):
    """
    1) 定位到图形验证码,保存图片
    2) Image.open 打开图像
    3) pytesseract 进行 OCR 识别
    """
    captcha_element = self.location_element(*locator)
    captcha_path = setting.FILE_PATH['screenshot'] + '/captcha.png'
    captcha_element.screenshot(captcha_path)

    captcha_image = Image.open(captcha_path)
    try:
        captcha_text = pytesseract.image_to_string(captcha_image)
        return captcha_text
    except pytesseract.pytesseract.TesseractNotFoundError:
        logs.error("找不到 tesseract,pytesseract 依赖 TesseractOCR 引擎!")
```

---

## 七、文件上传

### 7.1 标准 `<input type="file">`

直接 `send_keys(绝对路径)` 即可,**无需点击**。

```python
import os

file_path = os.path.abspath('./output_folder/image1.png')   # 必须绝对路径
upload_file = driver.find_element(By.CSS_SELECTOR, '#select_btn_1')
upload_file.send_keys(file_path)
```

> 注意:Selenium 的 `send_keys` **要求绝对路径**,不能用相对路径。

### 7.2 非 `<input>` 控件(系统文件对话框)

如 Element UI、按钮触发系统对话框,DOM 上没有 `input[type=file]`,只能借助第三方工具:

| 工具 | 平台 | 用法要点 |
| --- | --- | --- |
| **PyAutoGUI** | 跨平台 | `pyautogui.write(path); pyautogui.press('enter')`,需置顶 |
| **AutoIT** + `subprocess` | Windows | 编写 `.au3` 脚本编译为 `.exe`,Python 调起 |
| **pywinauto** | Windows | 通过控件树定位「文件名」输入框 |
| **SendKeys / keyboard** | Windows | 简单粗暴键盘流模拟 |

```python
import subprocess
driver.find_element(By.ID, 'uploadBtn').click()
subprocess.call(r'D:\scripts\upload.exe D:\file.png')
```

---

## 八、Selenium 常见异常清单(30+)

> 全部位于 `selenium.common.exceptions` 模块。

| # | 异常 | 触发场景 |
| --- | --- | --- |
| 1 | `WebDriverException` | 所有 WebDriver 异常的基类 |
| 2 | `NoSuchElementException` | `find_element` 找不到元素(★ 最常见) |
| 3 | `NoSuchAttributeException` | 元素上不存在指定属性 |
| 4 | `NoSuchFrameException` | `switch_to.frame()` 时未找到 frame |
| 5 | `NoSuchWindowException` | 切换到不存在的窗口句柄 |
| 6 | `NoSuchCookieException` | 通过 name 取 cookie 时未找到 |
| 7 | `NoAlertPresentException` | `switch_to.alert` 但当前没有 alert |
| 8 | `UnexpectedAlertPresentException` | 操作元素时弹出了意外的 alert |
| 9 | `UnexpectedTagNameException` | 类型不匹配(如 `Select` 包裹非 `<select>`) |
| 10 | `StaleElementReferenceException` | 元素已 “陈旧”(DOM 已刷新 / 重渲染) |
| 11 | `ElementNotVisibleException` | 元素存在但不可见(已废弃,4.x 中合并到下一项) |
| 12 | `ElementNotInteractableException` | 元素不可交互(被遮挡 / disabled) |
| 13 | `ElementClickInterceptedException` | 点击被其它元素拦截 |
| 14 | `ElementNotSelectableException` | 元素不可选中 |
| 15 | `InvalidElementStateException` | 元素状态非法(对 readonly 输入 clear 等) |
| 16 | `MoveTargetOutOfBoundsException` | ActionChains 目标坐标超出视口 |
| 17 | `TimeoutException` | 显式等待超时未满足条件 |
| 18 | `InvalidArgumentException` | API 参数非法 |
| 19 | `InvalidSelectorException` | 选择器非法(XPath/CSS 语法错) |
| 20 | `InvalidSessionIdException` | 会话已失效 |
| 21 | `InvalidSwitchToTargetException` | 切换目标无效 |
| 22 | `InvalidCookieDomainException` | 给非当前域名添加 cookie |
| 23 | `UnableToSetCookieException` | 浏览器拒绝设置 cookie |
| 24 | `InvalidCoordinatesException` | 坐标参数无效 |
| 25 | `SessionNotCreatedException` | driver 与浏览器版本不匹配 |
| 26 | `InsecureCertificateException` | 站点证书不安全且未忽略 |
| 27 | `JavascriptException` | `execute_script` 内 JS 抛错 |
| 28 | `ScreenshotException` | 截图失败 |
| 29 | `ImeActivationFailedException` | 输入法激活失败 |
| 30 | `ImeNotAvailableException` | 输入法不可用 |
| 31 | `ErrorInResponseException` | 服务端响应错误 |
| 32 | `RemoteDriverServerException` | 远端 driver 服务端错误 |
| 33 | `UnknownMethodException` | 调用了驱动不支持的方法 |

### 8.1 高频异常对策速记

| 异常 | 常见原因 | 解决思路 |
| --- | --- | --- |
| `NoSuchElementException` | 定位写错 / 在 iframe 内 / 未加载 | 校对定位、`switch_to.frame`、加显式等待 |
| `StaleElementReferenceException` | 拿到元素后页面刷新 / 重渲染 | 重新定位元素;不要缓存 element 太久 |
| `ElementClickInterceptedException` | 弹窗、遮罩、滚动位置 | 关闭遮罩 / 滚动到元素 / JS click |
| `ElementNotInteractableException` | 元素被隐藏 / disabled | 等待可点击 `EC.element_to_be_clickable` |
| `TimeoutException` | 网络慢 / 条件写错 | 增大 timeout、放宽条件 |
| `SessionNotCreatedException` | 驱动与浏览器版本不匹配 | 升级 / 降级 chromedriver |
| `InvalidSelectorException` | XPath / CSS 语法错 | 在浏览器 console 用 `$x()` / `$$` 验证 |
| `UnexpectedAlertPresentException` | 弹窗未处理就操作元素 | 先 `switch_to.alert.accept()` |

---

## 九、易错点 / 原文错误订正

| # | 原文表述 | 正确说法 | 说明 |
| --- | --- | --- | --- |
| 1 | “显**示**等待” | “显**式**等待” | 错别字。隐式对应 implicit,显式对应 explicit |
| 2 | “**知道**拿到某个元素就立即进行下一步” | “**直到**拿到某个元素就立即进行下一步” | 同音错字 |
| 3 | “找不到元素时**跑出**异常” | “**抛出**异常” | “抛出” 为正确术语 |
| 4 | “传递的文件路径是绝对路径,而是相对路径” | “传递的文件路径**必须是绝对路径,不能是相对路径**” | 原句残缺,语意颠倒 |
| 5 | `find_element(By.XPATH, '//input[@name="Submit"]')` 注释为「点击函数」 | 这只是**查找**,需要再 `.click()` | find 与 click 容易混淆 |
| 6 | `By.TAG_NAME, 'input'` 注释「查找第一个 `<input>`」 | `find_element` 默认返回**首个**;要全部用 `find_elements`(复数) | API 行为 |
| 7 | “`<frame>` 在 HTML5 中已被废弃,`<iframe>` 是 HTML 推荐使用的标签” | 描述基本正确,但 `<iframe>` 不是 HTML5 新引入,而是被保留 | 措辞略歧义 |
| 8 | CSS 表格层级 `tag>tbody>tr:first-child td:nth-child(2)` | 写法正确,但**`tbody` 是浏览器自动注入**;F12 看到的 DOM 才能这样写,源码可能没有 | 写定位务必参照浏览器 DOM 树 |
| 9 | 滑块 `offset = slider_width / 8` 注释为「拖动到一半」 | 8 分之一并非一半,文字与代码不符 | 按需求改 `/ 2` 才是「一半」 |
| 10 | `alert.accept()` 备注「点击确定」;`alert.dismiss()` 备注「点击取消」 | 正确;补充:`prompt` 用 `alert.send_keys()` 后再 `accept()` | 原文未提 prompt |
| 11 | OCR 步骤里 `pytesseract` 与系统级 `tesseract` 混用 | `pytesseract` 是 Python 封装,**底层依赖系统 `tesseract.exe`**,两者都要装 | 容易漏装其一 |
| 12 | 显式等待示例只给了 `WebDriverWait(driver, 10).until(...)` | 还可指定 `poll_frequency`(默认 0.5s)、`ignored_exceptions`(默认 `NoSuchElement`) | 补全 API 完整签名 |
| 13 | 隐式等待 + 显式等待混用 | 二者会叠加,实际等待时间难以预估 | 推荐统一使用显式等待,隐式等待设很小值或不设 |
| 14 | `ElementNotVisibleException` | 在 Selenium 4 中已**废弃**,合并到 `ElementNotInteractableException` | 4.x 代码建议直接捕获后者 |

---

## 附:统一引用模板

```python
import os
from time import sleep
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.support.ui import Select, WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import (
    NoSuchElementException, TimeoutException,
    StaleElementReferenceException, ElementClickInterceptedException,
    ElementNotInteractableException, UnexpectedAlertPresentException,
)

driver = webdriver.Chrome()
driver.maximize_window()
driver.implicitly_wait(5)
driver.get('http://www.leafground.com/')

try:
    el = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.ID, 'username'))
    )
    el.send_keys('demo')
except TimeoutException:
    driver.save_screenshot('err.png')
finally:
    driver.quit()
```
