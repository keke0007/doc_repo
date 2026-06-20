# python28 学完之后做什么 · 知识点梳理

> 原文档:`python/Article/PythonBasis/python28/1.md`
> 整理对象:Python 职业方向、技术栈路线、推荐项目、学习资源

---

## 一、三大方向总览

```
                Python 基础
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   ┌─────────┐ ┌─────────┐ ┌──────────────┐
   │ 数据/AI  │ │ Web 后端 │ │ 自动化/脚本   │
   │ ML      │ │         │ │ SRE/DevOps   │
   └─────────┘ └─────────┘ └──────────────┘
```

## 二、方向一:数据科学 / AI / 机器学习

### 核心库

| 库 | 用途 |
|----|------|
| **NumPy** | 多维数组、数值计算基石 |
| **Pandas** | 表格数据处理、ETL |
| **Polars** | 高性能 DataFrame(新兴，Rust 后端) |
| **Matplotlib** / **Seaborn** | 可视化 |
| **Plotly** | 交互式可视化 |
| **scikit-learn** | 传统机器学习 |
| **PyTorch** | 深度学习(研究首选) |
| **Transformers**(HuggingFace) | 预训练模型、LLM 应用 |
| **LangChain** / **LlamaIndex** | LLM 应用框架 |

### 3 个入门项目

1. **CSV 数据分析仪表盘**:Pandas + Streamlit 读取 CSV 生成交互图表。
2. **图片分类器**:PyTorch + torchvision 训练 CIFAR-10 分类。
3. **LLM 聊天机器人**:OpenAI/Anthropic SDK + FastAPI + Streamlit。

## 三、方向二:Web 后端

### 核心库

| 库 | 用途 |
|----|------|
| **FastAPI** | 现代异步 Web 框架(Python 首选) |
| **Django** | 全栈框架(ORM、Admin、Auth 内建) |
| **LiteStar** | 新兴异步框架，FastAPI 竞争者 |
| **Pydantic** | 数据校验(与 FastAPI 深度集成) |
| **SQLAlchemy** / **SQLModel** | ORM |
| **Alembic** | 数据库迁移 |
| **Celery** / **ARQ** | 异步任务队列 |
| **Redis** | 缓存 / 消息队列 |
| **PostgreSQL** / **SQLite** | 关系型数据库 |

### 3 个入门项目

1. **RESTful API 服务**:FastAPI + SQLModel + PostgreSQL 的 CRUD 应用。
2. **用户认证系统**:JWT + OAuth2，含注册/登录/权限角色。
3. **实时消息推送**:WebSocket + Redis pub/sub 或 SSE(Server-Sent Events)。

## 四、方向三:自动化 / 脚本 / DevOps

### 核心库

| 库 | 用途 |
|----|------|
| **typer** | CLI 工具 |
| **rich** | 终端美化(彩色输出、表格、进度条) |
| **textual** | 终端 TUI 应用 |
| **requests** / **httpx** | HTTP 客户端 |
| **beautifulsoup4** | HTML 解析 |
| **schedule** | 定时任务 |
| **APScheduler** | 高级任务调度 |
| **watchdog** | 文件系统监控 |
| **fabric** / **invoke** | 远程执行/任务自动化 |

### 3 个入门项目

1. **TODO CLI 工具**:typer + rich，支持添加/删除/标记完成，数据存 JSON/SQLite。
2. **网页爬虫**:httpx + beautifulsoup4 定时抓取目标网页并提取关键信息。
3. **文件整理脚本**:watchdog 监控目录变化，自动按扩展名分类移动文件。

## 五、跨方向通用技能

| 技能 | 重要性 | 说明 |
|------|--------|------|
| Git | 必须 | 版本控制，GitHub/GitLab 协作 |
| pytest | 必须 | 无论哪个方向都要写测试 |
| Docker | 强烈推荐 | 环境标准化，生产部署 |
| SQL | 强烈推荐 | 所有方向都可能和数据库打交道 |
| CI/CD | 推荐 | GitHub Actions 自动测试/部署 |
| Linux 基础 | 推荐 | 服务器环境大多 Linux |
| ruff / mypy | 推荐 | 代码质量和类型检查 |
| 英文阅读 | 必须 | 官方文档、StackOverflow、论文 |

## 六、学习资源

| 类型 | 资源 |
|------|------|
| 官方文档 | docs.python.org(Python), fastapi.tiangolo.com, doc.rust-lang.org |
| 练习平台 | LeetCode, Exercism, Advent of Code |
| 书籍 | 《Fluent Python》2nd edition(深入理解),《Robust Python》(工程化) |
| 社区 | PyCon 视频、Real Python、Python Weekly 邮件列表 |
| 源码 | 阅读好的开源项目代码(如 httpx, rich, textual 的源码) |

## 七、方向选择建议

```
你的兴趣在哪?
│
├── 喜欢数学、统计、模型 → 数据/AI 方向
│   入场: Pandas + scikit-learn → PyTorch → LLM
│
├── 喜欢搭系统、API、架构  → Web 后端方向
│   入场: FastAPI + Pydantic → 数据库 → 部署
│
└── 喜欢效率工具、脚本     → 自动化方向
    入场: typer + rich → requests → 系统管理
```

**三种方向不是互斥的**:Web 后端工程师也需要写脚本做部署自动化；数据科学家也需要 FastAPI 把模型部署成 API。打好基础后可以兼修。

---

## 八、与原文档差异速查

| 编号 | 出处 | 原文描述 | 正确说明 |
|------|------|----------|----------|
| 1 | 1.md | 技术栈较旧(未提 LLM 生态) | 补充了 Transformers、LangChain、LlamaIndex 等 2023+ 核心工具 |
| 2 | 1.md | Web 后端缺 Litestar | 补充了作为 FastAPI 竞争者的新兴框架 |
| 3 | 1.md | 数据处理缺 Polars | Polars 已成为 Pandas 的重要竞争者 |
| 4 | 1.md | 学习资源较窄 | 补充了 Exercism、Advent of Code、Real Python 等渠道 |
| 5 | 1.md | 三方向非互斥性未强调 | 实际工作中三种方向技能常交叉使用 |
