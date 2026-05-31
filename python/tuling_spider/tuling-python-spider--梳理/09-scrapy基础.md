# 第 9 章 scrapy 框架 — 基础操作

> 对应 9.1 ~ 9.13。scrapy 是多组件协作的异步爬虫框架,**必出流程图**。

---

## 1. 为什么用 scrapy

- 异步引擎(基于 Twisted)开箱即用,无需手撸 asyncio。
- 标准化项目结构 + 中间件 + 管道,**代码组织 = 工程化**。
- 内置去重、重试、限速、并发控制、调度、cookie、签名等基础设施。

⚠️ scrapy 解决"工程化",**反爬复杂的站点还是要靠 selenium/DrissionPage + 中间件配合**;两者不冲突。

---

## 2. 项目脚手架

```bash
pip install scrapy
scrapy startproject myproj            # 创建项目
cd myproj
scrapy genspider example example.com  # 生成爬虫
scrapy crawl example                  # 运行
scrapy crawl example -o out.jl        # 输出到 jsonl
scrapy crawl example -s JOBDIR=crawls/job1   # 暂停/恢复(增量)
```

项目目录:

```
myproj/
├── scrapy.cfg              # 部署配置
└── myproj/
    ├── items.py            # 数据结构定义
    ├── middlewares.py      # 下载/爬虫中间件
    ├── pipelines.py        # 数据处理 + 入库
    ├── settings.py         # 全局配置
    └── spiders/            # 业务爬虫
        └── example.py
```

---

## 3. scrapy 内部执行流程图(核心,必背)

```
                       ┌────────────┐
                       │  SPIDER    │ ← parse() / 自定义 callback
                       └──┬──────▲──┘
              ① yield Request │      ⑪ yield item / Request
                       ▼      │
                  ┌─────────────┐
                  │   ENGINE    │ ← 核心调度器,所有信号在此中转
                  └──┬──┬──▲──▲┘
                ②   │  │  │  │
                ┌───▼┐ │  │  ┌┴──────────┐
                │SCH │ │  │  │  PIPELINE │← ⑫ process_item → DB
                │EDU │ │  │  └───────────┘
                │LER │ │  │ ⑩ Response
                └─┬──┘ │  │
              ③   │    │④ Request
                  ▼    ▼  │
              ┌──────────────────┐
              │ DOWNLOADER MW    │ ← ⑤ process_request:换 UA/proxy/Cookie
              │  process_request │
              │  process_response│
              │  process_exception│
              └─────────┬────────┘
              ⑥ Request │  ▲ ⑨ Response
                        ▼  │
                   ┌──────────┐
                   │DOWNLOADER│  ⑦ HTTP(S) ⑧ Response
                   └─────┬────┘
                         │
                         ▼
                     目标网站
```

**12 步剧本**:① spider 产 Request → ② 经爬虫中间件 → ③ 入引擎 → ④ 调度器排队/去重 → ⑤ 出队回引擎 → ⑥ 经下载中间件(替 UA/代理/cookie) → ⑦ 下载器发请求 → ⑧ 拿到响应 → ⑨ 经下载中间件回程 → ⑩ 进引擎 → ⑪ spider 的 callback 解析 → ⑫ yield 出 item 进 pipeline 持久化。

---

## 4. Spider 写法

```python
import scrapy

class Top250Spider(scrapy.Spider):
    name = "top250"                                      # 唯一,启动用
    allowed_domains = ["douban.com", "doubanio.com"]     # 域名白名单
    start_urls = ["https://movie.douban.com/top250"]

    def parse(self, response, **kwargs):
        for li in response.xpath("//ol[@class='grid_view']/li"):
            yield {
                'title':  li.xpath(".//span[@class='title'][1]/text()").get(),
                'rating': li.xpath(".//span[@class='rating_num']/text()").get(),
            }
        # 翻页
        nxt = response.xpath("//span[@class='next']/a/@href").get()
        if nxt:
            yield scrapy.Request(response.urljoin(nxt), callback=self.parse)
```

要点:

- `response.xpath(...).get()` / `.getall()`:新版推荐,等价旧 `extract_first()` / `extract()`。
- `response.urljoin(rel)`:相对 URL 转绝对。
- 子请求传参用 `cb_kwargs={'k': v}`(`meta` 也行,但 `cb_kwargs` 更清晰)。

---

## 5. start_requests 与 dont_filter

