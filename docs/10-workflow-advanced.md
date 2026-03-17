# 10. 工作流进阶 — 高级触发器与实战案例

> **💡 Motto**: "让 AI 自动干活，你就躺着"

这节教程的目标：掌握高级触发器（条件、循环、异步）、错误处理，以及 3 个真实企业级案例。

---

## 🧩 这节学什么

| 技能 | 预计时间 | 难度 |
|------|---------|------|
| 高级触发器 | 10 分钟 | ⭐⭐ |
| 条件分支与循环 | 10 分钟 | ⭐⭐ |
| 错误处理与重试 | 10 分钟 | ⭐⭐ |
| 实战案例 | 20 分钟 | ⭐⭐⭐ |

---

## 🎯 目标：构建自动化工作流

学完这节，你将能：
- 设置定时任务
- 监听 GitHub/其他服务的事件
- 处理条件分支
- 自动重试失败任务

---

## 🚀 高级触发器

### 1. 定时触发（Cron）

```yaml
workflows:
  - name: daily-report
    trigger:
      type: cron
      expression: "0 9 * * *"  # 每天早上 9 点
    actions:
      - type: message
        channel: feishu
        content: "早上好！新的一天开始了"
```

### 2. 事件触发（GitHub/Webhook）

```yaml
workflows:
  - name: github-notify
    trigger:
      type: event
      source: github
      event: push  # push / issue / pull_request
    actions:
      - type: message
        channel: telegram
        content: |
          🚀 新提交！
          作者：{{ event.pusher.name }}
          消息：{{ event.commits[0].message }}
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

---

## 🚀 高级动作

### 1. 条件分支

```yaml
workflows:
  - name: process-upload
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
  - name: batch-notify
    trigger:
      type: manual
    actions:
      - type: loop
        over: "{{ user_list }}"
        do:
          - type: message
            channel: telegram
            content: "通知：{{ item.message }}"
            to: "{{ item.user_id }}"
```

### 3. 错误处理

```yaml
workflows:
  - name: safe-api-call
    trigger:
      type: manual
    actions:
      - type: try
        do:
          - type: http
            url: https://api.example.com/data
            method: GET
        catch:
          - type: message
            channel: feishu
            content: "❌ 操作失败：{{ error.message }}"
          - type: log
            level: error
            message: "{{ error }}"
```

### 4. 重试机制

```yaml
workflows:
  - name: reliable-api
    trigger:
      type: manual
    actions:
      - type: http
        url: https://api.example.com/data
        retry:
          maxAttempts: 3
          delay: 1000  # 1秒
          backoff: exponential
```

### 5. 异步执行

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
            content: "✅ 处理完成"
```

---

## 🚀 实战案例 1：自动周报

### 目标
每周五下午 6 点，自动生成并发送周报。

```yaml
workflows:
  - name: weekly-report
    trigger:
      type: cron
      expression: "0 18 * * Friday"  # 每周五 18:00
    actions:
      # 1. 获取本周 GitHub 数据
      - type: skill
        name: github-stats
        input:
          period: weekly
      
      # 2. 获取日历数据
      - type: skill
        name: calendar-summary
        input:
          days: 7
      
      # 3. 生成周报
      - type: skill
        name: report-generator
        input:
          type: weekly
          data:
            github: "{{ github-stats.result }}"
            calendar: "{{ calendar-summary.result }}"
      
      # 4. 发送到用户偏好的渠道
      - type: branch
        conditions:
          - if: "{{ user.channel }} == 'telegram'"
            then:
              - type: message
                channel: telegram
                content: "{{ report-generator.result }}"
          - else:
              - type: message
                channel: feishu
                content: "{{ report-generator.result }}"
```

### 周报生成技能示例

```javascript
// skills/report-generator/index.js

module.exports = {
  name: 'report-generator',
  description: '周报/月报生成',
  
  async handle(context) {
    const type = context.input.type;  // weekly / monthly
    const data = context.input.data;
    
    // 生成报告
    const report = this.generate(type, data);
    
    return report;
  },
  
  generate(type, data) {
    const title = type === 'weekly' ? '📅 周报' : '📊 月报';
    const period = type === 'weekly' ? '本周' : '本月';
    
    return `
${title} (${period})

## 📈 完成的工作

${data.github.commits} 次提交
${data.github.issues} 个 Issue
${data.github.prs} 个 PR

## 📅 日程

${data.calendar.events.map(e => `- ${e}`).join('\n')}

## 🔜 下周计划

- 
- 

---
自动生成于 ${new Date().toLocaleString()}
`.trim();
  }
};
```

---

## 🚀 实战案例 2：智能客服

### 目标
用户发消息 → 自动分类 → 处理 → 回复。

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
                priority: high
          - if: "{{ classification.type }} == 'question'"
            then:
              - type: skill
                name: qa-generator
          - else:
              - type: message
                content: "已收到，我们会尽快处理"
      
      # 3. 发送回复
      - type: message
        channel: "{{ event.channel }}"
        content: "{{ response }}"
```

---

## 🚀 实战案例 3：数据同步

### 目标
每 2 小时从外部 API 同步数据到本地。

```yaml
workflows:
  - name: data-sync
    trigger:
      type: cron
      expression: "0 */2 * * *"  # 每 2 小时
    actions:
      # 1. 获取外部数据
      - type: http
        name: fetch-data
        url: https://api.example.com/products
        method: GET
        headers:
          Authorization: "Bearer {{ secrets.API_TOKEN }}"
      
      # 2. 处理数据
      - type: skill
        name: data-processor
        input:
          data: "{{ fetch-data.body }}"
      
      # 3. 存储到本地
      - type: database
        operation: upsert
        table: products
        data: "{{ data-processor.output }}"
      
      # 4. 发送通知
      - type: message
        channel: feishu
        content: "✅ 数据同步完成\n新增：{{ data-processor.added }} 条\n更新：{{ data-processor.updated }} 条"
      
      # 5. 错误处理
      - type: if
        condition: "{{ data-processor.failed }} > 0"
        then:
          - type: message
            channel: feishu
            content: "⚠️ 部分数据同步失败：{{ data-processor.failed }} 条"
```

---

## 🧪 测试工作流

```bash
# 列出所有工作流
openclaw workflows list

# 查看状态
openclaw workflows status

# 手动触发
openclaw workflows run weekly-report

# 查看日志
openclaw workflows logs
openclaw workflows logs weekly-report --tail 50
```

---

## ⚙️ 工作流配置模板

```yaml
workflows:
  - name: my-workflow
    enabled: true
    
    # 触发器
    trigger:
      type: cron  # cron / event / webhook
      expression: "0 9 * * *"
      
    # 条件（可选）
    conditions:
      - key: hour
        operator: in
        value: [9, 18]
      
    # 动作
    actions:
      - type: message
        channel: feishu
        content: "Hello"
        
    # 错误处理
    errorHandler:
      - type: message
        channel: feishu
        content: "❌ 工作流执行失败：{{ error }}"
```

---

## 🎉 这节目标达成

✅ 会用高级触发器  
✅ 会用条件分支和循环  
✅ 会处理错误和重试  
✅ 能构建企业级工作流  

---

## ➡️ 下节学什么

- [11. 打造 AI 产品](./11-product.md) — 产品思维与变现模式
- [12. 企业级部署](./12-enterprise.md) — Docker 与运维
