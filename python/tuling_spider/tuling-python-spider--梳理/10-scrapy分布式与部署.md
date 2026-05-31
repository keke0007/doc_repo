# 第 10 章 scrapy 分布式与项目部署 — 知识点梳理

> 对应 10.1 ~ 10.5。scrapy-redis 分布式 + scrapyd/scrapydweb/gerapy 部署,多节点协作必附流程图。

---

## 1. 增量 vs 分布式 vs 部署

| 概念 | 解决什么 | 关键技术 |
|---|---|---|
| 增量爬虫 | 同一台机断点续爬 | `JOBDIR` / scrapy-redis(SCHEDULER_PERSIST) |
| 分布式爬虫 | 多机协同抓同一个任务 | scrapy-redis(共享队列+共享指纹集) |
| 项目部署 | 把爬虫放到服务器上跑 | scrapyd + scrapyd-client + scrapydweb / gerapy |

三件事独立,组合使用最常见:**scrapy-redis 做分布式 + scrapyd 做部署 + gerapy/scrapydweb 做可视化**。

---

## 2. scrapy-redis 安装/兼容性

```bash
# 版本敏感:scrapy-redis 老版本与 scrapy 2.11+ 不兼容
pip install "scrapy==2.6.3" scrapy-redis
# 或使用新版 scrapy-redis 0.8.x + scrapy 2.10+
```

⚠️ 原文锁死 `scrapy==2.6.3`,确实是当时社区主流配置;但 scrapy-redis 已发布 0.8.0 适配较新 scrapy,新项目可直接装最新版。版本不匹配会出 `request_fingerprinter` 相关报错。

---

## 3. scrapy-redis 工作流程图

```
                ┌──────────────────────────────────────────────┐
                │              Redis(任务总线)                  │
                │  - {spider}:requests   ZSET / List(待抓)      │
                │  - {spider}:dupefilter SET (URL 指纹)         │
                │  - {spider}:items      List(可选,存爬到的数据)│
                │  - {spider}:start_urls List(种子,RedisSpider用)│
                └─▲────────────────────────────────▲───────────┘
                  │ 拿任务/写结果                  │ 同上
        ┌─────────┴─────────┐         ┌──────────┴────────┐
        │   Slaver-A (爬虫) │   ...   │  Slaver-N (爬虫)  │
        │ scrapy + scrapy_  │         │                   │
        │ redis Scheduler   │         │                   │
        └───────────────────┘         └───────────────────┘
                │                                │
                ▼                                ▼
              目标站                          目标站
```

替换点:

- 原生 `Scheduler` → `scrapy_redis.scheduler.Scheduler`(队列入 Redis)
- 原生 `RFPDupeFilter` → `scrapy_redis.dupefilter.RFPDupeFilter`(指纹入 Redis SET)
- 可选 `scrapy_redis.pipelines.RedisPipeline`:把 item 也丢进 Redis,由消费者再分发到 MongoDB / 数据仓库。

---

## 4. settings.py 关键配置

```python
SCHEDULER         = "scrapy_redis.scheduler.Scheduler"
DUPEFILTER_CLASS  = "scrapy_redis.dupefilter.RFPDupeFilter"
SCHEDULER_PERSIST = True                          # 退出时不清空队列/指纹
SCHEDULER_QUEUE_CLASS = 'scrapy_redis.queue.PriorityQueue'  # 默认,有序集合
# PriorityQueue(ZSET,按优先级) / FifoQueue(List 队列) / LifoQueue(List 栈)

REDIS_URL = "redis://:pwd@127.0.0.1:6379/0"
# 或拆开:REDIS_HOST/REDIS_PORT/REDIS_PARAMS

# scrapy 2.7+ 必须显式声明 fingerprinter 版本
REQUEST_FINGERPRINTER_IMPLEMENTATION = "2.7"
TWISTED_REACTOR = "twisted.internet.asyncioreactor.AsyncioSelectorReactor"

# 布隆过滤器替代(可选)
# pip install scrapy-redis-bloomfilter
# DUPEFILTER_CLASS = 'scrapy_redis_bloomfilter.dupefilter.RFPDupeFilter'

ITEM_PIPELINES = {
    'myproj.pipelines.MongoPipeline': 300,
    # 'scrapy_redis.pipelines.RedisPipeline': 301,  # 可选,共享 item 列表
}
```

