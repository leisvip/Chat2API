# Chat2API 服务商模块深度分析报告

> 基于 Chat2API v1.2.0 源码通读，对全部 9 个内置服务商的配置、认证、协议、适配器进行逐项拆解。

---

## 目录

1. [总览](#1-总览)
2. [服务商注册机制](#2-服务商注册机制)
3. [逐服务商深度分析](#3-逐服务商深度分析)
   - 3.1 [DeepSeek](#31-deepseek)
   - 3.2 [GLM（智谱清言）](#32-glm智谱清言)
   - 3.3 [Kimi（月之暗面）](#33-kimi月之暗面)
   - 3.4 [MiniMax](#34-minimax)
   - 3.5 [Mimo（小米）](#35-mimo小米)
   - 3.6 [Perplexity](#36-perplexity)
   - 3.7 [Qwen（通义千问国内版）](#37-qwen通义千问国内版)
   - 3.8 [Qwen AI（国际版）](#38-qwen-ai国际版)
   - 3.9 [Z.ai（智谱海外）](#39-zai智谱海外)
4. [认证方式对比矩阵](#4-认证方式对比矩阵)
5. [协议差异对比](#5-协议差异对比)
6. [特殊能力支持矩阵](#6-特殊能力支持矩阵)
7. [工具调用（Function Calling）实现差异](#7-工具调用function-calling实现差异)
8. [流式响应协议对比](#8-流式响应协议对比)
9. [会话生命周期管理](#9-会话生命周期管理)
10. [扩展新服务商指南](#10-扩展新服务商指南)

---

## 1. 总览

Chat2API 通过**模拟用户在各大模型官方 Web 端的操作**，将网页聊天接口转换为标准 OpenAI 兼容 API。每个服务商需要三层适配：

| 层级 | 目录 | 职责 |
|------|------|------|
| **Provider Config** | `src/main/providers/builtin/` | 定义服务商元数据（ID、端点、模型列表、凭证字段） |
| **OAuth Adapter** | `src/main/oauth/adapters/` | 处理认证流程（登录、Token 提取、验证、刷新） |
| **Proxy Adapter** | `src/main/proxy/adapters/` | 实现请求转发协议（消息格式转换、流式解析、会话管理） |

---

## 2. 服务商注册机制

### 2.1 注册入口

```typescript
// src/main/providers/builtin/index.ts
export const builtinProviders: BuiltinProviderConfig[] = [
  deepseekConfig, glmConfig, kimiConfig, minimaxConfig,
  mimoConfig, perplexityConfig, qwenConfig, qwenAiConfig, zaiConfig,
]
```

### 2.2 BuiltinProviderConfig 接口

```typescript
interface BuiltinProviderConfig extends Omit<Provider, 'createdAt' | 'updatedAt'> {
  credentialFields: CredentialField[]     // 凭证字段定义
  tokenCheckEndpoint?: string             // Token 验证端点
  tokenCheckMethod?: 'GET' | 'POST'      // 验证请求方法
  modelsApiEndpoint?: string              // 动态模型列表 API
  modelsApiHeaders?: Record<string, string>
}
```

### 2.3 Provider 生命周期

1. **初始化**：`StoreManager.initializeDefaultProviders()` 读取内置配置，合并用户覆盖
2. **账户绑定**：用户通过 OAuth 或手动输入添加 Account，关联到 Provider
3. **请求路由**：`LoadBalancer` 根据模型名匹配 Provider，选择可用 Account
4. **请求转发**：`RequestForwarder` 根据 Provider 类型分发到对应 Adapter

---

## 3. 逐服务商深度分析

### 3.1 DeepSeek

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `deepseek` |
| **官网** | https://chat.deepseek.com |
| **API 端点** | `https://chat.deepseek.com/api` |
| **Chat 路径** | `/v0/chat/completion` |
| **认证方式** | `userToken` |
| **Token 来源** | Local Storage → `userToken` |

#### 支持的模型

| 显示名 | 实际模型 ID | 特殊能力 |
|--------|------------|---------|
| `DeepSeek-V3.2` | `deepseek-chat` | 基础对话 |
| `DeepSeek-Search` | `deepseek-chat` | 联网搜索 |
| `DeepSeek-R1` | `deepseek-chat` | 深度思考（推理） |
| `DeepSeek-R1-Search` | `deepseek-chat` | 深度思考 + 联网搜索 |

> **注意**：所有模型名最终映射到同一个 `deepseek-chat`，通过请求参数区分模式。

#### 认证流程

```
用户浏览器 → 登录 chat.deepseek.com → F12 → Application → Local Storage → userToken
                                                                          ↓
                                                            Chat2API 手动输入 / 应用内登录
```

**Token 生命周期**：
- 通过 `/v0/users/current` 验证有效性
- Token 过期后需重新获取
- 支持应用内浏览器自动提取（`inAppLogin`）

**Token 刷新机制**：
```typescript
// OAuth Adapter 中的 loginWithToken
async loginWithToken(providerId: string, token: string): Promise<OAuthResult> {
  const validation = await this.validateToken({ token })
  // 验证通过后返回 credentials
}
```

#### 协议详解

**请求格式**：
```json
{
  "chat_session_id": "session-uuid",
  "prompt": "<｜User｜>你好<｜Assistant｜>",
  "ref_file_ids": [],
  "search_enabled": true,
  "thinking_enabled": false,
  "model_type": "default"
}
```

**关键特性**：
1. **PoW 挑战（Proof of Work）**：每次请求前需计算 SHA3 哈希答案
   - 调用 `/v0/chat/create_pow_challenge` 获取挑战参数
   - 使用 WASM 模块（`sha3_wasm_bg.7b9ca65ddd.wasm`）计算答案
   - 答案通过 `X-Ds-Pow-Response` Header 传递
2. **消息格式**：使用特殊标记分隔角色
   ```
   <｜User｜>用户消息<｜Assistant｜>助手回复<｜end of sentence｜>
   ```
3. **Session 管理**：每次对话创建独立 session，支持删除
4. **Cookie 伪装**：每次请求生成随机 Cookie，模拟浏览器行为

**流式响应**：DeepSeek 始终返回流式响应，即使请求 `stream: false`，Adapter 内部收集后转换为非流式。

#### 代码架构

```
src/main/providers/builtin/deepseek.ts     ← Provider 配置
src/main/oauth/adapters/deepseek.ts         ← OAuth 适配器
src/main/proxy/adapters/deepseek.ts         ← 请求转发适配器
src/main/proxy/adapters/deepseek-stream.ts  ← 流式响应处理器
src/main/lib/challenge.ts                   ← PoW 挑战计算（WASM）
```

---

### 3.2 GLM（智谱清言）

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `glm` |
| **官网** | https://chatglm.cn |
| **API 端点** | `https://chatglm.cn/api` |
| **Chat 路径** | `/chatglm/backend-api/assistant/stream` |
| **认证方式** | `refresh_token` |
| **Token 来源** | Local Storage → `chatglm_refresh_token` |

#### 支持的模型

| 显示名 | 实际模型 ID |
|--------|------------|
| `GLM-5` | `glm-5` |

#### 认证流程

```
refresh_token → POST /user-api/user/refresh → { access_token, refresh_token }
                     ↓
              签名验证（X-Sign, X-Timestamp, X-Nonce）
```

**签名算法**：
```typescript
function generateSign() {
  const timestamp = Date.now().toString()
  const nonce = uuid()
  // 时间戳变形：去掉倒数第二位，替换为各位数字之和的个位
  const modifiedTimestamp = timestamp.substring(0, t-2) + (sum % 10) + timestamp.substring(t-1)
  const sign = md5(`${modifiedTimestamp}-${nonce}-${SIGN_SECRET}`)
  return { timestamp: modifiedTimestamp, nonce, sign }
}
```

**Token 刷新**：
- access_token 有效期 3600 秒
- refresh_token 可自动更新，新 token 会回写到账户凭证
- 签名密钥：`8a1317a7468aa3ad86e997d08f3f31cb`

#### 协议详解

**请求格式**：
```json
{
  "assistant_id": "65940acff94777010aa6b796",
  "conversation_id": "",
  "chat_type": "user_chat",
  "messages": [{ "role": "user", "content": [...] }],
  "meta_data": {
    "chat_mode": "zero",        // 深度思考模式
    "is_networking": true,       // 联网搜索
    "platform": "pc"
  }
}
```

**特殊能力**：
| 能力 | 触发方式 |
|------|---------|
| 深度思考 | `chat_mode: "zero"` 或模型名含 `think`/`zero` |
| 联网搜索 | `is_networking: true` 或 Header `X-Web-Search: true` |
| 深度研究 | `chat_mode: "deep_research"` 或 Header `X-Deep-Research: true` |
| 文件上传 | 支持图片和文件，通过 `file_upload` API 上传后引用 |
| 代码执行 | 支持 Python 代码块和执行结果 |

**签名验证**：每个请求需携带 `X-Sign`、`X-Timestamp`、`X-Nonce` Header。

**流式响应**：SSE 格式，使用 `eventsource-parser` 解析。响应中包含 `parts` 数组，每个 part 有独立的 `content` 和 `meta_data`。

#### 代码架构

```
src/main/providers/builtin/glm.ts          ← Provider 配置
src/main/oauth/adapters/glm.ts             ← OAuth 适配器
src/main/proxy/adapters/glm.ts             ← 请求转发 + 流式处理（约 800 行）
```

---

### 3.3 Kimi（月之暗面）

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `kimi` |
| **官网** | https://www.kimi.com |
| **API 端点** | `https://www.kimi.com` |
| **Chat 路径** | `/apiv2/kimi.gateway.chat.v1.ChatService/Chat` |
| **认证方式** | `jwt` |
| **Token 来源** | Network Tab → Authorization Header / Local Storage |

#### 支持的模型

| 显示名 | 实际模型 ID |
|--------|------------|
| `Kimi-K2.5` | `kimi-k2.5` |

#### 认证流程

**Token 类型检测**：
```typescript
function detectTokenType(token: string): 'jwt' | 'refresh' {
  if (token.startsWith('eyJ') && token.split('.').length === 3) {
    const payload = JSON.parse(Buffer.from(token.split('.')[1], 'base64').toString())
    if (payload.app_id === 'kimi' && payload.typ === 'access') return 'jwt'
  }
  return 'refresh'
}
```

- **JWT Token**：直接使用，从 payload 中提取 `sub` 作为 userId
- **Refresh Token**：通过 `/api/auth/token/refresh` 刷新

#### 协议详解

**gRPC-Web 协议**：Kimi 使用 gRPC-Web 格式通信，非标准 REST API。

**帧格式**：
```
[1 byte flag (0x00)] [4 bytes length (big-endian)] [JSON payload]
```

**请求格式**：
```json
{
  "scenario": "SCENARIO_K2D5",
  "chat_id": "",
  "tools": [{ "type": "TOOL_TYPE_SEARCH", "search": {} }],
  "message": {
    "parent_id": "",
    "role": "user",
    "blocks": [{ "message_id": "", "text": { "content": "..." } }],
    "scenario": "SCENARIO_K2D5"
  },
  "options": { "thinking": true }
}
```

**特殊能力**：
| 能力 | 触发方式 |
|------|---------|
| 思考模式 | `options.thinking: true` 或模型名含 `think`/`r1` |
| 联网搜索 | `tools` 中包含 `TOOL_TYPE_SEARCH` |

**流式响应**：gRPC-Web 帧格式，包含多阶段（multi-stage）：
- `STAGE_NAME_THINKING`：思考阶段，输出 `reasoning_content`
- `answer`：回答阶段，输出 `content`

**消息预处理**：
- URL 自动包装为 `<url>` 标签
- 系统消息提取并前置
- 注入 "Focus on the latest message from user" 引导语

#### 代码架构

```
src/main/providers/builtin/kimi.ts          ← Provider 配置
src/main/oauth/adapters/kimi.ts             ← OAuth 适配器
src/main/proxy/adapters/kimi.ts             ← gRPC-Web 协议实现（约 800 行）
```

---

### 3.4 MiniMax

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `minimax` |
| **官网** | https://www.minimaxi.com |
| **API 端点** | `https://agent.minimaxi.com` |
| **Chat 路径** | `/matrix/api/v1/chat/send_msg` |
| **认证方式** | `jwt`（支持 `realUserID+JWTtoken` 格式） |
| **Token 来源** | Local Storage → `token` |

#### 支持的模型

| 显示名 | 实际模型 ID |
|--------|------------|
| `MiniMax-M2.5` | `MiniMax-M2.5` |
| `MiniMax-M2.7` | `MiniMax-M2.7` |

#### 认证流程

**特殊 Token 格式**：
```
realUserID+JWTtoken  （用 + 号连接）
```

**JWT 解析**：
```typescript
function parseJWTUserID(jwtToken: string): string {
  const payload = JSON.parse(Buffer.from(jwtToken.split('.')[1], 'base64').toString())
  return payload.user?.id || ''
}
```

**设备注册**：
```
POST /v1/api/user/device/register
→ { device_id, user_id, uuid }
```

**Token 验证**：
```
GET /v1/api/user/info
→ { user_id, name, credits }
```

#### 协议详解

**请求流程**：
1. 获取 Chat ID（创建或获取最近的对话）
2. 构建请求体
3. 通过 HTTP/2 流式发送

**请求格式**：
```json
{
  "messages": [{ "role": "user", "content": "..." }],
  "model": "MiniMax-M2.5",
  "chat_id": 12345,
  "device_info": { ... }
}
```

**特殊能力**：
- MCP 多智能体协作
- 文件上传（图片、文档）
- 联网搜索

**流式响应**：使用 HTTP/2（`http2` 模块）+ SSE，非标准 axios 流。

**信用额度检查**：每次请求前检查用户信用额度，不足时拒绝。

#### 代码架构

```
src/main/providers/builtin/minimax.ts       ← Provider 配置（约 60 行）
src/main/oauth/adapters/minimax.ts          ← OAuth 适配器
src/main/proxy/adapters/minimax.ts          ← HTTP/2 协议实现（约 1400 行）
```

---

### 3.5 Mimo（小米）

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `mimo` |
| **官网** | https://aistudio.xiaomimimo.com |
| **API 端点** | `https://aistudio.xiaomimimo.com` |
| **Chat 路径** | `/open-apis/bot/chat` |
| **认证方式** | `cookie`（三件套） |
| **Token 来源** | Cookies → `serviceToken` + `userId` + `xiaomichatbot_ph` |

#### 支持的模型

| 显示名 | 实际模型 ID |
|--------|------------|
| `mimo-v2-pro` | `mimo-v2-pro` |
| `mimo-v2-flash-studio` | `mimo-v2-flash-studio` |
| `mimo-v2-omni` | `mimo-v2-omni` |

#### 认证流程

**三件套认证**：
```
serviceToken  ← Cookie: serviceToken
userId        ← Cookie: userId
phToken       ← Cookie: xiaomichatbot_ph
```

**验证方式**：直接检查三个 token 是否存在，无远程验证。

**应用内登录**：支持通过 `startInAppLogin` 打开应用内浏览器，自动拦截 Cookie。

#### 协议详解

**请求格式**：
```json
{
  "msgId": "uuid",
  "conversationId": "uuid",
  "query": "用户消息",
  "isEditedQuery": false,
  "modelConfig": {
    "enableThinking": false,
    "webSearchStatus": "disabled",
    "model": "mimo-v2-pro",
    "temperature": 0.8,
    "topP": 0.95
  },
  "multiMedias": []
}
```

**Cookie 传递**：
```
Cookie: serviceToken=xxx; userId=xxx; xiaomichatbot_ph=xxx
```

同时 `phToken` 也作为 URL 参数传递：
```
/open-apis/bot/chat?xiaomichatbot_ph=xxx
```

**思考模式**：模型名含 `think` 或 `r1` 时自动启用。

**流式响应**：标准 SSE 格式：
```
event: message
data: {"content": "部分文本"}

event: usage
data: {"promptTokens": 100, "completionTokens": 50}

event: dialogId
data: "对话ID"
```

**思考标签处理**：
- `<think>...</think>`：标准思考标签
- `<think>...</think>gt;`：变体标签
- 支持 `passthrough`（透传）、`strip`（剥离）、`separate`（分离为 `reasoning_content`）三种模式

**引用清理**：自动移除 `(citation:N)` 格式的引用标记。

**工具调用**：支持原生 `<tool_callgt;` 格式和 `<function_calls>` 格式。

#### 代码架构

```
src/main/providers/builtin/mimo.ts          ← Provider 配置
src/main/oauth/adapters/mimo.ts             ← Cookie 认证适配器
src/main/proxy/adapters/mimo.ts             ← SSE 协议实现（约 900 行）
```

---

### 3.6 Perplexity

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `perplexity` |
| **官网** | https://www.perplexity.ai |
| **API 端点** | `https://www.perplexity.ai` |
| **Chat 路径** | `/rest/sse/perplexity_ask` |
| **认证方式** | `cookie` |
| **Token 来源** | Cookie → `__Secure-next-auth.session-token` |

#### 支持的模型

| 显示名 | 实际模型 ID | 说明 |
|--------|------------|------|
| `Auto` | `auto` | 自动选择 |
| `Turbo` | `turbo` | 快速模式 |
| `PPLX-Pro` | `pplx_pro` | Perplexity 自研 |
| `GPT-5` | `gpt5` | OpenAI GPT-5 |
| `Gemini-2.5-Pro` | `gemini25pro` | Google Gemini |
| `Claude-Sonnet-4` | `claude4sonnet` | Anthropic Claude |
| `Claude-Opus-4` | `claude4opus` | Anthropic Claude |
| `Nemotron` | `nemotron` | NVIDIA Nemotron |

> **特色**：Perplexity 是唯一支持**多模型路由**的服务商，可通过它间接使用 GPT-5、Gemini、Claude 等模型。

#### 认证流程

**Session Token 获取**：
```
浏览器 DevTools → Application → Cookies → __Secure-next-auth.session-token
```

**应用内登录**：支持 `startInAppLogin` 自动提取 Cookie。

**特殊处理**：使用 Electron 的 `net` API 而非 axios，以绕过 Cloudflare 保护。

#### 协议详解

**请求格式**：
```json
{
  "query": "用户消息",
  "source": "default",
  "model": "turbo",
  "timezone": "Asia/Shanghai",
  "search_focus": "internet",
  "frontend_uuid": "uuid",
  "frontend_context_uuid": "uuid",
  "read_write_token": ""
}
```

**消息转换**：将 OpenAI messages 数组合并为单个 query 字符串：
```
[system]: 系统提示
---
[User]: 用户消息1
[Assistant]: 助手回复1
[User]: 用户消息2
```

**Session 管理**：
- 每次请求创建新的 `backend_uuid`、`frontend_uuid` 等
- 支持 `thread_url_slug` 进行对话关联
- 使用 `read_write_token` 维持会话状态

**流式响应**：标准 SSE，使用 Electron `net` API 绕过 CORS/Cloudflare。

#### 代码架构

```
src/main/providers/builtin/perplexity.ts    ← Provider 配置
src/main/oauth/adapters/perplexity.ts       ← OAuth 适配器
src/main/proxy/adapters/perplexity.ts       ← Electron net 实现
src/main/proxy/adapters/perplexity-stream.ts ← 流式处理器
```

---

### 3.7 Qwen（通义千问国内版）

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `qwen` |
| **官网** | https://www.qianwen.com |
| **API 端点** | `https://chat2.qianwen.com` |
| **Chat 路径** | `/api/v2/chat` |
| **认证方式** | `tongyi_sso_ticket` |
| **Token 来源** | Cookie → `tongyi_sso_ticket` |

#### 支持的模型

| 显示名 | 实际模型 ID |
|--------|------------|
| `Qwen3` | `tongyi-qwen3-max-model-agent` |
| `Qwen3-Max` | `tongyi-qwen3-max-model-agent` |
| `Qwen3-Max-Thinking` | `tongyi-qwen3-max-thinking-agent` |
| `Qwen3-Plus` | `tongyi-qwen-plus-agent` |
| `Qwen3.5-Plus` | `Qwen3.5-Plus` |
| `Qwen3-Flash` | `qwen3-flash` |
| `Qwen3-Coder` | `qwen3-coder-plus` |

#### 认证流程

**SSO Ticket 获取**：
```
浏览器 → www.qianwen.com → 登录 → F12 → Application → Cookies → tongyi_sso_ticket
```

**Ticket 验证**：通过创建对话接口间接验证。

#### 协议详解

**请求格式**（简化）：
```json
{
  "model": "tongyi-qwen3-max-model-agent",
  "messages": [{ "role": "user", "content": "..." }],
  "session_id": "uuid",
  "chat_id": "uuid"
}
```

**流式响应**：SSE 格式，支持 zstd 压缩（需特殊解压）。

**特殊能力**：
| 能力 | 触发方式 |
|------|---------|
| 深度思考 | 模型名含 `thinking` 或 `think` |
| 联网搜索 | Header `X-Web-Search: true` |

#### 代码架构

```
src/main/providers/builtin/qwen.ts          ← Provider 配置
src/main/oauth/adapters/qwen.ts             ← OAuth 适配器
src/main/proxy/adapters/qwen.ts             ← 协议实现（约 1000 行）
```

---

### 3.8 Qwen AI（国际版）

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `qwen-ai` |
| **官网** | https://chat.qwen.ai |
| **API 端点** | `https://chat.qwen.ai` |
| **Chat 路径** | `/api/v2/chat/completions` |
| **认证方式** | `jwt` |
| **Token 来源** | Local Storage → `token` |

#### 支持的模型（17 个）

| 显示名 | 实际模型 ID |
|--------|------------|
| `Qwen3.6-Plus` | `qwen3.6-plus` |
| `Qwen3.5-Plus` | `qwen3.5-plus` |
| `Qwen3.5-Omni-Plus` | `qwen3.5-omni-plus` |
| `Qwen3.5-Flash` | `qwen3.5-flash` |
| `Qwen3.5-Max-Preview` | `qwen3.5-max-2026-03-08` |
| `Qwen3.6-Plus-Preview` | `qwen3.6-plus-preview` |
| `Qwen3.5-397B-A17B` | `qwen3.5-397b-a17b` |
| `Qwen3.5-122B-A10B` | `qwen3.5-122b-a10b` |
| `Qwen3.5-Omni-Flash` | `qwen3.5-omni-flash` |
| `Qwen3.5-27B` | `qwen3.5-27b` |
| `Qwen3.5-35B-A3B` | `qwen3.5-35b-a3b` |
| `Qwen3-Max` | `qwen3-max-2026-01-23` |
| `Qwen3-235B-A22B-2507` | `qwen-plus-2025-07-28` |
| `Qwen3-Coder` | `qwen3-coder-plus` |
| `Qwen3-VL-235B-A22B` | `qwen3-vl-plus` |
| `Qwen3-Omni-Flash` | `qwen3-omni-flash-2025-12-01` |
| `Qwen2.5-Max` | `qwen-max-latest` |

> **特色**：模型数量最多（17 个），包含视觉（VL）、全模态（Omni）、代码（Coder）等变体。

#### 认证流程

**JWT Token**：
```
浏览器 → chat.qwen.ai → 登录 → F12 → Application → Local Storage → token
```

**可选 Cookie**：为提高兼容性，建议同时提供完整 Cookie 字符串。

**特殊 Header**：
```typescript
'bx-v': '2.5.36',
'bx-umidtoken': '...',
'bx-ua': '...',  // 风控相关
```

#### 协议详解

**请求格式**：
```json
{
  "model": "qwen3.5-plus",
  "messages": [...],
  "stream": true,
  "chat_id": "uuid"
}
```

**模型别名系统**：
```typescript
const MODEL_ALIASES = {
  'qwen': 'qwen3-max',
  'qwen3': 'qwen3-max',
  'qwen3.5': 'qwen3.5-plus',
  'qwen3-coder': 'qwen3-coder-plus',
  'qwen3-vl': 'qwen3-vl-235b-a22b',
  'qwen3-omni': 'qwen3-omni-flash',
  'qwen2.5': 'qwen2.5-max',
}
```

**思考模式后缀**：
- `-thinking`：强制开启思考模式
- `-fast`：强制关闭思考模式

**动态模型列表**：支持通过 `modelsApiEndpoint` (`/api/models`) 动态获取可用模型。

#### 代码架构

```
src/main/providers/builtin/qwen-ai.ts       ← Provider 配置（模型最多）
src/main/oauth/adapters/qwen-ai.ts          ← OAuth 适配器
src/main/proxy/adapters/qwen-ai.ts          ← 协议实现（约 750 行）
```

---

### 3.9 Z.ai（智谱海外）

#### 基本信息

| 字段 | 值 |
|------|-----|
| **ID** | `zai` |
| **官网** | https://chat.z.ai |
| **API 端点** | `https://chat.z.ai/api` |
| **Chat 路径** | `/v2/chat/completions` |
| **认证方式** | `jwt` |
| **Token 来源** | Cookie → JWT Token（以 `eyJ` 开头） |

#### 支持的模型

| 显示名 | 实际模型 ID |
|--------|------------|
| `GLM-5-Turbo` | `GLM-5-Turbo` |
| `glm-5` | `glm-5` |
| `glm-4.7` | `glm-4.7` |
| `glm-4.6v` | `glm-4.6v` |
| `glm-4.6` | `glm-4.6v` |
| `glm-4.5v` | `glm-4.5v` |
| `glm-4.5-air` | `glm-4.5-air` |

> **特色**：Z.ai 是智谱的海外版，提供 GLM 系列模型的免费访问。

#### 认证流程

**JWT Token**：
```
浏览器 → chat.z.ai → 登录 → F12 → Application → Cookie → JWT Token
```

**Token 验证**：`GET /api/v1/users/user/settings`

#### 协议详解

**请求格式**：
```json
{
  "model": "glm-5",
  "messages": [...],
  "stream": true,
  "chat_id": "uuid",
  "parent_id": "message-id"
}
```

**消息树结构**：Z.ai 使用消息树（parent_id 链）而非简单的消息数组。

**搜索引用清理**：自动移除 `【turn1search1】` 格式的引用标记。

**特殊能力**：
| 能力 | 触发方式 |
|------|---------|
| 联网搜索 | `web_search: true` 或 Header `X-Web-Search: true` |
| 深度思考 | `reasoning_effort` 参数 |

#### 代码架构

```
src/main/providers/builtin/zai.ts           ← Provider 配置
src/main/oauth/adapters/zai.ts              ← OAuth 适配器
src/main/proxy/adapters/zai.ts              ← 协议实现（约 950 行）
```

---

## 4. 认证方式对比矩阵

| 服务商 | 认证类型 | 凭证数量 | 凭证来源 | 自动刷新 | 应用内登录 | Token 验证端点 |
|--------|---------|---------|---------|---------|-----------|--------------|
| DeepSeek | `userToken` | 1 | LocalStorage | ❌ | ✅ | `/v0/users/current` |
| GLM | `refresh_token` | 1 | LocalStorage | ✅ | ✅ | `/user-api/user/refresh` |
| Kimi | `jwt` | 1 | Network/LocalStorage | ❌ | ✅ | `/api/auth/token/refresh` |
| MiniMax | `jwt` | 1-2 | LocalStorage | ❌ | ✅ | `/v1/api/user/info` |
| Mimo | `cookie` | 3 | Cookies | ❌ | ✅ | 直接检查 |
| Perplexity | `cookie` | 1 | Cookies | ❌ | ✅ | 隐式验证 |
| Qwen | `tongyi_sso_ticket` | 1 | Cookies | ❌ | ✅ | 隐式验证 |
| Qwen AI | `jwt` | 1-2 | LocalStorage | ❌ | ✅ | 隐式验证 |
| Z.ai | `jwt` | 1 | Cookie | ❌ | ✅ | `/api/v1/users/user/settings` |

### 认证类型说明

| 类型 | 说明 | 代表服务商 |
|------|------|-----------|
| `userToken` | 用户 Token，直接从 LocalStorage 获取 | DeepSeek |
| `refresh_token` | 刷新令牌，可自动换取 access_token | GLM |
| `jwt` | JWT 令牌，部分可直接使用，部分需刷新 | Kimi, MiniMax, Qwen AI, Z.ai |
| `cookie` | Cookie 认证，需多个 Cookie 值组合 | Mimo, Perplexity |
| `tongyi_sso_ticket` | 通义 SSO 专用票据 | Qwen |

---

## 5. 协议差异对比

| 服务商 | 传输协议 | 消息格式 | 请求 Content-Type | 响应格式 |
|--------|---------|---------|------------------|---------|
| DeepSeek | HTTPS | 自定义 prompt 拼接 | `application/json` | 流式 JSON |
| GLM | HTTPS | content 数组 + 文件引用 | `application/json` | SSE |
| Kimi | **gRPC-Web** | blocks 结构 | `application/connect+json` | gRPC-Web 帧 |
| MiniMax | **HTTP/2** | 标准 messages | `application/json` | SSE over HTTP/2 |
| Mimo | HTTPS | query 字符串 | `application/json` | SSE |
| Perplexity | Electron net | query 字符串 | `application/json` | SSE |
| Qwen | HTTPS | 标准 messages | `application/json` | SSE (zstd) |
| Qwen AI | HTTPS | 标准 messages | `application/json` | SSE |
| Z.ai | HTTPS | 标准 messages + 消息树 | `application/json` | SSE |

---

## 6. 特殊能力支持矩阵

| 服务商 | 联网搜索 | 深度思考 | 深度研究 | 文件上传 | 图片生成 | 代码执行 | 多模型路由 |
|--------|---------|---------|---------|---------|---------|---------|-----------|
| DeepSeek | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| GLM | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Kimi | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| MiniMax | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| Mimo | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Perplexity | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Qwen | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Qwen AI | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Z.ai | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

### 能力触发方式汇总

| 能力 | 触发参数 | Header 触发 | 模型名触发 |
|------|---------|------------|-----------|
| 联网搜索 | `web_search: true` | `X-Web-Search: true` | 模型名含 `search` |
| 深度思考 | `reasoning_effort: "medium"` | `X-Reasoning-Effort: medium` | 模型名含 `think`/`r1`/`thinking` |
| 深度研究 | `deep_research: true` | `X-Deep-Research: true` | 模型名含 `deepresearch` |

---

## 7. 工具调用（Function Calling）实现差异

### 7.1 实现策略

Chat2API 采用**双层策略**实现工具调用：

```
┌─────────────────────────────────────────────────┐
│  第一层：Prompt Injection Service                │
│  为不支持原生 function calling 的模型注入工具提示词  │
│  (DeepSeek, GLM, Kimi, Mimo, Perplexity, Z.ai)  │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  第二层：Tool Call Parsing                       │
│  从模型输出中解析工具调用，转换为 OpenAI 格式        │
└─────────────────────────────────────────────────┘
```

### 7.2 提示词注入格式

| 格式 | 示例 | 使用服务商 |
|------|------|-----------|
| **Bracket** | `[function_calls][call:tool_name]{"arg":"val"}[/call][/function_calls]` | DeepSeek, GLM, Kimi, 默认 |
| **XML** | `<tool_use><name>tool_name</name><arguments>{"arg":"val"}</arguments></tool_use>` | Perplexity |
| **MCP** | `<tool><name>...</name><description>...</description><arguments>...</arguments></tool>` | Cherry Studio 等客户端 |

### 7.3 工具调用解析器

```typescript
// src/main/proxy/utils/toolParser/index.ts
// 支持多种格式的工具调用解析：
// - Bracket 格式：[function_calls][call:name]args[/call]
// - XML 格式：<tool_use><name>...</name><arguments>...</arguments></tool_use>
// - Anthropic 格式：<invoke name="...">...</invoke>
// - JSON 格式：{"name": "...", "arguments": {...}}
```

### 7.4 各服务商工具调用处理

| 服务商 | 原生支持 | 注入策略 | 解析格式 | 特殊处理 |
|--------|---------|---------|---------|---------|
| DeepSeek | ❌ | 注入 bracket | bracket | PoW 挑战 + tool_call/tool_response 消息转换 |
| GLM | ❌ | 注入 bracket | bracket | GLM STRICT RULES 强化提示 |
| Kimi | ❌ | 注入 bracket | bracket | URL 包装为 `<url>` 标签 |
| MiniMax | ❌ | 注入 bracket | bracket | HTTP/2 流式工具调用拦截 |
| Mimo | ❌ | 自动检测 | XML/bracket | 原生 `<tool_callgt;` 格式支持 |
| Perplexity | ❌ | 注入 XML | XML | 专用 Perplexity 提示词 |
| Qwen | ❌ | 注入 bracket | bracket/JSON | zstd 压缩解码 |
| Qwen AI | ❌ | 自动检测 | XML | 原生工具调用支持 |
| Z.ai | ❌ | 注入 bracket | bracket | 消息树结构 |

---

## 8. 流式响应协议对比

### 8.1 DeepSeek

```
data: {"id":"...","choices":[{"delta":{"content":"你"},"finish_reason":null}]}
data: {"id":"...","choices":[{"delta":{"content":"好"},"finish_reason":null}]}
data: {"id":"...","choices":[{"delta":{},"finish_reason":"stop"}]}
data: [DONE]
```

### 8.2 GLM

```
event: message
data: {"parts":[{"content":[{"type":"text","text":"你好"}]}],"status":"finish","conversation_id":"..."}
```

### 8.3 Kimi（gRPC-Web）

```
[1 byte flag] [4 bytes length] [JSON payload]
// payload 示例：
{"op":"set","mask":"block.text","block":{"text":{"content":"你好"}}}
{"chat":{"id":"..."},"done":true}
```

### 8.4 MiniMax（HTTP/2 SSE）

```
data: {"choices":[{"delta":{"content":"你好"}}]}
data: [DONE]
```

### 8.5 Mimo

```
event: message
data: {"content":"你好"}

event: usage
data: {"promptTokens":100,"completionTokens":50}

event: dialogId
data: "dialog-id"
```

### 8.6 Perplexity

```
data: {"text":"你好","citations":["url1","url2"]}
data: [DONE]
```

### 8.7 Qwen / Qwen AI / Z.ai

标准 SSE 格式，与 OpenAI 类似：
```
data: {"choices":[{"delta":{"content":"你好"}}]}
data: [DONE]
```

---

## 9. 会话生命周期管理

### 9.1 会话创建时机

| 服务商 | 会话创建 | 会话 ID 来源 | 会话删除 |
|--------|---------|-------------|---------|
| DeepSeek | 每次请求前创建 | `/v0/chat_session/create` | ✅ 支持 |
| GLM | 无持久会话 | 每次请求生成 | ✅ 支持（conversation_id） |
| Kimi | 无持久会话 | 从响应中提取 `chat.id` | ✅ 支持 |
| MiniMax | 复用或创建 | Chat ID 管理 | ✅ 支持 |
| Mimo | 每次生成 | UUID 生成 | ✅ 支持（批量删除） |
| Perplexity | 每次生成 | UUID 生成 | ✅ 支持 |
| Qwen | 每次生成 | UUID 生成 | ✅ 支持 |
| Qwen AI | 每次生成 | UUID 生成 | ✅ 支持 |
| Z.ai | 每次生成 | UUID 生成 | ✅ 支持 |

### 9.2 会话清理策略

```typescript
// SessionManager 配置
interface SessionConfig {
  sessionTimeout: number           // 会话超时（分钟），默认 30
  maxMessagesPerSession: number    // 每会话最大消息数，默认 50
  deleteAfterTimeout: boolean      // 超时后是否删除
  maxSessionsPerAccount: number    // 每账户最大会话数，默认 3
}
```

**清理调度**：每 60 秒执行一次 `cleanExpiredSessions()`。

---

## 10. 扩展新服务商指南

### 10.1 需要创建的文件

```
src/main/providers/builtin/{provider}.ts      ← Provider 配置
src/main/oauth/adapters/{provider}.ts          ← OAuth 适配器
src/main/proxy/adapters/{provider}.ts          ← 请求转发适配器
src/main/proxy/adapters/{provider}-stream.ts   ← 流式处理器（可选）
src/renderer/src/assets/providers/{provider}.svg ← 图标
```

### 10.2 实现步骤

1. **Provider 配置**：
   ```typescript
   export const newProviderConfig: BuiltinProviderConfig = {
     id: 'new-provider',
     name: 'New Provider',
     type: 'builtin',
     authType: 'jwt',  // 选择合适的认证类型
     apiEndpoint: 'https://api.new-provider.com',
     chatPath: '/v1/chat',
     headers: { ... },
     enabled: true,
     supportedModels: ['Model-1', 'Model-2'],
     modelMappings: { 'Model-1': 'model-1' },
     credentialFields: [ ... ],
   }
   ```

2. **OAuth Adapter**：继承 `BaseOAuthAdapter`，实现 `validateToken` 和 `loginWithToken`。

3. **Proxy Adapter**：
   - 实现 `chatCompletion` 方法
   - 实现消息格式转换（OpenAI → 原生格式）
   - 实现流式响应解析（原生 → OpenAI SSE）
   - 实现会话管理（创建/删除）

4. **注册**：在 `src/main/providers/builtin/index.ts` 和 `src/main/oauth/adapters/index.ts` 中注册。

5. **Forwarder 路由**：在 `src/main/proxy/forwarder.ts` 的 `doForward` 方法中添加判断分支。

### 10.3 关键注意事项

- **浏览器伪装**：所有请求需携带完整的浏览器 Header（User-Agent、Sec-Ch-Ua 等）
- **反爬虫应对**：部分服务商（Perplexity）需要使用 Electron `net` API 绕过 Cloudflare
- **签名验证**：部分服务商（GLM、DeepSeek）需要计算请求签名或 PoW
- **流式协议适配**：不同服务商的 SSE 格式差异较大，需逐个适配
- **工具调用桥接**：需实现 prompt 注入和输出解析

---

## 附录：文件索引

| 文件路径 | 行数（约） | 说明 |
|---------|-----------|------|
| `src/main/providers/builtin/deepseek.ts` | 60 | DeepSeek 配置 |
| `src/main/providers/builtin/glm.ts` | 50 | GLM 配置 |
| `src/main/providers/builtin/kimi.ts` | 50 | Kimi 配置 |
| `src/main/providers/builtin/minimax.ts` | 60 | MiniMax 配置 |
| `src/main/providers/builtin/mimo.ts` | 60 | Mimo 配置 |
| `src/main/providers/builtin/perplexity.ts` | 55 | Perplexity 配置 |
| `src/main/providers/builtin/qwen.ts` | 50 | Qwen 配置 |
| `src/main/providers/builtin/qwen-ai.ts` | 80 | Qwen AI 配置（模型最多） |
| `src/main/providers/builtin/zai.ts` | 55 | Z.ai 配置 |
| `src/main/proxy/adapters/deepseek.ts` | 400 | DeepSeek 协议实现 |
| `src/main/proxy/adapters/glm.ts` | 800 | GLM 协议实现 |
| `src/main/proxy/adapters/kimi.ts` | 800 | Kimi gRPC-Web 实现 |
| `src/main/proxy/adapters/minimax.ts` | 1400 | MiniMax HTTP/2 实现 |
| `src/main/proxy/adapters/mimo.ts` | 900 | Mimo SSE 实现 |
| `src/main/proxy/adapters/perplexity.ts` | 600 | Perplexity Electron net 实现 |
| `src/main/proxy/adapters/qwen.ts` | 1000 | Qwen 协议实现 |
| `src/main/proxy/adapters/qwen-ai.ts` | 750 | Qwen AI 协议实现 |
| `src/main/proxy/adapters/zai.ts` | 950 | Z.ai 协议实现 |
| `src/main/lib/challenge.ts` | 150 | DeepSeek PoW WASM 模块 |
| `src/main/oauth/adapters/*.ts` | 各 100-200 | OAuth 适配器 |
