# 03. 渠道配置

OpenClaw 支持多种渠道接入，本文介绍如何配置各类渠道。

## 支持的渠道

| 渠道 | 状态 | 文档 |
|------|------|------|
| Telegram | ✅ 已支持 | 本文档 |
| Discord | ✅ 已支持 | 本文档 |
| 飞书 | ✅ 已支持 | 本文档 |
| WhatsApp | ✅ 已支持 | 本文档 |
| Signal | ✅ 已支持 | 本文档 |
| Webchat | ✅ 已支持 | 本文档 |

## Telegram 配置

### 1. 创建 Bot

1. 打开 [@BotFather](https://t.me/BotFather)
2. 发送 `/newbot` 创建新机器人
3. 按提示设置名称和用户名
4. 获取 Bot Token

### 2. 配置 OpenClaw

```yaml
# config.yaml
channels:
  - type: telegram
    enabled: true
    token: ${TELEGRAM_BOT_TOKEN}
```

### 3. 重启服务

```bash
openclaw restart
```

### 4. 验证

1. 打开你创建的 Bot
2. 发送 `/start` 或任意消息
3. 应该收到回复

---

## Discord 配置

### 1. 创建应用

1. 打开 [Discord Developer Portal](https://discord.com/developers/applications)
2. 创建新应用
3. 在 "Bot" 页面创建 Bot 用户
4. 获取 Token

### 2. 添加到服务器

1. 在 "OAuth2" → "URL Generator"
2. 勾选 `bot` 和 `messages.read`
3. 生成 URL 并邀请到服务器

### 3. 配置 OpenClaw

```yaml
channels:
  - type: discord
    enabled: true
    token: ${DISCORD_BOT_TOKEN}
    guildId: ${YOUR_GUILD_ID}
```

---

## 飞书配置

### 1. 创建应用

1. 打开 [飞书开放平台](https://open.feishu.cn/)
2. 创建企业自建应用
3. 获取 App ID 和 App Secret

### 2. 添加权限

需要的权限：
- `im:message:send_as_bot` — 发送消息
- `im:message:receive` — 接收消息

### 3. 配置 OpenClaw

```yaml
channels:
  - type: feishu
    enabled: true
    appId: ${FEISHU_APP_ID}
    appSecret: ${FEISHU_APP_SECRET}
```

---

## 环境变量

建议使用环境变量管理敏感信息：

```bash
# Linux / macOS
export TELEGRAM_BOT_TOKEN="your-token-here"
export DISCORD_BOT_TOKEN="your-token-here"

# Windows
set TELEGRAM_BOT_TOKEN=your-token-here
```

或在 `config.yaml` 中使用 `${VAR_NAME}` 语法引用。

---

## 下一步

- [04. 自定义技能](./04-custom-skill.md) — 创建你自己的技能
- [05. 记忆系统](./05-memory.md) — 让 OpenClaw 记住上下文
