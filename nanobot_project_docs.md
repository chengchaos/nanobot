# nanobot 项目概览

**nanobot** 是一个超轻量级的个人 AI 助手框架，灵感来源于 OpenClaw。核心优势是用约 4,000 行代码（实际 3,966 行）实现了完整的 agent 功能，比 Clawdbot（43万+行）减少了 99%。

## 项目结构

```
nanobot/
├── agent/              # 🧠 核心 agent 逻辑
│   ├── loop.py        # Agent loop（LLM ↔ 工具执行循环）
│   ├── context.py     # Prompt 构建器
│   ├── memory.py      # 持久化记忆系统（MEMORY.md + HISTORY.md）
│   ├── skills.py      # Skills 加载器
│   ├── subagent.py    # 后台任务执行器
│   └── tools/         # 内置工具集
│       ├── base.py           # 工具抽象基类
│       ├── registry.py       # 工具注册表
│       ├── filesystem.py     # 文件系统操作（读/写/编辑/列表）
│       ├── shell.py          # Shell 命令执行
│       ├── web.py            # Web 搜索和抓取
│       ├── message.py        # 跨频道消息发送
│       ├── spawn.py          # 子 agent 启动
│       ├── cron.py           # 定时任务工具
│       └── mcp.py            # MCP (Model Context Protocol) 集成
├── skills/             # 🎯 预置 skills（github, weather, tmux, clawhub等）
├── channels/           # 📱 聊天平台集成
│   ├── base.py        # 渠道抽象基类
│   ├── manager.py     # 渠道管理器
│   ├── telegram.py    # Telegram
│   ├── discord.py     # Discord
│   ├── whatsapp.py    # WhatsApp (Node.js bridge)
│   ├── feishu.py      # 飞书
│   ├── mochat.py      # MoChat (Claw IM)
│   ├── dingtalk.py    # 钉钉
│   ├── slack.py       # Slack
│   ├── email.py       # Email (IMAP/SMTP)
│   ├── qq.py          # QQ
│   └── matrix.py      # Matrix (Element)
├── bus/                # 🚌 消息路由
│   ├── events.py      # 消息事件定义
│   └── queue.py       # 异步消息队列
├── cron/               # ⏰ 定时任务
│   ├── service.py     # Cron 服务
│   └── types.py       # Cron 任务类型定义
├── heartbeat/          # 💓 主动唤醒服务
│   └── service.py     # 心跳服务（每30分钟检查 HEARTBEAT.md）
├── providers/          # 🤖 LLM 提供商
│   ├── base.py        # LLM 提供商抽象基类
│   ├── registry.py    # 提供商注册表
│   ├── litellm_provider.py  # LiteLLM（支持多种提供商）
│   ├── openai_codex_provider.py  # OpenAI Codex (OAuth)
│   ├── custom_provider.py        # 自定义 OpenAI 兼容端点
│   └── transcription.py   # 语音转文字
├── session/            # 💬 对话会话管理
│   └── manager.py     # Session 管理（JSONL 持久化）
├── config/             # ⚙️ 配置
│   ├── schema.py      # Pydantic 配置模式
│   └── loader.py      # 配置加载器
├── cli/                # 🖥️ 命令行接口
│   └── commands.py    # Typer CLI 命令
└── templates/          # 模板文件
    └── memory/        # MEMORY.md 模板
```

## 核心架构

### 1. Agent Loop (`agent/loop.py`)
核心处理引擎，负责：
- 从消息总线接收消息
- 构建上下文（历史记录、记忆、skills）
- 调用 LLM
- 执行工具调用
- 发送响应

关键参数：
- `max_iterations`: 40（防止无限循环）
- `temperature`: 0.1（低温度提高可靠性）
- `memory_window`: 100（上下文窗口大小）

### 2. 消息总线 (`bus/queue.py`)
异步队列实现 channel 和 agent 的解耦：
- `inbound`: 通道 → agent
- `outbound`: agent → 通道

### 3. 记忆系统 (`agent/memory.py`)
两层记忆：
- **MEMORY.md**: 长期事实存储
- **HISTORY.md**: 可 grep 搜索的日志

通过 LLM 工具调用自动 consolidation，将旧消息摘要到文件中。

### 4. 工具系统 (`agent/tools/`)
所有工具继承自 `Tool` 抽象基类，支持：
- 文件操作（读/写/编辑/列表）
- Shell 命令执行
- Web 搜索（Brave API）
- Web 抓取
- 跨频道消息
- 子 agent 启动
- 定时任务
- MCP 集成

