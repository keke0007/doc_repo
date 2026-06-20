# Pydantic AI 完整指南

## 一、什么是 Pydantic AI？

Pydantic AI 是由 Pydantic 团队开发的 Python Agent 框架，专门用于构建生产级的生成式 AI 应用程序。该框架的设计理念是将 FastAPI 的优雅体验带到 GenAI 应用和 Agent 开发中。

Pydantic AI 利用了 Pydantic 的验证层，这个验证层已经被 OpenAI、Anthropic、Google 等主要 AI SDK 所采用，因此它是"直接来源"而非"衍生实现"。

### 核心特点

1. **模型无关性（Model-agnostic）**
   - 支持 OpenAI、Anthropic、Gemini、DeepSeek、Cohere、Mistral、Ollama 等 20+ 个提供商
   - 支持 Azure AI Foundry、Amazon Bedrock 等平台
   - 可以实现自定义模型

2. **类型安全（Type Safety）**
   - 利用 Python 类型提示和 Pydantic 验证
   - 在编写时而非运行时捕获错误
   - 完整的 IDE 自动补全和静态类型检查支持

3. **可观测性（Observability）**
   - 与 Pydantic Logfire 深度集成
   - 基于 OpenTelemetry 的实时调试、性能监控和成本跟踪

4. **结构化输出（Structured Outputs）**
   - 使用 Pydantic 模型定义和验证 Agent 响应
   - 通过 JSON Schema 验证确保数据正确性

5. **可扩展架构（Extensible Architecture）**
   - 可组合的能力（Capabilities）：Web 搜索、思考模式、MCP
   - 工具、钩子和指令可打包成可重用单元

6. **高级功能**
   - 人机协作（Human-in-the-loop）：工具调用可要求用户批准
   - 持久化执行（Durable Execution）：跨故障保持进度
   - 流式输出：实时验证的结构化输出流
   - 图支持：基于类型提示的复杂工作流定义

7. **标准支持**
   - Model Context Protocol (MCP)
   - Agent-to-Agent 通信
   - UI 事件流

8. **系统化评估框架**
   - 与 Logfire 集成的测试和性能评估工具

## 二、安装与快速开始

### 安装

```bash
pip install pydantic-ai
```

### API 密钥配置

Pydantic AI 使用各个提供商的官方 SDK，API 密钥通过环境变量配置：

```bash
# OpenAI
export OPENAI_API_KEY='your-api-key'

# Anthropic
export ANTHROPIC_API_KEY='your-api-key'

# Google Gemini
export GEMINI_API_KEY='your-api-key'
```

### 最简单的示例

```python
from pydantic_ai import Agent

# 创建一个简单的 Agent（使用字符串模型名称）
agent = Agent(
    'anthropic:claude-sonnet-4-6',
    instructions='Be concise, reply with one sentence.'
)

# 同步运行
result = agent.run_sync('Where does "hello world" come from?')
print(result.output)
```

**说明：** 当使用字符串格式（如 `'anthropic:claude-sonnet-4-6'`）创建 Agent 时，Pydantic AI 会：
1. 自动从环境变量读取 API 密钥
2. 使用提供商的默认 API 端点
3. 无需手动配置 URL

### 自定义配置示例

如果需要自定义 API 端点、密钥或其他配置，可以使用 Provider 对象：

```python
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIProvider

# 方式 1: 使用自定义 API 密钥
provider = OpenAIProvider(api_key='your-custom-key')
agent = Agent('gpt-4', provider=provider)

# 方式 2: 使用自定义客户端（可配置 base_url）
from openai import AsyncOpenAI

custom_client = AsyncOpenAI(
    api_key='your-api-key',
    base_url='https://your-custom-endpoint.com/v1'
)
provider = OpenAIProvider(openai_client=custom_client)
agent = Agent('gpt-4', provider=provider)

# 方式 3: Anthropic 自定义配置
from pydantic_ai.models.anthropic import AnthropicProvider
from anthropic import AsyncAnthropicClient

custom_anthropic = AsyncAnthropicClient(
    api_key='your-api-key',
    base_url='https://your-proxy.com'
)
provider = AnthropicProvider(anthropic_client=custom_anthropic)
agent = Agent('claude-opus-4-7', provider=provider)
```

