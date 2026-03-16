# 10. 工作流进阶

本文介绍 OpenClaw 工作流的高级用法。

## 高级触发器

### 1. 定时触发

```yaml
workflows:
  - name: daily-report
    trigger:
      type: cron
      expression: "0 9 * * *"  # 每天早上9点
    actions:
      - type: message
        content: "早上好！开始新的一天"
```

### 2. 事件触发

```yaml
workflows:
  - name: github-notify
    trigger:
      type: event
      source: github
      event: push
    actions:
      - type: message
        channel: telegram
        content: "有新提交：{{ event.commits[0].message }}"
```

### 3. Webhook 触发

```yaml
workflows:
  - name: webhook-handler
    trigger:
      type: webhook
      path: /webhook/github
    actions:
      - type: log
        message: "收到 webhook: {{ event }}"
```

### 4. 条件触发

```yaml
workflows:
  - name: smart-notify
    trigger:
      type: cron
      expression: "0 * * * *"
    conditions:
      - key: hour
        operator: in
        value: [9, 12, 18]  # 只在早中晚发
    actions:
      - type: message
        content: "到饭点了！"
```

## 高级动作

### 1. 条件分支

```yaml
workflows:
  - name: process-image
    trigger:
      type: webhook
      path: /upload
    actions:
      - type: branch
        conditions:
          - if: "{{ file.type }} == 'image'"
            then:
              - type: skill
                name: image-processor
          - else:
              - type: skill
                name: file-processor
```

### 2. 循环处理

```yaml
workflows:
  - name: batch-process
    trigger:
      type: manual
    actions:
      - type: loop
        over: "{{ items }}"
        do:
          - type: http
            method: POST
            url: "{{ item.url }}"
```

### 3. 错误处理

```yaml
workflows:
  - name: safe-operation
    trigger:
      type: manual
    actions:
      - type: try
        do:
          - type: http
            url: https://api.example.com/data
        catch:
          - type: message
            content: "操作失败，请重试"
          - type: log
            level: error
            message: "{{ error }}"
```

### 4. 异步执行

```yaml
workflows:
  - name: async-task
    trigger:
      type: manual
    actions:
      - type: async
        do:
          - type: http
            url: https://slow-api.example.com/process
        then:
          - type: message
            content: "处理完成"
```

## 实战案例

### 案例 1：自动周报

```yaml
workflows:
  - name: weekly-report
    trigger:
      type: cron
      expression: "0 18 * * Friday"  # 每周五下午6点
    actions:
      # 1. 获取本周数据
      - type: skill
        name: git-stats
      
      # 2. 生成周报
      - type: skill
        name: report-generator
        input:
          type: weekly
      
      # 3. 发送到多个渠道
      - type: branch
        conditions:
          - if: "{{ user.preference.channel }} == 'telegram'"
            then:
              - type: message
                channel: telegram
                content: "{{ report }}"
          - if: "{{ user.preference.channel }} == 'discord'"
            then:
              - type: message
                channel: discord
                content: "{{ report }}"
```

### 案例 2：智能客服

```yaml
workflows:
  - name: customer-service
    trigger:
      type: webhook
      path: /webhook/chat
    actions:
      # 1. 分析问题类型
      - type: skill
        name: message-classifier
      
      # 2. 根据类型处理
      - type: branch
        conditions:
          - if: "{{ classification.type }} == 'bug'"
            then:
              - type: skill
                name: bug-handler
              - type: ticket
                action: create
          - if: "{{ classification.type }} == 'question'"
            then:
              - type: skill
                name: qa-generator
          - else:
              - type: message
                content: "已收到您的消息，我们会尽快处理"
      
      # 3. 发送回复
      - type: message
        channel: "{{ event.channel }}"
        content: "{{ response }}"
```

### 案例 3：数据同步

```yaml
workflows:
  - name: data-sync
    trigger:
      type: cron
      expression: "0 */2 * * *"  # 每2小时
    actions:
      # 1. 从外部获取数据
      - type: http
        name: fetch-data
        url: https://api.example.com/data
        method: GET
      
      # 2. 处理数据
      - type: skill
        name: data-processor
        input:
          data: "{{ fetch-data.result }}"
      
      # 3. 存储
      - type: database
        operation: upsert
        table: items
        data: "{{ data-processor.output }}"
      
      # 4. 通知
      - type: message
        content: "数据同步完成，共 {{ data-processor.count }} 条"
```

## 监控和调试

```bash
# 查看工作流执行日志
openclaw workflows logs

# 查看特定工作流
openclaw workflows logs my-workflow

# 测试工作流
openclaw workflows run my-workflow --input '{"key": "value"}'

# 列出所有工作流
openclaw workflows list
```

## 最佳实践

1. **日志记录** — 方便排查问题
2. **错误处理** — 优雅处理失败
3. **重试机制** — 网络不稳定时自动重试
4. **监控告警** — 失败时及时通知

## 下一步

- [11. 案例集](./11-cases.md) — 真实企业案例