### 5. LLM 提供商 (`providers/`)
支持 15+ LLM 提供商：
通过 **Provider Registry** 机制添加新提供商只需 2 步：
1. 在 `nanobot/providers/registry.py` 添加 `ProviderSpec`
2. 在 `nanobot/config/schema.py` 添加配置字段

支持的提供商：OpenRouter, Anthropic, OpenAI, DeepSeek, Groq, Gemini, MiniMax, Zhipu, Dashscope, VolcEngine, Moonshot, SiliconFlow, AihubMix, vLLM, Custom, OpenAI Codex (OAuth), GitHub Copilot (OAuth)

### 6. 渠道集成 (`channels/`)
支持 10+ 聊天平台：
- Telegram (推荐)
- Discord
- WhatsApp (Node.js bridge)
- 飞书 (WebSocket)
- MoChat (Claw IM)
- 钉钉 (Stream 模式)
- Slack (Socket 模式)
- Email (IMAP/SMTP)
- QQ (botpy SDK)
- Matrix (Element)

## 项目特点

### 1. 超轻量级
- ~4,000 行核心代码
- 99% smaller than Clawdbot (430k+ lines)
- 快速启动、低资源占用

### 2. 研究友好
- 代码清晰易读
- 模块化设计
- 易于修改和扩展

### 3. 易用性
```bash
pip install nanobot-ai
nanobot onboard          # 初始化配置
nanobot agent -m "Hi!"   # 单次消息
nanobot agent            # 交互模式
nanobot gateway          # 启动网关（多频道）
```

### 4. MCP 支持
支持 Model Context Protocol，可连接外部工具服务器：
```json
{
  "tools": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
      }
    }
  }
}
```

### 5. 安全性
- `restrictToWorkspace`: 限制 agent 访问工作区目录
- `allowFrom`: 白名单用户列表
- 支持 OAuth 认证

### 6. Docker 支持
```bash
docker run -v ~/.nanobot:/root/.nanobot nanobot gateway
```

## 使用场景

根据 README，主要应用场景：
1. **24/7 市场分析** - 实时监控和报道
2. **全栈软件工程师** - 编码、部署、扩展
3. **日常任务管理** - 计划、自动化、组织
4. **个人知识助手** - 学习、记忆、推理

## 版本信息
- 当前版本: `0.1.4.post2`
- Python 要求: ≥3.11
- Node.js 要求: ≥18（用于 WhatsApp bridge）

## 社区集成
支持加入 agent 社交网络：
- Moltbook
- ClawdChat

## 关键设计模式

### Provider Registry 模式
添加新提供商只需 2 步，无需修改 if-elif 链：
```python
# 1. 添加 ProviderSpec 到 registry.py
ProviderSpec(
    name="myprovider",
    keywords=("myprovider", "mymodel"),
    env_key="MYPROVIDER_API_KEY",
    display_name="My Provider",
    litellm_prefix="myprovider",
    skip_prefixes=("myprovider/",),
)

# 2. 添加字段到 ProvidersConfig schema.py
class ProvidersConfig(BaseModel):
    myprovider: ProviderConfig = ProviderConfig()
```

### 工具注册模式
所有工具继承 `Tool` 基类，实现统一接口：
```python
class MyTool(Tool):
    @property
    def name(self) -> str: ...

    @property
    def description(self) -> str: ...

    @property
    def parameters(self) -> dict: ...

    async def execute(self, **kwargs) -> str: ...
```

### 消息总线模式
通道和 agent 通过异步队列解耦：
```python
# 通道推送消息
await bus.publish_inbound(msg)

# Agent 消费消息
msg = await bus.consume_inbound()

# Agent 发送响应
await bus.publish_outbound(response)
```

## 命令行接口

| 命令 | 描述 |
|------|------|
| `nanobot onboard` | 初始化配置 & 工作空间 |
| `nanobot agent -m "..."` | 发送单条消息 |
| `nanobot agent` | 交互式聊天模式 |
| `nanobot gateway` | 启动网关（多频道） |
| `nanobot status` | 显示状态 |
| `nanobot cron list/add/remove` | 定时任务管理 |
| `nanobot channels login/status` | 渠道管理 |

## 配置文件

位置：`~/.nanobot/config.json`

主要配置段：
- `agents.defaults`: 模型、提供商、温度、token限制
- `providers.*`: 各LLM提供商的API密钥
- `channels.*`: 各聊天平台的连接配置
- `tools`: 工具配置（Web搜索、MCP服务器等）
- `gateway`: 网关设置（端口、心跳间隔）
