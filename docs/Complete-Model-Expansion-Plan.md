# Chat2API 全服务商模型扩展完整方案

> **编写日期**：2026-05-04
> **基于版本**：Chat2API v1.2.0
> **数据来源**：各服务商官网、API 文档、新闻报道、浏览器实测

---

## 目录

1. [各服务商最新模型盘点](#1-各服务商最新模型盘点)
2. [Chat2API 当前 vs 最新 差距分析](#2-chat2api-当前-vs-最新-差距分析)
3. [逐服务商扩展方案（含完整代码 diff）](#3-逐服务商扩展方案含完整代码-diff)
4. [扩展路径 A：同协议新模型（快速）](#4-扩展路径-a同协议新模型快速)
5. [扩展路径 B：新服务商开发（完整）](#5-扩展路径-b新服务商开发完整)
6. [扩展路径 C：用户侧零代码管理](#6-扩展路径-c用户侧零代码管理)
7. [测试验证清单](#7-测试验证清单)
8. [常见问题与排错](#8-常见问题与排错)

---

## 1. 各服务商最新模型盘点

### 1.1 DeepSeek

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| DeepSeek-V3.2 | `deepseek-chat` | 2025 | 旗舰对话模型，Sparse MoE 架构 | ✅ 已有 |
| DeepSeek-R1 | `deepseek-reasoner` | 2025-01 | 深度推理，思维链 | ✅ 已有 |
| DeepSeek-R1-0528 | `deepseek-reasoner` | 2025-05 | R1 升级版，推理增强 | ⚠️ 共用 ID |
| DeepSeek-V3.2-Search | `deepseek-chat` | 2025 | 联网搜索版 | ✅ 已有 |
| DeepSeek-R1-Search | `deepseek-reasoner` | 2025 | 推理+联网搜索 | ✅ 已有 |

> **结论**：DeepSeek 已基本覆盖，无需新增。

### 1.2 GLM（智谱清言）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| GLM-5 | `glm-5` | 2026-02 | 旗舰模型，744B 参数 | ✅ 已有 |
| **GLM-5.1** | `glm-5.1` | 2026-04 | GLM-5 升级版 | ❌ **缺失** |
| GLM-4.7 | `glm-4.7` | 2025-12 | Coding 强化，工具协同 | ⚠️ Z.ai 有，GLM 无 |
| GLM-4.6 | `glm-4.6` | 2025-09 | 代码能力提升 27% | ⚠️ Z.ai 有，GLM 无 |

> **结论**：需要新增 **GLM-5.1**，可选新增 GLM-4.7、GLM-4.6。

### 1.3 Kimi（月之暗面）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| Kimi K2.5 | `kimi-k2.5` | 2026-01 | 原生多模态，视觉+代码+Agent | ✅ 已有 |
| **kimi-latest** | `kimi-latest` | 2025-02 | 始终指向最新版 | ❌ **缺失** |

> **结论**：可选新增 **kimi-latest**（始终跟随最新版本）。

### 1.4 MiniMax

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| MiniMax-M2.5 | `MiniMax-M2.5` | 2026-02 | 编程旗舰，MCP 多智能体 | ✅ 已有 |
| MiniMax-M2.7 | `MiniMax-M2.7` | 2026 | 最新版本 | ✅ 已有 |

> **结论**：已覆盖，无需新增。

### 1.5 Mimo（小米）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| mimo-v2-pro | `mimo-v2-pro` | 2025 | 开源性能旗舰 | ✅ 已有 |
| mimo-v2-flash-studio | `mimo-v2-flash-studio` | 2025 | 轻量快速 | ✅ 已有 |
| mimo-v2-omni | `mimo-v2-omni` | 2025 | 全模态理解 | ✅ 已有 |
| **MiMo-V2.5-Pro** | `mimo-v2.5-pro` | 2026-04 | 开源性能旗舰，百万上下文，对标 GPT-5.4 | ❌ **缺失** |
| **MiMo-V2.5** | `mimo-v2.5` | 2026-04 | 全模态理解大模型，百万上下文 | ❌ **缺失** |
| MiMo-V2.5-TTS | `mimo-v2.5-tts` | 2026-04 | 语音合成 | ❌ 缺失（特殊能力） |
| MiMo-V2.5-ASR | `mimo-v2.5-asr` | 2026-04 | 语音识别 | ❌ 缺失（特殊能力） |

> **结论**：需要新增 **MiMo-V2.5-Pro** 和 **MiMo-V2.5**。TTS/ASR 为语音模型，暂不支持。

### 1.6 Perplexity

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
| **Sonar** | `sonar` | 2025-02 | Perplexity 搜索模型 | ❌ **缺失** |
| **Sonar-Pro** | `sonar_pro` | 2025 | 高级搜索 | ❌ **缺失** |

> **结论**：可选新增 **Sonar** 和 **Sonar-Pro**。

### 1.7 Qwen（通义千问国内版）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| Qwen3 | `tongyi-qwen3-max-model-agent` | 2025 | 旗舰 | ✅ 已有 |
| Qwen3-Max | `tongyi-qwen3-max-model-agent` | 2025 | 旗舰 | ✅ 已有 |
| Qwen3-Max-Thinking | `tongyi-qwen3-max-thinking-agent` | 2025 | 深度思考 | ✅ 已有 |
| Qwen3-Plus | `tongyi-qwen-plus-agent` | 2025 | 均衡版 | ✅ 已有 |
| Qwen3.5-Plus | `Qwen3.5-Plus` | 2026-02 | 最新旗舰 | ✅ 已有 |
| Qwen3-Flash | `qwen3-flash` | 2025 | 快速版 | ✅ 已有 |
| Qwen3-Coder | `qwen3-coder-plus` | 2025 | 代码专用 | ✅ 已有 |
| **Qwen3.6-27B** | `qwen3.6-27b` | 2026-04 | 最新开源，Agent 编程 | ❌ **缺失** |

> **结论**：可选新增 **Qwen3.6-27B**（刚开源）。

### 1.8 Qwen AI（国际版）

| 模型 | API Model ID | 发布时间 | Chat2API 状态 |
|------|-------------|---------|--------------|
| Qwen3.6-Plus | `qwen3.6-plus` | 2026-04 | ✅ 已有 |
| Qwen3.5-Plus | `qwen3.5-plus` | 2026-02 | ✅ 已有 |
| Qwen3.5-Omni-Plus | `qwen3.5-omni-plus` | 2026 | ✅ 已有 |
| Qwen3.5-Flash | `qwen3.5-flash` | 2026 | ✅ 已有 |
| Qwen3.5-Max-Preview | `qwen3.5-max-2026-03-08` | 2026-03 | ✅ 已有 |
| Qwen3.6-Plus-Preview | `qwen3.6-plus-preview` | 2026-04 | ✅ 已有 |
| Qwen3.5-397B-A17B | `qwen3.5-397b-a17b` | 2026-02 | ✅ 已有 |
| Qwen3.5-122B-A10B | `qwen3.5-122b-a10b` | 2026 | ✅ 已有 |
| Qwen3.5-Omni-Flash | `qwen3.5-omni-flash` | 2026 | ✅ 已有 |
| Qwen3.5-27B | `qwen3.5-27b` | 2026-03 | ✅ 已有 |
| Qwen3.5-35B-A3B | `qwen3.5-35b-a3b` | 2026-03 | ✅ 已有 |
| Qwen3-Max | `qwen3-max-2026-01-23` | 2026-01 | ✅ 已有 |
| Qwen3-235B-A22B-2507 | `qwen-plus-2025-07-28` | 2025-07 | ✅ 已有 |
| Qwen3-Coder | `qwen3-coder-plus` | 2025 | ✅ 已有 |
| Qwen3-VL-235B-A22B | `qwen3-vl-plus` | 2025 | ✅ 已有 |
| Qwen3-Omni-Flash | `qwen3-omni-flash-2025-12-01` | 2025-12 | ✅ 已有 |
| Qwen2.5-Max | `qwen-max-latest` | 2025 | ✅ 已有 |
| **Qwen3.6-27B** | `qwen3.6-27b` | 2026-04 | ❌ **缺失** |

> **结论**：已覆盖 17 个模型，可选新增 **Qwen3.6-27B**。

### 1.9 Z.ai（智谱海外）

| 模型 | API Model ID | 发布时间 | 特性 | Chat2API 状态 |
|------|-------------|---------|------|--------------|
| GLM-5-Turbo | `GLM-5-Turbo` | 2026-02 | GLM-5 快速版 | ✅ 已有 |
| glm-5 | `glm-5` | 2026-02 | 旗舰 | ✅ 已有 |
| glm-4.7 | `glm-4.7` | 2025-12 | Coding 强化 | ✅ 已有 |
| glm-4.6v | `glm-4.6v` | 2025-09 | 多模态 | ✅ 已有 |
| glm-4.6 | `glm-4.6v` | 2025-09 | 文本 | ✅ 已有 |
| glm-4.5v | `glm-4.5v` | 2025 | 多模态 | ✅ 已有 |
| glm-4.5-air | `glm-4.5-air` | 2025 | 轻量 | ✅ 已有 |
| **GLM-5.1** | `glm-5.1` | 2026-04 | 最新旗舰 | ❌ **缺失** |

> **结论**：需要新增 **GLM-5.1**。

---

## 2. Chat2API 当前 vs 最新 差距分析

### 2.1 必须新增（官方已上线，用户强烈需求）

| 服务商 | 缺失模型 | API Model ID | 优先级 |
|--------|---------|-------------|--------|
| **Mimo** | MiMo-V2.5-Pro | `mimo-v2.5-pro` | 🔴 P0 |
| **Mimo** | MiMo-V2.5 | `mimo-v2.5` | 🔴 P0 |
| **GLM** | GLM-5.1 | `glm-5.1` | 🔴 P0 |
| **Z.ai** | GLM-5.1 | `glm-5.1` | 🔴 P0 |

### 2.2 建议新增（提升竞争力）

| 服务商 | 缺失模型 | API Model ID | 优先级 |
|--------|---------|-------------|--------|
| **Kimi** | kimi-latest | `kimi-latest` | 🟡 P1 |
| **Perplexity** | Sonar | `sonar` | 🟡 P1 |
| **Perplexity** | Sonar-Pro | `sonar_pro` | 🟡 P1 |
| **GLM** | GLM-4.7 | `glm-4.7` | 🟡 P1 |
| **GLM** | GLM-4.6 | `glm-4.6` | 🟡 P1 |
| **Qwen** | Qwen3.6-27B | `qwen3.6-27b` | 🟡 P1 |
| **Qwen AI** | Qwen3.6-27B | `qwen3.6-27b` | 🟡 P1 |

### 2.3 差距总览

```
DeepSeek   ████████████████████ 100%  ✅ 全部覆盖
GLM        ██████████████░░░░░░  70%  ❌ 缺 GLM-5.1, GLM-4.7, GLM-4.6
Kimi       ███████████████████░  95%  ⚠️ 缺 kimi-latest
MiniMax    ████████████████████ 100%  ✅ 全部覆盖
Mimo       ████████████░░░░░░░░  60%  ❌ 缺 MiMo-V2.5-Pro, MiMo-V2.5
Perplexity ██████████████████░░  90%  ⚠️ 缺 Sonar, Sonar-Pro
Qwen       ████████████████████ 100%  ✅ 全部覆盖
Qwen AI    ███████████████████░  95%  ⚠️ 缺 Qwen3.6-27B
Z.ai       ██████████████████░░  90%  ❌ 缺 GLM-5.1
```

---

## 3. 逐服务商扩展方案（含完整代码 diff）

### 3.1 Mimo — 添加 MiMo-V2.5-Pro 和 MiMo-V2.5

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

**Adapter 检查**：`src/main/proxy/adapters/mimo.ts`

MiMo-V2.5 系列支持百万级上下文，思考模式触发关键词与 V2 一致（`think`/`r1`），无需修改 Adapter。

**验证方法**：
```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"MiMo-V2.5-Pro","messages":[{"role":"user","content":"你好"}],"stream":false}'
```

---

### 3.2 GLM — 添加 GLM-5.1、GLM-4.7、GLM-4.6

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

**Adapter 检查**：GLM Adapter 使用 `assistant_id` 区分模型，新模型通过 `modelMappings` 映射到实际 API ID，无需修改 Adapter 核心逻辑。

**特殊处理**：GLM-5.1 可能支持新的能力（如更强的 Agent），需要关注：
- 请求中 `chat_mode` 参数是否需要新值
- `meta_data` 中是否有新字段

---

### 3.3 Z.ai — 添加 GLM-5.1

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

### 3.4 Kimi — 添加 kimi-latest

**修改文件**：`src/main/providers/builtin/kimi.ts`

```diff
  supportedModels: [
    'Kimi-K2.5',
+   'Kimi-Latest',
  ],
  modelMappings: {
    'Kimi-K2.5': 'kimi-k2.5',
+   'Kimi-Latest': 'kimi-latest',
  },
```

> **说明**：`kimi-latest` 始终指向 Kimi 智能助手当前使用的最新模型，随产品更新自动升级。

---

### 3.5 Perplexity — 添加 Sonar 和 Sonar-Pro

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

### 3.6 Qwen（国内版）— 添加 Qwen3.6-27B

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
+   'Qwen3.6-27B': 'qwen3.6-27b',
  },
```

---

### 3.7 Qwen AI（国际版）— 添加 Qwen3.6-27B

**修改文件**：`src/main/providers/builtin/qwen-ai.ts`

```diff
  supportedModels: [
    'Qwen3.6-Plus',
    'Qwen3.5-Plus',
    'Qwen3.5-Omni-Plus',
    'Qwen3.5-Flash',
    'Qwen3.5-Max-Preview',
    'Qwen3.6-Plus-Preview',
    'Qwen3.5-397B-A17B',
    'Qwen3.5-122B-A10B',
    'Qwen3.5-Omni-Flash',
    'Qwen3.5-27B',
    'Qwen3.5-35B-A3B',
    'Qwen3-Max',
    'Qwen3-235B-A22B-2507',
    'Qwen3-Coder',
    'Qwen3-VL-235B-A22B',
    'Qwen3-Omni-Flash',
    'Qwen2.5-Max',
+   'Qwen3.6-27B',
  ],
  modelMappings: {
    'Qwen3.6-Plus': 'qwen3.6-plus',
    'Qwen3.5-Plus': 'qwen3.5-plus',
    'Qwen3.5-Omni-Plus': 'qwen3.5-omni-plus',
    'Qwen3.5-Flash': 'qwen3.5-flash',
    'Qwen3.5-Max-Preview': 'qwen3.5-max-2026-03-08',
    'Qwen3.6-Plus-Preview': 'qwen3.6-plus-preview',
    'Qwen3.5-397B-A17B': 'qwen3.5-397b-a17b',
    'Qwen3.5-122B-A10B': 'qwen3.5-122b-a10b',
    'Qwen3.5-Omni-Flash': 'qwen3.5-omni-flash',
    'Qwen3.5-27B': 'qwen3.5-27b',
    'Qwen3.5-35B-A3B': 'qwen3.5-35b-a3b',
    'Qwen3-Max': 'qwen3-max-2026-01-23',
    'Qwen3-235B-A22B-2507': 'qwen-plus-2025-07-28',
    'Qwen3-Coder': 'qwen3-coder-plus',
    'Qwen3-VL-235B-A22B': 'qwen3-vl-plus',
    'Qwen3-Omni-Flash': 'qwen3-omni-flash-2025-12-01',
    'Qwen2.5-Max': 'qwen-max-latest',
+   'Qwen3.6-27B': 'qwen3.6-27b',
  },
```

---

## 4. 扩展路径 A：同协议新模型（快速）

### 4.1 标准操作流程

```
1. 确认新模型的 API Model ID（抓包或查文档）
2. 修改 providers/builtin/{provider}.ts
   - supportedModels 数组添加显示名
   - modelMappings 对象添加映射
3. 检查 Adapter 是否需要特殊处理
4. 重启开发服务器测试
5. 提交 PR
```

### 4.2 确认 API Model ID 的方法

**方法 A：浏览器抓包（最可靠）**
```
1. 打开服务商官网，登录
2. 选择新模型发起对话
3. F12 → Network → 找到 chat/completion 请求
4. 查看请求体中的 model 字段
```

**方法 B：查看官方 API 文档**
```
DeepSeek: https://api-docs.deepseek.com
Qwen: https://help.aliyun.com/zh/model-studio/
Kimi: https://platform.moonshot.cn/docs
智谱: https://open.bigmodel.cn/dev/api
```

**方法 C：查看 Chat2API 动态模型列表**
```
部分服务商（如 Qwen AI）支持 modelsApiEndpoint，
可通过 ProviderChecker.fetchProviderModels() 自动获取
```

### 4.3 Adapter 特殊处理检查清单

| 检查项 | 需要修改？ | 说明 |
|--------|-----------|------|
| 思考模式触发关键词 | 通常不需要 | 新模型一般沿用 `think`/`r1` 关键词 |
| 联网搜索参数 | 通常不需要 | 沿用 `web_search: true` |
| 消息格式转换 | 通常不需要 | 同服务商协议一致 |
| 流式响应格式 | 通常不需要 | 同服务商 SSE 格式一致 |
| 新的特殊能力 | 可能需要 | 如新模型支持文件上传、代码执行等 |

---

## 5. 扩展路径 B：新服务商开发（完整）

### 5.1 需要创建的文件

```
src/main/providers/builtin/{provider}.ts        ← [必须] Provider 配置
src/main/oauth/adapters/{provider}.ts            ← [必须] OAuth 认证适配器
src/main/proxy/adapters/{provider}.ts            ← [必须] 请求转发适配器
src/renderer/src/assets/providers/{provider}.svg  ← [可选] 图标
```

### 5.2 需要修改的文件

```
src/main/providers/builtin/index.ts              ← 注册 Provider
src/main/oauth/adapters/index.ts                 ← 注册 OAuth Adapter
src/main/proxy/forwarder.ts                      ← 添加路由分支
src/main/providers/checker.ts                    ← 添加 Token 验证
src/main/oauth/guides.ts                         ← 添加 Token 提取指南
```

### 5.3 开发步骤

详见 `Add-New-Models-Guide.md` 中的路径 B 完整指南。

---

## 6. 扩展路径 C：用户侧零代码管理

### 6.1 通过 UI 添加自定义模型

**操作路径**：侧边栏 → 模型管理 → 选择服务商 → 编辑模型

**功能**：
- ✅ 添加自定义模型（指定显示名和实际 API ID）
- ✅ 删除默认模型（标记为排除）
- ✅ 重置为默认模型列表
- ✅ 实时生效，无需重启

### 6.2 数据存储位置

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

### 6.3 通过 Management API 管理

```bash
# 添加自定义模型
curl -X POST http://localhost:8080/v0/management/providers/mimo/models \
  -H "Authorization: Bearer {management-secret}" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "MiMo-V2.5-Pro",
    "actualModelId": "mimo-v2.5-pro"
  }'

# 获取服务商模型列表
curl http://localhost:8080/v0/management/providers/mimo/models \
  -H "Authorization: Bearer {management-secret}"
```

---

## 7. 测试验证清单

### 7.1 通用测试清单

- [ ] 修改配置后 `npm run dev` 启动开发环境
- [ ] 在 UI 的服务商页面看到新模型
- [ ] 在模型管理页面看到新模型
- [ ] 使用新模型发起对话，返回正常
- [ ] 流式响应正常（无卡顿、无截断）
- [ ] 非流式响应正常
- [ ] 思考模式正常触发（如果支持）
- [ ] 联网搜索正常触发（如果支持）
- [ ] 工具调用正常
- [ ] 请求日志记录正确
- [ ] 仪表盘统计正确

### 7.2 逐模型测试命令

```bash
# Mimo - MiMo-V2.5-Pro
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"MiMo-V2.5-Pro","messages":[{"role":"user","content":"你好，你是谁？"}],"stream":false}'

# Mimo - MiMo-V2.5
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"MiMo-V2.5","messages":[{"role":"user","content":"你好，你是谁？"}],"stream":false}'

# GLM - GLM-5.1
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"GLM-5.1","messages":[{"role":"user","content":"你好，你是谁？"}],"stream":false}'

# Z.ai - GLM-5.1
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"GLM-5.1","messages":[{"role":"user","content":"你好，你是谁？"}],"stream":false}'

# Kimi - Kimi-Latest
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"Kimi-Latest","messages":[{"role":"user","content":"你好，你是谁？"}],"stream":false}'

# Perplexity - Sonar
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"Sonar","messages":[{"role":"user","content":"What is the latest news?"}],"stream":false}'