```python
def start_requests(self):
    for page in range(1, 6):
        yield scrapy.Request(
            f'https://x.com/list?page={page}',
            callback=self.parse,
            dont_filter=False,          # 是否绕过 dupefilter
            cookies={'k': 'v'},
            meta={'proxy': '...'},
        )
```

`start_urls` 默认 **`dont_filter=True`**(不过滤),自己 `yield Request` 默认 **`dont_filter=False`**。增量场景请重写 `start_requests` 以便起始 URL 也参与去重。

---

## 6. POST 请求

```python
# 表单
yield scrapy.FormRequest(url=u, formdata={'k': 'v'})

# JSON 载荷
from scrapy.http import JsonRequest
yield JsonRequest(url=u, data={'k': 'v'})   # 自动 Content-Type: application/json

# 原始
yield scrapy.Request(u, method='POST', body=json.dumps(payload),
                     headers={'Content-Type': 'application/json'})
```

---

## 7. Item 与 ItemLoader

```python
# items.py
class JobItem(scrapy.Item):
    title = scrapy.Field()
    company = scrapy.Field()
```

`Item` ≈ 受约束的字典:未声明字段直接报错,适合大型项目做契约。小项目用 `dict` 即可。

---

## 8. Pipeline(管道)

```python
class MongoPipeline:
    def open_spider(self, spider):
        self.client = pymongo.MongoClient()
        self.col = self.client['db']['col']

    def process_item(self, item, spider):
        self.col.insert_one(dict(item))
        return item                      # 必须 return,否则下一管道收到 None

    def close_spider(self, spider):
        self.client.close()
```

settings.py:

```python
ITEM_PIPELINES = {
    'myproj.pipelines.CleanPipeline': 100,    # 数字越小越先执行
    'myproj.pipelines.MongoPipeline': 300,
}
```

跳过某条数据:`raise DropItem('reason')`。

---

## 9. 中间件(下载中间件最常用)

```python
class RandomUAMiddleware:
    UAS = [...]
    def process_request(self, request, spider):
        request.headers['User-Agent'] = random.choice(self.UAS)
        # return None → 继续往下一个中间件 / 下载器
        # return Response → 短路,不下载,直接交给 spider
        # return Request → 重新入调度器
        # raise IgnoreRequest → 丢弃

    def process_response(self, request, response, spider):
        if response.status != 200:
            return request           # 重发
        return response
```

settings.py:

```python
DOWNLOADER_MIDDLEWARES = {
    'myproj.middlewares.RandomUAMiddleware': 400,
    'myproj.middlewares.ProxyMiddleware': 410,
}
```

权重越小越先执行 `process_request`;`process_response` 反向执行。

**selenium 接入** = 自定义下载中间件,在 `process_request` 里用浏览器拿到 HTML 后返回 `HtmlResponse(url, body=html.encode(), request=request)`,后续不走真实下载器。

---

## 10. 关键 settings 速查

| 配置 | 作用 |
|---|---|
| `ROBOTSTXT_OBEY = False` | 不遵守 robots.txt(爬虫场景常用) |
| `CONCURRENT_REQUESTS = 16` | 并发请求总数 |
| `CONCURRENT_REQUESTS_PER_DOMAIN = 8` | 单域并发 |
| `DOWNLOAD_DELAY = 3` | 请求间隔(自动 × 0.5~1.5 随机) |
| `RANDOMIZE_DOWNLOAD_DELAY = True` | 默认 True |
| `DOWNLOAD_TIMEOUT = 30` | 下载超时 |
| `RETRY_ENABLED = True` / `RETRY_TIMES = 2` | 失败重试 |
| `LOG_LEVEL = 'WARNING'` | 日志级别 |
| `USER_AGENT = ...` | 全局 UA |
| `DEFAULT_REQUEST_HEADERS = {...}` | 全局请求头 |
| `COOKIES_ENABLED = True` | 跨请求自动维护 cookie |
| `HTTPCACHE_ENABLED = True` | 本地 HTTP 缓存(调试神器) |
| `AUTOTHROTTLE_ENABLED = True` | 自动限速(按响应延迟动态调) |

---

## 11. 去重 / 增量

**自带 dupefilter**:基于 URL 的 SHA1,默认存内存。配合 `-s JOBDIR=crawls/job1` 持久化到磁盘 = 暂停/恢复。

```bash
scrapy crawl spider -s JOBDIR=crawls/job1
# Ctrl+C 一次让其优雅退出;再次同命令恢复
```

