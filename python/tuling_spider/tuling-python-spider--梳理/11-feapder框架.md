# 第 11 章 feapder 框架 — 知识点梳理

> 对应 11.1 ~ 11.2。feapder 国产爬虫框架,内置浏览器渲染池/XHR 拦截/MysqlDB 直连,定位"开箱即用"。

---

## 1. 四种爬虫模板

| 模板 | 场景 | 关键能力 |
|---|---|---|
| `AirSpider` | 单机轻量、学习入门 | 无任务队列,内存调度,启动即跑 |
| `Spider` | 分布式 | Redis 任务队列、断点续爬、报警、自动入库 |
| `TaskSpider` | 分布式 + 种子来源多样 | 从 redis/mysql/自定义源取任务 |
| `BatchSpider` | 批次/周期采集 | 自动按周期划分批次(如每 7 天全量) |

学习路径:**AirSpider → Spider → BatchSpider**。生产分布式选 Spider/BatchSpider。

⚠️ 与 scrapy 对比:scrapy 上层灵活、生态广;feapder 内置了更多"业务级"东西(MysqlDB、报警、监控、浏览器池),适合"赶项目"。

---

## 2. 架构 & 执行流程

```
                ┌───────────────────────────────┐
                │           spider              │ 调度核心
                └─┬─────────┬─────────┬─────────┘
                  │         │         │
   ① 触发         ▼         ▼         ▼
       ┌──────────────┐ ┌────────────┐ ┌────────────┐
       │start_requests│ │ collector  │ │parser_ctrl │ (多线程)
       └──────┬───────┘ └─────▲──────┘ └──┬──────▲──┘
              │ ② Request     │           │ ⑥ 调度 │
              ▼               │ ④ 批量拉  │       │
       ┌──────────────┐       │           ▼       │
       │request_buffer│       │      ┌────────────┐
       └──────┬───────┘       │      │  request   │
              │ ③ 批量入队    │      │ (~requests)│
              ▼               │      └──┬─────────┘
       ┌──────────────────────┴────┐    │ ⑦ HTTP
       │  任务队列(Redis/MySQL)    │    ▼
       └───────────────────────────┘  目标站
                                       │
                                       ▼ ⑧
                                  ┌──────────┐
                                  │ response │ 自动解码/封装
                                  └────┬─────┘
                                       │ ⑨
                                       ▼
                                  ┌──────────┐
                                  │  parser  │ 业务解析(用户写)
                                  └────┬─────┘
                            ⑩ yield item / Request
                                       │
                          ┌────────────┴────────────┐
                          ▼                         ▼
                  ┌──────────────┐         ┌───────────────┐
                  │ item_buffer  │         │request_buffer │
                  └──────┬───────┘         └───────┬───────┘
                         │ 批量                    │ 新 Request
                         ▼                         ▼
                     数据库                     任务队列
```

12 步:`start_requests` 生产任务 → `request_buffer` 批量入任务队列 → `collector` 拉到内存 → `parser_control` 取任务调 `request` → 下载 → 封装 `response` → 回 `parser_control` → 调用对应 `parser` → 解析产出 `item` / 新 `Request` → 分发到 buffer → 批量入库。

AirSpider 简化版:**无任务队列**,任务直接走内存,因此最轻量但不支持分布式。

---

## 3. 安装 & 项目脚手架

```bash
conda create -n feapder_base python=3.9 && conda activate feapder_base
pip install "feapder[all]"          # 含浏览器渲染、报警等

# 单文件爬虫
feapder create -s douban            # 选 AirSpider / Spider / TaskSpider / BatchSpider

# 完整项目
feapder create -p wp_shop
# 生成:
# wp_shop/
# ├── items/        # Item 与数据库表映射
# ├── spiders/      # 爬虫脚本
# ├── main.py       # 运行入口
# └── setting.py    # 配置

# 基于数据表生成 Item(文件名 = 数据表名)
feapder create -i douban_feapder

# 单独生成 setting.py
feapder create --setting
```

⚠️ `feapder create -i <name>` 要求**数据表 `<name>` 已存在**,否则报"找不到表"。生成后 `__table_name__` 自动写入。

---

## 4. AirSpider 最小骨架

```python
import feapder

class Douban(feapder.AirSpider):
    def start_requests(self):
        for page in range(10):
            yield feapder.Request(
                f"https://movie.douban.com/top250?start={page*25}&filter="
            )

    def parse(self, request, response):
        for li in response.xpath('//ol/li/div[@class="item"]'):
            item = {
                'title':      li.xpath('.//span[1]/text()').extract_first(),
                'detail_url': li.xpath('.//a/@href').extract_first(),
                'score':      li.xpath('.//span[@class="rating_num"]/text()').extract_first(),
            }
            # 通过 item= 把数据带到下一级回调
            yield feapder.Request(item['detail_url'],
                                  callback=self.parse_detail, item=item)

    def parse_detail(self, request, response):
        request.item['detail_text'] = response.xpath('//div[@class="indent"]//text()').extract_first('').strip()
        yield request.item       # yield Item 自动入库

if __name__ == "__main__":
    Douban(thread_count=5).start()
```

