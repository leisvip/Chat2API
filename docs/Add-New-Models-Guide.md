# Chat2API 扩展服务商与模型开发方案

> 本文档提供两种扩展路径：
> - **路径 A**：向现有内置服务商添加新模型（快速，无需改协议）
> - **路径 B**：开发全新的自定义服务商模块（完整适配）
>
> 以 Mimo 添加 `MiMo-V2.5-Pro` 和 `MiMo-V2.5` 为贯穿示例。

---

## 目录

1. [扩展路径总览](#1-扩展路径总览)
2. [路径 A：向内置服务商添加新模型](#2-路径-a向内置服务商添加新模型)
3. [路径 B：开发全新自定义服务商](#3-路径-b开发全新自定义服务商)
4. [路径 C：用户侧动态模型管理（零代码）](#4-路径-c用户侧动态模型管理零代码)
5. [模型映射机制详解](#5-模型映射机制详解)
6. [完整代码示例](#6-完整代码示例)
7. [测试验证清单](#7-测试验证清单)
8. [常见问题与排错](#8-常见问题与排错)

---

## 1. 扩展路径总览

```
┌──────────────────────────────────────────────────────────────┐
│                    扩展需求分析                                │
│  "官方已上线新模型，Chat2API 还没有"                            │
└──────────────┬───────────────────────────────┬───────────────┘
               │                               │
       ┌───────▼────────┐              ┌───────▼────────┐
       │  新模型使用相同  │              │  新模型使用不同  │
       │  API 协议/端点  │              │  API 协议/端点  │
       └───────┬────────┘              └───────┬────────┘
               │                               │
       ┌───────▼────────┐              ┌───────▼────────┐
       │   路径 A        │              │   路径 B        │
       │  修改配置文件    │              │  开发新 Adapter  │
       │  （约 5 分钟）   │              │  （约 2-8 小时）  │
       └────────────────┘              └────────────────┘

       另外：路径 C（零代码，用户侧操作）
       ┌────────────────┐
       │  通过 UI 界面    │
       │  添加/编辑模型   │
       │  （无需改代码）   │
       └────────────────┘
```

### 判断标准

| 情况 | 选择 | 示例 |
|------|------|------|
| 新模型与已有模型使用**相同 API 端点和协议** | **路径 A** | Mimo 添加 MiMo-V2.5-Pro |
| 新模型使用**全新的 API 端点或协议** | **路径 B** | 新增一个全新服务商 |
| 不想改代码，只是临时用一下 | **路径 C** | 通过 UI 手动添加 |

---

## 2. 路径 A：向内置服务商添加新模型

### 2.1 场景分析

**当前状态**：
```
Mimo 内置模型：
├── mimo-v2-pro          ← 开源性能旗舰
├── mimo-v2-flash-studio ← 轻量快速
└── mimo-v2-omni         ← 全模态理解
```

**目标状态**：
```
Mimo 内置模型：
├── mimo-v2-pro          ← 开源性能旗舰
├── mimo-v2-flash-studio ← 轻量快速
├── mimo-v2-omni         ← 全模态理解
├── MiMo-V2.5-Pro        ← 新增：开源性能旗舰（新版）
└── MiMo-V2.5            ← 新增：全模态理解大模型（新版）
```

### 2.2 需要修改的文件（共 1 个）

**仅需修改**：`src/main/providers/builtin/mimo.ts`

### 2.3 修改步骤

#### 步骤 1：确认新模型的 API Model ID

首先需要确认新模型在 Mimo API 中的实际 model ID。有两种方法：

**方法 A：浏览器抓包**
```
1. 打开 https://aistudio.xiaomimimo.com
2. 登录后选择新模型发起对话
3. F12 → Network → 找到 /open-apis/bot/chat 请求
4. 查看请求体中的 modelConfig.model 字段
```

**方法 B：查阅官方文档**
```
查看 Mimo 官方文档或 API 说明中的模型 ID 列表
```

假设抓包得到：
- `MiMo-V2.5-Pro` 的 API ID 为 `mimo-v2.5-pro`
- `MiMo-V2.5` 的 API ID 为 `mimo-v2.5`

#### 步骤 2：修改 Provider 配置

```typescript
// src/main/providers/builtin/mimo.ts

import type { BuiltinProviderConfig } from '../../store/types'

export const mimoConfig: BuiltinProviderConfig = {
  id: 'mimo',
  name: 'Mimo',
  type: 'builtin',
  authType: 'cookie',
  apiEndpoint: 'https://aistudio.xiaomimimo.com',
  chatPath: '/open-apis/bot/chat',
  headers: {
    // ... 保持不变
  },
  enabled: true,
  description: 'XiaomiMIMO - Xiaomi General Intelligence Foundation Model',

  // ====== 修改 supportedModels ======
  supportedModels: [
    'mimo-v2-pro',
    'mimo-v2-flash-studio',
    'mimo-v2-omni',
    'MiMo-V2.5-Pro',   // ← 新增
    'MiMo-V2.5',        // ← 新增
  ],

  // ====== 修改 modelMappings ======
  modelMappings: {
    'mimo-v2-pro': 'mimo-v2-pro',
    'mimo-v2-flash-studio': 'mimo-v2-flash-studio',
    'mimo-v2-omni': 'mimo-v2-omni',
    'MiMo-V2.5-Pro': 'mimo-v2.5-pro',  // ← 新增：显示名 → API ID
    'MiMo-V2.5': 'mimo-v2.5',          // ← 新增：显示名 → API ID
  },

  credentialFields: [
    // ... 保持不变
  ],
}

export default mimoConfig
```

#### 步骤 3（可选）：更新中文 README

```markdown
# README_CN.md 中的支持服务商表格

| Mimo | Cookie | 是 | mimo-v2-pro, mimo-v2-flash-studio, mimo-v2-omni, **MiMo-V2.5-Pro**, **MiMo-V2.5** |
```

### 2.4 关键概念：`supportedModels` vs `modelMappings`

```typescript
// supportedModels：显示给用户的模型名列表（UI 展示用）
supportedModels: ['MiMo-V2.5-Pro', 'MiMo-V2.5']

// modelMappings：显示名 → API 实际 ID 的映射
modelMappings: {
  'MiMo-V2.5-Pro': 'mimo-v2.5-pro',  // 用户看到 "MiMo-V2.5-Pro"，实际调用 "mimo-v2.5-pro"
  'MiMo-V2.5': 'mimo-v2.5',          // 用户看到 "MiMo-V2.5"，实际调用 "mimo-v2.5"
}

// 如果显示名和 API ID 相同，可以不写映射
// 例如 'mimo-v2-pro' 没有映射，因为 supportedModels 中的名字就是 API ID
```

### 2.5 新模型特殊能力处理

如果新模型支持新的特殊能力（如新的思考模式、新的输入格式），需要检查并可能修改 Adapter。

**Mimo Adapter 中的能力检测逻辑**：

```typescript
// src/main/proxy/adapters/mimo.ts → chatCompletion()

const modelLower = request.model.toLowerCase()
let enableThinking = false

// 当前仅检测 "think" 和 "r1" 关键词
if (modelLower.includes('think') || modelLower.includes('r1')) {
  enableThinking = true
}

// 如果新模型使用不同的关键词触发思考模式，需要添加
// 例如 MiMo-V2.5-Pro 可能用 "reasoning" 关键词：
if (modelLower.includes('think') || modelLower.includes('r1') || modelLower.includes('reasoning')) {
  enableThinking = true
}
```

**检查清单**：
- [ ] 新模型是否支持思考模式？触发关键词是什么？
- [ ] 新模型是否支持联网搜索？触发参数是什么？
- [ ] 新模型是否支持文件上传？格式是否变化？
- [ ] 新模型的流式响应格式是否有变化？
- [ ] 新模型是否支持工具调用？格式是否变化？

### 2.6 完整 diff 示例（Mimo 添加新模型）

```diff
// src/main/providers/builtin/mimo.ts

  supportedModels: [
    'mimo-v2-pro',
    'mimo-v2-flash-studio',
    'mimo-v2-omni',
+   'MiMo-V2.5-Pro',
+   'MiMo-V2.5',
  ],
  modelMappings: {
    'mimo-v2-pro': 'mimo-v2-pro',
    'mimo-v2-flash-studio': 'mimo-v2-flash-studio',
    'mimo-v2-omni': 'mimo-v2-omni',
+   'MiMo-V2.5-Pro': 'mimo-v2.5-pro',
+   'MiMo-V2.5': 'mimo-v2.5',
  },
```

**总计修改**：2 行配置，约 30 秒。

---

## 3. 路径 B：开发全新自定义服务商

### 3.1 需要创建的文件

```
src/main/providers/builtin/{provider}.ts        ← [必须] Provider 配置
src/main/oauth/adapters/{provider}.ts            ← [必须] OAuth 认证适配器
src/main/proxy/adapters/{provider}.ts            ← [必须] 请求转发适配器
src/main/proxy/adapters/{provider}-stream.ts     ← [可选] 独立流式处理器
src/renderer/src/assets/providers/{provider}.svg  ← [可选] 图标
```

### 3.2 需要修改的文件

```
src/main/providers/builtin/index.ts              ← 注册 Provider
src/main/oauth/adapters/index.ts                 ← 注册 OAuth Adapter
src/main/proxy/forwarder.ts                      ← 添加路由分支
src/main/providers/checker.ts                    ← 添加 Token 验证
src/main/oauth/guides.ts                         ← 添加 Token 提取指南
```

### 3.3 开发步骤详解

#### 步骤 1：创建 Provider 配置

```typescript
// src/main/providers/builtin/new-provider.ts

import type { BuiltinProviderConfig } from '../../store/types'

export const newProviderConfig: BuiltinProviderConfig = {
  // === 基础信息 ===
  id: 'new-provider',                    // 唯一标识符，小写英文
  name: 'New Provider',                  // 显示名称
  type: 'builtin',                       // 'builtin' | 'custom'
  authType: 'jwt',                       // 认证类型

  // === API 端点 ===
  apiEndpoint: 'https://api.new-provider.com',
  chatPath: '/v1/chat/completions',      // Chat API 路径

  // === 请求头（模拟浏览器）===
  headers: {
    'Content-Type': 'application/json',
    'Accept': '*/*',
    'Origin': 'https://new-provider.com',
    'Referer': 'https://new-provider.com/',
    'User-Agent': 'Mozilla/5.0 ...',
    // ... 其他浏览器指纹 Header
  },

  // === 模型定义 ===
  enabled: true,
  description: 'New AI Provider',
  supportedModels: [
    'Model-A',           // 显示名
    'Model-B-Thinking',  // 带思考模式的模型
  ],
  modelMappings: {
    'Model-A': 'model-a',              // 显示名 → API ID
    'Model-B-Thinking': 'model-b',     // 显示名 → API ID（思考模式通过参数控制）
  },

  // === 凭证字段定义 ===
  credentialFields: [
    {
      name: 'token',                    // 凭证 key（与 credentials 对象对应）
      label: 'Access Token',           // UI 显示标签
      type: 'password',                 // 'text' | 'password' | 'textarea'
      required: true,
      placeholder: 'Enter your token',
      helpText: 'Get from browser DevTools → Application → Local Storage',
    },
  ],

  // === Token 验证（可选）===
  tokenCheckEndpoint: '/v1/user/info',   // 验证端点
  tokenCheckMethod: 'GET',               // 验证方法

  // === 动态模型列表（可选）===
  modelsApiEndpoint: 'https://api.new-provider.com/v1/models',
  modelsApiHeaders: {
    'Accept': 'application/json',
  },
}

export default newProviderConfig
```

#### 步骤 2：创建 OAuth Adapter

```typescript
// src/main/oauth/adapters/new-provider.ts

import { BaseOAuthAdapter } from './base'
import {
  OAuthResult,
  OAuthOptions,
  TokenValidationResult,
  CredentialInfo,
  AdapterConfig,
} from '../types'

export class NewProviderAdapter extends BaseOAuthAdapter {
  constructor(config: AdapterConfig) {
    super({
      ...config,
      providerType: 'new-provider',
      authMethods: ['manual'],           // 'manual' | 'cookie' | 'oauth'
      loginUrl: 'https://new-provider.com',
      apiUrl: 'https://api.new-provider.com',
    })
  }

  /**
   * 启动登录流程
   * 打开浏览器让用户手动获取 Token
   */
  async startLogin(options: OAuthOptions): Promise<OAuthResult> {
    this.emitProgress('pending', 'Opening browser...')

    try {
      // 打开浏览器
      await this.openBrowser('https://new-provider.com')
      this.emitProgress('pending', 'Please log in and extract token manually')

      return {
        success: false,
        providerId: options.providerId,
        providerType: 'new-provider',
        error: 'Please log in via browser and enter token manually',
      }
    } catch (error) {
      const errorMessage = error instanceof Error ? error.message : 'Failed to open browser'
      this.emitProgress('error', errorMessage)
      return {
        success: false,
        providerId: options.providerId,
        providerType: 'new-provider',
        error: errorMessage,
      }
    }
  }

  /**
   * 使用手动输入的 Token 完成认证
   */
  async loginWithToken(providerId: string, token: string): Promise<OAuthResult> {
    this.emitProgress('pending', 'Validating token...')

    try {
      const validation = await this.validateToken({ token })

      if (!validation.valid) {
        return {
          success: false,
          providerId,
          providerType: 'new-provider',
          error: validation.error || 'Token validation failed',
        }
      }

      this.emitProgress('success', 'Token validated successfully')

      return {
        success: true,
        providerId,
        providerType: 'new-provider',
        credentials: { token },
        accountInfo: validation.accountInfo,
      }
    } catch (error) {
      const errorMessage = error instanceof Error ? error.message : 'Validation failed'
      this.emitProgress('error', errorMessage)
      return {
        success: false,
        providerId,
        providerType: 'new-provider',
        error: errorMessage,
      }
    }
  }

  /**
   * 验证 Token 有效性
   */
  async validateToken(credentials: Record<string, string>): Promise<TokenValidationResult> {
    const { token } = credentials

    if (!token) {
      return { valid: false, error: 'Token is required' }
    }

    try {
      // 调用服务商 API 验证 Token
      const axios = (await import('axios')).default
      const response = await axios.get(
        'https://api.new-provider.com/v1/user/info',
        {
          headers: {
            Authorization: `Bearer ${token}`,
          },
          timeout: 15000,
          validateStatus: () => true,
        }
      )

      if (response.status === 200 && response.data?.user) {
        return {
          valid: true,
          accountInfo: {
            name: response.data.user.name,
            email: response.data.user.email,
          },
        }
      }

      if (response.status === 401) {
        return { valid: false, error: 'Token expired or invalid' }
      }

      return { valid: false, error: `Validation failed: HTTP ${response.status}` }
    } catch (error) {
      return {
        valid: false,
        error: error instanceof Error ? error.message : 'Connection failed',
      }
    }
  }

  /**
   * 刷新 Token（如果支持）
   */
  async refreshToken(credentials: Record<string, string>): Promise<CredentialInfo | null> {
    // 如果服务商支持 Token 刷新，在此实现
    // 否则返回 null
    return null
  }
}

export default NewProviderAdapter
```

#### 步骤 3：创建 Proxy Adapter（核心）

```typescript
// src/main/proxy/adapters/new-provider.ts

import axios, { AxiosResponse } from 'axios'
import { PassThrough } from 'stream'
import { Account, Provider } from '../store/types'

const API_BASE = 'https://api.new-provider.com'

interface ChatCompletionRequest {
  model: string
  messages: any[]
  stream?: boolean
  temperature?: number
  // ... 其他参数
}

export class NewProviderAdapter {
  private provider: Provider
  private account: Account

  constructor(provider: Provider, account: Account) {
    this.provider = provider
    this.account = account
  }

  /**
   * 判断是否为该服务商的 Provider
   */
  static isNewProvider(provider: Provider): boolean {
    return provider.id === 'new-provider' ||
           provider.apiEndpoint.includes('new-provider.com')
  }

  /**
   * 获取认证 Token
   */
  private getToken(): string {
    return this.account.credentials.token || ''
  }

  /**
   * 消息格式转换
   * 将 OpenAI 格式转换为服务商原生格式
   */
  private convertMessages(messages: any[]): any[] {
    return messages.map(msg => {
      if (msg.role === 'system') {
        // 某些服务商不支持 system role，需要合并到 user 消息
        return { role: 'user', content: `[System] ${msg.content}` }
      }
      if (Array.isArray(msg.content)) {
        // 处理多模态内容
        const textParts = msg.content
          .filter((p: any) => p.type === 'text')
          .map((p: any) => p.text)
        return { ...msg, content: textParts.join('\n') }
      }
      return msg
    })
  }

  /**
   * 核心方法：发送 Chat Completion 请求
   */
  async chatCompletion(request: ChatCompletionRequest): Promise<{
    response: AxiosResponse
    sessionId: string
  }> {
    const token = this.getToken()
    const sessionId = this.generateSessionId()

    // 转换消息格式
    const convertedMessages = this.convertMessages(request.messages)

    // 构建请求体（根据服务商 API 文档）
    const requestBody = {
      model: request.model,
      messages: convertedMessages,
      stream: request.stream ?? true,
      temperature: request.temperature ?? 0.7,
      // ... 其他服务商特定参数
    }

    const response = await axios.post(
      `${API_BASE}/v1/chat/completions`,
      requestBody,
      {
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`,
          // ... 浏览器伪装 Header
        },
        timeout: 120000,
        validateStatus: () => true,
        responseType: request.stream ? 'stream' : 'json',
      }
    )

    return { response, sessionId }
  }

  /**
   * 删除会话（如果支持）
   */
  async deleteSession(sessionId: string): Promise<boolean> {
    // 实现会话清理逻辑
    return true
  }

  private generateSessionId(): string {
    return `session-${Date.now()}-${Math.random().toString(36).slice(2, 11)}`
  }
}

/**
 * 流式响应处理器
 * 将服务商的 SSE 格式转换为 OpenAI 标准格式
 */
export class NewProviderStreamHandler {
  private model: string

  constructor(model: string) {
    this.model = model
  }

  /**
   * 处理流式响应
   * 将服务商的 SSE 流转换为 OpenAI 格式的 PassThrough 流
   */
  async handleStream(stream: any): Promise<PassThrough> {
    const transStream = new PassThrough()
    const created = Math.floor(Date.now() / 1000)
    const id = `chatcmpl-${Date.now().toString(36)}`

    // 写入初始 role chunk
    transStream.write(
      `data: ${JSON.stringify({
        id,
        model: this.model,
        object: 'chat.completion.chunk',
        choices: [{
          index: 0,
          delta: { role: 'assistant', content: '' },
          finish_reason: null,
        }],
        created,
      })}\n\n`
    )

    // 解析服务商的 SSE 流
    let buffer = ''
    stream.on('data', (chunk: Buffer) => {
      buffer += chunk.toString()
      const lines = buffer.split('\n')
      buffer = lines.pop() || ''

      for (const line of lines) {
        if (line.startsWith('data:')) {
          const data = line.slice(5).trim()
          if (data === '[DONE]') {
            transStream.write(
              `data: ${JSON.stringify({
                id,
                model: this.model,
                object: 'chat.completion.chunk',
                choices: [{ index: 0, delta: {}, finish_reason: 'stop' }],
                created,
              })}\n\n`
            )
            transStream.end('data: [DONE]\n\n')
            return
          }

          try {
            const parsed = JSON.parse(data)
            // 转换为 OpenAI chunk 格式
            const content = parsed.choices?.[0]?.delta?.content ||
                           parsed.content ||
                           parsed.text || ''

            if (content) {
              transStream.write(
                `data: ${JSON.stringify({
                  id,
                  model: this.model,
                  object: 'chat.completion.chunk',
                  choices: [{
                    index: 0,
                    delta: { content },
                    finish_reason: null,
                  }],
                  created,
                })}\n\n`
              )
            }
          } catch {
            // 跳过无效 JSON
          }
        }
      }
    })

    stream.on('error', (err: Error) => {
      transStream.end()
    })

    stream.on('close', () => {
      if (!transStream.closed) {
        transStream.end('data: [DONE]\n\n')
      }
    })

    return transStream
  }

  /**
   * 处理非流式响应
   * 收集完整响应并转换为 OpenAI 格式
   */
  async handleNonStream(stream: any): Promise<any> {
    return new Promise((resolve, reject) => {
      let content = ''
      let buffer = ''

      stream.on('data', (chunk: Buffer) => {
        buffer += chunk.toString()
        // 解析并收集内容
        // ...
      })

      stream.on('end', () => {
        resolve({
          id: `chatcmpl-${Date.now().toString(36)}`,
          object: 'chat.completion',
          created: Math.floor(Date.now() / 1000),
          model: this.model,
          choices: [{
            index: 0,
            message: { role: 'assistant', content },
            finish_reason: 'stop',
          }],
          usage: { prompt_tokens: 0, completion_tokens: 0, total_tokens: 0 },
        })
      })

      stream.on('error', reject)
    })
  }
}

export const newProviderAdapter = {
  NewProviderAdapter,
  NewProviderStreamHandler,
}
```

#### 步骤 4：注册到系统

**4a. 注册 Provider 配置**：
```typescript
// src/main/providers/builtin/index.ts

import newProviderConfig from './new-provider'  // ← 新增导入

export const builtinProviders: BuiltinProviderConfig[] = [
  deepseekConfig,
  glmConfig,
  kimiConfig,
  minimaxConfig,
  mimoConfig,
  perplexityConfig,
  qwenConfig,
  qwenAiConfig,
  zaiConfig,
  newProviderConfig,  // ← 新增注册
]

export const builtinProviderMap: Record<string, BuiltinProviderConfig> = {
  // ... 已有映射
  'new-provider': newProviderConfig,  // ← 新增映射
}
```

**4b. 注册 OAuth Adapter**：
```typescript
// src/main/oauth/adapters/index.ts

import { NewProviderAdapter } from './new-provider'  // ← 新增导入

export function createAdapter(
  providerType: ProviderType,
  config: AdapterConfig
): BaseOAuthAdapter {
  switch (providerType) {
    // ... 已有 case
    case 'new-provider':                // ← 新增 case
      return new NewProviderAdapter(config)
    default:
      throw new Error(`Unsupported provider type: ${providerType}`)
  }
}
```

**4c. 添加 Forwarder 路由**：
```typescript
// src/main/proxy/forwarder.ts → doForward()

import { NewProviderAdapter } from './adapters/new-provider'

// 在 doForward() 方法中添加判断分支：
if (NewProviderAdapter.isNewSeekProvider(provider)) {
  return this.forwardNewProvider(request, account, provider, actualModel, startTime)
}

// 添加转发方法：
private async forwardNewProvider(
  request: ChatCompletionRequest,
  account: Account,
  provider: Provider,
  actualModel: string,
  startTime: number
): Promise<ForwardResult> {
  try {
    const transformed = this.transformRequestForPromptToolUse(request, provider)
    const adapter = new NewProviderAdapter(provider, account)
    const { response, sessionId } = await adapter.chatCompletion({
      model: actualModel,
      messages: transformed.messages,
      stream: request.stream,
      temperature: request.temperature,
    })

    const latency = Date.now() - startTime

    if (response.status >= 400) {
      return { success: false, status: response.status, error: 'Request failed', latency }
    }

    const handler = new NewProviderStreamHandler(actualModel)

    if (request.stream) {
      const transformedStream = await handler.handleStream(response.data)
      return {
        success: true,
        status: response.status,
        stream: transformedStream,
        skipTransform: true,
        latency,
        providerSessionId: sessionId,
      }
    }

    const result = await handler.handleNonStream(response.data)
    return {
      success: true,
      status: response.status,
      body: result,
      latency,
      providerSessionId: sessionId,
    }
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error',
      latency: Date.now() - startTime,
    }
  }
}
```

**4d. 添加 Token 验证**（可选）：
```typescript
// src/main/providers/checker.ts → checkAccountToken()

case 'new-provider':
  return this.checkNewProviderToken(account.credentials.token)
```

**4e. 添加 Token 提取指南**（可选）：
```typescript
// src/main/oauth/guides.ts

newProvider: {
  loginUrl: 'https://new-provider.com',
  steps: [
    '1. Click the button below to open New Provider website',
    '2. Log in to your account',
    '3. Press F12 to open Developer Tools',
    '4. Switch to the Application tab',
    '5. Find Local Storage → new-provider.com',
    '6. Find the token field and copy its value',
  ],
  tokenKey: 'token',
  tokenLabel: 'Token',
  storageType: 'localStorage',
  placeholder: 'Paste the token from New Provider',
},
```

---

## 4. 路径 C：用户侧动态模型管理（零代码）

### 4.1 通过 UI 添加自定义模型

Chat2API 内置了 **Model Editor** 组件，允许用户在不修改代码的情况下添加/删除模型。

**操作路径**：侧边栏 → 模型管理 → 选择服务商 → 编辑模型

**功能**：
- ✅ 添加自定义模型（指定显示名和实际 API ID）
- ✅ 删除默认模型（标记为排除）
- ✅ 重置为默认模型列表
- ✅ 实时生效，无需重启

### 4.2 通过 UI 创建自定义服务商

**操作路径**：侧边栏 → 服务商 → 添加服务商 → 自定义

**可配置项**：
- 服务商名称
- API 端点
- 认证类型
- 请求 Header
- 支持的模型列表
- 凭证字段定义

### 4.3 通过 Management API 管理

如果启用了 Management API（设置 → 高级 → 启用管理 API），可以通过 HTTP 接口管理：

```bash
# 添加自定义模型
curl -X POST http://localhost:8080/v0/management/providers/{providerId}/models \
  -H "Authorization: Bearer {management-secret}" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "MiMo-V2.5-Pro",
    "actualModelId": "mimo-v2.5-pro"
  }'

# 获取服务商模型列表
curl http://localhost:8080/v0/management/providers/{providerId}/models \
  -H "Authorization: Bearer {management-secret}"
```

### 4.4 数据存储位置

用户自定义的模型覆盖存储在：
```
~/.chat2api/data.json → userModelOverrides 字段
```

```json
{
  "userModelOverrides": {
    "mimo": {
      "addedModels": [
        { "displayName": "MiMo-V2.5-Pro", "actualModelId": "mimo-v2.5-pro" },
        { "displayName": "MiMo-V2.5", "actualModelId": "mimo-v2.5" }
      ],
      "excludedModels": []
    }
  }
}
```

---

## 5. 模型映射机制详解

### 5.1 映射优先级

```
用户请求 model: "MiMo-V2.5-Pro"
        │
        ▼
┌───────────────────────────────┐
│ 1. 全局 ModelMapping 匹配     │  config.modelMappings["MiMo-V2.5-Pro"]
│    精确匹配 → 通配符匹配       │  支持 preferredProviderId
└───────────┬───────────────────┘
            │ 未命中
            ▼
┌───────────────────────────────┐
│ 2. Provider Effective Models  │  storeManager.getEffectiveModels(providerId)
│    合并默认模型 + 用户覆盖      │  displayName 匹配
└───────────┬───────────────────┘
            │ 未命中
            ▼
┌───────────────────────────────┐
│ 3. 原样返回                   │  model = "MiMo-V2.5-Pro"
└───────────────────────────────┘
```

### 5.2 Effective Models 合并逻辑

```typescript
// storeManager.getEffectiveModels(providerId)

function getEffectiveModels(providerId: string): EffectiveModel[] {
  const provider = getProviderById(providerId)
  const defaultModels = provider.supportedModels || []      // 内置默认
  const modelMappings = provider.modelMappings || {}
  const overrides = getUserModelOverrides(providerId)       // 用户覆盖

  const result: EffectiveModel[] = []

  // 1. 默认模型（排除被用户排除的）
  for (const displayName of defaultModels) {
    if (!overrides.excludedModels.includes(displayName)) {
      result.push({
        displayName,
        actualModelId: modelMappings[displayName] || displayName,
        isCustom: false,
      })
    }
  }

  // 2. 用户添加的自定义模型
  for (const customModel of overrides.addedModels) {
    result.push({
      displayName: customModel.displayName,
      actualModelId: customModel.actualModelId,
      isCustom: true,
    })
  }

  return result
}
```

### 5.3 通配符映射

```typescript
// config.modelMappings 支持通配符
modelMappings: {
  'gpt-4*': { actualModel: 'deepseek-chat', preferredProviderId: 'deepseek' },
  'claude-*': { actualModel: 'glm-5', preferredProviderId: 'glm' },
  '*': { actualModel: 'kimi-k2.5', preferredProviderId: 'kimi' },  // 兜底
}
```

---

## 6. 完整代码示例

### 6.1 示例：Mimo 添加 MiMo-V2.5-Pro 和 MiMo-V2.5

**修改文件**：`src/main/providers/builtin/mimo.ts`

```typescript
// 完整修改后的内容

import type { BuiltinProviderConfig } from '../../store/types'

export const mimoConfig: BuiltinProviderConfig = {
  id: 'mimo',
  name: 'Mimo',
  type: 'builtin',
  authType: 'cookie',
  apiEndpoint: 'https://aistudio.xiaomimimo.com',
  chatPath: '/open-apis/bot/chat',
  headers: {
    'Content-Type': 'application/json',
    'Accept': '*/*',
    'Accept-Encoding': 'gzip, deflate, br, zstd',
    'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
    'Cache-Control': 'no-cache',
    'Origin': 'https://aistudio.xiaomimimo.com',
    'Referer': 'https://aistudio.xiaomimimo.com/',
    'Pragma': 'no-cache',
    'Sec-Ch-Ua': '"Chromium";v="144", "Not(A:Brand";v="8", "Google Chrome";v="144"',
    'Sec-Ch-Ua-Mobile': '?0',
    'Sec-Ch-Ua-Platform': '"Windows"',
    'Sec-Fetch-Dest': 'empty',
    'Sec-Fetch-Mode': 'cors',
    'Sec-Fetch-Site': 'same-origin',
    'X-Timezone': 'Asia/Shanghai',
  },
  enabled: true,
  description: 'XiaomiMIMO - Xiaomi General Intelligence Foundation Model',
  supportedModels: [
    // 原有模型
    'mimo-v2-pro',
    'mimo-v2-flash-studio',
    'mimo-v2-omni',
    // 新增模型
    'MiMo-V2.5-Pro',
    'MiMo-V2.5',
  ],
  modelMappings: {
    // 原有映射
    'mimo-v2-pro': 'mimo-v2-pro',
    'mimo-v2-flash-studio': 'mimo-v2-flash-studio',
    'mimo-v2-omni': 'mimo-v2-omni',
    // 新增映射（显示名 → API 实际 ID）
    'MiMo-V2.5-Pro': 'mimo-v2.5-pro',
    'MiMo-V2.5': 'mimo-v2.5',
  },
  credentialFields: [
    {
      name: 'service_token',
      label: 'Service Token',
      type: 'password',
      required: true,
      placeholder: 'Enter serviceToken from Cookie',
      helpText: 'Found in browser DevTools -> Application -> Cookies -> serviceToken',
    },
    {
      name: 'user_id',
      label: 'User ID',
      type: 'text',
      required: true,
      placeholder: 'Enter userId from Cookie',
      helpText: 'Found in browser DevTools -> Application -> Cookies -> userId',
    },
    {
      name: 'ph_token',
      label: 'PH Token',
      type: 'password',
      required: true,
      placeholder: 'Enter xiaomichatbot_ph from Cookie',
      helpText: 'Found in browser DevTools -> Application -> Cookies -> xiaomichatbot_ph',
    },
  ],
}

export default mimoConfig
```

### 6.2 示例：Mimo Adapter 添加新模型特殊处理

如果 `MiMo-V2.5-Pro` 支持新的 `reasoning` 模式（假设），修改 Adapter：

```typescript
// src/main/proxy/adapters/mimo.ts → chatCompletion()

const modelLower = request.model.toLowerCase()
let enableThinking = false

// 原有检测
if (modelLower.includes('think') || modelLower.includes('r1')) {
  enableThinking = true
}

// 新增：MiMo-V2.5-Pro 使用 "reasoning" 关键词
if (modelLower.includes('reasoning')) {
  enableThinking = true
  console.log('[Mimo] Reasoning mode enabled for MiMo-V2.5-Pro')
}
```

---

## 7. 测试验证清单

### 7.1 路径 A 测试清单

- [ ] 修改配置后 `npm run dev` 启动开发环境
- [ ] 在 UI 的服务商页面看到新模型
- [ ] 在模型管理页面看到新模型
- [ ] 使用新模型发起对话，返回正常
- [ ] 流式响应正常（无卡顿、无截断）
- [ ] 非流式响应正常
- [ ] 思考模式正常触发（如果支持）
- [ ] 联网搜索正常触发（如果支持）
- [ ] 工具调用正常（如果有客户端使用 tools）
- [ ] 请求日志记录正确
- [ ] 仪表盘统计正确

### 7.2 路径 B 测试清单

- [ ] Provider 配置加载无报错
- [ ] OAuth 登录流程正常
- [ ] Token 验证通过
- [ ] Token 提取指南显示正确
- [ ] Chat Completion 请求成功
- [ ] 流式响应正确转换为 OpenAI 格式
- [ ] 非流式响应正确转换
- [ ] 错误处理正常（Token 过期、网络超时等）
- [ ] 会话管理正常（创建/删除）
- [ ] 负载均衡正常（多账户场景）
- [ ] 模型映射正常
- [ ] 工具调用正常

### 7.3 自动化测试

```bash
# 运行现有测试
npm test

# 手动测试 API
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MiMo-V2.5-Pro",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": false
  }'