### 支持的模型格式

```python
# 字符串格式（推荐用于标准配置）
agent = Agent('openai:gpt-4')
agent = Agent('anthropic:claude-sonnet-4-6')
agent = Agent('gemini:gemini-1.5-pro')

# Provider 对象格式（用于自定义配置）
from pydantic_ai.models.openai import OpenAIProvider, OpenAIChatModel

provider = OpenAIProvider(api_key='key')
model = OpenAIChatModel('gpt-4', provider=provider)
agent = Agent(model)
```

## 三、核心概念与架构

### 1. Agent（代理）

Agent 是泛型容器，通过依赖类型和输出类型参数化：`Agent[DependencyType, OutputType]`。它们协调 LLM 交互、工具调用和指令执行。

```python
from pydantic_ai import Agent
from pydantic import BaseModel

class OutputSchema(BaseModel):
    answer: str
    confidence: float

agent = Agent(
    'openai:gpt-4',
    output_type=OutputSchema,
    instructions='Provide answers with confidence scores.'
)
```

### 2. 依赖注入（Dependency Injection）

`RunContext` 将依赖项传递到工具函数和动态指令中，实现可测试、类型安全的自定义，无需全局状态。

```python
from dataclasses import dataclass
from pydantic_ai import Agent, RunContext

@dataclass
class AppDependencies:
    database: DatabaseConnection
    api_key: str
    user_id: int

agent = Agent(
    'anthropic:claude-sonnet-4-6',
    deps_type=AppDependencies
)
```

### 3. 工具（Tools）

使用 `@agent.tool` 装饰器将函数注册为 LLM 可调用的工具。文档字符串自动填充工具描述，Pydantic 验证参数并处理错误。

```python
@agent.tool
async def get_user_balance(
    ctx: RunContext[AppDependencies],
    include_pending: bool = False
) -> float:
    """获取用户的账户余额。
    
    Args:
        include_pending: 是否包含待处理交易
    """
    return await ctx.deps.database.get_balance(
        user_id=ctx.deps.user_id,
        include_pending=include_pending
    )
```

### 4. 指令（Instructions）

- **静态指令**：配置 Agent 行为的固定指令
- **动态指令**：通过 `@agent.instructions` 访问依赖项并根据上下文调整

```python
@agent.instructions
def dynamic_instructions(ctx: RunContext[AppDependencies]) -> str:
    return f"You are assisting user {ctx.deps.user_id}. Be helpful and professional."
```

## 四、完整应用示例

### 银行客服 Agent

```python
from dataclasses import dataclass
from pydantic import BaseModel, Field
from pydantic_ai import Agent, RunContext

# 定义依赖
@dataclass
class SupportDependencies:
    customer_id: int
    db: DatabaseConn

# 定义结构化输出
class SupportOutput(BaseModel):
    support_advice: str = Field(description="给客户的建议")
    block_card: bool = Field(description="是否需要冻结银行卡")
    risk: int = Field(ge=0, le=10, description="风险评分 0-10")

# 创建 Agent
support_agent = Agent(
    'openai:gpt-4',
    deps_type=SupportDependencies,
    output_type=SupportOutput,
    instructions='''你是我们银行的客服代理。
    帮助客户解决账户问题，评估风险，必要时建议冻结银行卡。
    始终保持专业和同理心。'''
)

# 注册工具
@support_agent.tool
async def customer_balance(
    ctx: RunContext[SupportDependencies],
    include_pending: bool = False
) -> float:
    """返回客户当前账户余额。
    
    Args:
        include_pending: 是否包含待处理交易
    """
    return await ctx.deps.db.customer_balance(
        id=ctx.deps.customer_id,
        include_pending=include_pending
    )

@support_agent.tool
async def recent_transactions(
    ctx: RunContext[SupportDependencies],
    limit: int = 10
) -> list[dict]:
    """获取最近的交易记录。
    
    Args:
        limit: 返回的交易数量
    """
    return await ctx.deps.db.get_transactions(
        customer_id=ctx.deps.customer_id,
        limit=limit
    )

# 使用 Agent
async def handle_support_request(customer_id: int, query: str):
    deps = SupportDependencies(
        customer_id=customer_id,
        db=get_database_connection()
    )
    
    result = await support_agent.run(query, deps=deps)
    
    print(f"建议: {result.output.support_advice}")
    print(f"冻结卡片: {result.output.block_card}")
    print(f"风险评分: {result.output.risk}")
    
    return result.output
```

