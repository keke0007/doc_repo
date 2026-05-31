# Pi Agent 完整指南

## 一、什么是 Pi Agent？

Pi 是由 Mario Zechner 开发的自扩展 AI 编码代理工具包（Self-extensible Coding Agent Toolkit），专门用于构建交互式开发工作流的 AI 助手。它是一个基于 TypeScript 的 monorepo 项目，提供了完整的 AI 代理基础设施。

### 项目定位

Pi 的核心目标是创建一个**交互式编码代理 CLI**，让开发者能够通过自然语言与 AI 助手协作完成编码任务，包括文件读写、代码编辑、命令执行等。

### 核心特点

1. **自扩展能力（Self-extensible）**
   - 代理可以根据需求动态扩展功能
   - 支持自定义工具和提供商
   - 灵活的插件架构

2. **多提供商 LLM 支持**
   - OpenAI（标准、Completions、Codex）
   - Anthropic
   - Google（Generative AI 和 Vertex）
   - Mistral
   - Azure OpenAI
   - AWS Bedrock
   - 支持自动模型发现和提供商配置

3. **丰富的工具集**
   - 文件操作：read、write、edit
   - 搜索功能：grep、find、ls
   - 系统执行：bash
   - 工具调用和状态管理

4. **终端 UI**
   - 差异化渲染（Differential Rendering）
   - 交互式命令行界面
   - 实时反馈和状态显示

5. **供应链安全**
   - 精确版本锁定（Exact Version Pinning）
   - Shrinkwrap 文件控制传递依赖
   - Pre-commit 钩子防止意外更改
   - `--ignore-scripts` 安全安装

6. **生产级架构**
   - 传输抽象（Transport Abstraction）
   - 状态管理
   - 附件支持
   - 会话管理

## 二、项目架构

### Monorepo 结构

Pi 采用 npm workspaces 组织的 monorepo 架构，包含以下核心包：

```
pi/
├── packages/
│   ├── ai/                    # @earendil-works/pi-ai
│   │   └── 统一的 LLM API 抽象层
│   ├── agent/                 # @earendil-works/pi-agent-core
│   │   └── 通用代理运行时引擎
│   ├── coding-agent/          # @earendil-works/pi-coding-agent
│   │   └── 交互式编码代理 CLI
│   ├── tui/                   # 终端用户界面库
│   └── examples/              # 示例和扩展
│       ├── extensions/
│       ├── sandbox/
│       └── custom-providers/
```

### 核心包详解

#### 1. @earendil-works/pi-ai (v0.78.0)

**功能：** 统一的 LLM API，支持自动模型发现和提供商配置

**支持的提供商：**
- Anthropic SDK (0.91.1)
- OpenAI SDK (6.26.0)
- Google Generative AI (1.52.0)
- Google Vertex AI
- Mistral
- Azure OpenAI
- AWS Bedrock Runtime

**特性：**
- OAuth 支持
- CLI 工具（`pi-ai` 命令）
- 多种导出路径对应不同提供商
- 自动配置和模型发现

#### 2. @earendil-works/pi-agent-core (v0.78.0)

**功能：** 通用代理库，提供传输抽象、状态管理和附件支持

**核心能力：**
- 传输层抽象（Transport Abstraction）
- 状态管理（State Management）
- 附件支持（Attachment Support）
- 工具调用框架
- 会话管理

**依赖：**
- @earendil-works/pi-ai
- typebox (1.1.38) - Schema 验证
- yaml (2.9.0) - YAML 解析
- ignore (7.0.5) - 文件过滤

#### 3. @earendil-works/pi-coding-agent (v0.78.0)

**功能：** 交互式编码代理 CLI，提供完整的开发工作流支持

**核心工具：**
- **read** - 读取文件内容
- **write** - 创建或修改文件
- **edit** - 对现有文件进行精确编辑
- **bash** - 执行 Shell 命令
- **grep** - 按模式搜索文件内容
- **find** - 按名称或条件定位文件
- **ls** - 列出目录内容

**特性：**
- 会话管理
- 语法高亮
- 剪贴板支持
- 文件 glob 匹配
- 可编译为独立可执行文件（通过 Bun）

