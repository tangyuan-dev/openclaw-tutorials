# 09. 记忆系统进阶 — 向量存储、语义搜索

> **💡 Motto**: "让 AI 记住得更多、理解得更深"

这节教程的目标：深入掌握记忆系统，学会向量存储、语义搜索、跨会话上下文等高级功能。

---

## 🧩 这节学什么

| 技能 | 预计时间 | 难度 |
|------|---------|------|
| 会话/用户/项目记忆 | 5 分钟 | ⭐ |
| 记忆搜索 | 10 分钟 | ⭐⭐ |
| 记忆总结 | 10 分钟 | ⭐⭐ |
| 向量语义搜索 | 20 分钟 | ⭐⭐⭐ |

---

## 🎯 目标：构建"第二大脑"

学完这节，AI 将能：
- 记住你告诉它的一切
- 按语义搜索记忆
- 自动总结对话
- 跨会话保持上下文

---

## 🚀 记忆类型详解

### 1. 会话记忆（当前会话）

```javascript
// 当前会话内的临时记忆
const session = context.memory.session;

// 存储
session.set('last_topic', 'weather');
session.set('pending_action', { type: 'confirm', value: 'yes' });

// 读取
const lastTopic = session.get('last_topic');
```

### 2. 用户记忆（永久）

```javascript
// 用户级别的长期记忆
await context.memory.user.set('name', '杜博士');
await context.memory.user.set('language', 'python');
await context.memory.user.set('favorite_theme', 'dark');

// 读取
const name = await context.memory.user.get('name');
```

### 3. 项目记忆（项目级别）

```javascript
// 项目相关记忆
await context.memory.project.set('current_task', '开发用户模块');
await context.memory.project.set('tech_stack', ['React', 'Node.js']);
await context.memory.project.set('database', 'PostgreSQL');

// 读取
const task = await context.memory.project.get('current_task');
```

---

## 🚀 记忆搜索

### 1. 关键词搜索

```javascript
// 搜索记忆
const results = await context.memory.search('用户认证');

results.forEach(result => {
  console.log(result.content);  // 记忆内容
  console.log(result.score);     // 相关度 0-1
  console.log(result.timestamp); // 时间
});
```

### 2. 按时间范围

```javascript
// 查询最近 7 天
const weekAgo = new Date();
weekAgo.setDate(weekAgo.getDate() - 7);

const memories = await context.memory.query({
  timeRange: {
    start: weekAgo,
    end: new Date()
  }
});
```

### 3. 按标签查询

```javascript
// 搜索带标签的记忆
const memories = await context.memory.query({
  tags: ['important', 'project']
});
```

---

## 🚀 记忆总结（自动摘要）

### 场景
每次会话结束，AI 自动总结并保存关键信息。

```javascript
// skills/session-summary/index.js

module.exports = {
  name: 'session-summary',
  description: '会话总结技能',
  
  // 会话结束时自动调用
  async onSessionEnd(context) {
    // 1. 获取会话所有消息
    const messages = await context.memory.session.getAll();
    
    // 2. 用 AI 总结
    const summary = await this.summarize(messages);
    
    // 3. 保存到用户记忆
    await context.memory.user.append('session_summaries', {
      date: new Date().toISOString(),
      summary: summary
    });
    
    console.log('会话总结已保存:', summary);
  },
  
  // 用 LLM 总结
  async summarize(messages) {
    const content = messages.map(m => m.content).join('\n');
    
    // 实际项目中调用 LLM
    // const response = await llm.chat(`
    //   请总结以下对话的要点：
    //   ${content}
    // `);
    
    // 模拟
    return `
## 会话总结
    
完成的任务：
- 设置了用户偏好
- 查询了北京天气
    
重要信息：
- 用户叫杜博士
- 喜欢 Python
    
下次继续：
- 继续开发博客项目
`.trim();
  }
};
```

---

## 🚀 跨会话上下文

### 场景：记住上次做到哪了

```javascript
// skills/continue-task/index.js

