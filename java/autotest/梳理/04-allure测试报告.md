# Allure 测试报告(知识点梳理)

> 源文档:`allure报告环境配置.html` / `allure测试报告常用功能.html` / `allure测试报告换行展示设置.html`

---

## 零、pytest → allure-results → allure CLI → html 报告 流程图

```
 +----------------------+      +-----------------------+      +---------------------+      +---------------------+
 |  pytest 用例集       |      |  allure-results/      |      |  allure CLI         |      |  allure-report/     |
 |  test_xxx.py         | ===> |  *-result.json        | ===> |  generate / serve   | ===> |  index.html         |
 |  @allure.* 装饰器    |      |  *-attachment.png     |      |  (Java 运行时)      |      |  浏览器查看         |
 +----------------------+      +-----------------------+      +---------------------+      +---------------------+
        |                              ^                              |                            ^
        | pytest --alluredir=          |                              |                            |
        |   ./allure-results           |                              |                            |
        +------------------------------+                              +----------------------------+
                                                                       allure generate <results> -o <report> --clean
                                                                       allure serve <results>
                                                                       allure open <report>
```

要点:
- **allure-pytest** 负责在 pytest 跑用例时把结果写成 `allure-results/` 下的 json/附件
- **allure CLI**(基于 Java)负责把 `allure-results` 渲染成静态 html 报告
- 因此 **Java 运行时 + allure 命令行 + allure-pytest 插件** 三者缺一不可

---

## 一、环境配置

### 1.1 常见失败现象
- `allure: 不是内部或外部命令`(PATH 没配)
- `Cannot find Java`(JDK 未安装或 `JAVA_HOME` 未配)

### 1.2 完整配置步骤(订正后)

1. **安装 JDK 并配置环境变量**
   Allure CLI 是 Java 程序,需要安装 JDK(JDK 8 及以上),并配置 `JAVA_HOME`,在 `PATH` 中追加 `%JAVA_HOME%\bin`。
2. **下载 allure 命令行**
   解压 `allure-2.13.8.zip`,解压后通常会有两层同名目录,**只保留一层** `allure-2.13.8`(里面应直接包含 `bin/`、`config/`、`plugins/` 等)。
3. **放置目录**(原文做法是放到 Python 的 `Lib\site-packages` 下,**这不是必须的**,放在任意目录都行,只要 `bin` 进了 PATH 即可)
4. **配置 PATH**
   把 `allure-2.13.8\bin` 的绝对路径追加到系统 `Path` 变量
5. **安装 pytest 插件**(原文遗漏)
   ```bash
   pip install allure-pytest -i https://pypi.tuna.tsinghua.edu.cn/simple/
   ```
6. **重启 PyCharm / 终端**,在新终端执行 `allure --version` 验证

---

## 二、生成与查看报告

### 2.1 收集结果

```bash
pytest -s -v --alluredir=./allure-results --clean-alluredir
```

- `--alluredir`:指定结果目录
- `--clean-alluredir`:每次运行前清空旧结果(避免历史脏数据)

### 2.2 渲染 html 报告

```bash
# 方式 A:本地起一个临时服务,自动打开浏览器(推荐调试时用)
allure serve ./allure-results

# 方式 B:生成静态 html 报告到 ./allure-report
allure generate ./allure-results -o ./allure-report --clean
allure open ./allure-report
```

> 直接双击生成目录里的 `index.html` 多半看到空白,因为浏览器同源策略禁止本地加载 json,必须用 `allure open` 或起一个 http 服务。

---

## 三、装饰器与标签(常用功能)

### 3.1 `@allure.feature('登录模块')`

定义测试用例所属的 **feature**(顶层模块)。可加在类或函数上。

```python
import allure
import pytest

@allure.feature('登录模块')
class TestLogin:

    @pytest.mark.parametrize('data', read_json('./data/login_success.json'))
    def test_login_success(self, not_login_driver, data):
        username, password = data
        login_page = LoginPage(not_login_driver)
        login_page.login(username, password)
        login_page.assert_is_element_present(login_page.assert_result)
        sleep(1)
```

### 3.2 `@allure.story('用户登录成功')`

**feature 下的子分类**,级别低于 feature。

### 3.3 `@allure.title('登录成功')`

定义用例在报告中显示的标题,支持参数化变量,如 `@allure.title('登录成功-{data}')`。