## 三、安装与配置

### 系统要求

- **Node.js**: >= 22.19.0
- **npm**: 最新版本
- **操作系统**: Linux、macOS、Windows

### 开发环境安装

```bash
# 克隆仓库
git clone https://github.com/earendil-works/pi.git
cd pi

# 安全安装依赖（跳过脚本执行）
npm install --ignore-scripts

# 构建所有包
npm run build

# 运行检查
npm run check

# 运行测试（跳过需要 API 密钥的 LLM 测试）
./test.sh
```

### 作为 npm 包使用

```bash
# 安装编码代理 CLI
npm install -g @earendil-works/pi-coding-agent

# 或者在项目中使用
npm install @earendil-works/pi-coding-agent
npm install @earendil-works/pi-agent-core
npm install @earendil-works/pi-ai
```

### API 密钥配置

根据使用的 LLM 提供商，配置相应的环境变量：

```bash
# OpenAI
export OPENAI_API_KEY='your-api-key'

# Anthropic
export ANTHROPIC_API_KEY='your-api-key'

# Google Gemini
export GOOGLE_API_KEY='your-api-key'

# AWS Bedrock
export AWS_ACCESS_KEY_ID='your-access-key'
export AWS_SECRET_ACCESS_KEY='your-secret-key'
export AWS_REGION='us-east-1'

# Azure OpenAI
export AZURE_OPENAI_API_KEY='your-api-key'
export AZURE_OPENAI_ENDPOINT='your-endpoint'
```

## 四、使用指南

### 1. 使用编码代理 CLI

```bash
# 启动交互式编码代理
pi

# 从任何目录测试 CLI
./pi-test.sh
```

**工作流示例：**

```
用户: 帮我创建一个 TypeScript 项目的基本结构

代理: [使用 write 工具创建 package.json]
     [使用 write 工具创建 tsconfig.json]
     [使用 bash 工具运行 npm install]
     
用户: 在 src 目录下创建一个 Hello World 程序

代理: [使用 write 工具创建 src/index.ts]
     [使用 read 工具验证文件内容]
```

### 2. 工具集使用

#### 只读工具集（Read-only Tools）

适用于安全检查和代码审查：

```typescript
import { createReadOnlyTools } from '@earendil-works/pi-coding-agent';

const tools = createReadOnlyTools();
// 包含: read, grep, find, ls
```

#### 编码工具集（Coding Tools）

适用于开发任务：

```typescript
import { createCodingTools } from '@earendil-works/pi-coding-agent';

const tools = createCodingTools();
// 包含: read, bash, edit, write
```

#### 完整工具集（All Tools）

```typescript
import { createAllTools } from '@earendil-works/pi-coding-agent';

const tools = createAllTools();
// 包含所有 7 个工具: read, write, edit, bash, grep, find, ls
```

### 3. 使用 pi-ai 包

```typescript
import { createClient } from '@earendil-works/pi-ai';

// 自动发现和配置提供商
const client = await createClient({
  provider: 'openai',
  model: 'gpt-4',
  apiKey: process.env.OPENAI_API_KEY
});

// 发送请求
const response = await client.chat({
  messages: [
    { role: 'user', content: 'Hello, world!' }
  ]
});

console.log(response.content);
```

### 4. 使用 pi-agent-core

```typescript
import { Agent } from '@earendil-works/pi-agent-core';
import { createCodingTools } from '@earendil-works/pi-coding-agent';

// 创建代理实例
const agent = new Agent({
  model: 'gpt-4',
  tools: createCodingTools(),
  systemPrompt: 'You are a helpful coding assistant.'
});

// 运行代理
const result = await agent.run({
  message: 'Create a new TypeScript file with a hello world function'
});

console.log(result);
```

## 五、核心概念

### 1. 工具（Tools）

工具是代理可以调用的函数，用于执行特定任务。每个工具都有：

- **名称**：工具的唯一标识符
- **描述**：工具的功能说明（用于 LLM 理解）
- **参数 Schema**：使用 TypeBox 定义的参数验证
- **执行函数**：实际执行工具逻辑的函数

**工具示例：**