更强去重:**布隆过滤器**或自定义 `DUPEFILTER_CLASS`;数据级去重在 pipeline 用 Redis SADD(参见第 5 章)。

⚠️ 原文 9.10 在 spider 的 `start_requests` 里手动用 Redis 给 URL 去重,**与框架自带 dupefilter 功能重复**;实际只需开 JOBDIR 即可,真要做分布式去重看第 10 章 scrapy-redis。

---

## 12. 原文需要纠正/补充的点

| # | 原文 | 问题 | 正确做法 |
|---|------|------|---------|
| 1 | 多次出现 `extract_first()` / `extract()` | 旧 API,scrapy 1.x 起推荐 `.get()` / `.getall()` | 用新 API,语义更清晰、空值返回 `None` 而非异常 |
| 2 | `if __name__ == '__main__': cmdline.execute('scrapy crawl xxx'.split())` 写在爬虫文件里 | 仅用于本机调试方便,**不要部署到 scrapyd 等环境** | 部署用 `scrapy crawl` 命令或 `CrawlerProcess` |
| 3 | "爬虫中间件和下载中间件作用重复"(9.5 注解) | 不准确,二者位置/作用对象不同 | 下载中间件作用于 **Request/Response**;爬虫中间件作用于 **spider 输入输出的 Request/Item**(比如统一处理 spider 抛出的异常、过滤 item) |
| 4 | 9.10 在 `start_requests` 里用 Redis 自建 URL 去重 | scrapy 已有 dupefilter,功能重复 | 单机增量用 `-s JOBDIR=...`,分布式用 scrapy-redis 自带 RFPDupeFilter |
| 5 | "用 `__del__` 关 Redis 连接" | `__del__` 不可靠 | 在 spider 的 `closed(self, reason)` 信号里关 |
| 6 | 9.8 selenium 中间件直接 `webdriver.Chrome()` 放 `__init__` | 多 spider 共享/scrapyd 部署会冲突 | 用 `from_crawler` 工厂方法 + spider 关闭信号关浏览器 |
| 7 | `scrapy genspider top250 https://movie.douban.com/top250?start=0&filter=` | `genspider` 第二参数应是 **域名**,不是完整 URL | `scrapy genspider top250 movie.douban.com`,生成后再改 `start_urls` |
| 8 | 9.7 `DEFAULT_REQUEST_HEADERS` 里写 UA | 与单独的 `USER_AGENT` 同时设置时,**`USER_AGENT` 优先**;别两边都改 | 二选一,通常只设 `USER_AGENT`,UA 池放下载中间件 |
| 9 | 翻页用 `start_requests` 一次性生成 1~5 页 | 静态翻页可行,但页数未知时不能用 | 推荐"边解析边 yield 下一页"或 `CrawlSpider` 规则 |
| 10 | 9.11 "Ctrl+C 不能按两次" | 准确说:两次会强杀,丢失暂停状态 | 一次让 scrapy 写完 JOBDIR 再退出,耐心等 |
| 11 | `pipelines.py` 中 `os.getcwd()` 拼路径 | scrapyd 运行时 cwd 可能不是项目目录 | 用 `scrapy.utils.project.data_path()` 或绝对路径 |
| 12 | `LOG_LEVEL = 'WARNING'` 仅用于"少看日志" | 调试时该开 `DEBUG` | 视场景 |
| 13 | 9.13 `scrapy.FormRequest` 与 `JsonRequest` 没有强调差异 | 容易混 | form → `application/x-www-form-urlencoded`;`JsonRequest` → `application/json` |
| 14 | 蜻蜓 FM 把图片二进制塞进 item 经 pipeline 落盘 | 阻塞 pipeline + 内存占用高 | 用 scrapy 内置 **`ImagesPipeline` / `FilesPipeline`**,自动异步下载、去重、缩略图 |

---

## 13. 速记

- 流程图 12 步要背:Spider → Engine → Scheduler → DownloaderMW → Downloader → 回程 → Pipeline。
- 子请求传参用 `cb_kwargs`,提取数据用 `.get()/.getall()`。
- 起始 URL 默认不过滤,自己 yield 的 Request 默认过滤;增量看 `JOBDIR`,分布式看 scrapy-redis。
- 替 UA / 代理 / 注入 cookie:**下载中间件**统一干;权重越小越先。
- 多管道 + `return item` 串行;丢弃用 `DropItem`;开启数据库连接放 `open_spider`。
- 文件/图片下载首选自带 `ImagesPipeline` / `FilesPipeline`,不要把 body 塞进普通 item。
