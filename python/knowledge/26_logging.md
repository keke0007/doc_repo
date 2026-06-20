# python26 logging · 知识点梳理

> 原文档:`python/Article/PythonBasis/python26/1.md`
> 整理对象:logging 模块、层级、Handler、Formatter、`dictConfig`、结构化日志

---

## 一、日志五级别

```python
import logging

logging.debug("调试信息")
logging.info("常规信息")
logging.warning("警告信息")        # 默认级别
logging.error("错误信息")
logging.critical("严重错误")
```

| 级别 | 数值 | 使用场景 |
|------|------|---------|
| DEBUG | 10 | 开发调试详情 |
| INFO | 20 | 关键业务流程节点 |
| WARNING | 30 | 预期外的但可处理的情况(默认) |
| ERROR | 40 | 需要人工介入的错误 |
| CRITICAL | 50 | 系统级致命故障 |

## 二、`basicConfig` 快速配置

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
    handlers=[
        logging.FileHandler("app.log", encoding="utf-8"),
        logging.StreamHandler(),
    ],
)
logger = logging.getLogger(__name__)
```

> ⚠️ **关键**:`basicConfig` 只在第一次调用时生效，后续调用被忽略。应在程序入口点调用一次。

## 三、Logger 层级与传播

```
root logger (级别 WARNING)
│
├── "app" (级别 INFO, propagate=True)
│   ├── logger = getLogger("app")         ← 继承 root
│   ├── "app.api"                         ← 继承 "app" 的级别
│   └── "app.db"                          ← 继承 "app" 的级别
│
└── "third_party" (级别 WARNING)
```

- `getLogger(__name__)` 获取与模块名同名的 logger(如 `my_app.api.handlers`)。
- 名字用 `.` 分隔形成层级:`my_app.api.handlers` 是 `my_app.api` 的子 logger。
- `propagate=True`(默认):日志向上传播给父 logger，可能造成重复输出。
- **推荐每个模块**:`logger = logging.getLogger(__name__)`

## 四、Handler — 日志输出目标

```python
# 流处理器
handler = logging.StreamHandler()                  # stderr

# 文件处理器
handler = logging.FileHandler("app.log", encoding="utf-8")

# 按大小轮转
from logging.handlers import RotatingFileHandler
handler = RotatingFileHandler(
    "app.log", maxBytes=10*1024*1024, backupCount=5
)

