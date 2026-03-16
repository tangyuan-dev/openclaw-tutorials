# 09. 记忆系统进阶

本文深入介绍 OpenClaw 记忆系统的高级用法。

## 记忆类型详解

### 1. 会话记忆（Session Memory）

```javascript
// 获取当前会话
const session = context.memory.session;

// 存储临时数据
session.set('last_topic', 'weather');
session.set('pending_action', { type: 'confirm', value: 'yes' });

// 读取
const lastTopic = session.get('last_topic');
```

### 2. 用户记忆（User Memory）

```javascript
// 存储用户偏好
await context.memory.user.set('language', 'zh-CN');
await context.memory.user.set('timezone', 'Asia/Shanghai');
await context.memory.user.set('name', '张三');

// 读取
const name = await context.memory.user.get('name');
```

### 3. 项目记忆（Project Memory）

```javascript
// 存储项目信息
await context.memory.project.set('current_task', '开发用户模块');
await context.memory.project.set('tech_stack', ['React', 'Node.js']);

// 读取
const task = await context.memory.project.get('current_task');
```

## 记忆查询

### 1. 关键词搜索

```javascript
// 搜索记忆
const results = await context.memory.search('用户认证');

results.forEach(result => {
  console.log(result.content);
  console.log(result.score); // 相关度分数
});
```

### 2. 时间范围查询

```javascript
// 查询最近7天的记忆
const weekAgo = new Date();
weekAgo.setDate(weekAgo.getDate() - 7);

const memories = await context.memory.query({
  timeRange: {
    start: weekAgo,
    end: new Date()
  },
  type: 'user'
});
```

### 3. 标签查询

```javascript
const memories = await context.memory.query({
  tags: ['important', 'project']
});
```

## 记忆持久化

### 1. 自动持久化

```yaml
# config.yaml
memory:
  autoPersist: true
  persistInterval: 60000  # 毫秒
```

### 2. 手动持久化

```javascript
// 立即保存
await context.memory.persist();

// 保存特定类型的记忆
await context.memory.user.persist();
```

## 高级用法

### 1. 记忆总结

```javascript
// 自动总结长对话
module.exports = {
  async onSessionEnd(context) {
    const memories = await context.memory.session.getAll();
    
    // 提取关键信息
    const summary = await this.summarize(memories);
    
    // 存储为长期记忆
    await context.memory.user.append('session_summaries', summary);
  },
  
  async summarize(memories) {
    // 调用 LLM 总结
    return '用户今天完成了：1. 设置偏好 2. 查询天气';
  }
};
```

### 2. 跨会话上下文

```javascript
// 记住之前的任务进度
module.exports = {
  async handle(context) {
    // 检查之前的任务
    const task = await context.memory.user.get('current_task');
    
    if (task && task.status === 'in_progress') {
      return `继续之前的任务：${task.name}\n进度：${task.progress}`;
    }
    
    // 新任务
    return '请告诉我你想做什么？';
  }
};
```

### 3. 偏好学习

```javascript
// 学习用户偏好
module.exports = {
  async handle(context) {
    const message = context.message;
    
    // 检测偏好
    if (message.includes('我喜欢')) {
      const preference = this.extractPreference(message);
      await context.memory.user.set(`pref_${preference.key}`, preference.value);
    }
    
    // 应用偏好
    const language = await context.memory.user.get('pref_language') || '中文';
    const response = await this.generateResponse(context, language);
    
    return response;
  },
  
  extractPreference(message) {
    // 提取偏好
    if (message.includes('我喜欢英文')) {
      return { key: 'language', value: 'English' };
    }
    return null;
  }
};
```

## 配置选项

```yaml
memory:
  # 启用记忆
  enabled: true
  
  # 存储类型
  storage: sqlite  # local / sqlite / redis
  
  # SQLite 配置
  sqlite:
    path: ~/.openclaw/memory.db
    
  # Redis 配置（企业版）
  redis:
    host: localhost
    port: 6379
    
  # 记忆保留策略
  retention:
    session: 24h      # 会话记忆保留24小时
    user: 30d         # 用户记忆保留30天
    project: 90d       # 项目记忆保留90天
    
  # 自动清理
  autoCleanup:
    enabled: true
    interval: 86400   # 每天清理
```

## 最佳实践

1. **不要存储敏感信息** — 密码、密钥等
2. **定期清理无用记忆** — 保持性能
3. **使用标签分类** — 便于检索
4. **合理设置过期时间** — 平衡性能和准确性

## 下一步

- [10. 工作流自动化进阶](./10-workflow-advanced.md) — 高级工作流