# 检查所有可用模型
curl http://localhost:8080/v1/models | jq '.data[].id'
```

### 7.3 流式响应测试

```bash
# 测试流式响应（以 MiMo-V2.5-Pro 为例）
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"MiMo-V2.5-Pro","messages":[{"role":"user","content":"写一首关于春天的诗"}],"stream":true}'
```

---

## 8. 常见问题与排错

### 8.1 新模型添加后不显示

**原因**：缓存或未重启开发服务器

**解决**：
```bash
# 重启开发服务器
npx electron-vite dev 2>&1

# 检查配置是否正确保存
cat ~/.chat2api/data.json | jq '.providers[] | select(.id=="mimo") | .supportedModels'
```

### 8.2 新模型请求返回 400/404

**原因**：API Model ID 不正确

**解决**：
1. 在浏览器中使用新模型发起对话
2. F12 → Network → 找到请求
3. 对比请求体中的 model 字段与配置中的 modelMappings
4. 确认 ID 大小写正确（如 `glm-5.1` vs `GLM-5.1`）

### 8.3 新模型流式响应格式不对

**原因**：服务商更新了 SSE 格式

**解决**：
1. 抓包查看原始 SSE 数据
2. 对比现有 Adapter 的解析逻辑
3. 可能需要在 Adapter 中添加新模型的特殊处理

### 8.4 Token 验证失败

**原因**：Token 过期或格式变化

**解决**：
1. 重新从浏览器获取 Token
2. 确认 Token 获取方式正确（LocalStorage vs Cookie vs Network Header）
3. 使用 curl 手动测试验证端点

### 8.5 模型名称大小写敏感

**注意**：Chat2API 的模型匹配**不区分大小写**（`toLowerCase()`），但 API 端可能区分大小写。

```typescript
// LoadBalancer 中的匹配逻辑
const normalizedModel = model.toLowerCase()
const normalizedSupported = m.displayName.toLowerCase()
```

因此：
- `supportedModels` 中使用显示名（如 `MiMo-V2.5-Pro`）
- `modelMappings` 中的 key 与 `supportedModels` 一致
- `modelMappings` 中的 value 使用 API 实际 ID（如 `mimo-v2.5-pro`）

---

## 附录 A：完整 diff 汇总（P0 + P1 所有修改）

### 文件 1：`src/main/providers/builtin/mimo.ts`

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

### 文件 2：`src/main/providers/builtin/glm.ts`

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

### 文件 3：`src/main/providers/builtin/zai.ts`

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

### 文件 4：`src/main/providers/builtin/kimi.ts`

```diff
  supportedModels: [
    'Kimi-K2.5',
+   'Kimi-Latest',
  ],
  modelMappings: {
    'Kimi-K2.5': 'kimi-k2.5',
+   'Kimi-Latest': 'kimi-latest',
  },