```typescript
const readTool = {
  name: 'read',
  description: 'Read the contents of a file',
  parameters: Type.Object({
    path: Type.String({ description: 'File path to read' })
  }),
  execute: async (params) => {
    const content = await fs.readFile(params.path, 'utf-8');
    return content;
  }
};
```

### 2. 传输抽象（Transport Abstraction）

Pi 的传输层抽象允许代理通过不同的通信方式工作：

- **CLI 传输**：命令行界面交互
- **HTTP 传输**：Web API 集成
- **WebSocket 传输**：实时双向通信
- **自定义传输**：可扩展的传输实现

### 3. 状态管理（State Management）

代理维护会话状态，包括：

- **对话历史**：用户和代理的交互记录
- **工具调用历史**：已执行的工具及其结果
- **上下文信息**：项目配置、文件状态等
- **附件**：文件、图片等二进制数据

### 4. 会话管理（Session Management）

Pi 支持持久化会话：

- 保存和恢复对话历史
- 跨会话共享上下文
- 会话隔离和安全性
- 多会话并发支持

## 六、应用场景

### 1. 交互式代码开发

**场景描述：** 通过自然语言与 AI 协作编写代码

**典型工作流：**
```
开发者: 创建一个 Express.js API 服务器
代理: [创建项目结构] → [安装依赖] → [编写服务器代码] → [创建路由]

开发者: 添加用户认证中间件
代理: [分析现有代码] → [创建认证模块] → [集成到路由] → [编写测试]

开发者: 运行测试
代理: [执行 npm test] → [显示结果] → [修复失败的测试]
```

### 2. 代码审查和重构

**场景描述：** 使用只读工具集分析代码质量

**典型操作：**
- 使用 `grep` 查找代码模式和潜在问题
- 使用 `find` 定位特定文件
- 使用 `read` 分析代码结构
- 使用 `ls` 了解项目组织

### 3. 自动化脚本生成

**场景描述：** 生成和执行自动化脚本

**示例：**
```
开发者: 创建一个脚本来清理旧的日志文件
代理: [编写 bash 脚本] → [测试脚本] → [添加错误处理] → [创建 cron 任务]
```

### 4. 文档生成

**场景描述：** 自动生成项目文档

**工作流：**
- 读取源代码文件
- 分析代码结构和注释
- 生成 Markdown 文档
- 创建 API 参考

### 5. 测试驱动开发（TDD）

**场景描述：** AI 辅助的测试驱动开发

**流程：**
```
开发者: 为用户服务编写测试
代理: [创建测试文件] → [编写测试用例]

开发者: 实现功能使测试通过
代理: [分析测试要求] → [实现功能] → [运行测试] → [迭代修复]
```

### 6. 项目初始化和脚手架

**场景描述：** 快速搭建项目结构

**能力：**
- 创建标准项目模板
- 配置构建工具
- 设置 CI/CD 流程
- 初始化 Git 仓库

### 7. 调试和问题诊断

**场景描述：** 协助定位和修复 bug

**工作流：**
- 分析错误日志
- 搜索相关代码
- 提出修复建议
- 应用补丁并验证

## 七、与其他工具的对比

### Pi vs Cursor

**Pi Agent:**
- 开源且可自托管
- 命令行优先
- 完全可定制的工具集
- 多提供商支持
- 适合自动化和脚本化

**Cursor:**
- 商业产品
- IDE 集成
- 图形界面
- 主要使用 OpenAI
- 适合交互式编辑

### Pi vs GitHub Copilot

**Pi Agent:**
- 完整的代理框架
- 可执行系统命令
- 文件级操作
- 会话管理
- 适合复杂任务编排

**GitHub Copilot:**
- 代码补全专注
- IDE 内联建议
- 实时代码生成
- 上下文感知
- 适合快速编码

### Pi vs Aider

**Pi Agent:**
- TypeScript 生态系统
- 模块化架构
- 可编程 API
- 企业级安全特性
- 适合集成到现有工具链

**Aider:**
- Python 实现
- Git 集成优先
- 命令行工具
- 简单易用
- 适合快速原型开发

### Pi vs LangChain

**Pi Agent:**
- 专注于编码任务
- 内置开发工具
- 终端 UI
- 会话持久化
- 适合软件开发工作流

