---
layout: post
title: "五大模型平台API全方位对比：DeepSeek、OpenAI、Gemini、Claude、Kimi"
date: 2026-06-17 14:00:00 +0800
categories: [AI, 大模型]
tags: [DeepSeek, OpenAI, Gemini, Claude, Kimi, API, 大模型]
description: "详细对比 DeepSeek、OpenAI、Gemini、Claude、Kimi 五家主流大模型平台的 Chat API，涵盖请求参数、响应格式、模型列表、定价策略和代码示例。"
toc: true
---

此篇内容由 AI 生成，仅供参考。

## 前言

随着大语言模型（LLM）的快速发展，DeepSeek、OpenAI、Google Gemini、Anthropic Claude 和 Moonshot Kimi 已成为开发者最常接触的五大模型平台。虽然各家 API 在设计上都参考了 OpenAI 的 Chat Completions 格式，但在参数细节、响应结构、模型能力和定价策略上仍有显著差异。

本文将从 **请求参数、响应格式、模型列表、定价** 等多个维度进行全面对比，帮助开发者在做技术选型时快速找到最适合的方案。文中数据均核对自 2026 年 6 月各平台官方文档。

---

## 一、基本信息速览

| 对比维度 | DeepSeek | OpenAI | Gemini | Claude | Kimi |
|---------|----------|--------|--------|--------|------|
| **API 端点** | `https://api.deepseek.com/v1/chat/completions` | `https://api.openai.com/v1/chat/completions` | 原生：`generativelanguage.googleapis.com/v1beta/models/{model}:generateContent`<br>兼容层：`.../v1beta/openai/chat/completions` | `https://api.anthropic.com/v1/messages` | `https://api.moonshot.cn/v1/chat/completions` |
| **协议兼容** | ✅ OpenAI 兼容<br>✅ Anthropic 兼容 | - 原生 | ✅ OpenAI 兼容层<br>❌ 原生格式独立 | ❌ 独立协议 | ✅ OpenAI 兼容 |
| **认证方式** | `Bearer $API_KEY` | `Bearer $API_KEY` | `Authorization: Bearer $API_KEY`（兼容层）<br>或 `?key=$API_KEY`（原生） | `x-api-key: $API_KEY` | `Bearer $API_KEY` |
| **系统提示** | `messages[0].role="system"` | `messages[0].role="system"` | 兼容层：`role="system"`<br>原生：`systemInstruction` | `system` 独立参数 | `messages[0].role="system"` |
| **多模态支持** | ❌ 文本 | ✅ 文本+图片+音频 | ✅ 文本+图片+音频+视频 | ✅ 文本+图片+PDF | ✅ 文本+图片+视频 |
| **思考模式** | ✅ `thinking` + `reasoning_effort` | ✅ `reasoning_effort`（GPT-5.x） | ✅ `reasoning_effort`（兼容层）<br>`thinkingConfig`（原生） | ✅ `thinking` / `adaptive`<br>+ `output_config.effort` | ✅ `thinking` 参数 |

> 💡 **关键发现**：DeepSeek、Kimi 和 Gemini（兼容层）均可通过 OpenAI SDK 接入，只需修改 `base_url`。DeepSeek 还额外支持 Anthropic 格式（`https://api.deepseek.com/anthropic`）。Claude 仍使用独立的 Messages API，迁移成本最高。

---

## 二、请求参数全面对比

### 2.1 核心生成参数

