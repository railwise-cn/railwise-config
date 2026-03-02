# RAILWISE-CLI 配置指南

[RAILWISE-CLI](https://github.com/railwise-cn/RAILWISE-CLI) 的配置文件模板与详细说明。

本仓库包含两个配置文件：

| 文件 | 用途 | 放置位置 |
|------|------|---------|
| `railwise.jsonc` | RAILWISE-CLI 主配置 | 项目根目录 `.railwise/railwise.jsonc` |
| `oh-my-opencode.json` | oh-my-opencode 插件配置 | 项目根目录 `.railwise/oh-my-opencode.json` |

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
curl -o .railwise/oh-my-opencode.json https://raw.githubusercontent.com/railwise-cn/railwise-config/main/oh-my-opencode.json
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

## oh-my-opencode.json 详解

这是 [oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode) 插件的配置文件，用于给 RAILWISE-CLI 的内置子代理（subagent）分配不同的模型。

### 什么是子代理？

RAILWISE-CLI 内部有 6 个后台子代理（不同于上面的 7 个领域智能体），负责执行系统级任务：

| 子代理 | 职责 | 推荐模型 |
|--------|------|---------|
| `librarian` | 搜索外部文档、查找 API 用法 | 免费模型即可（GLM-4 Flash） |
| `explore` | 搜索项目代码、理解代码结构 | 需要一定推理能力（DeepSeek V3） |
| `oracle` | 架构设计、疑难问题咨询 | 需要最强推理（Kimi K2.5） |
| `frontend-ui-ux-engineer` | 前端界面设计与实现 | 需要创造力（Kimi K2.5） |
| `document-writer` | 技术文档编写 | 需要写作能力（Kimi K2.5） |
| `multimodal-looker` | 图片/PDF 分析 | 需要多模态能力（GLM-4 Flash） |

### 配置格式

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "agent-name": {
      "model": "provider/model-id"
    }
  }
}
```

模型 ID 格式与 `railwise.jsonc` 中的 `model` 字段一致：`provider/model-id`。

### 模型选择思路

```
高频 + 低成本任务（librarian、multimodal-looker）
  → 用免费模型：zhipuai/glm-4-flash-250414

中等复杂度任务（explore）
  → 用高性价比模型：deepseek/deepseek-chat

高复杂度任务（oracle、frontend、document-writer）
  → 用最强可用模型：kimi/kimi-k2.5 或 anthropic/claude-sonnet-4-20250514
```

---

## 典型配置方案

### 方案一：零成本（全免费）

只注册智谱 GLM，所有模型用免费的 `glm-4-flash`：

**railwise.jsonc**:
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

**oh-my-opencode.json**:
```json
{
  "agents": {
    "librarian": { "model": "zhipuai/glm-4-flash-250414" },
    "explore": { "model": "zhipuai/glm-4-flash-250414" },
    "oracle": { "model": "zhipuai/glm-4-flash-250414" },
    "frontend-ui-ux-engineer": { "model": "zhipuai/glm-4-flash-250414" },
    "document-writer": { "model": "zhipuai/glm-4-flash-250414" },
    "multimodal-looker": { "model": "zhipuai/glm-4-flash-250414" }
  }
}
```

### 方案二：国产低价（推荐）

注册 DeepSeek + Kimi + 智谱，按场景分配：

**railwise.jsonc**:
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

**oh-my-opencode.json**（本仓库提供的默认配置）:
```json
{
  "agents": {
    "librarian": { "model": "zhipuai/glm-4-flash-250414" },
    "explore": { "model": "deepseek/deepseek-chat" },
    "oracle": { "model": "kimi/kimi-k2.5" },
    "frontend-ui-ux-engineer": { "model": "kimi/kimi-k2.5" },
    "document-writer": { "model": "kimi/kimi-k2.5" },
    "multimodal-looker": { "model": "zhipuai/glm-4-flash-250414" }
  }
}
```

### 方案三：付费旗舰

使用 Claude + GPT 获得最佳效果：

**railwise.jsonc**:
```jsonc
{
  "model": "anthropic/claude-sonnet-4-20250514",
  "provider": {
    "anthropic": { "options": { "baseURL": "https://api.anthropic.com", "apiKey": "..." } },
    "openai": { "options": { "baseURL": "https://api.openai.com/v1", "apiKey": "..." } }
  }
}
```

**oh-my-opencode.json**:
```json
{
  "agents": {
    "librarian": { "model": "zhipuai/glm-4-flash-250414" },
    "explore": { "model": "anthropic/claude-sonnet-4-20250514" },
    "oracle": { "model": "openai/gpt-4.1" },
    "frontend-ui-ux-engineer": { "model": "anthropic/claude-sonnet-4-20250514" },
    "document-writer": { "model": "anthropic/claude-sonnet-4-20250514" },
    "multimodal-looker": { "model": "openai/gpt-4.1" }
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

### Q: oh-my-opencode.json 中的子代理和 .railwise/agent/ 中的智能体有什么区别？

两者完全独立：

- `.railwise/agent/*.md` 中的 7 个智能体是**业务层**的领域专家（项目总工、方案设计师等），直接面向用户
- `oh-my-opencode.json` 中的 6 个子代理是**系统层**的后台助手（代码搜索、文档查找等），由系统自动调用

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
- [oh-my-opencode 插件](https://github.com/code-yeongyu/oh-my-opencode)
- [opencode 框架](https://github.com/sst/opencode)

## 许可

MIT