**LangChain:**
- 通用 AI 应用框架
- 丰富的集成生态
- 链式组合
- 文档处理
- 适合多样化 AI 应用

### Pi vs Pydantic AI

**Pi Agent (TypeScript):**
- TypeScript/Node.js 生态
- 编码代理专注
- CLI 优先
- 系统命令执行
- 适合前端和全栈开发

**Pydantic AI (Python):**
- Python 生态系统
- 类型安全优先
- FastAPI 风格 API
- 结构化输出验证
- 适合数据科学和后端开发

**共同点：**
- 都强调类型安全
- 多提供商支持
- 生产级设计
- 可扩展架构
- 开源 MIT 许可

## 八、高级特性

### 1. 自定义工具

创建自定义工具扩展代理能力：

```typescript
import { Tool } from '@earendil-works/pi-agent-core';
import { Type } from '@sinclair/typebox';

const customTool: Tool = {
  name: 'database_query',
  description: 'Execute a SQL query on the database',
  parameters: Type.Object({
    query: Type.String({ description: 'SQL query to execute' }),
    database: Type.String({ description: 'Database name' })
  }),
  execute: async (params) => {
    // 实现数据库查询逻辑
    const result = await db.query(params.query);
    return JSON.stringify(result);
  }
};
```

### 2. 自定义提供商

实现自定义 LLM 提供商：

```typescript
import { Provider } from '@earendil-works/pi-ai';

class CustomProvider implements Provider {
  async chat(messages, options) {
    // 实现自定义 LLM API 调用
    const response = await fetch('https://your-api.com/chat', {
      method: 'POST',
      body: JSON.stringify({ messages, ...options })
    });
    return response.json();
  }
  
  async stream(messages, options) {
    // 实现流式响应
  }
}
```

### 3. HTTP 调度器配置

Pi 支持自定义 HTTP 请求行为：

```typescript
// 在 cli.ts 中配置
import { configureHttpDispatcher } from './core/http-dispatcher';

// 应用运行时设置
configureHttpDispatcher({
  timeout: 30000,
  retries: 3,
  proxy: process.env.HTTP_PROXY
});
```

### 4. 项目级配置

在项目根目录创建 `.pi/config.json`：

```json
{
  "model": "gpt-4",
  "provider": "openai",
  "tools": ["read", "write", "edit", "bash"],
  "systemPrompt": "You are a helpful coding assistant for this project.",
  "maxTokens": 4096,
  "temperature": 0.7
}
```

### 5. 扩展和插件

Pi 支持通过扩展添加功能：

```typescript
// packages/examples/extensions/custom-extension.ts
export class CustomExtension {
  name = 'custom-extension';
  
  async onSessionStart(session) {
    console.log('Session started:', session.id);
  }
  
  async onToolCall(tool, params) {
    console.log('Tool called:', tool.name);
  }
  
  async onSessionEnd(session) {
    console.log('Session ended:', session.id);
  }
}
```

## 九、开发最佳实践

### 1. 代码质量

- **避免 `any` 类型**：使用具体的 TypeScript 类型
- **完整读取文件**：在进行大范围编辑前完整读取文件
- **使用可擦除语法**：在检查的代码中只使用可擦除的 TypeScript 语法
- **询问后再删除**：删除有意功能前先询问用户

### 2. 命令执行

- **运行检查**：代码更改后运行 `npm run check`
- **使用测试脚本**：使用 `./test.sh` 而非完整测试套件
- **不自动提交**：除非用户明确要求，否则不提交代码

### 3. Git 安全

- **明确路径**：使用 `git add` 时指定明确路径
- **避免危险命令**：
  - 不使用 `git add -A`
  - 不使用 `git reset --hard`
  - 不使用 force push
- **冲突处理**：如果未修改的文件出现冲突，中止并询问用户

### 4. 依赖管理

- **精确版本**：外部直接依赖使用精确版本锁定
- **安全安装**：本地使用 `npm install --ignore-scripts`
- **CI 安装**：CI 中使用 `npm ci --ignore-scripts`
- **工作区灵活性**：内部工作区包保持灵活性

### 5. 测试策略