| 参数 | DeepSeek | OpenAI | Gemini | Claude | Kimi |
|------|----------|--------|--------|--------|------|
| **model** (必填) | ✅ | ✅ | 路径参数 / body | ✅ | ✅ |
| **messages/contents** (必填) | ✅ `messages` | ✅ `messages` | ✅ `contents` / `messages` | ✅ `messages` | ✅ `messages` |
| **temperature** | 0-2，默认 1 | 0-2，默认 1 | 0-2，默认 1 | 0-1，默认 1 | K2.x 未公开默认值；V1 默认 0 |
| **top_p** | 0-1，默认 1 | 0-1，默认 1 | 0-1，默认 0.95 | 0-1，默认 1 | 0-1，默认 1 |
| **top_k** | ❌ | ❌ | ✅ 默认 40 | ❌ | ❌ |
| **max_tokens** | ✅ `max_tokens` | ⚠️ 已弃用，改用 `max_completion_tokens` | ✅ `maxOutputTokens` | ✅ **必填** | ✅ |
| **stop** | ✅ string/array（最多 16 个） | ✅ string/array（最多 4 个） | ✅ `stopSequences` | ✅ `stop_sequences` | ✅ |
| **stream** | ✅ SSE | ✅ SSE | ✅ SSE | ✅ SSE | ✅ SSE |
| **frequency_penalty** | ⚠️ 已弃用 | ✅ -2.0~2.0 | ❌ | ❌ | ✅（V1 系列） |
| **presence_penalty** | ⚠️ 已弃用 | ✅ -2.0~2.0 | ❌ | ❌ | ✅（V1 系列） |
| **seed** | ❌ | ✅ | ❌ | ❌ | ❌ |
| **logprobs** | ✅ | ✅ | ❌ | ❌ | ❌ |
| **n**（多候选） | ❌ | ✅ | ❌ | ❌ | ✅（V1 系列） |

> ⚠️ DeepSeek 在思考模式下，`temperature`、`top_p`、`frequency_penalty`、`presence_penalty` 传入后不生效（不报错）。

### 2.2 消息结构对比

#### DeepSeek / OpenAI / Kimi / Gemini（OpenAI 兼容层）

```json
{
  "messages": [
    {"role": "system", "content": "你是一个有用的助手"},
    {"role": "user", "content": "你好"},
    {"role": "assistant", "content": "你好！有什么可以帮助你的？"},
    {"role": "user", "content": "今天天气怎么样？"}
  ]
}
```

支持角色：`system`、`user`、`assistant`、`tool`

Kimi K2.x 还支持多模态 content 数组（`text` / `image_url` / `video_url`）。

#### Claude（独立格式）

```json
{
  "system": "你是一个有用的助手",
  "messages": [
    {"role": "user", "content": "你好"},
    {"role": "assistant", "content": "你好！有什么可以帮助你的？"},
    {"role": "user", "content": "今天天气怎么样？"}
  ]
}
```

**关键区别**：
- Claude 的 `system` 是**顶层参数**，不在 `messages` 数组中
- Claude 的 `content` 支持数组格式（文本+图片混合）
- Claude 的 `messages` 必须以 `user` 开头，不支持 `system` 角色

#### Gemini（原生格式）

```json
{
  "systemInstruction": {
    "parts": [{"text": "你是一个有用的助手"}]
  },
  "contents": [
    {
      "role": "user",
      "parts": [{"text": "你好"}]
    },
    {
      "role": "model",
      "parts": [{"text": "你好！有什么可以帮助你的？"}]
    },
    {
      "role": "user",
      "parts": [{"text": "今天天气怎么样？"}]
    }
  ]
}
```

**关键区别**：
- 使用 `contents` 而非 `messages`
- 助手角色为 `model` 而非 `assistant`
- 内容通过 `parts` 数组组织
- `systemInstruction` 为独立顶层字段

### 2.3 工具调用（Function Calling）

| 功能 | DeepSeek | OpenAI | Gemini | Claude | Kimi |
|------|----------|--------|--------|--------|------|
| **支持工具调用** | ✅ 最多 128 个函数 | ✅ | ✅ `functionDeclarations` | ✅ `tools` | ✅ |
| **并行调用** | ❌ | ✅ `parallel_tool_calls` | ❌ | ⚠️ 默认开启，可 `disable_parallel_tool_use` | ❌ |
| **强制调用** | ✅ `tool_choice: "required"` | ✅ | ✅ `toolConfig` | ✅ `tool_choice` | ✅ |
| **严格模式** | ✅ `strict: true`（Beta） | ✅ `strict: true` | ❌ | ❌ | ❌ |

**工具定义对比**：