### 天气查询 Agent

```python
from pydantic_ai import Agent
import httpx

# 创建简单的天气 Agent
weather_agent = Agent(
    'anthropic:claude-sonnet-4-6',
    instructions='提供准确的天气信息，使用摄氏度。'
)

@weather_agent.tool
async def get_weather(city: str) -> dict:
    """获取指定城市的天气信息。
    
    Args:
        city: 城市名称
    """
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f'https://api.weatherapi.com/v1/current.json',
            params={'key': 'YOUR_API_KEY', 'q': city}
        )
        return response.json()

# 使用
result = await weather_agent.run('北京今天天气怎么样？')
print(result.output)
```

## 五、应用场景

### 1. 客户支持系统
- 自动化客户查询响应
- 访问客户数据和历史记录
- 风险评估和决策支持
- 多轮对话管理

### 2. 数据分析助手
- SQL 查询生成
- 数据可视化建议
- 报告生成
- 异常检测和解释

### 3. 内容生成
- 结构化内容创建（博客、文档）
- 代码生成和审查
- 翻译和本地化
- 摘要和提取

### 4. 工作流自动化
- 多步骤任务编排
- 跨系统集成
- 审批流程（人机协作）
- 长时间运行的任务（持久化执行）

### 5. RAG（检索增强生成）应用
- 文档问答系统
- 知识库查询
- 上下文感知响应
- 引用和来源追踪

### 6. 企业应用集成
- CRM 系统集成
- ERP 数据访问
- API 编排
- 业务流程自动化

## 六、Pydantic AI 与 LangChain 的差异

### 设计理念

**Pydantic AI:**
- 类型安全优先，利用 Python 类型系统
- "FastAPI 式"的开发体验
- 由 Pydantic 团队构建，是"源头"而非"衍生"
- 简洁、直接的 API 设计

**LangChain:**
- 链式组合优先，强调模块化
- 丰富的预构建组件和集成
- 社区驱动，生态系统庞大
- 更多抽象层和概念

### 类型安全

**Pydantic AI:**
- 原生 Pydantic 集成，完整的类型检查
- IDE 自动补全支持优秀
- 编译时错误检测
- 结构化输出验证内置

**LangChain:**
- 使用 Pydantic 但作为依赖
- 类型安全程度较低
- 更多运行时错误可能
- 需要额外配置输出解析器

### 依赖注入

**Pydantic AI:**
- 内置的 `RunContext` 依赖注入系统
- 类型安全的依赖传递
- 清晰的关注点分离
- 易于测试和模拟

**LangChain:**
- 通过回调和内存传递状态
- 依赖管理较为隐式
- 需要更多样板代码

### 可观测性

**Pydantic AI:**
- 与 Pydantic Logfire 深度集成
- OpenTelemetry 原生支持
- 实时成本跟踪
- 内置性能监控

**LangChain:**
- 通过 LangSmith 提供可观测性
- 需要额外配置
- 社区工具支持

### 模型支持

**Pydantic AI:**
- 模型无关设计
- 统一的模型接口
- 支持 20+ 提供商
- 易于添加自定义模型

**LangChain:**
- 广泛的模型集成
- 每个提供商有特定包装器
- 更成熟的生态系统
- 更多预构建集成

### 工具和能力

**Pydantic AI:**
- 简单的 `@agent.tool` 装饰器
- 可组合的能力系统
- MCP（Model Context Protocol）支持
- 清晰的工具定义

**LangChain:**
- 丰富的工具生态系统
- 预构建的工具集成
- 更多现成的连接器
- 工具定义较为复杂

### 学习曲线

**Pydantic AI:**
- 较平缓的学习曲线
- 熟悉 FastAPI 的开发者容易上手
- 概念较少，更直观
- 文档清晰简洁

