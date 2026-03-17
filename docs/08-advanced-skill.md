# 08. 高级 Skill 开发 — 多模态、长对话、外部 API

> **💡 Motto**: "从会做到做精，就是进阶"

这节教程的目标：学会处理图片/文件、调用外部 API、实现多轮对话等高级功能。

---

## 🧩 这节学什么

| 技能 | 预计时间 | 难度 |
|------|---------|------|
| 处理图片/附件 | 10 分钟 | ⭐⭐ |
| 调用外部 API | 10 分钟 | ⭐⭐ |
| 多轮对话 | 15 分钟 | ⭐⭐⭐ |
| 向量知识库 | 20 分钟 | ⭐⭐⭐ |

---

## 🎯 目标：做出更智能的技能

学完这节，你的 AI 将能：
- 分析上传的图片
- 搜索 GitHub 仓库
- 进行多轮对话（点餐、客服）
- 构建私人知识库

---

## 🚀 技能 1：图片/文件分析器

### 场景
用户上传图片或文件，AI 自动分析内容。

### 代码

```javascript
// skills/file-analyzer/index.js
const fs = require('fs');
const path = require('path');

module.exports = {
  name: 'file-analyzer',
  description: '文件分析器 - 分析图片、PDF、文档',
  version: '1.0.0',
  
  // 触发：上传了附件
  match(context) {
    return (context.attachments?.length > 0) || 
           context.message.includes('分析文件');
  },
  
  async handle(context) {
    const attachments = context.attachments || [];
    
    if (attachments.length === 0) {
      return '请上传要分析的文件（图片、PDF、文档）';
    }
    
    const results = [];
    for (const file of attachments) {
      const result = await this.analyze(file);
      results.push(result);
    }
    
    return this.formatResults(results);
  },
  
  async analyze(file) {
    const ext = path.extname(file.name).toLowerCase();
    
    // 图片分析
    if (['.jpg', '.jpeg', '.png', '.gif', '.webp'].includes(ext)) {
      return await this.analyzeImage(file);
    }
    
    // PDF 分析
    if (ext === '.pdf') {
      return await this.analyzePDF(file);
    }
    
    // 文档分析
    if (['.txt', '.md', '.json', '.yaml'].includes(ext)) {
      return await this.analyzeDoc(file);
    }
    
    return { name: file.name, type: 'file', result: '不支持的文件类型' };
  },
  
  // 分析图片
  async analyzeImage(file) {
    // 实际项目中调用 AI 图片识别 API
    // 如：OpenAI Vision、腾讯云图片分析
    
    // 模拟
    await new Promise(r => setTimeout(r, 500));
    
    return {
      name: file.name,
      type: 'image',
      result: '📸 图片分析结果：\n' +
        '- 格式：' + file.type + '\n' +
        '- 大小：' + this.formatSize(file.size) + '\n' +
        '- AI 识别：图片内容描述...'
    };
  },
  
  // 分析 PDF
  async analyzePDF(file) {
    // 实际项目中用 pdf-parse 解析
    
    return {
      name: file.name,
      type: 'pdf',
      result: '📄 PDF 分析结果：\n' +
        '- 页数：10+\n' +
        '- 提取文本：...\n' +
        '- 总结：文档内容摘要...'
    };
  },
  
  // 分析文档
  async analyzeDoc(file) {
    const content = fs.readFileSync(file.path, 'utf-8');
    const lines = content.split('\n').length;
    const words = content.split(/\s+/).length;
    
    return {
      name: file.name,
      type: 'document',
      result: `📝 文档分析结果：
- 行数：${lines}
- 词数：${words}
- 字符数：${content.length}
- 编码：UTF-8`
    };
  },
  
  formatResults(results) {
    return results.map(r => 
      `📎 ${r.name}\n${r.result}`
    ).join('\n\n---\n\n');
  },
  
  formatSize(bytes) {
    if (bytes < 1024) return bytes + ' B';
    if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
    return (bytes / 1024 / 1024).toFixed(1) + ' MB';
  }
};
```