module.exports = {
  name: 'continue-task',
  description: '继续之前的任务',
  
  keywords: ['继续', '之前', '上次的'],
  
  async handle(context) {
    const message = context.message;
    const userId = context.userId;
    
    // 检查是否有未完成的任务
    const task = await context.memory.user.get('current_task');
    
    if (!task) {
      return '我没有找到你之前未完成的任务';
    }
    
    // 用户想继续
    if (message.includes('继续') || message.includes('是的')) {
      return `好的，继续 "${task.name}"
      
进度：${task.progress}
      
下一步：${task.next_step}`;
    }
    
    // 用户想知道状态
    return `📋 你之前的任务：
      
名称：${task.name}
进度：${task.progress}
时间：${task.updatedAt}
      
继续这个任务请说"继续"
`;
  },
  
  // 更新任务进度（可在其他技能中调用）
  async updateProgress(userId, task) {
    task.updatedAt = new Date().toISOString();
    await context.memory.user.set('current_task', task);
  }
};
```

---

## 🚀 向量语义搜索（高级）

### 场景
用自然语言搜索记忆，不只是关键词匹配。

```javascript
// skills/semantic-memory/index.js
// 需要安装向量数据库：npm install chromadb

const { ChromaClient } = require('chromadb');

module.exports = {
  name: 'semantic-memory',
  description: '语义记忆搜索',
  
  async initialize() {
    // 初始化向量数据库
    this.client = new ChromaClient();
    this.collection = await client.getOrCreateCollection('memories');
  },
  
  // 存储记忆（自动向量化）
  async store(context, content, metadata = {}) {
    // 实际项目中：
    // 1. 用 embedding API 获取向量
    // 2. 存储到 ChromaDB
    
    // 简化版：直接存文本
    const id = 'mem_' + Date.now();
    await this.collection.add({
      ids: [id],
      documents: [content],
      metadatas: [{ ...metadata, userId: context.userId }]
    });
    
    return id;
  },
  
  // 语义搜索
  async search(query, userId, topK = 3) {
    // 1. 把查询转成向量
    // const queryEmbedding = await getEmbedding(query);
    
    // 2. 向量搜索
    // const results = await this.collection.query({
    //   queryTexts: [query],
    //   nResults: topK,
    //   where: { userId: userId }
    // });
    
    // 模拟
    return [
      { content: '用户偏好 Python', score: 0.95 },
      { content: '用户叫杜博士', score: 0.88 },
      { content: '项目用 React + Node.js', score: 0.82 }
    ];
  },
  
  // 处理搜索请求
  async handle(context) {
    const message = context.message;
    const userId = context.userId;
    
    // 存储当前对话（可选）
    if (message.length > 20) {
      await this.store(context, message, { type: 'conversation' });
    }
    
    // 语义搜索
    const results = await this.search(message, userId);
    
    if (results.length === 0) {
      return '我没有找到相关的记忆';
    }
    
    // 返回结果
    const list = results.map((r, i) => 
      `${i + 1}. ${r.content} (${(r.score * 100).toFixed(0)}% 匹配)`
    ).join('\n');
    
    return `🔍 找到 ${results.length} 条相关记忆：\n\n${list}`;
  }
};
```

### 配置向量数据库

```yaml
# config.yaml
memory:
  vector:
    enabled: true
    provider: chromadb  # 或 pinecone, weaviate
    collection: memories
    
  storage: sqlite
```

---

## 🧪 测试记忆功能

```bash
openclaw restart

# 测试存储
"记住我叫杜博士"
"记住我偏好 Python"
"我现在在做博客项目，已经完成了用户模型"

# 测试读取
"我的名字是什么"
"继续之前的项目"

# 测试搜索
"我之前做了什么"
"搜索关于天气的记忆"
```

---

## ⚙️ 记忆配置

```yaml
memory:
  enabled: true
  storage: sqlite  # local / sqlite / redis
  
  # 存储位置
  sqlite:
    path: ~/.openclaw/memory.db
    
  # 记忆保留时间
  retention:
    session: 24h   # 会话 24 小时
    user: 30d     # 用户 30 天
    project: 90d   # 项目 90 天
    
  # 自动清理
  autoCleanup:
    enabled: true
    interval: 86400  # 每天
```

---

## 🎉 这节目标达成

✅ 会用三种记忆类型  
✅ 会搜索记忆  
✅ 会自动总结会话  
✅ 了解向量语义搜索  

---

## ➡️ 下节学什么

- [10. 工作流进阶](./10-workflow-advanced.md) — 高级触发器实战
- [11. 打造 AI 产品](./11-product.md) — 产品思维与变现
