# 01. 10 分钟做出你的第一个 AI 助手

> **💡 Motto**: "装好就能用，3 句话让 AI 帮你干活"

这节教程的目标：安装 OpenClaw，3 分钟让它跑起来，3 句话让它帮你完成第一个任务。

---

## 🧩 这节学什么

| 目标 | 预计时间 |
|------|---------|
| 安装 OpenClaw | 3 分钟 |
| 配置 AI 模型 | 2 分钟 |
| 发送第一条指令 | 1 分钟 |
| **总计** | **6 分钟** |

---

## 🚀 第一步：安装（Windows/macOS/Linux 通用）

```bash
# 1. 安装 Node.js (如果没有)
# Windows: https://nodejs.org/ 下载 LTS 版本
# macOS:  brew install node

# 2. 一行命令安装 OpenClaw
npm install -g openclaw

# 3. 验证安装
openclaw --version
```

**✅ 预期输出：**
```
openclaw v1.0.0
```

---

## ⚙️ 第二步：初始化（1 分钟）

```bash
# 初始化配置
openclaw init
```

会让你选择：
1. **AI 模型**：推荐选 `minimax`（国内免费速度快）或 `openai`
2. **API Key**：粘贴你的密钥

### 获取 API Key

| 模型 | 获取地址 |
|------|---------|
| MiniMax | https://platform.minimaxi.com/ |
| OpenAI | https://platform.openai.com/api-keys |
| Anthropic | https://console.anthropic.com/ |

> 💡 **学生/免费用户推荐 MiniMax**，国内直连，免费额度够用

---

## 🎯 第三步：启动并发送第一条指令

```bash
# 启动
openclaw start
```

启动成功后，终端会显示：
```
🚀 OpenClaw 已启动
📡 渠道: terminal
💡 输入你的第一条指令...
```

### 立即发送这 3 条指令：

**指令 1：让 AI 帮你写代码**
```
写一个 Python 脚本，读取当前文件夹所有 .txt 文件，统计每个文件有多少行
```

**指令 2：让 AI 帮你操作文件**
```
创建一个新文件叫 demo.py，内容是上面的代码
```

**指令 3：让 AI 帮你运行**
```
运行 demo.py
```

---

## 🔧 配置文件说明

安装后会在 `~/.openclaw/` 目录生成：

```
~/.openclaw/
├── config.yaml      # 主配置（模型、渠道、记忆）
├── skills/          # 你的技能扩展
├── memory/         # 记忆存储
└── logs/           # 运行日志
```

### config.yaml 完整示例

```yaml
# ~/.openclaw/config.yaml
model:
  provider: minimax
  apiKey: your-api-key-here
  model: abab6.5s-chat

# 可选：配置渠道
channels:
  - type: terminal  # 默认，已启用

# 可选：启用记忆（让 AI 记住你的偏好）
memory:
  enabled: true
```

---

## ❓ 常见问题

### Q: 提示 "command not found"
```bash
# Windows: 重启 PowerShell
# macOS/Linux: 检查 PATH
echo $PATH
```

### Q: API Key 报错
- 确认密钥复制完整（sk-xxx 开头）
- 检查余额是否充足
- 尝试切换模型

### Q: 响应太慢
- 使用 MiniMax（国内网络）
- 检查网络代理设置

---

## 🎉 这节目标达成

✅ 会安装 OpenClaw  
✅ 会配置 AI 模型  
✅ 已完成第一个任务  

---

## ➡️ 下节学什么

- [02. 你的第一个任务](./02-first-task.md) — 学习用自然语言指挥 AI 干各种活
- [03. 把 AI 接入微信/飞书/Telegram](./03-channels.md) — 让 AI 随时随地响应

---
