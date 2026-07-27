<div align="center">

# AI 客户端配置合集

**20+ 主流 AI 客户端的 Base URL 配置模板 —— 复制、替换、开跑**

[![License](https://img.shields.io/badge/License-MIT-2ea44f?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-1a73e8?style=flat-square)](CONTRIBUTING.md)
[![中文](https://img.shields.io/badge/文档-简体中文-D97757?style=flat-square)](README.md)

</div>

---

## 怎么用这份文档

所有第三方 AI 网关（one-api / new-api / sub2api / LiteLLM / 各类托管服务）的接入方式都是同一套逻辑：

1. 把客户端的 **Base URL** 从官方地址改成网关地址
2. 把 **API Key** 换成网关签发的 Key
3. 确认客户端用的 **协议** 和网关暴露的端点对得上

难点只在第 3 步 —— 不同客户端用的协议不一样，端点填错了就报 404 或 401。

### 先搞清楚三种端点

| 端点 | 协议 | 谁在用 |
|---|---|---|
| `/v1/messages` | Anthropic Messages | Claude Code、Anthropic SDK、Cline（Anthropic 模式） |
| `/v1/chat/completions` | OpenAI Chat Completions | 绝大多数第三方客户端；**Gemini 与 Grok 模型也走这个** |
| `/v1/responses` | OpenAI Responses | Codex CLI、Codex Desktop |

> 下文示例统一用 `https://ai.svtun.cn` 作为网关地址（[Leonis AI](https://ai.svtun.cn)，支持上述全部三种端点，
> 覆盖 Claude / GPT / Gemini / Grok 共 114 个模型）。换成你自己的网关地址即可，配置结构完全通用。
>
> 📋 模型名可在 [在线清单](https://leonis-ai.github.io/models.html) 搜索并点击复制。

---

## 目录

**CLI 工具** · [Claude Code](#claude-code) · [Codex CLI](#codex-cli) · [Aider](#aider) · [curl](#curl)

**编辑器插件** · [Cline](#cline) · [Roo Code](#roo-code) · [Continue](#continue) · [Cursor](#cursor) · [Zed](#zed)

**桌面客户端** · [Cherry Studio](#cherry-studio) · [ChatBox](#chatbox) · [LobeChat](#lobechat) · [NextChat](#nextchat)

**Web / 自托管** · [Open WebUI](#open-webui) · [Dify](#dify)

**SDK** · [OpenAI Python](#openai-sdk--python) · [OpenAI Node](#openai-sdk--nodejs) · [Anthropic Python](#anthropic-sdk--python) · [Anthropic Node](#anthropic-sdk--nodejs) · [LangChain](#langchain) · [LlamaIndex](#llamaindex)

**自动化** · [n8n](#n8n)

---

## CLI 工具

### Claude Code

协议：**Anthropic Messages** · 端点：`/api`

```bash
export ANTHROPIC_BASE_URL="https://ai.svtun.cn"
export ANTHROPIC_AUTH_TOKEN="sk-your-key"

claude
```

写进 `~/.zshrc` 或 `~/.bashrc` 持久化：

```bash
echo 'export ANTHROPIC_BASE_URL="https://ai.svtun.cn"' >> ~/.zshrc
echo 'export ANTHROPIC_AUTH_TOKEN="sk-your-key"' >> ~/.zshrc
source ~/.zshrc
```

Windows PowerShell：

```powershell
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "https://ai.svtun.cn", "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", "sk-your-key", "User")
```

> ⚠️ 变量名是 `ANTHROPIC_AUTH_TOKEN`，不是 `ANTHROPIC_API_KEY`。
> 如果两个都设了，`ANTHROPIC_API_KEY` 优先级更高会走官方 —— 先 `unset ANTHROPIC_API_KEY`。
>
> 📖 完整教程：[claude-code-guide](https://github.com/leonis-ai/claude-code-guide)
> 🔀 同时管理多套配置：[cc-switch-guide](https://github.com/leonis-ai/cc-switch-guide)

---

### Codex CLI

协议：**OpenAI Responses** · 端点：`/api/v1`

`~/.codex/config.toml`：

```toml
model_provider = "leonis"
model = "gpt-5.6-sol"

[model_providers.leonis]
name = "Leonis AI"
base_url = "https://ai.svtun.cn/v1"
wire_api = "responses"
env_key = "LEONIS_API_KEY"
```

```bash
export LEONIS_API_KEY="sk-your-key"
codex
```

> 📖 `config.toml` 全字段详解、多 Provider、沙箱与 MCP 配置：[codex-cli-guide](https://github.com/leonis-ai/codex-cli-guide)
>
> ⚠️ `wire_api` 是最容易配错的字段。网关支持 `/v1/responses` 就填 `responses`，只支持
> `/v1/chat/completions` 就填 `chat`。填错的表现是 `404 Not Found`。

---

### Aider

协议：**OpenAI Chat** · 端点：`/api/v1`

```bash
export OPENAI_API_BASE="https://ai.svtun.cn/v1"
export OPENAI_API_KEY="sk-your-key"

aider --model gpt-5.6-sol
```

用 Claude 模型：

```bash
aider --model openai/claude-sonnet-5
```

---

### curl

**Anthropic 协议：**

```bash
curl https://ai.svtun.cn/v1/messages \
  -H "x-api-key: sk-your-key" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-5",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

**OpenAI 协议：**

```bash
curl https://ai.svtun.cn/v1/chat/completions \
  -H "Authorization: Bearer sk-your-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-5.6-sol",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

---

## 编辑器插件

### Cline

VS Code 扩展。支持两种模式。

**Anthropic 模式（推荐用 Claude 时）：**

| 字段 | 值 |
|---|---|
| API Provider | `Anthropic` |
| Anthropic API Key | `sk-your-key` |
| Use custom base URL | ✅ 勾选 |
| Base URL | `https://ai.svtun.cn` |

**OpenAI Compatible 模式：**

| 字段 | 值 |
|---|---|
| API Provider | `OpenAI Compatible` |
| Base URL | `https://ai.svtun.cn/v1` |
| API Key | `sk-your-key` |
| Model ID | `claude-sonnet-5` 或 `gpt-5.6-sol` |

---

### Roo Code

VS Code 扩展（Cline 的分支）。配置方式相同：

| 字段 | 值 |
|---|---|
| API Provider | `OpenAI Compatible` |
| Base URL | `https://ai.svtun.cn/v1` |
| API Key | `sk-your-key` |
| Model | 手动填模型名 |

---

### Continue

VS Code / JetBrains 扩展。`~/.continue/config.json`：

```json
{
  "models": [
    {
      "title": "Claude Sonnet 5",
      "provider": "openai",
      "model": "claude-sonnet-5",
      "apiBase": "https://ai.svtun.cn/v1",
      "apiKey": "sk-your-key"
    },
    {
      "title": "GPT-5.6",
      "provider": "openai",
      "model": "gpt-5.6-sol",
      "apiBase": "https://ai.svtun.cn/v1",
      "apiKey": "sk-your-key"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Haiku",
    "provider": "openai",
    "model": "claude-haiku-4-5",
    "apiBase": "https://ai.svtun.cn/v1",
    "apiKey": "sk-your-key"
  }
}
```

---

### Cursor

`Settings → Models → OpenAI API Key → Override OpenAI Base URL`

| 字段 | 值 |
|---|---|
| Base URL | `https://ai.svtun.cn/v1` |
| API Key | `sk-your-key` |

然后在模型列表里 **Add Model**，手动填入模型名（如 `claude-sonnet-5`）。

> ⚠️ Cursor 覆盖 Base URL 后，其内置的 Composer / Tab 补全仍走官方通道，只有对话走你的网关。

---

### Zed

`~/.config/zed/settings.json`：

```json
{
  "language_models": {
    "openai": {
      "api_url": "https://ai.svtun.cn/v1",
      "available_models": [
        { "name": "claude-sonnet-5", "max_tokens": 200000 },
        { "name": "gpt-5.6-sol", "max_tokens": 128000 }
      ]
    }
  }
}
```

API Key 通过 `Assistant → Settings` 面板填入。

---

## 桌面客户端

### Cherry Studio

`设置 → 模型服务 → 添加提供商`

| 字段 | 值 |
|---|---|
| 提供商类型 | `OpenAI` |
| API 地址 | `https://ai.svtun.cn` |
| API 密钥 | `sk-your-key` |

> 💡 Cherry Studio 会自动补 `/v1`，所以这里填到域名就行，**不要**再写 `/v1`。

添加模型时手动输入模型 ID（`claude-opus-5`、`gpt-5.6-sol` 等）。

---

### ChatBox

`设置 → 模型提供方 → OpenAI API`

| 字段 | 值 |
|---|---|
| API 域名 | `https://ai.svtun.cn` |
| API 密钥 | `sk-your-key` |
| 模型 | 自定义填写 |

---

### LobeChat

环境变量方式（Docker 部署）：

```bash
OPENAI_API_KEY=sk-your-key
OPENAI_PROXY_URL=https://ai.svtun.cn/v1
OPENAI_MODEL_LIST=+claude-opus-5,+claude-sonnet-5,+gpt-5.6-sol,+gpt-5.5
```

Docker Compose：

```yaml
services:
  lobe-chat:
    image: lobehub/lobe-chat
    ports:
      - "3210:3210"
    environment:
      OPENAI_API_KEY: sk-your-key
      OPENAI_PROXY_URL: https://ai.svtun.cn/v1
      OPENAI_MODEL_LIST: "+claude-opus-5,+claude-sonnet-5,+gpt-5.6-sol"
      ACCESS_CODE: your-access-code
```

---

### NextChat

环境变量：

```bash
OPENAI_API_KEY=sk-your-key
BASE_URL=https://ai.svtun.cn
CUSTOM_MODELS=+claude-opus-5,+claude-sonnet-5,+gpt-5.6-sol
```

或在网页设置里填「接口地址」和「API Key」。

---

## Web / 自托管

### Open WebUI

`Settings → Connections → OpenAI API`

| 字段 | 值 |
|---|---|
| API Base URL | `https://ai.svtun.cn/v1` |
| API Key | `sk-your-key` |

Docker 部署时直接给环境变量：

```bash
docker run -d -p 3000:8080 \
  -e OPENAI_API_BASE_URL=https://ai.svtun.cn/v1 \
  -e OPENAI_API_KEY=sk-your-key \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main
```

---

### Dify

`设置 → 模型供应商 → OpenAI-API-compatible`

| 字段 | 值 |
|---|---|
| 模型名称 | `claude-sonnet-5` |
| API Key | `sk-your-key` |
| API endpoint URL | `https://ai.svtun.cn/v1` |
| Completion mode | `Chat` |
| 模型上下文长度 | `200000` |
| 最大 token 上限 | `8192` |

每个模型都要单独添加一次。

---

## SDK

### OpenAI SDK · Python

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://ai.svtun.cn/v1",
    api_key="sk-your-key",
)

resp = client.chat.completions.create(
    model="gpt-5.6-sol",
    messages=[{"role": "user", "content": "Hello"}],
)
print(resp.choices[0].message.content)
```

流式：

```python
stream = client.chat.completions.create(
    model="claude-sonnet-5",
    messages=[{"role": "user", "content": "写一首短诗"}],
    stream=True,
)
for chunk in stream:
    delta = chunk.choices[0].delta.content
    if delta:
        print(delta, end="", flush=True)
```

---

### OpenAI SDK · Node.js

```javascript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://ai.svtun.cn/v1",
  apiKey: process.env.LEONIS_API_KEY,
});

const resp = await client.chat.completions.create({
  model: "gpt-5.6-sol",
  messages: [{ role: "user", content: "Hello" }],
});

console.log(resp.choices[0].message.content);
```

---

### Anthropic SDK · Python

```python
from anthropic import Anthropic

client = Anthropic(
    base_url="https://ai.svtun.cn",
    auth_token="sk-your-key",
)

msg = client.messages.create(
    model="claude-opus-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}],
)
print(msg.content[0].text)
```

带 Prompt 缓存（长上下文场景强烈建议）：

```python
msg = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1024,
    system=[{
        "type": "text",
        "text": very_long_context,
        "cache_control": {"type": "ephemeral"},   # 标记为可缓存
    }],
    messages=[{"role": "user", "content": "基于上面的内容回答..."}],
)
```

---

### Anthropic SDK · Node.js

```javascript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  baseURL: "https://ai.svtun.cn",
  authToken: process.env.LEONIS_API_KEY,
});

const msg = await client.messages.create({
  model: "claude-opus-5",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello" }],
});

console.log(msg.content[0].text);
```

---

### LangChain

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    base_url="https://ai.svtun.cn/v1",
    api_key="sk-your-key",
    model="claude-sonnet-5",
    temperature=0.7,
)

print(llm.invoke("Hello").content)
```

用 Anthropic 原生协议：

```python
from langchain_anthropic import ChatAnthropic

llm = ChatAnthropic(
    anthropic_api_url="https://ai.svtun.cn",
    anthropic_api_key="sk-your-key",
    model="claude-opus-5",
)
```

---

### LlamaIndex

```python
from llama_index.llms.openai_like import OpenAILike

llm = OpenAILike(
    api_base="https://ai.svtun.cn/v1",
    api_key="sk-your-key",
    model="claude-sonnet-5",
    context_window=200000,
    is_chat_model=True,
)

print(llm.complete("Hello"))
```

---

## 自动化

### n8n

添加 **OpenAI** 凭据：

| 字段 | 值 |
|---|---|
| API Key | `sk-your-key` |
| Base URL | `https://ai.svtun.cn/v1` |

之后 `OpenAI Chat Model` 节点里手动填模型名即可。

---

## 排错

| 现象 | 原因 | 解决 |
|---|---|---|
| `404 Not Found` | 端点路径不对 | 检查是否多写或少写了 `/v1`；Cherry Studio 会自动补 `/v1` |
| `401 Unauthorized` | Key 错误或变量名不对 | Claude Code 用 `ANTHROPIC_AUTH_TOKEN` 而非 `ANTHROPIC_API_KEY` |
| 配了 Base URL 还是走官方 | 存在优先级更高的变量或本地登录凭据 | `unset ANTHROPIC_API_KEY`；`claude logout` |
| `400 model not found` | 模型名拼写错误 | 用完整官方模型名，如 `claude-sonnet-5` |
| 流式输出卡住 | 反代未正确透传 SSE | 确认网关支持 `text/event-stream` 且未开缓冲 |
| 环境变量不生效 | 未重载配置 | `source ~/.zshrc` 或重开终端；IDE 需重启 |

**快速自检：**

```bash
curl -s https://ai.svtun.cn/v1/chat/completions \
  -H "Authorization: Bearer $YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-5.6-sol","messages":[{"role":"user","content":"hi"}],"max_tokens":16}'
```

能返回 JSON 就说明网关和 Key 都没问题，剩下的是客户端配置的事。

---

## 相关项目

| 仓库 | 说明 |
|---|---|
| [claude-code-guide](https://github.com/leonis-ai/claude-code-guide) | Claude Code 中文完全指南 |
| [codex-cli-guide](https://github.com/leonis-ai/codex-cli-guide) | Codex CLI 完全配置手册 |
| [gemini-api-guide](https://github.com/leonis-ai/gemini-api-guide) | Gemini API 中文配置手册 |
| [cc-switch-guide](https://github.com/leonis-ai/cc-switch-guide) | 多配置一键切换 |
| [awesome-ai-api-gateway](https://github.com/leonis-ai/awesome-ai-api-gateway) | AI 网关与中转生态精选 |
| [ai-api-pricing](https://github.com/leonis-ai/ai-api-pricing) | 成本计算与缓存经济学 |
| [ai-client-configs](https://github.com/leonis-ai/ai-client-configs) | 20+ 客户端配置模板 |
| [全部 114 个模型](https://leonis-ai.github.io/models.html) | 在线可搜索模型清单 |

---

## 贡献

缺了你在用的客户端？欢迎提 PR。请附上：客户端名称、协议类型、配置字段与截图或代码片段。

## License

[MIT](LICENSE)

---

<sub>

**关键词** · AI 客户端配置 · Base URL 配置 · Claude Code 配置 · Codex CLI 配置 · Cline 配置 ·
Cherry Studio 配置 · ChatBox 配置 · LobeChat · NextChat · Open WebUI · Dify · AI 中转 · API 中转 · 反代配置

</sub>