- **测试文件**：对于测试文件，运行并迭代直到通过
- **使用 Faux Provider**：`packages/coding-agent/test/suite/` 测试使用 faux provider
- **无需真实 API**：不使用真实 API 密钥或付费令牌

### 6. 发布流程

- **锁步版本**：所有包使用锁步版本控制
- **更新变更日志**：发布前通过 `/cl` 提示确认变更日志已更新
- **本地冒烟测试**：发布前运行本地冒烟测试
- **版本命令**：
  ```bash
  PI_ALLOW_LOCKFILE_CHANGE=1 npm run release:patch
  PI_ALLOW_LOCKFILE_CHANGE=1 npm run release:minor
  PI_ALLOW_LOCKFILE_CHANGE=1 npm run release:major
  ```

## 十、社区和贡献

### 项目信息

- **GitHub**: https://github.com/earendil-works/pi
- **作者**: Mario Zechner
- **许可证**: MIT
- **Stars**: 58k+
- **发布版本**: 225+

### 贡献指南

1. **Fork 项目**并创建特性分支
2. **遵循代码规范**：使用 Biome 进行格式化和 lint
3. **编写测试**：为新功能添加测试
4. **运行检查**：提交前运行 `npm run check`
5. **提交 PR**：清晰描述变更内容

### 社区资源

- **Discord**: 活跃的社区频道
- **Hugging Face**: 发布真实世界代理会话数据
- **文档**: 持续更新的文档和示例
- **Issue 跟踪**: 维护者每日审查新贡献者的自动关闭 issue

### 数据共享

Pi 团队鼓励分享开源编码会话数据到 Hugging Face，以：
- 改进代理性能
- 使用真实世界数据而非玩具基准
- 支持研究和开发

## 十一、故障排除

### 常见问题

**1. 安装失败**

```bash
# 清理并重新安装
rm -rf node_modules package-lock.json
npm install --ignore-scripts
npm run build
```

**2. API 密钥未识别**

```bash
# 确保环境变量已设置
echo $OPENAI_API_KEY
echo $ANTHROPIC_API_KEY

# 或在 .env 文件中配置
cat > .env << EOF
OPENAI_API_KEY=your-key
ANTHROPIC_API_KEY=your-key
EOF
```

**3. 工具执行失败**

- 检查文件路径是否正确
- 确认有足够的文件系统权限
- 查看错误日志获取详细信息

**4. 模型响应慢**

- 检查网络连接
- 考虑使用更快的模型
- 调整 timeout 配置

**5. 构建错误**

```bash
# 清理构建缓存
npm run clean
npm run build

# 检查 TypeScript 版本
npm list typescript
```

## 十二、未来展望

### 计划中的特性

- 更多 LLM 提供商支持
- 增强的会话管理
- 图形用户界面选项
- 更丰富的工具生态系统
- 企业级功能（审计、权限控制）

### 社区驱动发展

Pi 项目欢迎社区贡献，特别是：
- 新工具实现
- 提供商集成
- 文档改进
- 使用案例分享
- 性能优化

## 十三、总结

Pi Agent 是一个强大的、生产级的 AI 编码代理工具包，特别适合：

- **TypeScript/Node.js 开发者**：原生 TypeScript 支持
- **需要自动化的团队**：可编程 API 和 CLI 工具
- **重视安全的组织**：供应链安全和精确依赖管理
- **多云环境**：支持多个 LLM 提供商
- **开源项目**：MIT 许可，完全可定制

与 Pydantic AI 相比，Pi Agent 更专注于编码任务和 TypeScript 生态系统，而 Pydantic AI 则在 Python 生态中提供更广泛的 AI 应用框架。两者都是优秀的开源项目，选择取决于你的技术栈和具体需求。

## 参考资源

- **GitHub 仓库**: https://github.com/earendil-works/pi
- **npm 包**: 
  - @earendil-works/pi-coding-agent
  - @earendil-works/pi-agent-core
  - @earendil-works/pi-ai
- **社区**: Discord 频道
- **数据集**: Hugging Face 上的真实会话数据

---

*文档生成日期: 2026-05-31*
*基于 Pi Agent v0.78.0*