# 按时间轮转
from logging.handlers import TimedRotatingFileHandler
handler = TimedRotatingFileHandler(
    "app.log", when="midnight", backupCount=30
)
```

## 五、Formatter — 控制输出格式

```python
formatter = logging.Formatter(
    fmt="%(asctime)s [%(levelname)-8s] %(name)s:%(lineno)d - %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)
handler.setFormatter(formatter)
```

常用占位符:
| 占位符 | 含义 |
|--------|------|
| `%(asctime)s` | 时间 |
| `%(levelname)s` | 级别名称 |
| `%(name)s` | logger 名称 |
| `%(module)s` | 模块名 |
| `%(funcName)s` | 函数名 |
| `%(lineno)d` | 行号 |
| `%(message)s` | 日志消息 |
| `%(pathname)s` | 源文件完整路径 |
| `%(process)d` | 进程 ID |
| `%(thread)d` | 线程 ID |

## 六、YAML 配置(`dictConfig`)

```yaml
# logging_config.yaml
version: 1
disable_existing_loggers: false

formatters:
  default:
    format: "%(asctime)s [%(levelname)s] %(name)s: %(message)s"
  detailed:
    format: "%(asctime)s [%(levelname)s] %(name)s:%(lineno)d - %(message)s"

handlers:
  console:
    class: logging.StreamHandler
    formatter: default
    level: INFO
  file:
    class: logging.handlers.RotatingFileHandler
    formatter: detailed
    filename: app.log
    maxBytes: 10485760
    backupCount: 5

loggers:
  my_app:
    level: DEBUG
    handlers: [console, file]
    propagate: false
  sqlalchemy.engine:
    level: WARNING
    handlers: [console]
```

```python
import yaml
import logging.config

with open("logging_config.yaml") as f:
    config = yaml.safe_load(f)
logging.config.dictConfig(config)
```

## 七、`logger.exception()` vs `logger.error()`

```python
try:
    1 / 0
except Exception:
    logger.exception("计算失败")   # ✅ 自动附带 traceback
    # vs
    logger.error("计算失败")       # ❌ 没有 traceback

    # 或者手动:
    logger.error("计算失败", exc_info=True)
```

## 八、结构化日志

```python
# python-json-logger: 输出 JSON 格式，方便 ELK/Loki 收集
import logging
from python_json_logger import jsonlogger

handler = logging.StreamHandler()
handler.setFormatter(jsonlogger.JsonFormatter())
logging.basicConfig(handlers=[handler], level=logging.INFO)

# 结构化消息(推荐 structlog)
import structlog

logger = structlog.get_logger()
logger.info("order_created", order_id="ORD-123", amount=99.9, user_id=42)
# 输出: {"event": "order_created", "order_id": "ORD-123", "amount": 99.9, "user_id": 42, "timestamp": "..."}
```

## 九、Logger 层级与 Handler 管道(ASCII 图)

```
日志记录 → 处理流程:

    logger.debug("message")
        │
        ▼
    ┌─────────────────────┐
    │ 1. 级别过滤          │
    │   logger.level       │
    │   消息级别 >= 设置级别? │
    │   DEBUG < INFO → 丢弃 │
    └────────┬────────────┘
             │ 通过
             ▼
    ┌─────────────────────┐
    │ 2. 创建 LogRecord    │
    │   包含: msg, level,  │
    │   name, lineno,      │
    │   pathname, time...  │
    └────────┬────────────┘
             │
             ├───────────────────────────────┐
             ▼                               ▼
    ┌──────────────────┐            ┌──────────────────┐
    │ 3a. Handler 处理  │            │ 3b. 传播到父级   │
    │  (当前 logger)    │            │ (如果 propagate) │
    │                  │            │                  │
    │ Handler.level     │            │ 父 logger        │
    │  → 过滤           │            │  → 步骤 3a 重复  │
    │  → Formatter      │            │                  │
    │  → emit() 输出    │            │ root logger      │
    │    (文件/控制台/   │            │  → 最后 stop     │
    │     socket/...)   │            │                  │
    └──────────────────┘            └──────────────────┘


典型部署结构(不同环境不同 Handler):

    ┌───────────────┐
    │  my_app       │  (DEBUG, propagate=True)
    │  ├── api      │
    │  ├── services │
    │  └── db       │
    └───────┬───────┘
            │
    ┌───────▼───────────────────────────────────┐
    │  root logger 配置                          │
    │                                            │
    │  开发环境:                                  │
    │    StreamHandler(sys.stderr, level=DEBUG)  │
    │                                            │
    │  生产环境:                                  │
    │    RotatingFileHandler(app.log)            │
    │    + jsonlogger.JsonFormatter              │
    │    + SysLogHandler(远程日志服务器)          │
    └────────────────────────────────────────────┘
```

---

## 十、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 未强调 `basicConfig` 只生效一次 | 未注意此特性可能导致配置"失效"却不报错 |
| 2 | 1.md | 缺少结构化日志(structlog/python-json-logger) | 生产环境日志收集(ELK/Loki/CloudWatch)的标准做法 |
| 3 | 1.md | 未提 `exc_info=True` 替代方案 | `logger.exception()` 更简洁 |
| 4 | 1.md | YAML 配置示例不完整 | 补充了 handler/lgger/formatter 三要素完整示例 |
| 5 | 1.md | 未明确 `propagate=False` 的使用场景 | 防止日志重复输出的关键设置 |