要点:
- `feapder.Request` 兼容 `requests` 全部参数,**额外支持** `callback` / `item` / `download_midware` / `render` 等。
- `response.xpath(...).extract_first()` / `.extract()`:与旧 scrapy 风格一致(parsel)。
- 跨回调透传数据用 `item=...`(在 `request.item` 读取),不要再用 `meta`。

---

## 5. Item + MysqlDB(自动入库)

```python
from feapder import Item

class DoubanFeapderItem(Item):
    __table_name__ = "douban_feapder"        # 必须与数据库表名一致
    def __init__(self, *a, **kw):
        # 不要写 id(自增列)
        self.title = None
        self.score = None
        self.detail_url = None
        self.detail_text = None
```

在 `setting.py`:

```python
MYSQL_IP = "localhost"
MYSQL_PORT = 3306
MYSQL_DB = "py_spider"
MYSQL_USER_NAME = "root"
MYSQL_USER_PASS = "root"
```

`yield item` 即落库,内部走 **`insert ignore`**(主键冲突静默跳过);需要更新策略可改 `update_columns`。

直接用 `MysqlDB`(脚本场景):

```python
from feapder.db.mysqldb import MysqlDB
db = MysqlDB(ip='localhost', port=3306, user_name='root', user_pass='root', db='py_spider')
db.execute("create table if not exists ...")
db.add("insert ignore into douban_feapder(...) values(...)")
```

`MysqlDB` 内部基于 `DBUtils` 连接池 + `pymysql`,**线程安全**。

---

## 6. 下载中间件

**默认中间件**——重写 `download_midware` 全局生效:

```python
class Douban(feapder.AirSpider):
    def download_midware(self, request):
        request.headers = {'User-Agent': 'xxx'}
        request.proxies = {'http': 'http://127.0.0.1:7890'}
        return request           # 必须 return request
```

**单请求中间件**——只对某条 Request 生效:

```python
yield feapder.Request(url, download_midware=self.custom_mw)

def custom_mw(self, request):
    request.headers['Cookie'] = '...'
    return request
```

注意:与 scrapy 不同,feapder **每个 Request 只走一个中间件**(默认 or 自定义,二选一),不存在中间件链。

---

## 7. 响应校验 + 自动重试

```python
def validate(self, request, response):
    # 返回值约定:
    #  True/None  → 进入 parse
    #  False      → 丢弃该请求
    #  raise ...  → 触发重试(默认最多 100 次)
    if response.status_code != 200:
        raise Exception("状态码异常")
```

修改重试上限:`setting.py` 中 `SPIDER_MAX_RETRY_TIMES = 5`(以及对应失败入库行为)。

⚠️ 默认 100 次重试太多——遇到 403/封禁会疯狂打目标站;生产建议降到 3~5 次并配合代理切换。

---

## 8. 浏览器渲染池(内置)

```python
# Request 里加 render=True 即可走浏览器
yield feapder.Request("https://news.qq.com/", render=True)

def parse(self, request, response):
    from feapder.utils.webdriver import WebDriver
    browser: WebDriver = response.browser     # 拿到底层 selenium 实例
    browser.find_element(By.ID, 'kw').send_keys('feapder')
```

`setting.py` 关键配置:

```python
WEBDRIVER = dict(
    pool_size=1,                # 池里浏览器实例数
    load_images=True,
    user_agent=None,            # 字符串 或 无参函数
    proxy=None,                 # ip:port 或 函数
    headless=False,
    driver_type="CHROME",       # CHROME/EDGE/FIREFOX/PHANTOMJS(已废)
    timeout=30,
    window_size=(1024, 800),
    executable_path=None,       # 驱动路径,留 None 走默认
    render_time=0,              # 打开后再等 N 秒取源码
    custom_argument=[
        "--ignore-certificate-errors",
        "--disable-blink-features=AutomationControlled",
    ],
    xhr_url_regexes=None,       # 拦截哪些 XHR(数组,正则)
    auto_install_driver=True,   # 自动下载匹配驱动(国内可能失败)
    download_path=None,
    use_stealth_js=False,       # stealth.min.js 反检测
)
```

⚠️ `auto_install_driver=True` 在国内网络环境下经常拉不下来驱动二进制;建议关掉,手动放好 `chromedriver`,显式指定 `executable_path`。

---

## 9. XHR 拦截(免 JS 逆向利器)

```python
# 1. 在 setting.py 写正则
WEBDRIVER = dict(..., xhr_url_regexes=["/api/job/search"])

# 2. 业务里取数据
browser = response.browser
text  = browser.xhr_text("/api/job/search")
data  = browser.xhr_json("/api/job/search")
xhr   = browser.xhr_response("/api/job/search")
xhr.request.url; xhr.request.headers; xhr.request.data
xhr.headers;     xhr.url;             xhr.content
```

效果与 DrissionPage 的 `page.listen.start` 类似——**让浏览器替你跑掉加密,你只取结果**。

实际示例(应届生招聘):浏览器打开列表页,后台 XHR 命中 `open/noauth/job/search` → 直接 `browser.xhr_json(...)` 拿到 JSON,无需解析 DOM。