### 测试

```
# 上传一张图片
"分析这个文件"

# 或上传 PDF
"看看这个 PDF 讲了什么"
```

---

## 🚀 技能 2：GitHub 仓库搜索

### 场景
用自然语言搜索 GitHub 仓库。

### 代码

```javascript
// skills/github-search/index.js
const axios = require('axios');

module.exports = {
  name: 'github-search',
  description: 'GitHub 仓库搜索',
  version: '1.0.0',
  
  keywords: ['搜索仓库', '找项目', 'github', '仓库'],
  
  async handle(context) {
    const query = this.extractQuery(context.message);
    
    if (!query) {
      return '请告诉我你想搜索什么仓库？\n比如："搜索 AI 相关的仓库"';
    }
    
    // 搜索
    const repos = await this.searchRepos(query);
    
    if (repos.length === 0) {
      return `没有找到与 "${query}" 相关的仓库`;
    }
    
    return this.formatRepos(query, repos);
  },
  
  extractQuery(message) {
    return message
      .replace(/搜索仓库|找项目|github|仓库/g, '')
      .trim();
  },
  
  async searchRepos(query) {
    try {
      const response = await axios.get(
        'https://api.github.com/search/repositories',
        {
          params: { 
            q: query, 
            sort: 'stars', 
            order: 'desc',
            per_page: 5
          },
          headers: { 
            Accept: 'application/vnd.github.v3+json',
            // 建议添加 GitHub Token 避免限流
            // Authorization: 'token YOUR_TOKEN'
          }
        }
      );
      
      return response.data.items;
    } catch (error) {
      console.error('GitHub API 错误:', error.message);
      return [];
    }
  },
  
  formatRepos(query, repos) {
    const list = repos.map(r => {
      const stars = r.stargazers_count >= 1000 
        ? (r.stargazers_count / 1000).toFixed(1) + 'k'
        : r.stargazers_count;
      
      return `
⭐ **${r.full_name}** (${stars} ⭐)
${r.description || '无描述'}
- 语言：${r.language || '未知'}
- Fork：${r.forks_count} | Watch：${r.watchers_count}
[查看仓库](${r.html_url})
`;
    }).join('\n---\n');
    
    return `🔍 搜索 "${query}" 的热门仓库：\n\n${list}`;
  }
};
```

### 配置（可选 GitHub Token）

```yaml
# config.yaml
skills:
  github-search:
    token: ${GITHUB_TOKEN}  # 可选，避免限流
```

### 测试

```
"搜索 AI 相关的仓库"
"找一些 Python 教程仓库"
"github 上有哪些火的前端项目"
```

---

## 🚀 技能 3：点餐助手（多轮对话）

### 场景
用户说"我要点餐"，然后一步步选择、确认、完成订单。

### 代码

```javascript
// skills/order/index.js
const fs = require('fs');
const path = require('path');

module.exports = {
  name: 'order',
  description: '点餐助手 - 多轮对话示例',
  version: '1.0.0',
  
  keywords: ['点餐', '订餐', '外卖', '下单'],
  
  async handle(context) {
    const message = context.message;
    const userId = context.userId;
    
    // 获取用户当前状态
    const state = await this.getState(userId);
    
    // 根据状态处理
    if (state?.step === 'selecting') {
      return await this.handleSelection(context, state);
    }
    
    if (state?.step === 'address') {
      return await this.handleAddress(context, state);
    }
    
    if (state?.step === 'confirming') {
      return await this.handleConfirm(context, state);
    }
    
    // 新订单
    return this.startOrder(context);
  },
  
  // 开始点餐
  startOrder(context) {
    const menu = `
🍔 今日菜单：

1. **香辣鸡腿堡** - ¥25
2. **巨无霸** - ¥28
3. **麦香鱼** - ¥22
4. **薯条(大)** - ¥12
5. **可乐(大)** - ¥8

