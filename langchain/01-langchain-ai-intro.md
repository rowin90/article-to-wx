# 第1章：LangChain.js 与 AI 应用开发入门

## 🎯 本章目标

通过本章学习，您将：
- 理解 LangChain.js 的核心概念和设计理念
- 掌握 LangChain.js 的安装和基础配置
- 了解 AI 应用开发的基本流程
- 完成第一个 LangChain.js 应用
- 理解 LangChain.js 在 AI 生态系统中的定位

## 📖 什么是 LangChain.js

### 定义与核心理念

LangChain.js 是一个专为构建大语言模型（LLM）应用而设计的 JavaScript/TypeScript 框架。它的核心理念是让开发者能够轻松地将 LLM 与其他数据源和计算资源连接起来，构建智能、上下文感知的应用程序。

### 核心特性

🔗 **链式组合（Chaining）**
- 将多个组件串联起来，形成复杂的处理流程
- 支持条件分支和并行处理
- 提供丰富的预构建链模板

🧠 **记忆管理（Memory）**
- 维护对话历史和上下文状态
- 支持多种记忆策略（短期、长期、摘要等）
- 自动管理 token 限制和成本优化

🔌 **模块化设计**
- 可插拔的组件架构
- 支持自定义组件开发
- 丰富的第三方集成

⚡ **异步优先**
- 原生支持 Promise 和 async/await
- 流式处理和实时响应
- 高性能并发处理

## 🏗️ LangChain.js 架构概览

### 核心组件层次

┌─────────────────────────────────────┐
│           应用层 (Applications)      │
├─────────────────────────────────────┤
│           链层 (Chains)              │
├─────────────────────────────────────┤
│         组件层 (Components)          │
│  ┌─────────┬─────────┬─────────────┐│
│  │ Models  │ Prompts │   Memory    ││
│  ├─────────┼─────────┼─────────────┤│
│  │ Indexes │ Agents  │ Callbacks   ││
│  └─────────┴─────────┴─────────────┘│
├─────────────────────────────────────┤
│          基础层 (Foundation)         │
└─────────────────────────────────────┘



### 主要模块说明

**🤖 Models（模型层）**
- LLM 接口：OpenAI、Anthropic、Hugging Face 等
- Chat Models：专门用于对话的模型接口
- Embeddings：文本向量化模型

**📝 Prompts（提示层）**
- Prompt Templates：动态提示模板
- Few-shot Examples：少样本学习示例
- Output Parsers：结构化输出解析

**🧠 Memory（记忆层）**
- Conversation Memory：对话记忆
- Vector Store Memory：向量存储记忆
- Summary Memory：摘要记忆

**🔍 Indexes（索引层）**
- Document Loaders：文档加载器
- Text Splitters：文本分割器
- Vector Stores：向量数据库

**🤖 Agents（代理层）**
- Tool Integration：工具集成
- Decision Making：决策制定
- Action Execution：动作执行

## 🛠️ 环境搭建与安装

### 系统要求

- **Node.js**: 18.0.0 或更高版本
- **npm**: 8.0.0 或更高版本（或 yarn 1.22.0+）
- **TypeScript**: 4.5.0 或更高版本（推荐）

### 创建新项目

```bash
# 1. 创建项目目录
mkdir langchain-tutorial
cd langchain-tutorial

# 2. 初始化 npm 项目
npm init -y

# 3. 安装 TypeScript 开发环境
npm install -D typescript @types/node tsx nodemon

# 4. 创建 TypeScript 配置
npx tsc --init
```

### 安装 LangChain.js

```bash
# 核心包
npm install @langchain/core

# 社区集成包
npm install @langchain/community

# OpenAI 集成
npm install @langchain/openai

# 其他常用集成
npm install @langchain/anthropic
npm install @langchain/google-genai
npm install @langchain/huggingface
```

### 配置 TypeScript

更新 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 环境变量配置

创建 `.env` 文件：

```bash
# OpenAI API 配置
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_BASE_URL=https://api.openai.com/v1

# Anthropic API 配置
ANTHROPIC_API_KEY=your_anthropic_api_key_here

# 其他配置
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your_langsmith_api_key_here
```

安装环境变量支持：

```bash
npm install dotenv
```

## 🚀 第一个 LangChain.js 应用

### 基础示例：简单对话

创建 `src/basic-chat.ts`：

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { HumanMessage, SystemMessage } from "@langchain/core/messages";
import * as dotenv from "dotenv";

// 加载环境变量
dotenv.config();

// 初始化模型
const model = new ChatOpenAI({
  openAIApiKey: process.env.OPENAI_API_KEY,
  modelName: "gpt-3.5-turbo",
  temperature: 0.7,
  maxTokens: 1000,
});

async function basicChat() {
  try {
    // 创建消息
    const messages = [
      new SystemMessage("你是一个专业的前端开发助手，擅长 JavaScript 和 React 开发。"),
      new HumanMessage("请解释什么是 React Hooks，并给出一个简单的使用示例。")
    ];

    // 发送请求并获取响应
    const response = await model.invoke(messages);

    console.log("AI 助手回复：");
    console.log(response.content);

  } catch (error) {
    console.error("发生错误：", error);
  }
}