```json
// DeepSeek / OpenAI / Kimi (OpenAI 兼容格式)
{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "查询天气",
    "parameters": {
      "type": "object",
      "properties": {
        "city": {"type": "string", "description": "城市名"}
      },
      "required": ["city"]
    }
  }
}

// Claude 格式
{
  "name": "get_weather",
  "description": "查询天气",
  "input_schema": {
    "type": "object",
    "properties": {
      "city": {"type": "string", "description": "城市名"}
    },
    "required": ["city"]
  }
}
```

> ⚠️ **注意**：Claude 使用 `input_schema` 而非 `parameters`，且工具定义不需要 `type: "function"` 包裹。

### 2.4 思考/推理模式

| 平台 | 参数 | 说明 |
|------|------|------|
| **DeepSeek** | `thinking: {type: "enabled"/"disabled"}` + `reasoning_effort: "high"/"max"` | V4 默认开启思考；`low`/`medium` 映射为 `high`，`xhigh` 映射为 `max` |
| **OpenAI** | `reasoning_effort: "none"/"low"/"medium"/"high"/"xhigh"` | GPT-5.4 / GPT-5.5 等旗舰模型支持；GPT-5.5 默认 `medium` |
| **Gemini** | 兼容层：`reasoning_effort`；原生：`thinkingConfig` | 2.5 系列映射为 `thinking_budget`；3.x 系列映射为 `thinking_level` |
| **Claude** | `thinking: {type: "enabled", budget_tokens: N}` 或 `{type: "adaptive"}` + `output_config.effort` | Opus 4.8 使用自适应思考（始终开启）；Sonnet 4.6 支持扩展思考 |
| **Kimi** | `thinking: {type: "enabled"/"disabled", keep: "all"}` | K2.6 默认开启；K2.7 Code **仅支持** `enabled`，始终思考 |

DeepSeek / Kimi 思考模式下，响应会额外返回 `reasoning_content` 字段（与 `content` 同级）。

---

## 三、响应格式对比

### 3.1 非流式响应

#### DeepSeek / OpenAI / Kimi

```json
{
  "id": "chatcmpl-xxx",
  "object": "chat.completion",
  "created": 1712345678,
  "model": "deepseek-v4-flash",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "你好！今天天气不错。",
        "reasoning_content": "（思考模式下可选返回）"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 8,
    "total_tokens": 18
  }
}
```

DeepSeek 额外返回 `prompt_cache_hit_tokens` / `prompt_cache_miss_tokens`；Kimi 返回 `cached_tokens`。

#### Claude

```json
{
  "id": "msg_xxx",
  "type": "message",
  "role": "assistant",
  "model": "claude-sonnet-4-6",
  "content": [
    {
      "type": "text",
      "text": "你好！今天天气不错。"
    }
  ],
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 10,
    "output_tokens": 8
  }
}
```

#### Gemini（原生）

```json
{
  "candidates": [
    {
      "content": {
        "parts": [{"text": "你好！今天天气不错。"}],
        "role": "model"
      },
      "finishReason": "STOP",
      "index": 0
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 10,
    "candidatesTokenCount": 8,
    "totalTokenCount": 18
  }
}
```

### 3.2 finish_reason / stop_reason 对比

| 含义 | DeepSeek | OpenAI | Gemini | Claude | Kimi |
|------|----------|--------|--------|--------|------|
| 正常结束 | `stop` | `stop` | `STOP` | `end_turn` | `stop` |
| 达到长度限制 | `length` | `length` | `MAX_TOKENS` | `max_tokens` | `length` |
| 工具调用 | `tool_calls` | `tool_calls` | `TOOL_CALLS` | `tool_use` | `tool_calls` |
| 内容过滤 | `content_filter` | `content_filter` | `SAFETY` | - | `content_filter` |
| 资源不足 | `insufficient_system_resource` | - | - | - | - |

### 3.3 流式响应对比

| 平台 | 事件格式 | 结束标志 | usage 包含方式 |
|------|---------|---------|---------------|
| **DeepSeek** | SSE `data: {...}` | `data: [DONE]` | `stream_options.include_usage` |
| **OpenAI** | SSE `data: {...}` | `data: [DONE]` | `stream_options.include_usage` |
| **Gemini** | SSE `data: {...}` | 流结束 | 最后 chunk |
| **Claude** | SSE `event: content_block_delta` | `event: message_stop` | `event: message_delta` |
| **Kimi** | SSE `data: {...}` | `data: [DONE]` | 最后 chunk 含 usage |