---

## 5. 两种 Spider 基类

```python
# A. 普通 Spider + scrapy-redis 配置 → 单节点增量
class MySpider(scrapy.Spider):
    start_urls = ['...']

# B. RedisSpider → 多节点分布式;种子从 Redis 取
from scrapy_redis.spiders import RedisSpider
class MySpider(RedisSpider):
    name = 'top250'
    redis_key = 'top250:start_urls'      # 种子 key,所有节点共享
```

启动节点之后,**外部往 Redis 推种子**就开跑:

```bash
redis-cli> LPUSH top250:start_urls "https://movie.douban.com/top250"
```

类似还有 `RedisCrawlSpider`(规则爬虫的分布式版)。

---

## 6. 主从架构(Master / Slaver)

```
Master(只跑 Redis,不爬)        Slaver*N(跑 scrapy 节点)
  ├── 任务队列                   ├── 从队列取 Request
  ├── 指纹集合                   ├── 下载/解析
  └── (可选)结果列表             └── 把新 Request 回灌 / item 入库
```

注意:

- **种子可推到 Master 任一台** = 立刻分发。
- Item 持久化既可走"每 Slaver 直连 Mongo",也可走 RedisPipeline + 单独消费者。前者并发高,后者解耦清晰。

---

## 7. scrapyd 部署

```
本地开发机                                 服务器
┌────────────────┐  scrapyd-deploy        ┌──────────────────────┐
│ scrapy 项目    │ ─── HTTP egg 上传 ───▶ │  scrapyd  daemon      │
│ scrapy.cfg     │                        │  :6800                │
└────────────────┘                        │  /schedule.json (跑)  │
        │                                 │  /cancel.json   (停)  │
        │                                 │  /listjobs.json (查)  │
        │  curl                            └──────────┬───────────┘
        └─────────────────────────────────────────────┘
                                                     │ 启动子进程
                                                     ▼
                                              scrapy crawl xxx
```

服务端:

```bash
pip install scrapyd
# 让外网访问:在工作目录新建 scrapyd.conf
# [scrapyd]
# bind_address = 0.0.0.0
# http_port    = 6800
scrapyd
```

客户端:

```bash
pip install "scrapyd-client==1.2.3"
# 修改项目的 scrapy.cfg:
# [deploy:ubuntu-1]
# url = http://192.168.70.82:6800/
# project = myproj
scrapyd-deploy -l                              # 列出可用 target
scrapyd-deploy ubuntu-1 -p myproj              # 打 egg 上传

# 启动/停止/查询
curl http://ip:6800/schedule.json -d project=myproj -d spider=top250
curl http://ip:6800/cancel.json   -d project=myproj -d job=<jobid>
curl http://ip:6800/listjobs.json?project=myproj
```

scrapy.cfg 同一段可写多 `[deploy:xxx]`,实现一键多机部署。

---

## 8. scrapydweb(scrapyd 的 Web 管理界面)

```bash
pip install scrapydweb       # 后端 flask
scrapyd                      # 必须先开 scrapyd
scrapydweb                   # 首次会失败生成配置,改完再启
# 默认 http://localhost:5000
```

优点:可视化部署、运行、日志、定时任务、邮件告警;支持挂多个 scrapyd 节点(节点列表 + 批量发布)。

---

## 9. gerapy(国产可视化,中文)

```bash
conda create -n spider_deploy python=3.9
pip install "gerapy==0.9.12"
gerapy init && cd gerapy
gerapy migrate
gerapy createsuperuser
gerapy runserver 0.0.0.0:8000
```