// 运行示例
basicChat();
```

### 使用 Prompt 模板

创建 `src/prompt-template.ts`：

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { PromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";
import * as dotenv from "dotenv";

dotenv.config();

const model = new ChatOpenAI({
  openAIApiKey: process.env.OPENAI_API_KEY,
  modelName: "gpt-3.5-turbo",
});

// 创建 Prompt 模板
const promptTemplate = PromptTemplate.fromTemplate(`
你是一个{role}，专门帮助用户解决{domain}相关的问题。

用户问题：{question}

请提供详细、专业的回答，包含以下要素：
1. 问题分析
2. 解决方案
3. 代码示例（如果适用）
4. 最佳实践建议

回答：
`);

// 创建输出解析器
const outputParser = new StringOutputParser();

// 构建处理链
const chain = promptTemplate.pipe(model).pipe(outputParser);

async function promptTemplateExample() {
  try {
    const result = await chain.invoke({
      role: "资深前端工程师",
      domain: "React 性能优化",
      question: "如何优化 React 应用的渲染性能？"
    });

    console.log("优化建议：");
    console.log(result);

  } catch (error) {
    console.error("处理失败：", error);
  }
}

promptTemplateExample();
```

### 流式响应处理

创建 `src/streaming-response.ts`：

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { HumanMessage } from "@langchain/core/messages";
import * as dotenv from "dotenv";

dotenv.config();

const model = new ChatOpenAI({
  openAIApiKey: process.env.OPENAI_API_KEY,
  modelName: "gpt-3.5-turbo",
  streaming: true, // 启用流式响应
});

async function streamingExample() {
  try {
    console.log("AI 正在思考中...");
    console.log("回复：");

    const stream = await model.stream([
      new HumanMessage("请详细介绍 LangChain.js 的主要特性和使用场景。")
    ]);

    // 逐块处理流式响应
    for await (const chunk of stream) {
      process.stdout.write(chunk.content);
    }

    console.log("\n\n--- 回复完成 ---");

  } catch (error) {
    console.error("流式处理失败：", error);
  }
}

streamingExample();
```

### 运行脚本

更新 `package.json` 添加运行脚本：

```json
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "basic-chat": "tsx src/basic-chat.ts",
    "prompt-template": "tsx src/prompt-template.ts",
    "streaming": "tsx src/streaming-response.ts"
  }
}
```

运行示例：

```bash
# 运行基础对话示例
npm run basic-chat

# 运行 Prompt 模板示例
npm run prompt-template

# 运行流式响应示例
npm run streaming
```

## 🔧 核心概念深入

### 1. Runnable 接口

LangChain.js 中的所有组件都实现了 `Runnable` 接口，这是框架的核心抽象：

```typescript
interface Runnable<Input, Output> {
  invoke(input: Input): Promise<Output>;
  stream(input: Input): AsyncGenerator<Output>;
  batch(inputs: Input[]): Promise<Output[]>;
  pipe<NewOutput>(next: Runnable<Output, NewOutput>): Runnable<Input, NewOutput>;
}
```

### 2. 链式组合（Chaining）

```typescript
import { PromptTemplate } from "@langchain/core/prompts";
import { ChatOpenAI } from "@langchain/openai";
import { StringOutputParser } from "@langchain/core/output_parsers";

// 创建处理链
const prompt = PromptTemplate.fromTemplate("翻译以下文本到{language}：{text}");
const model = new ChatOpenAI();
const parser = new StringOutputParser();

// 使用 pipe 方法组合
const translationChain = prompt.pipe(model).pipe(parser);

// 使用链
const result = await translationChain.invoke({
  language: "英文",
  text: "你好，世界！"
});
```

### 3. 错误处理和重试

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { HumanMessage } from "@langchain/core/messages";

const model = new ChatOpenAI({
  maxRetries: 3, // 最大重试次数
  timeout: 30000, // 超时时间（毫秒）
});

async function robustChat() {
  try {
    const response = await model.invoke([
      new HumanMessage("解释量子计算的基本原理")
    ]);

    console.log(response.content);
  } catch (error) {
    if (error.name === 'TimeoutError') {
      console.error("请求超时，请稍后重试");
    } else if (error.status === 429) {
      console.error("API 调用频率限制，请稍后重试");
    } else {
      console.error("未知错误：", error.message);
    }
  }
}
```

## 📊 实际应用场景