### 3.4 `@allure.link / issue / testcase`

- `@allure.link(url='http://www.baidu.com', name='百度')`:任意链接
- `@allure.issue(url, name)`:关联缺陷
- `@allure.testcase(url, name)`:关联用例管理系统

### 3.5 `@allure.description('描述信息')`

给用例追加描述,显示在用例详情页的 Description 区域。

### 3.6 `@allure.severity(allure.severity_level.CRITICAL)`

严重等级:`BLOCKER`、`CRITICAL`、`NORMAL`、`MINOR`、`TRIVIAL`

### 3.7 层级关系

严格层级是 **Epic > Feature > Story**:

```
@allure.epic("xxx 项目")
└── @allure.feature("登录模块")
    └── @allure.story("用户登录成功")
        └── @allure.title("登录成功")
```

---

## 四、附件与截图

### 4.1 `allure.attach(...)` 是函数,不是装饰器

```python
allure.attach(body, name='截图', attachment_type=allure.attachment_type.TEXT)
allure.attach.file(source, name='日志', attachment_type=allure.attachment_type.TEXT)
```

`attachment_type` 常用:
- `TEXT`、`HTML`、`JSON`、`XML`、`URI_LIST`
- `PNG`、`JPG`、`GIF`、`SVG`
- `MP4`、`OGG`、`WEBM`、`PCAP`

### 4.2 页面截屏

在 `BasePage` 封装截屏方法:

```python
def screenshots_png(self):
    """页面截屏,返回 PNG 二进制"""
    return self.__driver.get_screenshot_as_png()
```

在业务方法里调用:

```python
import allure
from selenium.webdriver.common.by import By
from util_tools.basePage import BasePage


class FilesUploadPage(BasePage):
    url = 'https://www.leafground.com/file.xhtml'
    files_input = (By.XPATH, '//*[@id="j_idt88:j_idt89_input"]')

    def files_upload(self):
        self.open_url(self.url)
        allure.attach(self.url, '打开测试页面',
                      attachment_type=allure.attachment_type.TEXT)
        file_path = r'C:\Users\xxx\Desktop\文件上传.txt'
        allure.attach(file_path, '上传文件',
                      attachment_type=allure.attachment_type.TEXT)
        self.send_keys(self.files_input, file_path)
        allure.attach(self.screenshots_png(), '文件上传截屏',
                      attachment_type=allure.attachment_type.PNG)
```

---

## 五、动态报告(动态标题 / feature / story)

当用例数据来自 yaml/json/excel 参数化、标题想随数据变化时,装饰器写死的值无法动态计算,要在用例方法里用 `allure.dynamic.*`:

```python
import allure
import pytest

@allure.feature('公用接口,供调试使用')
class TestLogin:

    @allure.story("用户登录")
    @pytest.mark.parametrize('base_info,testcase',
                             get_testcase_yaml('./testcase/public/loginName.yml'))
    def test_login_true(self, base_info, testcase):
        allure.dynamic.feature(base_info['api_name'])
        allure.dynamic.title(testcase['case_name'])
        print(f'接口请求地址信息:{base_info}')
        print(f'测试用例数据:{testcase}')
```

常用动态接口:`allure.dynamic.feature/story/title/description/link/issue/testcase/severity`

---

## 六、换行展示与显示问题

### 6.1 修改 allure 报告 logo

在 `allure-2.13.8\config\allure.yml` 的 `plugins:` 列表末尾追加 `custom-logo-plugin`:

```yaml
plugins:
  - junit-xml-plugin
  - xunit-xml-plugin
  - trx-plugin
  - behaviors-plugin
  - packages-plugin
  - screen-diff-plugin
  - xctest-plugin
  - jira-plugin
  - xray-plugin
  - custom-logo-plugin
```

把自定义 logo 图片放到:

```
<allure 安装目录>\plugins\custom-logo-plugin\static\
```

修改同目录下的 `styles.css`(**注意:CSS 注释只能用 `/* ... */`,不能用 `#`**):

```css
.side-nav__brand {
  background: url('sinoiov.png') no-repeat left center !important;
  /* url() 内填自己 logo 文件名 */
  margin-left: 10px;
  height: 40px;
  background-size: contain !important;
}
.side-nav__brand span {
  display: none;
}
.side-nav__brand:after {
  content: "Test Report";
  /* logo 右侧文字,不写则默认显示 Allure */
  margin-left: 20px;
}
```

