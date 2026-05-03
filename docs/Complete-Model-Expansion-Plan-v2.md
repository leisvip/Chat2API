# Chat2API 全服务商模型扩展完整方案（修订版）

> **编写日期**：2026-05-04（修订）
> **基于版本**：Chat2API v1.2.0
> **数据来源**：各服务商官网、API 文档、新闻报道（截至 2026-04-28）

---

## 目录

1. [重大遗漏自查](#1-重大遗漏自查)
2. [各服务商最新模型盘点（修订版）](#2-各服务商最新模型盘点修订版)
3. [差距分析与优先级](#3-差距分析与优先级)
4. [逐服务商扩展方案（含完整 diff）](#4-逐服务商扩展方案含完整-diff)
5. [扩展路径说明](#5-扩展路径说明)
6. [测试验证](#6-测试验证)

---

## 1. 重大遗漏自查

上一版方案存在以下**严重遗漏**，本次修订全部修正：

| 遗漏项 | 严重程度 | 说明 |
|--------|---------|------|
| **DeepSeek V4** | 🔴 致命 | V4 预览版已于 2026-04-24 上线，API ID: `deepseek-v4-pro`/`deepseek-v4-flash` |
| **Kimi K2.6** | 🔴 严重 | K2.6 已于 2026-04-20 发布并开源，最强代码模型 |
| **MiniMax M2.7** | 🟡 已有但需确认 | 2026-04-12 开源，Chat2API 配置中已有但需验证 |
| **MiniMax M3** | 🟡 即将发布 | 大摩报告确认 M3 升级在即 |
| **MiMo-V2.5 系列** | 🔴 严重 | 2026-04-23 公测，2026-04-28 开源 |
| **GLM-5.1** | 🔴 严重 | 2026-04-08 发布，Code Arena 榜首 |
| **Qwen3.6-Plus** | 🟡 已有但需确认 | Chat2API 已有，但 Qwen3.6-27B 缺失 |

---

## 2. 各服务商最新模型盘点（修订版）

### 2.1 DeepSeek

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| DeepSeek-V3.2 | `deepseek-chat` | 2025 | 上一代旗舰 | ✅ 已有 |
| DeepSeek-R1 | `deepseek-reasoner` | 2025-01 | 推理模型 | ✅ 已有 |
| **DeepSeek-V4-Pro** | `deepseek-v4-pro` | **2026-04-24** | **1.6T 参数 MoE，百万上下文，Agent 能力国内领先** | ❌ **缺失** |
| **DeepSeek-V4-Flash** | `deepseek-v4-flash` | **2026-04-24** | **V4 轻量版，速度更快** | ❌ **缺失** |

> **关键信息**：V4 于 2026-04-24 上线，同步开源（MIT 许可），已上线官网 chat.deepseek.com 和官方 App。V4 拥有百万字超长上下文，在 Agent 能力、世界知识和推理性能上均实现国内与开源领先。不再有 R2（V4 继承混合架构，定位旗舰编程模型）。

### 2.2 GLM（智谱清言）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| GLM-5 | `glm-5` | 2026-02 | 旗舰，744B 参数 | ✅ 已有 |
| **GLM-5.1** | `glm-5.1` | **2026-04-08** | **Code Arena 榜首，编程能力全球领先** | ❌ **缺失** |
| **GLM-4.7** | `glm-4.7` | 2025-12 | Coding 强化，工具协同 | ❌ **缺失** |
| GLM-4.6 | `glm-4.6` | 2025-09 | 代码能力 +27% | ❌ 缺失 |

### 2.3 Kimi（月之暗面）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| Kimi K2.5 | `kimi-k2.5` | 2026-01 | 原生多模态，视觉+代码+Agent | ✅ 已有 |
| **Kimi K2.6** | `kimi-k2.6` | **2026-04-20** | **最强代码模型，长程编码 13 小时，4000+ 行代码** | ❌ **缺失** |
| kimi-latest | `kimi-latest` | 2025-02 | 始终指向最新版 | ❌ 缺失 |

### 2.4 MiniMax

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| MiniMax-M2.5 | `MiniMax-M2.5` | 2026-02 | 编程旗舰 | ✅ 已有 |
| MiniMax-M2.7 | `MiniMax-M2.7` | 2026-04-12 | 开源，自我进化，SWE-Pro 56.22% | ✅ 已有 |
| **MiniMax-M3** | `MiniMax-M3` | 即将发布 | 大摩报告确认升级在即 | ⏳ 待发布 |

### 2.5 Mimo（小米）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| mimo-v2-pro | `mimo-v2-pro` | 2025 | 开源性能旗舰 | ✅ 已有 |
| mimo-v2-flash-studio | `mimo-v2-flash-studio` | 2025 | 轻量快速 | ✅ 已有 |
| mimo-v2-omni | `mimo-v2-omni` | 2025 | 全模态理解 | ✅ 已有 |
| **MiMo-V2.5-Pro** | `mimo-v2.5-pro` | **2026-04-23** | **开源性能旗舰，百万上下文，对标 GPT-5.4，MIT 许可** | ❌ **缺失** |
| **MiMo-V2.5** | `mimo-v2.5` | **2026-04-23** | **全模态理解大模型，百万上下文，MIT 许可** | ❌ **缺失** |
| MiMo-V2.5-TTS | - | 2026-04-23 | 语音合成（非对话模型） | ⚠️ 不适用 |
| MiMo-V2.5-ASR | - | 2026-04-23 | 语音识别（非对话模型） | ⚠️ 不适用 |

### 2.6 Perplexity

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| Auto | `auto` | - | 自动选择 | ✅ 已有 |
| Turbo | `turbo` | - | 快速模式 | ✅ 已有 |
| PPLX-Pro | `pplx_pro` | - | Perplexity 自研 | ✅ 已有 |
| GPT-5 | `gpt5` | 2025 | OpenAI GPT-5 | ✅ 已有 |
| Gemini-2.5-Pro | `gemini25pro` | 2025 | Google Gemini | ✅ 已有 |
| Claude-Sonnet-4 | `claude4sonnet` | 2025 | Anthropic Claude | ✅ 已有 |
| Claude-Opus-4 | `claude4opus` | 2025 | Anthropic Claude | ✅ 已有 |
| Nemotron | `nemotron` | 2025 | NVIDIA | ✅ 已有 |
| **Sonar** | `sonar` | 2025-02 | Perplexity 搜索模型，1200 token/s | ❌ **缺失** |
| **Sonar-Pro** | `sonar_pro` | 2025 | 高级搜索 | ❌ **缺失** |

### 2.7 Qwen（通义千问国内版）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| Qwen3 | `tongyi-qwen3-max-model-agent` | 2025 | 旗舰 | ✅ 已有 |
| Qwen3-Max | `tongyi-qwen3-max-model-agent` | 2025 | 旗舰 | ✅ 已有 |
| Qwen3-Max-Thinking | `tongyi-qwen3-max-thinking-agent` | 2025 | 深度思考 | ✅ 已有 |
| Qwen3-Plus | `tongyi-qwen-plus-agent` | 2025 | 均衡版 | ✅ 已有 |
| Qwen3.5-Plus | `Qwen3.5-Plus` | 2026-02 | 旗舰 | ✅ 已有 |
| Qwen3-Flash | `qwen3-flash` | 2025 | 快速版 | ✅ 已有 |
| Qwen3-Coder | `qwen3-coder-plus` | 2025 | 代码专用 | ✅ 已有 |
| **Qwen3.6-Plus** | `qwen3.6-plus` | **2026-04-02** | **新一代旗舰，原生多模态，编程接近全球最强** | ❌ **缺失** |
| **Qwen3.6-27B** | `qwen3.6-27b` | **2026-04-22** | **最新开源，Agent 编程** | ❌ **缺失** |

### 2.8 Qwen AI（国际版）

| 模型 | API Model ID | 发布时间 | Chat2API 状态 |
|------|-------------|---------|--------------|
| Qwen3.6-Plus | `qwen3.6-plus` | 2026-04 | ✅ 已有 |
| Qwen3.5-Plus | `qwen3.5-plus` | 2026-02 | ✅ 已有 |
| 其他 15 个模型 | - | - | ✅ 已有 |
| **Qwen3.6-27B** | `qwen3.6-27b` | **2026-04-22** | ❌ **缺失** |

### 2.9 Z.ai（智谱海外）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| GLM-5-Turbo | `GLM-5-Turbo` | 2026-02 | GLM-5 快速版 | ✅ 已有 |
| glm-5 | `glm-5` | 2026-02 | 旗舰 | ✅ 已有 |
| glm-4.7 | `glm-4.7` | 2025-12 | Coding 强化 | ✅ 已有 |
| glm-4.6v | `glm-4.6v` | 2025-09 | 多模态 | ✅ 已有 |
| glm-4.6 | `glm-4.6v` | 2025-09 | 文本 | ✅ 已有 |
| glm-4.5v | `glm-4.5v` | 2025 | 多模态 | ✅ 已有 |
| glm-4.5-air | `glm-4.5-air` | 2025 | 轻量 | ✅ 已有 |
| **GLM-5.1** | `glm-5.1` | **2026-04-08** | **Code Arena 榜首** | ❌ **缺失** |

---

## 3. 差距分析与优先级

### 3.1 P0 必须新增（2026-04 最新发布，用户强烈需求）

| # | 服务商 | 缺失模型 | API Model ID | 发布日期 | 依据 |
|---|--------|---------|-------------|---------|------|
| 1 | **DeepSeek** | DeepSeek-V4-Pro | `deepseek-v4-pro` | 2026-04-24 | 官网已上线接口文档 |
| 2 | **DeepSeek** | DeepSeek-V4-Flash | `deepseek-v4-flash` | 2026-04-24 | 官网已上线接口文档 |
| 3 | **Mimo** | MiMo-V2.5-Pro | `mimo-v2.5-pro` | 2026-04-23 | 公测+开源（MIT） |
| 4 | **Mimo** | MiMo-V2.5 | `mimo-v2.5` | 2026-04-23 | 公测+开源（MIT） |
| 5 | **GLM** | GLM-5.1 | `glm-5.1` | 2026-04-08 | Code Arena 榜首 |
| 6 | **Z.ai** | GLM-5.1 | `glm-5.1` | 2026-04-08 | 同步上线 |
| 7 | **Kimi** | Kimi K2.6 | `kimi-k2.6` | 2026-04-20 | 最强代码模型 |

### 3.2 P1 建议新增（近期发布，提升竞争力）

| # | 服务商 | 缺失模型 | API Model ID | 发布日期 | 依据 |
|---|--------|---------|-------------|---------|------|
| 8 | **GLM** | GLM-4.7 | `glm-4.7` | 2025-12 | Coding 强化 |
| 9 | **GLM** | GLM-4.6 | `glm-4.6` | 2025-09 | 代码能力 +27% |
| 10 | **Kimi** | kimi-latest | `kimi-latest` | 2025-02 | 自动跟随最新 |
| 11 | **Perplexity** | Sonar | `sonar` | 2025-02 | 搜索模型 |
| 12 | **Perplexity** | Sonar-Pro | `sonar_pro` | 2025 | 高级搜索 |
| 13 | **Qwen** | Qwen3.6-Plus | `qwen3.6-plus` | 2026-04-02 | 新一代旗舰 |
| 14 | **Qwen** | Qwen3.6-27B | `qwen3.6-27b` | 2026-04-22 | 最新开源 |
| 15 | **Qwen AI** | Qwen3.6-27B | `qwen3.6-27b` | 2026-04-22 | 最新开源 |

### 3.3 差距总览（修订版）

```
DeepSeek   ████████████░░░░░░░░  60%  ❌ 缺 DeepSeek-V4-Pro, V4-Flash（最严重！）
GLM        ██████████████░░░░░░  70%  ❌ 缺 GLM-5.1, GLM-4.7, GLM-4.6
Kimi       ██████████████████░░  90%  ❌ 缺 K2.6, kimi-latest
MiniMax    ████████████████████ 100%  ✅ 全部覆盖
Mimo       ████████████░░░░░░░░  60%  ❌ 缺 MiMo-V2.5-Pro, V2.5
Perplexity ██████████████████░░  90%  ⚠️ 缺 Sonar, Sonar-Pro
Qwen       ██████████████████░░  90%  ❌ 缺 Qwen3.6-Plus, Qwen3.6-27B
Qwen AI    ███████████████████░  95%  ⚠️ 缺 Qwen3.6-27B
Z.ai       ██████████████████░░  90%  ❌ 缺 GLM-5.1
```

---

## 4. 逐服务商扩展方案（含完整 diff）

### 4.1 DeepSeek — 添加 V4-Pro 和 V4-Flash（🔴 最高优先级）

**修改文件**：`src/main/providers/builtin/deepseek.ts`

```diff
  supportedModels: [
    'DeepSeek-V3.2',
    'DeepSeek-Search',
    'DeepSeek-R1',
    'DeepSeek-R1-Search',
+   'DeepSeek-V4-Pro',
+   'DeepSeek-V4-Flash',
  ],
  modelMappings: {
    'DeepSeek-V3.2': 'deepseek-chat',
    'DeepSeek-Search': 'deepseek-chat',
    'DeepSeek-R1': 'deepseek-chat',
    'DeepSeek-R1-Search': 'deepseek-chat',
+   'DeepSeek-V4-Pro': 'deepseek-v4-pro',
+   'DeepSeek-V4-Flash': 'deepseek-v4-flash',
  },
```

**Adapter 影响分析**：

DeepSeek V4 使用**全新的 API Model ID**（`deepseek-v4-pro`/`deepseek-v4-flash`），但 Chat2API 的 DeepSeek Adapter 使用的是 Web API（`/api/v0/chat/completion`），不是官方 API。需要确认：

1. **Web API 是否支持 V4**：V4 已上线 chat.deepseek.com，Web API 的 `model_type` 参数可能需要新值
2. **PoW 挑战是否变化**：V4 可能使用新的 PoW 算法
3. **Session 创建是否变化**：V4 可能需要新的 session 类型

**需要额外检查**：
```typescript
// src/main/proxy/adapters/deepseek.ts → chatCompletion()
// 当前 model_type 固定为 'default'，V4 可能需要 'v4' 或其他值
const response = await axios.post(
  `${DEEPSEEK_API_BASE}/v0/chat/completion`,
  {
    chat_session_id: sessionId,
    prompt,
    model_type: 'default',  // ← V4 可能需要修改此处
    // ...
  }
)
```

**建议**：先通过浏览器抓包确认 V4 的请求格式，再决定是否需要修改 Adapter。

---

### 4.2 Mimo — 添加 MiMo-V2.5-Pro 和 MiMo-V2.5

**修改文件**：`src/main/providers/builtin/mimo.ts`

```diff
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

**Adapter 影响**：无。V2.5 系列使用相同的 `/open-apis/bot/chat` 端点，协议一致。

---

### 4.3 GLM — 添加 GLM-5.1、GLM-4.7、GLM-4.6

**修改文件**：`src/main/providers/builtin/glm.ts`

```diff
  supportedModels: [
    'GLM-5',
+   'GLM-5.1',
+   'GLM-4.7',
+   'GLM-4.6',
  ],
  modelMappings: {
    'GLM-5': 'glm-5',
+   'GLM-5.1': 'glm-5.1',
+   'GLM-4.7': 'glm-4.7',
+   'GLM-4.6': 'glm-4.6',
  },
```

**Adapter 影响**：无。GLM Adapter 使用 `assistant_id` 区分模型，新模型通过 modelMappings 映射。

---

### 4.4 Z.ai — 添加 GLM-5.1

**修改文件**：`src/main/providers/builtin/zai.ts`

```diff
  supportedModels: [
    'GLM-5-Turbo',
    'glm-5',
    'glm-4.7',
    'glm-4.6v',
    'glm-4.6',
    'glm-4.5v',
    'glm-4.5-air',
+   'GLM-5.1',
  ],
  modelMappings: {
    'GLM-5-Turbo': 'GLM-5-Turbo',
    'glm-5': 'glm-5',
    'glm-4.7': 'glm-4.7',
    'glm-4.6v': 'glm-4.6v',
    'glm-4.6': 'glm-4.6v',
    'glm-4.5v': 'glm-4.5v',
    'glm-4.5-air': 'glm-4.5-air',
+   'GLM-5.1': 'glm-5.1',
  },
```

---

### 4.5 Kimi — 添加 K2.6 和 kimi-latest

**修改文件**：`src/main/providers/builtin/kimi.ts`

```diff
  supportedModels: [
    'Kimi-K2.5',
+   'Kimi-K2.6',
+   'Kimi-Latest',
  ],
  modelMappings: {
    'Kimi-K2.5': 'kimi-k2.5',
+   'Kimi-K2.6': 'kimi-k2.6',
+   'Kimi-Latest': 'kimi-latest',
  },
```

**Adapter 影响**：无。K2.6 使用相同的 gRPC-Web 协议。

---

### 4.6 Perplexity — 添加 Sonar 和 Sonar-Pro

**修改文件**：`src/main/providers/builtin/perplexity.ts`

```diff
  supportedModels: [
    'Auto',
    'Turbo',
    'PPLX-Pro',
    'GPT-5',
    'Gemini-2.5-Pro',
    'Claude-Sonnet-4',
    'Claude-Opus-4',
    'Nemotron',
+   'Sonar',
+   'Sonar-Pro',
  ],
  modelMappings: {
    'Auto': 'auto',
    'Turbo': 'turbo',
    'PPLX-Pro': 'pplx_pro',
    'GPT-5': 'gpt5',
    'Gemini-2.5-Pro': 'gemini25pro',
    'Claude-Sonnet-4': 'claude4sonnet',
    'Claude-Opus-4': 'claude4opus',
    'Nemotron': 'nemotron',
+   'Sonar': 'sonar',
+   'Sonar-Pro': 'sonar_pro',
  },
```

---

### 4.7 Qwen（国内版）— 添加 Qwen3.6-Plus 和 Qwen3.6-27B

**修改文件**：`src/main/providers/builtin/qwen.ts`

```diff
  supportedModels: [
    'Qwen3',
    'Qwen3-Max',
    'Qwen3-Max-Thinking',
    'Qwen3-Plus',
    'Qwen3.5-Plus',
    'Qwen3-Flash',
    'Qwen3-Coder',
+   'Qwen3.6-Plus',
+   'Qwen3.6-27B',
  ],
  modelMappings: {
    'Qwen3': 'tongyi-qwen3-max-model-agent',
    'Qwen3-Max': 'tongyi-qwen3-max-model-agent',
    'Qwen3-Max-Thinking': 'tongyi-qwen3-max-thinking-agent',
    'Qwen3-Plus': 'tongyi-qwen-plus-agent',
    'Qwen3.5-Plus': 'Qwen3.5-Plus',
    'Qwen3-Flash': 'qwen3-flash',
    'Qwen3-Coder': 'qwen3-coder-plus',
+   'Qwen3.6-Plus': 'qwen3.6-plus',
+   'Qwen3.6-27B': 'qwen3.6-27b',
  },
```

---

### 4.8 Qwen AI（国际版）— 添加 Qwen3.6-27B

**修改文件**：`src/main/providers/builtin/qwen-ai.ts`

```diff
  supportedModels: [
    'Qwen3.6-Plus',
    'Qwen3.5-Plus',
    // ... 其他 15 个模型
    'Qwen2.5-Max',
+   'Qwen3.6-27B',
  ],
  modelMappings: {
    'Qwen3.6-Plus': 'qwen3.6-plus',
    'Qwen3.5-Plus': 'qwen3.5-plus',
    // ... 其他映射
    'Qwen2.5-Max': 'qwen-max-latest',
+   'Qwen3.6-27B': 'qwen3.6-27b',
  },
```

---

## 5. 扩展路径说明

### 5.1 路径 A：同协议新模型（本次所有修改均属此类）

**操作流程**：
```
1. 确认新模型的 API Model ID（抓包或查文档）
2. 修改 providers/builtin/{provider}.ts
   - supportedModels 数组添加显示名
   - modelMappings 对象添加映射
3. 检查 Adapter 是否需要特殊处理
4. 重启开发服务器测试
```

**本次所有修改均无需修改 Adapter 代码**，仅改配置文件。

**唯一例外**：DeepSeek V4 可能需要确认 Web API 的 `model_type` 参数是否需要新值。

### 5.2 路径 B：新服务商开发

详见 `Add-New-Models-Guide.md`。

### 5.3 路径 C：用户侧零代码管理

通过 UI 界面（模型管理 → 编辑模型）或 Management API 添加自定义模型，无需改代码。

---

## 6. 测试验证

### 6.1 完整测试清单

```bash
# DeepSeek V4-Pro
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"DeepSeek-V4-Pro","messages":[{"role":"user","content":"你好"}],"stream":false}'

# DeepSeek V4-Flash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"DeepSeek-V4-Flash","messages":[{"role":"user","content":"你好"}],"stream":false}'

# MiMo-V2.5-Pro
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"MiMo-V2.5-Pro","messages":[{"role":"user","content":"你好"}],"stream":false}'

# MiMo-V2.5
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"MiMo-V2.5","messages":[{"role":"user","content":"你好"}],"stream":false}'

# GLM-5.1
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"GLM-5.1","messages":[{"role":"user","content":"你好"}],"stream":false}'

# Kimi K2.6
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"Kimi-K2.6","messages":[{"role":"user","content":"你好"}],"stream":false}'

# Qwen3.6-Plus
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"Qwen3.6-Plus","messages":[{"role":"user","content":"你好"}],"stream":false}'