**LangChain:**
- 较陡峭的学习曲线
- 更多概念需要理解（Chain、Agent、Memory 等）
- 文档丰富但可能overwhelming
- 需要时间掌握最佳实践

### 适用场景对比

**选择 Pydantic AI 当你需要:**
- 强类型安全和 IDE 支持
- 简洁、直接的 API
- 与 FastAPI 风格一致的开发体验
- 内置的可观测性和监控
- 生产级的结构化输出验证
- 清晰的依赖注入

**选择 LangChain 当你需要:**
- 丰富的预构建集成
- 成熟的社区和生态系统
- 大量的示例和教程
- 复杂的链式组合
- 特定的工具和连接器
- 已有的 LangChain 代码库

### 代码对比示例

**Pydantic AI 风格:**
```python
from pydantic_ai import Agent, RunContext
from pydantic import BaseModel

class Output(BaseModel):
    answer: str
    confidence: float

agent = Agent('openai:gpt-4', output_type=Output)

@agent.tool
async def search(ctx: RunContext, query: str) -> str:
    """搜索工具"""
    return await do_search(query)

result = await agent.run('查询问题')
```

**LangChain 风格:**
```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain.tools import tool
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate

@tool
def search(query: str) -> str:
    """搜索工具"""
    return do_search(query)

llm = ChatOpenAI(model="gpt-4")
tools = [search]
prompt = ChatPromptTemplate.from_messages([...])

agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

result = agent_executor.invoke({"input": "查询问题"})
```

## 七、最佳实践

### 1. 使用类型提示
```python
from typing import Annotated
from pydantic import Field

@agent.tool
async def calculate(
    amount: Annotated[float, Field(gt=0, description="金额必须大于0")],
    rate: Annotated[float, Field(ge=0, le=1, description="费率 0-1")]
) -> float:
    """计算费用"""
    return amount * rate
```

### 2. 结构化输出验证
```python
from pydantic import BaseModel, Field, field_validator

class AnalysisResult(BaseModel):
    summary: str = Field(min_length=10, max_length=500)
    sentiment: str = Field(pattern="^(positive|negative|neutral)$")
    score: float = Field(ge=0, le=1)
    
    @field_validator('summary')
    def validate_summary(cls, v):
        if not v.strip():
            raise ValueError('摘要不能为空')
        return v
```

### 3. 错误处理
```python
from pydantic_ai import Agent, ModelRetry

agent = Agent('openai:gpt-4')

@agent.tool
async def risky_operation(ctx: RunContext) -> str:
    try:
        result = await perform_operation()
        return result
    except SpecificError as e:
        # 让模型重试
        raise ModelRetry(f'操作失败: {e}，请重试')
    except CriticalError:
        # 不可恢复的错误
        return "操作失败，无法继续"
```

### 4. 流式输出
```python
async def stream_response():
    async with agent.run_stream('生成长文本') as stream:
        async for chunk in stream.stream_text():
            print(chunk, end='', flush=True)
```

### 5. 测试
```python
import pytest
from unittest.mock import Mock

@pytest.mark.asyncio
async def test_agent():
    mock_db = Mock()
    mock_db.get_balance.return_value = 1000.0
    
    deps = SupportDependencies(customer_id=123, db=mock_db)
    result = await support_agent.run('我的余额是多少？', deps=deps)
    
    assert '1000' in result.output.support_advice
    mock_db.get_balance.assert_called_once()
```

## 八、总结

Pydantic AI 是一个现代化的 AI Agent 框架，特别适合：

- 需要强类型安全的生产环境
- 熟悉 FastAPI 开发模式的团队
- 重视代码质量和可维护性的项目
- 需要深度可观测性的企业应用
- 追求简洁 API 和清晰架构的开发者

与 LangChain 相比，Pydantic AI 提供了更简洁、类型安全的开发体验，但生态系统相对较新。选择哪个框架取决于你的具体需求、团队背景和项目要求。

## 参考资源

- GitHub 仓库: https://github.com/pydantic/pydantic-ai
- 官方文档: https://ai.pydantic.dev
- Pydantic 文档: https://docs.pydantic.dev
- Pydantic Logfire: https://pydantic.dev/logfire

---

*文档生成日期: 2026-05-31*
