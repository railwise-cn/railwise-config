# RAILWISE-CLI 配置指南

[RAILWISE-CLI](https://github.com/railwise-cn/RAILWISE-CLI) 的配置文件模板与详细说明。

本仓库包含配置文件模板：

| 文件 | 用途 | 放置位置 |
|------|------|---------|
| `railwise.jsonc` | RAILWISE-CLI 主配置 | 项目根目录 `.railwise/railwise.jsonc` |

---

## 快速开始

### 1. 安装 RAILWISE-CLI

```bash
npm install -g railwise-ai
```

### 2. 复制配置文件

```bash
# 进入你的项目目录
cd your-project

# 创建配置目录
mkdir -p .railwise

# 下载配置文件
curl -o .railwise/railwise.jsonc https://raw.githubusercontent.com/railwise-cn/railwise-config/main/railwise.jsonc
```

### 3. 填入 API Key

编辑 `.railwise/railwise.jsonc`，在对应 provider 的 `apiKey` 字段填入你的密钥。

### 4. 启动

```bash
railwise
```

---

## railwise.jsonc 详解

这是 RAILWISE-CLI 的主配置文件，控制模型选择、provider 接入和系统行为。

### 配置加载顺序（低 → 高优先级）

```
远程 .well-known/railwise
  ↓
全局 ~/.config/railwise/railwise.json
  ↓
环境变量 RAILWISE_CONFIG 指向的文件
  ↓
项目根目录 railwise.jsonc → railwise.json
  ↓
.railwise/ 目录下的配置
  ↓
环境变量 RAILWISE_CONFIG_CONTENT
```

高优先级配置会覆盖低优先级的同名字段。

### 核心字段

#### `model`

全局默认模型，格式为 `provider/model-id`。

```jsonc
// 付费（推荐，效果最好）
"model": "anthropic/claude-sonnet-4-20250514"

// 免费（零成本起步）
"model": "deepseek/deepseek-chat"
"model": "zhipuai/glm-4-flash-250414"
```

#### `default_agent`

启动后默认使用的智能体。RAILWISE-CLI 内置 7 个领域智能体：

| 智能体 | 默认模型 | 职责 |
|--------|---------|------|
| `chief_manager` | Kimi K2.5 | 项目总工，任务调度 |
| `solution_architect` | Kimi K2.5 | 方案设计 |
| `data_analyst` | DeepSeek V3 | 平差计算、数据分析 |
| `qa_inspector` | DeepSeek V3 | 外业数据质检 |
| `qa_reviewer` | Kimi K2.5 | 报告终审 |
| `technical_writer` | Kimi K2.5 | 报告编写 |
| `commercial_specialist` | Kimi K2.5 | 商务投标 |

> 每个智能体的默认模型已在其 `.railwise/agent/*.md` 的 frontmatter 中配置，无需在此文件设置。

#### `enabled_providers`

控制模型选择列表中显示哪些 provider。未列出的 provider 不会出现在 TUI 的模型切换菜单中。

```jsonc
"enabled_providers": ["anthropic", "openai", "gemini", "deepseek", "minimax", "kimi", "zhipuai"]
```

> 注意：`zhipuai` 和 `kimi` 的免费模型会自动绕过此过滤，即使不在列表中也会显示。

#### `disabled_providers`

禁用指定 provider（优先级最高，会覆盖 `enabled_providers`）。

```jsonc
"disabled_providers": ["openai"]
```

### Provider 配置

每个 provider 的基本结构：

```jsonc
"provider-id": {
  "options": {
    "baseURL": "API 端点地址",
    "apiKey": "你的密钥"
  },
  "models": {
    "model-id": {
      "name": "显示名称",
      "limit": { "context": 上下文长度, "output": 最大输出 },
      "modalities": { "input": ["text"], "output": ["text"] }
    }
  }
}
```

#### 使用 API 代理

如果直连国际模型有网络问题，可以将 `baseURL` 改为代理地址：

```jsonc
"anthropic": {
  "options": {
    "baseURL": "https://你的代理地址/anthropic",
    "apiKey": "代理平台的密钥"
  }
}
```

#### 国产模型特殊配置

**MiniMax 和 Kimi** 需要额外设置 `npm` 字段，因为它们使用 OpenAI 兼容接口但不在 models.dev 数据库中：

```jsonc
"minimax": {
  "npm": "@ai-sdk/openai-compatible",
  "api": "https://api.minimax.chat/v1",
  "options": { ... },
  "models": {
    "MiniMax-M1": {
      "provider": { "npm": "@ai-sdk/openai-compatible" },
      ...
    }
  }
}
```

### 免费模型一览

| 厂商 | 模型 | 免费额度 | 注册地址 |
|------|------|---------|---------|
| 智谱 GLM | `glm-4-flash-250414`、`glm-z1-flash` | **永久免费** | [open.bigmodel.cn](https://open.bigmodel.cn) |
| DeepSeek | `deepseek-chat` | 注册送 500 万 tokens | [platform.deepseek.com](https://platform.deepseek.com) |
| MiniMax | `MiniMax-M1`、`MiniMax-T1` | 注册送免费额度 | [platform.minimaxi.com](https://platform.minimaxi.com) |
| Kimi | `kimi-k2.5`、`moonshot-v1-auto` | 注册送免费额度 | [platform.moonshot.cn](https://platform.moonshot.cn) |

> **零成本方案**：只注册智谱 GLM（永久免费），将 `model` 设为 `"zhipuai/glm-4-flash-250414"` 即可开始使用。

### MCP 配置

MCP（Model Context Protocol）用于接入外部服务。最常用的是飞书集成：

```bash
# 自动配置飞书 MCP（交互式引导）
railwise feishu
```

手动配置示例：

```jsonc
"mcp": {
  "feishu": {
    "command": "npx",
    "args": ["-y", "@larksuiteoapi/lark-mcp", "--app-id", "YOUR_APP_ID", "--app-secret", "YOUR_APP_SECRET"]
  }
}
```

---

## 典型配置方案

### 方案一：零成本（全免费）

只注册智谱 GLM，所有模型用免费的 `glm-4-flash`：

```jsonc
{
  "model": "zhipuai/glm-4-flash-250414",
  "provider": {
    "zhipuai": {
      "options": {
        "baseURL": "https://open.bigmodel.cn/api/paas/v4",
        "apiKey": "你的智谱密钥"
      }
    }
  }
}
```

### 方案二：国产低价（推荐）

注册 DeepSeek + Kimi + 智谱，按场景分配：

```jsonc
{
  "model": "deepseek/deepseek-chat",
  "enabled_providers": ["deepseek", "kimi", "zhipuai"],
  "provider": {
    "deepseek": { "options": { "baseURL": "https://api.deepseek.com/v1", "apiKey": "..." } },
    "kimi": { "npm": "@ai-sdk/openai-compatible", "api": "https://api.moonshot.cn/v1", "options": { "baseURL": "https://api.moonshot.cn/v1", "apiKey": "..." } },
    "zhipuai": { "options": { "baseURL": "https://open.bigmodel.cn/api/paas/v4", "apiKey": "..." } }
  }
}
```

### 方案三：付费旗舰

使用 Claude + GPT 获得最佳效果：

```jsonc
{
  "model": "anthropic/claude-sonnet-4-20250514",
  "provider": {
    "anthropic": { "options": { "baseURL": "https://api.anthropic.com", "apiKey": "..." } },
    "openai": { "options": { "baseURL": "https://api.openai.com/v1", "apiKey": "..." } }
  }
}
```

---

## 常见问题

### Q: `railwise.json` 和 `railwise.jsonc` 有什么区别？

`.jsonc` 支持注释（`// ...`），`.json` 不支持。两者都能被系统识别，优先加载 `.jsonc`。推荐使用 `.jsonc`。

### Q: API Key 怎么安全存储？

将 `.railwise/railwise.jsonc` 和 `.railwise/railwise.json` 加入 `.gitignore`，不要提交到版本控制。或者使用首次启动时的交互式向导配置密钥（存储在 `~/.local/share/railwise/auth.json`）。

### Q: 智能体的模型和全局模型是什么关系？

每个智能体可以在 `.railwise/agent/*.md` 的 YAML frontmatter 中指定 `model` 字段。如果指定的模型可用，使用指定模型；如果不可用，回退到全局 `"model"` 字段。

### Q: 国内网络无法访问 models.dev 怎么办？

设置环境变量指向本地缓存文件：

```bash
export MODELS_DEV_API_JSON=~/.cache/railwise/models.json
```

首次需要手动下载 `https://models.dev/api.json` 到该路径。

### Q: npm 安装慢怎么办？

使用国内镜像：

```bash
npm config set registry https://registry.npmmirror.com
```

---

## 相关链接

- [RAILWISE-CLI 主仓库](https://github.com/railwise-cn/RAILWISE-CLI)
- [opencode 框架](https://github.com/sst/opencode)

## 许可

MIT