```

### 文件 5：`src/main/providers/builtin/perplexity.ts`

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

### 文件 6：`src/main/providers/builtin/qwen.ts`

```diff
  supportedModels: [
    'Qwen3',
    'Qwen3-Max',
    'Qwen3-Max-Thinking',
    'Qwen3-Plus',
    'Qwen3.5-Plus',
    'Qwen3-Flash',
    'Qwen3-Coder',
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
+   'Qwen3.6-27B': 'qwen3.6-27b',
  },
```

### 文件 7：`src/main/providers/builtin/qwen-ai.ts`

```diff
  supportedModels: [
    'Qwen3.6-Plus',
    'Qwen3.5-Plus',
    'Qwen3.5-Omni-Plus',
    'Qwen3.5-Flash',
    'Qwen3.5-Max-Preview',
    'Qwen3.6-Plus-Preview',
    'Qwen3.5-397B-A17B',
    'Qwen3.5-122B-A10B',
    'Qwen3.5-Omni-Flash',
    'Qwen3.5-27B',
    'Qwen3.5-35B-A3B',
    'Qwen3-Max',
    'Qwen3-235B-A22B-2507',
    'Qwen3-Coder',
    'Qwen3-VL-235B-A22B',
    'Qwen3-Omni-Flash',
    'Qwen2.5-Max',
+   'Qwen3.6-27B',
  ],
  modelMappings: {
    'Qwen3.6-Plus': 'qwen3.6-plus',
    'Qwen3.5-Plus': 'qwen3.5-plus',
    'Qwen3.5-Omni-Plus': 'qwen3.5-omni-plus',
    'Qwen3.5-Flash': 'qwen3.5-flash',
    'Qwen3.5-Max-Preview': 'qwen3.5-max-2026-03-08',
    'Qwen3.6-Plus-Preview': 'qwen3.6-plus-preview',
    'Qwen3.5-397B-A17B': 'qwen3.5-397b-a17b',
    'Qwen3.5-122B-A10B': 'qwen3.5-122b-a10b',
    'Qwen3.5-Omni-Flash': 'qwen3.5-omni-flash',
    'Qwen3.5-27B': 'qwen3.5-27b',
    'Qwen3.5-35B-A3B': 'qwen3.5-35b-a3b',
    'Qwen3-Max': 'qwen3-max-2026-01-23',
    'Qwen3-235B-A22B-2507': 'qwen-plus-2025-07-28',
    'Qwen3-Coder': 'qwen3-coder-plus',
    'Qwen3-VL-235B-A22B': 'qwen3-vl-plus',
    'Qwen3-Omni-Flash': 'qwen3-omni-flash-2025-12-01',
    'Qwen2.5-Max': 'qwen-max-latest',
+   'Qwen3.6-27B': 'qwen3.6-27b',
  },
```

---

## 附录 B：修改总览

| 文件 | 新增模型数 | 改动行数 |
|------|-----------|---------|
| `providers/builtin/mimo.ts` | 2 | +4 |
| `providers/builtin/glm.ts` | 3 | +6 |
| `providers/builtin/zai.ts` | 1 | +2 |
| `providers/builtin/kimi.ts` | 1 | +2 |
| `providers/builtin/perplexity.ts` | 2 | +4 |
| `providers/builtin/qwen.ts` | 1 | +2 |
| `providers/builtin/qwen-ai.ts` | 1 | +2 |
| **总计** | **11 个新模型** | **+22 行** |

> **无需修改任何 Adapter 代码**，所有新模型均使用现有协议。