### 1. 智能客服助手

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { PromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";

class CustomerServiceBot {
  private chain;

  constructor() {
    const prompt = PromptTemplate.fromTemplate(`
    你是一个专业的客服助手，请根据以下信息回答用户问题：

    公司信息：{companyInfo}
    用户问题：{userQuestion}
    用户历史：{userHistory}

    请提供友好、专业的回答：
    `);

    const model = new ChatOpenAI({
      modelName: "gpt-3.5-turbo",
      temperature: 0.3, // 较低的温度确保回答一致性
    });

    this.chain = prompt.pipe(model).pipe(new StringOutputParser());
  }

  async handleUserQuery(question: string, userHistory: string = "") {
    return await this.chain.invoke({
      companyInfo: "我们是一家专业的软件开发公司，提供 Web 和移动应用开发服务。",
      userQuestion: question,
      userHistory: userHistory
    });
  }
}

// 使用示例
const bot = new CustomerServiceBot();
const response = await bot.handleUserQuery("你们的服务价格是多少？");
console.log(response);
```

### 2. 代码审查助手

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { PromptTemplate } from "@langchain/core/prompts";

class CodeReviewAssistant {
  private model;
  private reviewPrompt;

  constructor() {
    this.model = new ChatOpenAI({
      modelName: "gpt-4", // 使用更强的模型进行代码审查
      temperature: 0.1,
    });

    this.reviewPrompt = PromptTemplate.fromTemplate(`
    请审查以下 {language} 代码，并提供详细的反馈：

    代码：
    \`\`\`{language}
    {code}
    \`\`\`

    请从以下方面进行评估：
    1. 代码质量和可读性
    2. 性能优化建议
    3. 安全性考虑
    4. 最佳实践建议
    5. 潜在的 bug 或问题

    审查报告：
    `);
  }

  async reviewCode(code: string, language: string = "javascript") {
    const chain = this.reviewPrompt.pipe(this.model);

    const result = await chain.invoke({
      code,
      language
    });

    return result.content;
  }
}

// 使用示例
const reviewer = new CodeReviewAssistant();
const codeToReview = `
function calculateTotal(items) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total += items[i].price * items[i].quantity;
  }
  return total;
}
`;

const review = await reviewer.reviewCode(codeToReview, "javascript");
console.log(review);
```

## 🎯 最佳实践

### 1. API 密钥管理

```typescript
// ❌ 错误做法：硬编码 API 密钥
const model = new ChatOpenAI({
  openAIApiKey: "sk-...", // 永远不要这样做！
});

// ✅ 正确做法：使用环境变量
const model = new ChatOpenAI({
  openAIApiKey: process.env.OPENAI_API_KEY,
});

// ✅ 更好的做法：添加验证
if (!process.env.OPENAI_API_KEY) {
  throw new Error("OPENAI_API_KEY 环境变量未设置");
}
```

### 2. 成本控制

```typescript
const model = new ChatOpenAI({
  modelName: "gpt-3.5-turbo", // 选择合适的模型
  maxTokens: 500, // 限制输出长度
  temperature: 0.7, // 适当的随机性
});

// 监控 token 使用
const response = await model.invoke(messages, {
  callbacks: [{
    handleLLMEnd: (output) => {
      console.log(`Token 使用情况:`, output.llmOutput?.tokenUsage);
    }
  }]
});
```

### 3. 错误处理策略

```typescript
class RobustLLMService {
  private model;
  private maxRetries = 3;
  private retryDelay = 1000;

  constructor() {
    this.model = new ChatOpenAI({
      maxRetries: this.maxRetries,
      timeout: 30000,
    });
  }

  async safeInvoke(messages: any[], retryCount = 0): Promise<string> {
    try {
      const response = await this.model.invoke(messages);
      return response.content;
    } catch (error) {
      if (retryCount < this.maxRetries) {
        console.log(`重试第 ${retryCount + 1} 次...`);
        await this.delay(this.retryDelay * Math.pow(2, retryCount)); // 指数退避
        return this.safeInvoke(messages, retryCount + 1);
      }
      throw error;
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

## 🔍 调试和监控

### 启用详细日志

```typescript
import { ChatOpenAI } from "@langchain/openai";
import { ConsoleCallbackHandler } from "@langchain/core/callbacks/console";

const model = new ChatOpenAI({
  callbacks: [new ConsoleCallbackHandler()], // 启用控制台日志
  verbose: true, // 详细模式
});
```

### 使用 LangSmith 追踪

```bash
# 在 .env 文件中添加
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your_langsmith_api_key
LANGCHAIN_PROJECT=your_project_name
```

## 📚 本章总结

通过本章学习，我们掌握了：

✅ **核心概念理解**
- LangChain.js 的设计理念和架构
- Runnable 接口和链式组合
- 主要组件的作用和关系

✅ **实践技能获得**
- 环境搭建和项目配置
- 基础 API 调用和响应处理
- Prompt 模板的使用
- 流式响应的处理

✅ **最佳实践掌握**
- API 密钥安全管理
- 错误处理和重试策略
- 成本控制和性能优化
- 调试和监控方法

## 🎯 下章预告

在下一章《Prompt Engineering 与模板系统深度实践》中，我们将深入探讨：

- Prompt 设计的艺术与科学
- 高级模板技术和动态生成
- Few-shot Learning 和 Chain-of-Thought
- 输出解析和结构化数据处理
- Prompt 优化和 A/B 测试