---

## 四、模型列表与能力

### 4.1 DeepSeek

| 模型 | 上下文 | 最大输出 | 思考模式 | 特点 |
|------|--------|---------|---------|------|
| `deepseek-v4-flash` | 1M | 384K | 默认开启，可切换 | 高性价比，通用对话 |
| `deepseek-v4-pro` | 1M | 384K | 默认开启，可切换 | 旗舰性能，复杂推理 |

> ⚠️ `deepseek-chat` / `deepseek-reasoner` 将于 **2026 年 7 月 24 日** 弃用，分别对应 V4 Flash 的非思考与思考模式。

### 4.2 OpenAI

| 模型 | 上下文 | 特点 |
|------|--------|------|
| `gpt-5.5` | 1.05M | 最新旗舰，复杂专业任务，默认 `reasoning_effort: medium` |
| `gpt-5.4` | 1.05M | 高性价比旗舰，支持 none/low/medium/high/xhigh |
| `gpt-5.4-mini` | 1.05M | 最强 mini 模型，适合子 Agent 和代码 |
| `gpt-4o` | 128K | 上一代多模态旗舰（仍可用） |
| `o3` / `o4-mini` | 200K | 专用推理模型 |

> GPT-5.4 / GPT-5.5 在输入超过 272K tokens 时，整段会话按 2 倍输入、1.5 倍输出计费。

### 4.3 Gemini

| 模型 | 上下文 | 特点 |
|------|--------|------|
| `gemini-3.5-flash` | 1M | 最新快速模型，前沿智能+搜索增强 |
| `gemini-3.1-pro-preview` | 1M | 最强多模态与 Agent 能力 |
| `gemini-3.1-flash-lite` | 1M | 极致性价比，高吞吐 Agent 任务 |
| `gemini-2.5-flash` | 1M | 混合推理，支持 thinking budget |
| `gemini-2.5-pro` | 1M | 稳定强推理，多模态 |
| `gemini-2.5-flash-lite` | 1M | 2.5 系列最低成本 |

### 4.4 Claude

| 模型 | 上下文 | 特点 |
|------|--------|------|
| `claude-opus-4-8` | 1M | 最新 Opus，自适应思考，128K 最大输出 |
| `claude-sonnet-4-6` | 1M | 速度与智能最佳平衡，支持扩展思考 |
| `claude-haiku-4-5` | 200K | 最快速度，近前沿智能 |
| `claude-fable-5` | 1M | 2026 年 6 月 GA 的新旗舰（$10/$50 MTok） |

> Claude 4.6 及以后模型 ID 采用无日期格式（如 `claude-sonnet-4-6`），均为固定快照版本。

### 4.5 Kimi

| 模型 | 上下文 | 特点 |
|------|--------|------|
| `kimi-k2.7-code` | 256K | 最新编码模型，始终思考，支持文本+图片+视频 |
| `kimi-k2.7-code-highspeed` | 256K | 高速版，~180 tokens/s |
| `kimi-k2.6` | 256K | 旗舰通用模型，支持思考开关 |
| `kimi-k2.5` | 256K | 上一代多模态，性价比更高 |
| `moonshot-v1-128k` | 128K | V1 经典系列 |

---

## 五、定价对比

> 数据采集于 **2026 年 6 月**。DeepSeek、Kimi 为官方人民币标价；OpenAI、Gemini、Claude 为官方美元标价，按 **1 USD ≈ 7.2 CNY** 换算供横向参考（实际汇率以账单为准）。

