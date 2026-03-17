# 03. 把 AI 接入微信/飞书/Telegram

> **💡 Motto**: "让 AI 随时随地响应你"

这节教程的目标：把 OpenClaw 接入飞书/Telegram/Discord，让 AI 随时在微信/飞书上帮你干活。

---

## 🧩 这节学什么

| 渠道 | 预计时间 | 难度 |
|------|---------|------|
| 飞书 | 10 分钟 | ⭐ |
| Telegram | 10 分钟 | ⭐ |
| Discord | 15 分钟 | ⭐⭐ |

---

## 🎯 场景：为什么要接渠道？

| 不接渠道 | 接入渠道 |
|---------|---------|
| 只能在终端里用 | 随时随地微信/飞书发指令 |
| 断开了就中断 | 24 小时在线 |
| 只能自己用 | 可以分享给朋友/同事 |

---

## 🚀 方式一：接入飞书（推荐，国内用户首选）

飞书是最佳选择，国内网络流畅，配置简单。

### 第一步：创建飞书应用

1. 打开 [飞书开放平台](https://open.feishu.cn/) → "创建应用"
2. 应用名称：`我的 AI 助手`
3. 获取 `App ID` 和 `App Secret`

### 第二步：添加权限

进入应用 → "权限管理"，添加：
- ✅ `im:message:send_as_bot` — 发送消息
- ✅ `im:message:receive` — 接收消息
- ✅ `im:chat:create` — 创建群聊（可选）

### 第三步：配置 OpenClaw

```yaml
# ~/.openclaw/config.yaml
channels:
  - type: feishu
    enabled: true
    appId: your-feishu-app-id
    appSecret: your-feishu-app-secret
```

### 第四步：启动并验证

```bash
openclaw restart
```

**验证方法：**
1. 打开飞书 → "工作台" → "自建应用"
2. 找到你的应用 → 点击进入
3. 发送消息，比如：`帮我写一个 hello world`
4. 应该收到 AI 回复

---

## 🚀 方式二：接入 Telegram

### 第一步：创建 Bot

1. 打开 [@BotFather](https://t.me/BotFather)
2. 发送 `/newbot`
3. 按提示设置名称（xxx_bot）
4. **复制 Bot Token**（形如 `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`）

### 第二步：配置 OpenClaw

```yaml
# ~/.openclaw/config.yaml
channels:
  - type: telegram
    enabled: true
    token: 你的-telegram-bot-token
```

### 第三步：启动并验证

```bash
openclaw restart
```

**验证方法：**
1. 打开 Telegram
2. 搜索你的 bot 名字
3. 发送 `/start` 或 `你好`
4. 应该收到回复

---

## 🚀 方式三：接入 Discord

### 第一步：创建应用

1. 打开 [Discord Developer Portal](https://discord.com/developers/applications)
2. 点击 "New Application"
3. 名称：`AI Assistant`

### 第二步：创建 Bot

1. 左侧点击 "Bot"
2. 点击 "Reset Token" 获取 Token
3. 开启 **Message Content Intent**（重要！）

### 第三步：邀请到服务器

1. 左侧 "OAuth2" → "URL Generator"
2. 勾选 `bot`
3. 权限输入：`10304`（读取消息 + 发送消息）
4. 复制生成的 URL，浏览器打开并选择服务器

### 第四步：配置 OpenClaw

```yaml
# ~/.openclaw/config.yaml
channels:
  - type: discord
    enabled: true
    token: 你的-discord-bot-token
```

### 第五步：启动并验证

```bash
openclaw restart
```

**验证方法：**
1. 进入服务器
2. 发送消息到频道
3. @你的 bot 名字
4. 应该收到回复

---

## 🔧 多渠道同时开启

```yaml
# ~/.openclaw/config.yaml
channels:
  - type: feishu
    enabled: true
    appId: xxx
    appSecret: xxx
    
  - type: telegram
    enabled: true
    token: xxx
    
  - type: discord
    enabled: true
    token: xxx
```

---

## ❓ 常见问题

### Q: 收不到消息回复
- 检查权限是否都开启了
- 重启服务：`openclaw restart`
- 查看日志：`openclaw logs`

### Q: 消息发不出去
- 检查 App ID/Secret 是否正确
- 检查是否已发布应用（飞书）

### Q: 如何 @机器人
- Telegram: 直接发消息，不用 @
- Discord: 需要 @机器人名字
- 飞书: 在应用消息里直接发

---

## 🎉 这节目标达成

✅ 会配置飞书/Telegram/Discord 渠道  
✅ AI 能在微信/飞书上响应  
✅ 了解多渠道配置  

---

## ➡️ 下节学什么

- [04. 自定义技能](./04-custom-skill.md) — 教 AI 更多技能
- [05. 记忆系统](./05-memory.md) — 让 AI 记住你的偏好