# 检查所有可用模型
curl http://localhost:8080/v1/models | jq '.data[].id'
```

---

## 附录：修改总览（修订版）

| 文件 | 新增模型数 | 改动行数 | 优先级 |
|------|-----------|---------|--------|
| `providers/builtin/deepseek.ts` | 2 (V4-Pro, V4-Flash) | +4 | 🔴 P0 |
| `providers/builtin/mimo.ts` | 2 (V2.5-Pro, V2.5) | +4 | 🔴 P0 |
| `providers/builtin/glm.ts` | 3 (5.1, 4.7, 4.6) | +6 | 🔴 P0 |
| `providers/builtin/zai.ts` | 1 (GLM-5.1) | +2 | 🔴 P0 |
| `providers/builtin/kimi.ts` | 2 (K2.6, Latest) | +4 | 🔴 P0 |
| `providers/builtin/perplexity.ts` | 2 (Sonar, Sonar-Pro) | +4 | 🟡 P1 |
| `providers/builtin/qwen.ts` | 2 (3.6-Plus, 3.6-27B) | +4 | 🟡 P1 |
| `providers/builtin/qwen-ai.ts` | 1 (3.6-27B) | +2 | 🟡 P1 |
| **总计** | **15 个新模型** | **+30 行** | |

> **无需修改任何 Adapter 代码**（DeepSeek V4 需额外验证 Web API 的 model_type 参数）。
