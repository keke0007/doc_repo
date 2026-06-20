# auto 目录知识点梳理总目录

把 `auto/` 下 19 份印象笔记导出 HTML 按主题重新组织、订正错误,并对涉及跨文件 / 跨工具协作的流程绘制 ASCII 流程图。

## 目录

| 文件 | 主题 | 关键流程图 |
| --- | --- | --- |
| [01-selenium简介.md](./01-selenium简介.md) | 安装、驱动、Selenium 4 新特性 | 客户端库 → ChromeDriver(HTTP) → Chrome(CDP) |
| [02-selenium元素操作.md](./02-selenium元素操作.md) | 定位、API、等待、iframe、弹框、验证码、文件上传、异常 | 显式等待回调、iframe 多层切换、OCR 管道 |
| [03-pytest框架.md](./03-pytest框架.md) | 用例、fixture、参数化、标记、conftest、钩子、pytest.ini | 钩子执行顺序、conftest 查找链 |
| [04-allure测试报告.md](./04-allure测试报告.md) | 环境、装饰器、附件、动态报告、换行设置 | pytest → allure-results → CLI → html |
| [05-jenkins持续集成.md](./05-jenkins持续集成.md) | 部署、向导、构建、Allure 集成、cron、触发器 | 代码提交 → Jenkins → pytest → Allure → 通知 |
| [06-POM设计模式.md](./06-POM设计模式.md) | 分层目录、BasePage、Page、TestCase、数据 / 工具分离 | TestCase → Page → BasePage → Selenium API → 浏览器 |
| [07-requirements.md](./07-requirements.md) | 生成、安装、版本约束、镜像、venv | A 机 freeze → 提交 → B 机 install |

## 重点订正速览

- **selenium 4 相对定位**:Python 是 `to_left_of` 等 snake_case,**不是** Java 的 `toLeftOf`(原文错)
- **`find_element_by_xxx`** 系列在 selenium 4 已**移除**,统一 `find_element(By.XXX, value)`
- **`ElementNotVisibleException`** 在 selenium 4 已废弃,合并到 `ElementNotInteractableException`
- **「显**示**等待」→ 「显**式**等待」**,「跑出异常」→ 「**抛出**异常」
- **pytest 钩子** `pytest_runtest_makereport` 签名必须 `(item, call)`;`coucome` → `outcome`
- **dict 参数化**:`username, password = data` 拿到的是 keys,不是 values
- **YAML 参数化**应使用 list 语法 `- [admin123, "123456"]`,不能整行当字符串
- **`python_functions = test`** 会误收 `testing_xxx`,要写 `test_*`
- **自定义 marker** 不在 `pytest.ini` 注册会触发 `PytestUnknownMarkWarning`
- **`allure.attach`** 是函数不是装饰器,且不是 `attachment`/`acctch`/`attch`
- **`styles.css` 注释**只能 `/* */`,**不能用 `#`**
- **Jenkins.war 不能跑在 Tomcat 10+** 上(javax/jakarta 包冲突),必须 **Tomcat 9.x**
- **Jenkins cron 是 5 字段无秒位**,`H` 是 Jenkins 独有的哈希占位符(标准 cron 没有)
- **`H 12 * * *` 不是 12:00 整**,而是 12 点这一小时内某分钟,严格整点用 `0 12 * * *`
- **`requestment.txt`** → `requirements.txt`(原文 typo)
- **`pip freeze`** 在全局环境会导出无关包,应配合 venv 使用
- **`pipreqs ./` 在 Windows 必须加 `--encoding=utf8`**,否则中文注释报 UnicodeDecodeError
