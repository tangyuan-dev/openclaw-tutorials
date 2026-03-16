# 06. 工作流自动化

本文介绍如何创建自动化工作流，让 OpenClaw 自动完成重复任务。

## 什么是工作流？

工作流是一系列自动化任务的组合，可以：
- 定时执行任务
- 响应特定事件
- 串联多个技能

## 基本示例

### 1. 定时任务

```yaml
# config.yaml
workflows:
  - name: daily-reminder
    trigger: cron("0 9 * * *")  # 每天早上 9 点
    actions:
      - type: message
        channel: telegram
        content: "早上好！今天的待办事项：..."
```

### 2. 事件触发

```yaml
workflows:
  - name: new-issue
    trigger: event(github.issue.created)
    actions:
      - type: webhook
        url: https://example.com/notify
        body: "{{ event }}"
```

## 工作流配置

### Cron 表达式

| 表达式 | 含义 |
|--------|------|
| `0 * * * *` | 每小时 |
| `0 9 * * *` | 每天早上 9 点 |
| `0 9 * * 1` | 每周一早上 9 点 |
| `*/15 * * * *` | 每 15 分钟 |

### 支持的触发器

```yaml
trigger:
  # 定时触发
  type: cron
  expression: "0 9 * * *"
  
  # 事件触发
  type: event
  source: github
  event: push
  
  # 手动触发
  type: manual
```

### 支持的操作

```yaml
actions:
  # 发送消息
  - type: message
    channel: telegram
    content: "任务完成！"
    
  # 调用 API
  - type: http
    method: POST
    url: https://example.com/api
    body:
      status: completed
      
  # 执行技能
  - type: skill
    name: daily-report
    input:
      date: "{{ date }}"
```

## 实战示例

### 示例 1：每日早报

```yaml
workflows:
  - name: morning-news
    trigger:
      type: cron
      expression: "0 8 * * *"
    actions:
      - type: skill
        name: fetch-news
      - type: message
        channel: telegram
        content: "{{ skill_result }}"
```

### 示例 2：GitHub 推送通知

```yaml
workflows:
  - name: github-push-notify
    trigger:
      type: event
      source: github
      event: push
    conditions:
      - branch: main
    actions:
      - type: message
        channel: discord
        content: |
          🚀 新提交！
          作者: {{ event.commit.author }}
          消息: {{ event.commit.message }}
```

### 示例 3：定时备份

```yaml
workflows:
  - name: backup-files
    trigger:
      type: cron
      expression: "0 2 * * *"  # 每天凌晨 2 点
    actions:
      - type: skill
        name: backup
        input:
          source: /data
          destination: /backup
```

## 监控和日志

```bash
# 查看工作流日志
openclaw workflows logs

# 查看特定工作流
openclaw workflows logs daily-reminder

# 测试工作流
openclaw workflows run daily-reminder
```

## 最佳实践

1. **设置合理的触发频率** — 避免过于频繁
2. **添加错误处理** — 失败时发送通知
3. **记录日志** — 方便调试和问题排查
4. **测试后再上线** — 先用手动触发测试

## 下一步

- [07. 安全配置](./07-security.md) — 企业级安全设置
