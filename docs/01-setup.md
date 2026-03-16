# 01. OpenClaw 安装与配置

本指南将帮助你完成 OpenClaw 的完整安装和配置。

## 环境要求

| 要求 | 最低版本 |
|------|---------|
| Node.js | 18.x |
| 操作系统 | Windows / macOS / Linux |
| 内存 | 4GB+ |
| 磁盘空间 | 1GB+ |

## 安装步骤

### 1. 安装 Node.js

**macOS / Linux:**
```bash
curl -fsSL https://fnm.vercel.app/install | bash
fnm install 20
fnm use 20
```

**Windows:**
```powershell
# 方法 1: 使用 winget
winget install OpenJS.NodeJS.LTS

# 方法 2: 使用 nvm-windows
# 下载安装: https://github.com/coreybutler/nvm-windows
nvm install 20
nvm use 20
```

验证安装：
```bash
node --version  # 应显示 v20.x.x
npm --version   # 应显示 10.x.x
```

### 2. 安装 OpenClaw

```bash
# 全局安装
npm install -g openclaw

# 验证安装
openclaw --version
```

### 3. 初始化配置

```bash
# 初始化
openclaw init

# 按提示完成配置
# - 选择 AI 模型 (OpenAI / Anthropic / Ollama / MiniMax)
# - 配置 API Key
# - 选择渠道 (Telegram / Discord / 飞书 等)
```

### 4. 配置文件说明

安装完成后，会在用户目录生成配置文件：

```
~/.openclaw/
├── config.yaml      # 主配置文件
├── skills/          # 技能目录
├── memory/         # 记忆存储
└── channels/       # 渠道配置
```

### 5. 配置示例

```yaml
# config.yaml
model:
  provider: openai  # 或 anthropic, ollama, minimax
  apiKey: ${OPENAI_API_KEY}
  
channels:
  - type: telegram
    enabled: true
    token: ${TELEGRAM_BOT_TOKEN}
    
memory:
  enabled: true
  type: local  # 或 sqlite
```

## 启动 OpenClaw

```bash
# 启动服务
openclaw start

# 或以后台模式运行
openclaw start --daemon
```

启动成功后，你会看到：

```
🚀 OpenClaw 已启动
📡 渠道状态:
   ✅ Telegram: 已连接
   ✅ Webchat: 已连接
💡 发送消息开始使用
```

## 常见问题

### Q: 提示 "command not found"
A: 检查 Node.js 和 npm 是否正确安装，或尝试重启终端

### Q: API Key 怎么获取？
A: 
- OpenAI: https://platform.openai.com/api-keys
- Anthropic: https://console.anthropic.com/
- MiniMax: https://platform.minimaxi.com/

### Q: 如何查看日志？
```bash
openclaw logs
openclaw logs --tail 100  # 查看最近 100 行
```

## 下一步

- [02. 第一个任务](./02-first-task.md) — 发送你的第一条指令
- [03. 渠道配置](./03-channels.md) — 配置更多渠道