特点:

- Web UI 配置 scrapyd 主机、批量部署、批量启动。
- 支持基于模板生成爬虫(可视化建项目)。
- 内嵌定时任务(基于 APScheduler)。

scrapydweb vs gerapy:scrapydweb 偏运维监控,gerapy 偏项目管理+定时调度。新项目两个都试再选。

---

## 10. 原文需要纠正/补充的点

| # | 原文 | 问题 | 正确做法 |
|---|------|------|---------|
| 1 | "Redis 默认 16 库,`/1` 是序号为 2 的库" | 错,**`/1` 就是 1 号库**,0 号库是默认 | `redis://host:port/<db_index>`,db_index 即库号 |
| 2 | scrapy-redis 锁 scrapy 2.6.3,注释"2.11.0 有兼容问题" | 部分版本兼容性是真的,但 scrapy-redis 0.8.x 已支持新 scrapy | 新项目直接装最新双方,出错再降级 |
| 3 | "`SCHEDULER_FLUSH_ON_START`:重爬" | 完整含义是"启动时清空 Redis 队列+指纹",分布式下慎用 | 仅在调试或重新跑全量时开;生产保持 `False` |
| 4 | 10.2 把图片二进制 `base64` 进 item 再走 RedisPipeline | 路径绕、Redis 内存占用高 | 用 `FilesPipeline` 异步落盘,或单独消费者拉 URL 下载 |
| 5 | 10.3 客户端机生成的 `setup.py` "可以删除" | 该文件是 scrapyd-deploy 用来打包 egg 的,删了下次发布会报错 | 保留;`.gitignore` 也别忽略 |
| 6 | 10.3 `curl ... -d job=<id>` 停止后又说"再次发命令断点续爬" | scrapyd 本身不做断点续爬,**断点续爬靠 scrapy-redis 共享 Redis 状态** | 两者结合:scrapy-redis 提供续爬能力,scrapyd 负责调度 |
| 7 | "scrapyd-client 1.2.3"锁版本 | 较新版本有更稳的 egg 打包流程 | 没必要锁;装不上再降级 |
| 8 | scrapyd.conf "在工作目录新建" 但没说优先级 | 实际查找顺序:`/etc/scrapyd/scrapyd.conf` → `~/.scrapyd.conf` → `./scrapyd.conf` 等 | 多机部署用 `/etc/scrapyd/scrapyd.conf` 更稳妥 |
| 9 | `bind_address = 0.0.0.0` 默认无鉴权 | 暴露公网会被滥用上传任意 egg 跑代码 | 至少加防火墙;或挂 Nginx 做 Basic Auth;或绑内网 IP |
| 10 | scrapydweb 配置 "报错重启即可" | 实际是首次缺 `scrapydweb_settings_v10.py`,跑一次生成后再编辑 | 编辑 `SCRAPYD_SERVERS = [...]` 写入要管理的节点 |
| 11 | 主从架构图中只有一个 Master | Master(Redis) 是单点风险 | 生产用 Redis 哨兵/集群;或冷备 |
| 12 | "Redis 效率非常高,所以用它而不用 MySQL/MongoDB" | 真正原因是**原子操作 + ZSET 优先级队列 + 内存延迟极低** | 在写技术文档时给出真实原因更专业 |

---

## 11. 速记

- 增量 = JOBDIR;分布式 = scrapy-redis 共享 Redis;部署 = scrapyd + 可视化层。
- 三件事独立可组合:**scrapy-redis + scrapyd + (scrapydweb / gerapy)** 是常见的全家桶。
- `redis://host:port/N` 中 `N` 就是库号,不要再"+1"。
- 把"等待抓"的种子推进 Redis `LPUSH {spider}:start_urls "url"`,等 Slaver 起来自己消费。
- scrapyd 端口 6800,scrapydweb 5000,gerapy 8000——三件套端口要记牢。
- 公网部署 scrapyd 一定要加鉴权/防火墙。