| 平台 | 模型 | 输入（缓存命中） | 输入（缓存未命中） | 输出 |
|------|------|:---:|:---:|:---:|
| **DeepSeek** | V4 Flash | ¥0.02 | ¥1.00 | ¥2.00 |
| **DeepSeek** | V4 Pro | ¥0.025 | ¥3.00 | ¥6.00 |
| **OpenAI** | GPT-5.5 | ¥3.60 | ¥36.00 | ¥216.00 |
| **OpenAI** | GPT-5.4 | ¥1.80 | ¥18.00 | ¥108.00 |
| **OpenAI** | GPT-5.4 mini | ¥0.54 | ¥5.40 | ¥32.40 |
| **Gemini** | 3.5 Flash | - | ¥10.80 | ¥64.80 |
| **Gemini** | 2.5 Flash | - | ¥2.16 | ¥18.00 |
| **Gemini** | 2.5 Pro | - | ¥9.00 | ¥72.00 |
| **Claude** | Haiku 4.5 | ¥0.72 | ¥7.20 | ¥36.00 |
| **Claude** | Sonnet 4.6 | ¥2.16 | ¥21.60 | ¥108.00 |
| **Claude** | Opus 4.8 | ¥3.60 | ¥36.00 | ¥180.00 |
| **Kimi** | K2.6 | ¥1.10 | ¥6.50 | ¥27.00 |
| **Kimi** | K2.7 Code | ¥1.30 | ¥6.50 | ¥27.00 |
| **Kimi** | K2.5 | ¥0.70 | ¥4.00 | ¥21.00 |

> 💡 **定价结论**：
> - **DeepSeek** 在缓存命中场景成本极低（¥0.02/百万 tokens），整体最具价格竞争力
> - **Gemini 2.5 Flash**（¥2.16 输入）和 **Kimi K2.5**（¥4.00 输入）是性价比较高的国际/国产选择
> - **Claude Opus 4.8** 已从旧版 Opus 4 的 $15/$75 降至 $5/$25，定价大幅下调
> - **OpenAI GPT-5.4 mini**（¥5.40 输入）在 GPT-5 系列中性价比最佳

---

## 六、代码示例

### 6.1 DeepSeek（OpenAI 兼容）

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-xxx",
    base_url="https://api.deepseek.com/v1"
)

response = client.chat.completions.create(
    model="deepseek-v4-flash",
    messages=[
        {"role": "system", "content": "你是一个有用的助手"},
        {"role": "user", "content": "介绍一下深度求索公司"}
    ],
    temperature=0.7,
    max_tokens=2048,
    extra_body={"thinking": {"type": "enabled"}},
    reasoning_effort="high"
)

print(response.choices[0].message.content)
# 思考模式还可访问 response.choices[0].message.reasoning_content
```

### 6.2 OpenAI

```python
from openai import OpenAI

client = OpenAI(api_key="sk-xxx")

response = client.chat.completions.create(
    model="gpt-5.4",
    messages=[
        {"role": "system", "content": "你是一个有用的助手"},
        {"role": "user", "content": "介绍一下OpenAI公司"}
    ],
    reasoning_effort="medium",
    max_completion_tokens=2048
)

print(response.choices[0].message.content)
```

### 6.3 Gemini（OpenAI 兼容层，推荐）

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-gemini-api-key",
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/"
)

response = client.chat.completions.create(
    model="gemini-3.5-flash",
    messages=[
        {"role": "system", "content": "你是一个有用的助手"},
        {"role": "user", "content": "介绍一下Google公司"}
    ],
    reasoning_effort="low",
    max_completion_tokens=2048
)

print(response.choices[0].message.content)
```

### 6.4 Claude

```python
import anthropic

client = anthropic.Anthropic(api_key="sk-ant-xxx")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,
    system="你是一个有用的助手",
    messages=[
        {"role": "user", "content": "介绍一下Anthropic公司"}
    ]
)

print(response.content[0].text)
```

### 6.5 Kimi（OpenAI 兼容）

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-xxx",
    base_url="https://api.moonshot.cn/v1"
)

response = client.chat.completions.create(
    model="kimi-k2.6",
    messages=[
        {"role": "system", "content": "你是一个有用的助手"},
        {"role": "user", "content": "介绍一下月之暗面公司"}
    ],
    max_tokens=2048,
    extra_body={"thinking": {"type": "enabled"}}
)