---

## 10. 完整项目目录结构

```
wp_shop/
├── items/
│   └── wp_shop_info_item.py        # Item 类
├── spiders/
│   └── wp_spider.py                # 爬虫
├── main.py                         # 入口:WpSpider().start()
└── setting.py                      # MYSQL_*/WEBDRIVER/SPIDER_MAX_RETRY_TIMES 等
```

`spiders/wp_spider.py`(唯品会示例骨架):

```python
import feapder, time
from random import randint
from items import wp_shop_info_item
from selenium.webdriver.common.by import By
from feapder.utils.webdriver import WebDriver

class WpSpider(feapder.AirSpider):
    def start_requests(self):
        url = 'https://category.vip.com/suggest.php?keyword=电脑&page={}'
        for page in range(1, 13):
            yield feapder.Request(url.format(page), render=True)

    def parse(self, request, response):
        browser: WebDriver = response.browser
        time.sleep(2)
        self.drop_down(browser)             # 触发懒加载

        for div in browser.find_elements(By.XPATH, '//section[@id="J_searchCatList"]/div'):
            item = wp_shop_info_item.WpShopInfoItem()
            item['title'] = div.find_element(By.XPATH, './/div[2]/div[2]').text
            item['price'] = div.find_element(By.XPATH, './/div[@class="...sale-price..."]').text
            yield item

    def drop_down(self, browser):
        for i in range(1, 12):
            browser.execute_script(f'document.documentElement.scrollTop = {i*1000}')
            time.sleep(randint(1, 2))
```

---

## 11. 原文需要纠正/补充的点

| # | 原文 | 问题 | 正确做法 |
|---|------|------|---------|
| 1 | 文档链接 `boris-code.gitee.io/feapder` | 官方主域已迁到 `feapder.com`,gitee 是镜像 | 以 `feapder.com` 为准 |
| 2 | `insert into ... values(0, ...)` 让 id 自增 | 显式写 `0` 会真的插 `0`,自增链断裂后会冲突 | 写 `insert ... (title, ...) values(...)`,自增列别填 |
| 3 | `MysqlDB(ip='localhost', user_name='root', ...)` 缺 `port` | 默认 3306,大多场景没问题;非默认端口必须传 | 显式传 `port=` |
| 4 | Item 里把 `self.id = None` 注释掉 | 注释方式可读性差 | 自增列**根本不要写**;feapder 会自动用 `insert ignore` |
| 5 | "默认重试 100 次" | 100 次会持续冲击目标站 | `SPIDER_MAX_RETRY_TIMES = 3~5` 更合理 |
| 6 | `auto_install_driver=True` | 国内拉 chromedriver 经常 403/超时 | 关掉,手动放驱动 + 显式 `executable_path` |
| 7 | `driver_type` 选项含 `PHANTOMJS` | PhantomJS 早已停止维护(2018) | 用 CHROME/EDGE/FIREFOX |
| 8 | 反检测仅靠 `--disable-blink-features=AutomationControlled` | 不够 | 同时开 `use_stealth_js=True` |
| 9 | `start_requests` 一次性 yield 全部页 | AirSpider 无任务队列,大量任务直接占内存 | Spider/BatchSpider 才合适大批量;AirSpider 控制好数量级 |
| 10 | `yield request.item` 之后才入库——但没说明列名必须与 Item 字段对齐 | 字段不匹配会报"unknown column" | Item 字段名 = MySQL 列名 + `__table_name__` = 表名 |
| 11 | `download_midware` 与 scrapy 中间件类比"链式" | feapder 的下载中间件**不是链**,每个请求只走一个 | 想要多步处理就在一个中间件里串行写 |
| 12 | "feapder 集成断点续爬" 解释在 AirSpider 一栏出现 | AirSpider **不支持**断点续爬(无队列) | 断点续爬要用 Spider / BatchSpider + Redis 队列 |
| 13 | `from douban_feapder_item import DoubanFeapderItem` 同级导入 | 完整项目结构里 Item 在 `items/` 包下 | 应 `from items.douban_feapder_item import DoubanFeapderItem` |
| 14 | `feapder.Request(..., proxies=...)` 静态写代理 | 单一 IP 易封 | 在 `download_midware` 里每次从代理池取,配合 `validate` 失败重试 |
| 15 | `validate` 内 `print('响应状态码:', response.status_code)` | 高并发下日志爆炸 | 用 `feapder.utils.log.log` 按级别打,设置 LOG_LEVEL |

---

## 12. 速记

- 选型口诀:**学习用 AirSpider,生产 Spider,周期 BatchSpider**。
- `feapder.Request` 三大利器:`callback` / `item`(透传) / `render=True`(浏览器渲染) / `download_midware`(单请求中间件)。
- 想入库?写 Item + `__table_name__` + `yield item`,框架走 `insert ignore`。
- 反爬终极方案:`render=True` + `xhr_url_regexes` + `browser.xhr_json(...)` = 跳过 JS 逆向。
- `validate` 返回值:**True/None 进解析、False 丢弃、抛异常重试**。
- 国内一律 `auto_install_driver=False`,自带驱动。