# 测试流式响应
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MiMo-V2.5",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true
  }'

# 检查模型列表
curl http://localhost:8080/v1/models | jq '.data[].id'
```

---

## 8. 常见问题与排错

### 8.1 新模型添加后不显示

**原因**：缓存或未重启开发服务器

**解决**：
```bash
# 重启开发服务器
npx electron-vite dev 2>&1

# 或检查配置是否正确保存
cat ~/.chat2api/data.json | jq '.providers[] | select(.id=="mimo") | .supportedModels'
```

### 8.2 新模型请求返回 400/404

**原因**：API Model ID 不正确

**解决**：
1. 在浏览器中使用新模型发起对话
2. F12 → Network → 找到请求
3. 对比请求体中的 model 字段与配置中的 modelMappings

### 8.3 新模型流式响应格式不对

**原因**：服务商的 SSE 格式与现有 Adapter 不兼容

**解决**：
1. 抓包查看原始 SSE 数据
2. 对比现有 Adapter 的解析逻辑
3. 可能需要在 Adapter 中添加新模型的特殊处理

### 8.4 自定义服务商的 CORS 问题

**原因**：浏览器直接请求被 CORS 阻止

**解决**：Electron 主进程不受 CORS 限制，确保请求从主进程发出（通过 IPC）。

### 8.5 Token 验证失败

**原因**：Token 格式或验证端点不正确

**解决**：
1. 确认 Token 获取方式正确
2. 使用 curl 手动测试验证端点
3. 检查请求 Header 是否完整（特别是浏览器伪装 Header）

---

## 附录：快速参考

### 文件修改速查表

| 操作 | 修改文件 | 改动量 |
|------|---------|--------|
| 添加新模型（同协议） | `providers/builtin/{provider}.ts` | 2 行 |
| 添加新模型（新能力） | 上述 + `proxy/adapters/{provider}.ts` | 10-50 行 |
| 添加新服务商 | 5+ 个文件 | 500-2000 行 |
| 用户侧添加模型 | UI 操作 | 0 行 |
| 用户侧添加服务商 | UI 操作 | 0 行 |

### 认证类型速查

| 类型 | 凭证字段 | 代表服务商 |
|------|---------|-----------|
| `userToken` | `token` | DeepSeek |
| `refresh_token` | `refresh_token` | GLM |
| `jwt` | `token` | Kimi, MiniMax, Qwen AI, Z.ai |
| `cookie` | `service_token` + `user_id` + `ph_token` | Mimo |
| `tongyi_sso_ticket` | `ticket` | Qwen |

### 模型 ID 命名建议

| 格式 | 示例 | 说明 |
|------|------|------|
| `provider-version` | `mimo-v2.5-pro` | API 实际 ID（小写+连字符） |
| `Provider-Version` | `MiMo-V2.5-Pro` | 显示名（可含大小写） |
| `provider_variant` | `mimo_v2_5_pro` | 备选格式（下划线） |
