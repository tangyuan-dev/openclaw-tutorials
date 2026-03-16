# 08. 高级 Skill 开发

本文介绍高级 Skill 开发技巧，包括多模态、长对话、外部 API 等。

## 高级技巧

### 1. 处理附件

```javascript
module.exports = {
  name: 'file-analyzer',
  description: '分析文件',
  
  match(context) {
    return context.attachments?.length > 0;
  },
  
  async handle(context) {
    const attachments = context.attachments;
    const results = [];
    
    for (const file of attachments) {
      const result = await this.analyze(file);
      results.push(result);
    }
    
    return this.format(results);
  },
  
  async analyze(file) {
    // 根据文件类型处理
    if (file.type.startsWith('image/')) {
      return { type: 'image', name: file.name, result: '图片分析结果...' };
    }
    if (file.type === 'application/pdf') {
      return { type: 'pdf', name: file.name, result: 'PDF 解析结果...' };
    }
    return { type: 'file', name: file.name, result: '已处理' };
  },
  
  format(results) {
    return results.map(r => 
      `📄 ${r.name}\n${r.result}`
    ).join('\n\n');
  }
};
```

### 2. 调用外部 API

```javascript
const axios = require('axios');

module.exports = {
  name: 'github-search',
  description: '搜索 GitHub 仓库',
  
  match(context) {
    return context.message.includes('搜索仓库') || 
           context.message.includes('找项目');
  },
  
  async handle(context) {
    const query = this.extractQuery(context.message);
    const repos = await this.searchRepos(query);
    
    return this.formatRepos(repos);
  },
  
  extractQuery(message) {
    return message
      .replace(/搜索仓库|找项目/g, '')
      .trim();
  },
  
  async searchRepos(query) {
    const response = await axios.get(
      `https://api.github.com/search/repositories`,
      {
        params: { q: query, sort: 'stars', order: 'desc' },
        headers: { Accept: 'application/vnd.github.v3+json' }
      }
    );
    
    return response.data.items.slice(0, 5);
  },
  
  formatRepos(repos) {
    return repos.map(r => 
      `⭐ ${r.stargazers_count} | [${r.full_name}](${r.html_url})\n${r.description}`
    ).join('\n\n');
  }
};
```

### 3. 多轮对话

```javascript
module.exports = {
  name: 'order',
  description: '点餐助手',
  
  match(context) {
    return context.message.includes('点餐') || 
           context.message.includes('订餐');
  },
  
  async handle(context) {
    // 检查是否有未完成的订单
    const order = await this.getOrder(context.userId);
    
    if (order) {
      // 继续未完成的订单
      return this.continueOrder(context, order);
    }
    
    // 新订单
    return this.startOrder(context);
  },
  
  async startOrder(context) {
    // 引导用户选择
    return `🍔 请选择餐品：
1. 汉堡
2. 披萨
3. 沙拉

请回复序号或名称`;
  },
  
  async continueOrder(context, order) {
    const choice = context.message;
    
    if (choice === '1') {
      order.items.push('汉堡');
    }
    // ...
    
    return `已添加到订单：${order.items.join(', ')}
    
确认下单请回复"确认"
继续添加请回复其他餐品`;
  },
  
  async getOrder(userId) {
    // 从数据库或缓存获取
    return null;
  }
};
```

### 4. 使用向量存储

```javascript
const { Chroma } = require('chromadb');

module.exports = {
  name: 'knowledge-base',
  description: '知识库问答',
  
  match(context) {
    return context.message.includes('?') || 
           context.message.includes('？');
  },
  
  async handle(context) {
    const query = context.message;
    
    // 1. 向量检索
    const results = await this.search(query);
    
    // 2. 构建上下文
    const context_text = results.map(r => r.text).join('\n');
    
    // 3. 调用 LLM 生成答案
    const answer = await this.askLLM(context_text, query);
    
    return answer;
  },
  
  async search(query) {
    // 使用向量数据库搜索
    const db = new Chroma();
    return await db.query('knowledge', query, 3);
  },
  
  async askLLM(context, query) {
    // 调用 OpenAI/Claude
    return '基于知识库的答案...';
  }
};
```

## Skill 进阶配置

### 优先级设置

```yaml
skills:
  # 数字越大优先级越高
  weather:
    priority: 100
    
  search:
    priority: 50
```

### 缓存配置

```yaml
skills:
  weather:
    cache:
      enabled: true
      ttl: 3600  # 秒
```

### 限流配置

```yaml
skills:
  search:
    rateLimit:
      enabled: true
      max: 10  # 每分钟最多 10 次
```

## 性能优化

### 1. 异步处理

```javascript
// ❌ 阻塞
async handle(context) {
  const result = await syncOperation();
  return result;
}

// ✅ 非阻塞（立即返回）
async handle(context) {
  // 立即返回处理中
  context.reply('正在处理，请稍候...');
  
  // 后台处理
  setTimeout(async () => {
    const result = await longOperation();
    context.reply(`处理完成：${result}`);
  }, 0);
  
  return null; // 不阻塞
}
```

### 2. 结果缓存

```javascript
const cache = new Map();

module.exports = {
  async handle(context) {
    const cacheKey = this.getCacheKey(context);
    
    // 检查缓存
    if (cache.has(cacheKey)) {
      return cache.get(cacheKey);
    }
    
    const result = await this.compute(context);
    
    // 存入缓存（1小时过期）
    cache.set(cacheKey, result);
    setTimeout(() => cache.delete(cacheKey), 3600000);
    
    return result;
  }
};
```

## 下一步

- [09. 记忆系统进阶](./09-memory-advanced.md) — 深入记忆系统