print(response.choices[0].message.content)
```

---

## 七、迁移指南：从 OpenAI 切换到其他平台

### 切换到 DeepSeek

```python
client = OpenAI(
    api_key="sk-your-deepseek-key",
    base_url="https://api.deepseek.com/v1"
)
# model 改为 deepseek-v4-flash 或 deepseek-v4-pro
# 注意：frequency_penalty / presence_penalty 已弃用
# 思考模式需通过 extra_body={"thinking": {"type": "enabled"}} 开启
```

### 切换到 Kimi

```python
client = OpenAI(
    api_key="sk-your-kimi-key",
    base_url="https://api.moonshot.cn/v1"
)
# model 改为 kimi-k2.6 或 kimi-k2.7-code
# K2.7 Code 始终开启思考，不支持 thinking.type="disabled"
```

### 切换到 Gemini（兼容层，低成本迁移）

```python
client = OpenAI(
    api_key="your-gemini-api-key",
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/"
)
# model 改为 gemini-3.5-flash 或 gemini-2.5-flash
# reasoning_effort 可直接映射 Gemini 思考强度
```

### 切换到 Claude（需要重构）

```python
import anthropic

client = anthropic.Anthropic(api_key="sk-ant-xxx")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=2048,                      # 必填！
    system="你是一个有用的助手",           # 独立参数
    messages=[...]
)
# content 是数组，需要 response.content[0].text
```

---

## 八、选型建议

| 场景 | 推荐平台 | 理由 |
|------|---------|------|
| **最低成本** | DeepSeek V4 Flash | 缓存命中仅 ¥0.02/百万 tokens |
| **最强推理** | Claude Opus 4.8 / OpenAI GPT-5.5 | 复杂推理与 Agent 任务表现最佳 |
| **多模态应用** | Gemini 3.1 Pro / Kimi K2.6 | 原生支持文本+图片+视频 |
| **代码生成** | Claude Sonnet 4.6 / Kimi K2.7 Code | 编码场景专项优化 |
| **超长上下文** | DeepSeek（1M）/ Claude Opus 4.8（1M） | 文档分析、长文本处理 |
| **快速集成** | DeepSeek / Kimi / Gemini 兼容层 | 均兼容 OpenAI SDK |
| **生态最丰富** | OpenAI | Responses API、MCP、Computer Use 等工具链最完善 |
| **中文最佳** | DeepSeek / Kimi | 国产模型，中文理解更自然 |
| **安全合规** | Claude | 内置多层次内容安全机制 |

---

## 九、总结

本文从 **请求参数、响应格式、模型能力、定价、迁移成本** 五个维度对比了五大模型平台。核心要点：

1. **DeepSeek、Kimi、Gemini（兼容层）** 均可通过 OpenAI SDK 接入，迁移成本较低
2. **OpenAI** 已全面转向 GPT-5 系列，参数最丰富（`seed`、`logprobs`、`parallel_tool_calls`、Responses API）
3. **Claude** 独立 API 设计，`system` 提示词和 `max_tokens` 位置独特；Opus 4.8 已升级至 1M 上下文
4. **Gemini** 除原生 API 外，还提供 OpenAI 兼容层，降低了接入门槛
5. **定价上 DeepSeek 仍断层领先**；Claude Opus 4.8 和 Gemini 2.5 Flash 定价也已显著下调

选择哪家平台，取决于你的具体需求：成本优先选 DeepSeek，性能优先选 Claude/OpenAI，多模态选 Gemini/Kimi，中文场景选 DeepSeek/Kimi。

---

> 📝 **参考文档**
> - [DeepSeek API 文档](https://api-docs.deepseek.com/zh-cn/api/create-chat-completion)
> - [DeepSeek 定价](https://api-docs.deepseek.com/zh-cn/quick_start/pricing)
> - [OpenAI API 文档](https://platform.openai.com/docs/api-reference/chat/create)
> - [OpenAI 定价](https://openai.com/api/pricing/)
> - [Gemini API 文档](https://ai.google.dev/api/generate-content)
> - [Gemini OpenAI 兼容层](https://ai.google.dev/gemini-api/docs/openai)
> - [Claude API 文档](https://docs.anthropic.com/en/api/messages)
> - [Claude 模型概览](https://docs.anthropic.com/en/docs/about-claude/models/overview)
> - [Kimi API 文档](https://platform.kimi.com/docs/api/chat)

> *本文数据采集于 2026 年 6 月，各平台参数和价格可能随时调整，请以官方文档为准。*