### 6.2 用例换行显示 / 取消标题尾部参数化字符串

**现象**:参数化用例会在标题尾部拼一长串 `[xxx-yyy-zzz]`,把标题挤成一行很难看。

**解决方法**:修改

```
<Python 安装路径>\Lib\site-packages\allure_pytest\listener.py
```

找到调用 `test_result.parameters.extend(...)`,把它改写为传入空列表(等价于不再追加参数到标题中):

```python
test_result.parameters.extend([])
```

> 这一步改的是第三方库源码,**升级 `allure-pytest` 后需要重新修改**。

### 6.3 改完后报告显示不全 / 缺少用例

原因:`allure-pytest` 版本过高(**> 2.13.2 一般不再支持上述改法**),需要降级。

```bash
pip uninstall allure-pytest
pip install allure-pytest==2.9.41 -i https://pypi.tuna.tsinghua.edu.cn/simple/
```

---

## 七、易错点 / 原文错误集中订正

| # | 原文写法 / 现象 | 问题 | 正确写法 / 说明 |
| --- | --- | --- | --- |
| 1 | 章节标题写 `allure.attachment` | API 名拼错 | 实际是 **`allure.attach`**(函数,不是装饰器) |
| 2 | 代码示例 `allure.acctch()` | 拼写错误 | **`allure.attach()`** |
| 3 | 文中 `allure.attch` 等变体 | 同上 | 统一拼成 `allure.attach` |
| 4 | 「页面类通过 allure.acctch() 调用即可」 | 拼写 + 表述 | 「在用例 / 页面方法内部通过 `allure.attach()` 调用」 |
| 5 | 把 allure-2.13.8 必须放到 `Lib\site-packages` | 描述偏死 | 任意路径均可,只要 `bin` 进 PATH;放 site-packages 仅是个人习惯 |
| 6 | 仅提「装 Java」,未提 `JAVA_HOME` | 步骤不完整 | 需要安装 JDK 并配 `JAVA_HOME`,再把 `%JAVA_HOME%\bin` 加入 PATH |
| 7 | 未提到 `pip install allure-pytest` | 步骤缺失 | pytest 端必须装 `allure-pytest` 插件,否则 `--alluredir` 不生效 |
| 8 | 未给出 `pytest --alluredir=...` 命令 | 步骤缺失 | 收集结果需要此参数;渲染用 `allure generate` 或 `allure serve` |
| 9 | `styles.css` 中行尾用 `#` 写注释 | CSS 语法错 | CSS 注释必须用 `/* ... */`,`#` 在 CSS 里是 id 选择器 |
| 10 | 截图代码 `r'C:\Users\yaozm\Desktop\文件上传.txt'` 路径示例 | 路径硬编码 | 建议改成相对路径或从配置读取,避免换机器即失效 |
| 11 | `@allure.link(url=..., name=...)` 用于跟踪 Bug | 没问题,但易混 | 跟踪 Bug 用 `@allure.issue`,用例管理用 `@allure.testcase`,语义更明确 |
| 12 | 描述把 story 说成「feature 下的 story,级别比 feature 低」 | 表述弱 | 严格层级是 **Epic > Feature > Story**,Story 在 Feature 之下 |
| 13 | 动态显示一节里图省略了 `import allure` | 易漏 | 使用 `allure.dynamic.*` 前必须 `import allure` |
| 14 | 「test_result.parameters.extend 方法注释」 | 措辞含糊 | 准确说法:把这一行调用替换为 `test_result.parameters.extend([])`,等价于清空参数 |
| 15 | 「版本是 2.13.2 以上的多半不支持」 | 信息可能过期 | 不同版本差异较大,实际改法应以当前安装版本的 `listener.py` 源码为准;若降级首选 2.9.x |
| 16 | 报告打开方式没提 | 易踩坑 | 不要直接双击 `index.html`,要用 `allure open` 或 `allure serve` |
| 17 | `@allure.severity(...)` 未列全等级 | 信息不全 | 完整等级:BLOCKER / CRITICAL / NORMAL / MINOR / TRIVIAL |
| 18 | 换行设置改完后「缺少用例」才提到版本 | 因果顺序混乱 | 推荐顺序:先确认 `allure-pytest` 版本(≤ 2.13.2),再修改 `listener.py`,避免来回踩坑 |
