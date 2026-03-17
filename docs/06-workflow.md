# 06. 工作流自动化 — 告别重复任务

> **💡 Motto**: "一次设置，AI 天天帮你干"

这节教程的目标：设置自动化工作流，让 AI 定时帮你做事，不用每次手动触发。

---

## 🧩 这节学什么

| 工作流 | 预计时间 | 难度 |
|--------|---------|------|
| 每日早报 | 5 分钟 | ⭐ |
| 定时提醒 | 5 分钟 | ⭐ |
| 自动回复 | 10 分钟 | ⭐⭐ |
| GitHub 通知 | 10 分钟 | ⭐⭐ |

---

## 🎯 目标：做几个实用工作流

学完这节，你可以设置：
- 每天早上 8 点自动发送新闻早报
- 定时提醒你喝水/休息
- GitHub 有新 issue 自动通知

---

## 🚀 场景 1：每日早报（每天早上 8 点自动发）

### 目标
每天早上 8 点，自动给你发送今日新闻/天气/待办。

### 配置

```yaml
# ~/.openclaw/config.yaml
workflows:
  - name: morning-report
    enabled: true
    trigger:
      type: cron
      expression: "0 8 * * *"  # 每天早上 8:00
    actions:
      - type: message
        channel: feishu  # 或 telegram, discord
        content: |
          ☀️ 早上好，杜博士！
          
          今日待办：
          - [ ] 查看邮件
          - [ ] 代码 review
          
          天气：北京 25°C 晴
```

### 启动

```bash
openclaw restart
```

**验证：** 等待第二天早上 8 点，或者手动测试：

```bash
# 手动触发测试
openclaw workflows run morning-report
```

---

## 🚀 场景 2：定时提醒（每小时提醒）

### 目标
每小时提醒你喝水、休息、活动一下。

### 配置

```yaml
workflows:
  - name: health-reminder
    enabled: true
    trigger:
      type: cron
      expression: "0 * * * *"  # 每小时整点
    actions:
      - type: message
        channel: feishu
        content: |
          💧 提醒：该喝水啦！
          
          已经工作 {{ elapsed_hours }} 小时了，休息一下？
```

---

## 🚀 场景 3：自动回复（收到特定消息自动响应）

### 目标
当有人在群里 @AI 时，自动回复。

### 配置

```yaml
workflows:
  - name: auto-reply
    enabled: true
    trigger:
      type: event
      source: feishu
      event: message
    conditions:
      - contains: "帮助"
    actions:
      - type: message
        channel: feishu
        content: |
          👋 你好！我是 AI 助手
          
          我可以帮你：
          - 查资料、写代码
          - 管理文件、执行命令
          - 设置提醒、管理任务
          
          直接告诉我要做什么就行！
```

---

## 🚀 场景 4：GitHub 通知（有人提 Issue 立即通知）

### 目标
当 GitHub 仓库有新 Issue 时，自动发送到飞书/Telegram。

### 配置

```yaml
workflows:
  - name: github-issue-notify
    enabled: true
    trigger:
      type: webhook
      path: /webhook/github
    actions:
      - type: condition
        expression: "{{ event.issue }} != null"
      - type: message
        channel: feishu
        content: |
          🐛 新 Issue！
          
          标题：{{ event.issue.title }}
          作者：{{ event.issue.user.login }}
          链接：{{ event.issue.html_url }}
```

### 配置 GitHub Webhook

1. 打开 GitHub 仓库 → Settings → Webhooks
2. 点击 "Add webhook"
3. Payload URL: `http://你的IP:端口/webhook/github`
4. Events: 选择 "Issues"

---

## 🚀 场景 5：定时总结（每周五下午总结本周工作）

### 目标
每周五下午 6 点，自动生成本周工作摘要。

### 配置

```yaml
workflows:
  - name: weekly-summary
    enabled: true
    trigger:
      type: cron
      expression: "0 18 * * 5"  # 每周五 18:00
    actions:
      - type: skill
        name: weekly-summary
      - type: message
        channel: feishu
        content: |
          📊 本周工作摘要
          
          {{ skill_result }}
          
          下周计划：
          - 
          -
```

---

## 🔧 Cron 表达式速查

| 表达式 | 含义 | 示例 |
|--------|------|------|
| `0 * * * *` | 每小时 | 每小时执行 |
| `0 9 * * *` | 每天 9 点 | 每日早报 |
| `0 9 * * 1-5` | 工作日 9 点 | 工作日提醒 |
| `0 18 * * 5` | 每周五 6 点 | 周总结 |
| `*/15 * * * *` | 每 15 分钟 | 频繁任务 |
| `0 0 1 * *` | 每月 1 号 | 月度报告 |

---

## 🧪 测试工作流

```bash
# 列出所有工作流
openclaw workflows list

# 查看工作流状态
openclaw workflows status

# 手动触发工作流
openclaw workflows run morning-report

# 查看工作流日志
openclaw workflows logs
openclaw workflows logs --tail 50
```

---

## ❓ 常见问题

### Q: 工作流没触发
- 检查 enabled 是否为 true
- 检查 cron 表达式是否正确
- 查看日志：`openclaw workflows logs`

### Q: 消息发不出去
- 检查渠道是否配置正确
- 检查 channel 名称是否正确

### Q: 如何关闭工作流？
```yaml
workflows:
  - name: xxx
    enabled: false  # 改为 false
```

---

## 🎉 这节目标达成

✅ 会设置定时任务（cron）  
✅ 会设置事件触发（webhook）  
✅ 会配置自动回复  
✅ 能监听 GitHub 通知  

---

## ➡️ 下节学什么

- [07. Skill 开发实战](./07-skill.md) — 更复杂的技能开发
- [08. 高级 Skill](./08-advanced-skill.md) — 多模态、外部 API