请回复序号选择（如：1,2）
`;

    // 保存状态
    this.saveState(context.userId, {
      step: 'selecting',
      items: []
    });

    return menu + '\n💡 回复"退出"取消订单';
  },
  
  // 处理选择
  async handleSelection(context, state) {
    const message = context.message;
    
    // 退出
    if (message.includes('退出') || message.includes('取消')) {
      await this.clearState(context.userId);
      return '❌ 订单已取消';
    }
    
    // 确认完成
    if (message.includes('确认') || message.includes('好了')) {
      if (state.items.length === 0) {
        return '你还没有选择任何商品';
      }
      
      state.step = 'address';
      await this.saveState(context.userId, state);
      
      return this.getOrderSummary(state) + '\n\n📍 请告诉我收货地址：';
    }
    
    // 解析选择
    const menu = {
      '1': { name: '香辣鸡腿堡', price: 25 },
      '2': { name: '巨无霸', price: 28 },
      '3': { name: '麦香鱼', price: 22 },
      '4': { name: '薯条(大)', price: 12 },
      '5': { name: '可乐(大)', price: 8 }
    };
    
    const choices = message.split(/[,，]/).map(s => s.trim());
    const added = [];
    
    for (const choice of choices) {
      const item = menu[choice];
      if (item) {
        state.items.push(item);
        added.push(item.name);
      }
    }
    
    await this.saveState(context.userId, state);
    
    return `✅ 已添加：${added.join(', ')}\n\n` + 
           this.getOrderSummary(state) + 
           '\n\n继续添加请回复序号，确认下单请回复"确认"';
  },
  
  // 处理地址
  async handleAddress(context, state) {
    state.address = context.message;
    state.step = 'confirming';
    await this.saveState(context.userId, state);
    
    return this.getOrderSummary(state) + 
           '\n\n📍 地址：' + state.address + 
           '\n\n✅ 确认下单请回复"确认"，修改地址请重新输入';
  },
  
  // 处理确认
  async handleConfirm(context, state) {
    if (context.message.includes('修改')) {
      state.step = 'address';
      await this.saveState(context.userId, state);
      return '📍 请重新输入收货地址：';
    }
    
    // 完成订单
    const orderId = 'ORD' + Date.now();
    await this.clearState(context.userId);
    
    return `
🎉 订单确认成功！

订单号：${orderId}
📦 商品：${state.items.map(i => i.name).join(', '')}
💰 总价：¥${state.items.reduce((sum, i) => sum + i.price, 0)}
📍 地址：${state.address}

感谢下单！🏃‍♂️ 骑手正在准备中...
`;
  },
  
  getOrderSummary(state) {
    if (state.items.length === 0) {
      return '🛒 当前购物车：空';
    }
    
    const items = state.items.map((i, idx) => 
      `${idx + 1}. ${i.name} - ¥${i.price}`
    ).join('\n');
    
    const total = state.items.reduce((sum, i) => sum + i.price, 0);
    
    return `🛒 购物车：
${items}

💰 总计：¥${total}`;
  },
  
  // 状态管理（简化版，实际用数据库）
  async getState(userId) {
    try {
      const data = fs.readFileSync(
        path.join(__dirname, 'data', `${userId}.json`), 
        'utf-8'
      );
      return JSON.parse(data);
    } catch {
      return null;
    }
  },
  
  async saveState(userId, state) {
    const dir = path.join(__dirname, 'data');
    if (!fs.existsSync(dir)) fs.mkdirSync(dir, { recursive: true });
    fs.writeFileSync(
      path.join(dir, `${userId}.json`),
      JSON.stringify(state)
    );
  },
  
  async clearState(userId) {
    try {
      fs.unlinkSync(path.join(__dirname, 'data', `${userId}.json`));
    } catch {}
  }
};
```

### 测试

```
"我要点餐"
# 回复 1,2
# 回复"确认"
# 输入地址
# 回复"确认"
```

---

## 🚀 技能 4：私人知识库（向量存储）

### 场景
上传文档/笔记，AI 基于这些内容回答问题。

### 代码

```javascript
// skills/knowledge-base/index.js
// 需要先安装：npm install chromadb

/*
# 使用说明：
# 1. 先用 add 命令添加文档
# 2. 然后用自然语言提问
*/

module.exports = {
  name: 'knowledge-base',
  description: '私人知识库 - 基于文档回答问题',
  version: '1.0.0',
  
  keywords: ['知识库', '问答', '基于文档', '根据文档'],
  
  async handle(context) {
    const message = context.message;
    
    // 添加文档
    if (message.includes('添加') || message.includes('上传')) {
      return await this.addDocument(context);
    }
    
    // 搜索/问答
    return await this.askQuestion(context);
  },
  
  // 添加文档到知识库
  async addDocument(context) {
    const attachments = context.attachments || [];
    
    if (attachments.length === 0) {
      return '请上传要添加到知识库的文档（支持 .txt, .md）';
    }
    
    const results = [];
    for (const file of attachments) {
      const content = await this.readFile(file);
      // 实际项目中：await this.addToVectorDB(file.name, content);
      results.push(`✅ 已添加：${file.name}（${content.length} 字符）`);
    }
    
    return '📚 知识库更新：\n\n' + results.join('\n');
  },
  
  // 问答
  async askQuestion(context) {
    const query = context.message;
    
    // 实际项目中：
    // 1. 向量检索：const results = await this.searchVectorDB(query);
    // 2. 构建 Prompt：const prompt = this.buildPrompt(results, query);
    // 3. 调用 LLM：const answer = await this.askLLM(prompt);
    
    // 模拟
    await new Promise(r => setTimeout(r, 500));
    
    return `
💡 基于知识库的回答：

根据您上传的文档，关于"${query}"的回答：

（这里会显示 AI 基于文档生成的回答）

---
📚 参考来源：
- document1.md
- notes.txt
`;
  },
  
  async readFile(file) {
    // 实际读取文件内容
    return '文档内容...';
  },
  
  // 向量搜索（示例，实际需要 ChromaDB / Pinecone）
  async searchVectorDB(query, topK = 3) {
    // const db = new ChromaClient();
    // const results = await db.collection('knowledge').query({
    //   queryTexts: [query],
    //   nResults: topK
    // });
    // return results;
    return [];
  },
  
  // 构建 Prompt
  buildPrompt(docs, query) {
    const context = docs.map(d => d.text).join('\n\n');
    return `基于以下文档回答问题：\n\n${context}\n\n问题：${query}\n回答：`;  },
  
  // 问 LLM
  async askLLM(prompt) {
    // 调用 OpenAI / Claude
    return '基于文档的答案...';
  }
};
```

---

## 🧪 测试高级技能

```bash
openclaw restart

# 测试文件分析
上传一张图片，然后说"分析这个"

# 测试 GitHub 搜索
"搜索 Python 机器学习仓库"

# 测试点餐
"我要点餐"
# 然后按提示选择

# 测试知识库
上传一个文档，然后问"文档里讲了什么"
```

---

## ⚙️ 技能配置进阶

### 优先级

```yaml
skills:
  weather:
    priority: 100  # 数字越大越优先
    
  order:
    priority: 50
```

### 缓存

```yaml
skills:
  github-search:
    cache:
      enabled: true
      ttl: 3600  # 1小时
```

### 限流

```yaml
skills:
  search:
    rateLimit:
      max: 10  # 每分钟 10 次
```

---

## 🎉 这节目标达成

✅ 会处理图片/文件附件  
✅ 会调用外部 API（GitHub）  
✅ 会实现多轮对话  
✅ 了解向量知识库概念  

---

## ➡️ 下节学什么

- [09. 记忆系统进阶](./09-memory-advanced.md) — 向量存储、语义搜索
- [10. 工作流进阶](./10-workflow-advanced.md) — 高级触发器
