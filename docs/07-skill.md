# 07. Skill 开发实战 — 做出你的第一个实用技能

> **💡 Motto**: "一个技能只做一件事，但要做到极致"

这节教程的目标：做出几个真正能用的技能（提醒、搜索、文件处理），掌握技能开发的核心模式。

---

## 🧩 这节学什么

| 技能 | 预计时间 | 难度 |
|------|---------|------|
| 天气查询技能 | 10 分钟 | ⭐ |
| 提醒助手 | 10 分钟 | ⭐⭐ |
| 搜索助手 | 10 分钟 | ⭐⭐ |
| 文件处理器 | 15 分钟 | ⭐⭐⭐ |

---

## 🎯 目标：做出 4 个实用技能

学完这节，你将拥有：
- 天气查询机器人
- 定时提醒助手
- 搜索工具
- 文件处理工具

---

## 🚀 技能开发模式

所有技能都遵循这个模式：

```
触发条件 → 解析输入 → 处理业务 → 返回结果
```

---

## 🎯 技能 1：天气查询（完整代码）

### 目录结构

```
~/.openclaw/skills/weather/
├── SKILL.md
└── index.js
```

### SKILL.md

```markdown
# weather - 天气预报技能

## 触发关键词
- "天气"、"weather"
- "...天气怎么样"、"...天气如何"

## 支持城市
北京、上海、广州、深圳、杭州、成都、重庆、武汉、西安、南京
```

### index.js（完整代码）

```javascript
const https = require('https');

module.exports = {
  name: 'weather',
  description: '查询天气预报',
  version: '1.0.0',
  
  // 触发条件
  match(context) {
    const msg = context.message;
    return msg.includes('天气') || msg.includes('weather');
  },
  
  // 处理请求
  async handle(context) {
    const city = this.extractCity(context.message);
    
    if (!city) {
      return '请告诉我你想查询哪个城市的天气？\n比如："北京天气怎么样"';
    }
    
    // 获取天气
    const weather = await this.getWeather(city);
    if (!weather) {
      return `暂时无法获取 ${city} 的天气信息`;
    }
    
    // 返回结果
    return this.formatReply(city, weather);
  },
  
  // 提取城市
  extractCity(message) {
    const cities = [
      '北京', '上海', '广州', '深圳', '杭州', 
      '成都', '重庆', '武汉', '西安', '南京',
      '苏州', '天津', '青岛', '长沙', '郑州'
    ];
    
    for (const city of cities) {
      if (message.includes(city)) return city;
    }
    return null;
  },
  
  // 获取天气（使用免费 API）
  async getWeather(city) {
    // 实际项目中替换为真实 API
    // 推荐：tianqiapi.com / qweather.com
    
    const mockData = {
      '北京': { temp: 25, condition: '晴', wind: '北风3级', humidity: '40%' },
      '上海': { temp: 20, condition: '多云', wind: '东风2级', humidity: '65%' },
      '广州': { temp: 28, condition: '晴', wind: '南风1级', humidity: '70%' },
      '深圳': { temp: 27, condition: '晴', wind: '东南风2级', humidity: '75%' },
      '杭州': { temp: 22, condition: '阴', wind: '东风3级', humidity: '60%' },
    };
    
    // 模拟 API 延迟
    await new Promise(r => setTimeout(r, 500));
    
    return mockData[city] || null;
  },
  
  // 格式化回复
  formatReply(city, weather) {
    const emoji = {
      '晴': '☀️', '多云': '⛅', '阴': '☁️', 
      '小雨': '🌧️', '中雨': '🌧️', '大雨': '⛈️',
      '雪': '❄️', '雾': '🌫️'
    };
    
    return `
🌤️ ${city}天气预报

温度：${weather.temp}°
天气：${emoji[weather.condition] || ''} ${weather.condition}
风速：${weather.wind}
湿度：${weather.humidity}

出行建议：${this.getAdvice(weather)}
`.trim();
  },
  
  // 出行建议
  getAdvice(weather) {
    if (weather.condition.includes('雨')) return '记得带伞！';
    if (weather.temp > 28) return '天气炎热，注意防暑';
    if (weather.temp < 10) return '天气较凉，注意保暖';
    return '天气不错，适合出行！';
  }
};
```

### 测试

```bash
openclaw restart
# 测试指令：
# "北京天气怎么样"
# "上海天气"
```

---

## 🎯 技能 2：提醒助手（完整代码）

### 目录结构

```
~/.openclaw/skills/reminder/
├── SKILL.md
└── index.js
```

### index.js

```javascript
const fs = require('fs');
const path = require('path');

module.exports = {
  name: 'reminder',
  description: '定时提醒助手',
  version: '1.0.0',
  
  // 触发关键词
  keywords: ['提醒', '闹钟', '定时', '通知'],
  
  async handle(context) {
    const message = context.message;
    const userId = context.userId;
    
    // 判断是设置提醒还是查看提醒
    if (message.includes('查看') || message.includes('我的提醒')) {
      return this.listReminders(userId);
    }
    
    if (message.includes('删除') || message.includes('取消')) {
      const id = this.extractId(message);
      return this.deleteReminder(userId, id);
    }
    
    // 解析提醒
    const reminder = this.parseReminder(message);
    if (!reminder.time || !reminder.content) {
      return this.getHelp();
    }
    
    // 保存提醒
    return this.saveReminder(userId, reminder);
  },
  
  // 解析提醒内容
  parseReminder(message) {
    // 匹配时间：今天/明天/具体时间
    let time = null;
    let content = message;
    
    // 明天早上 9 点
    if (message.includes('明天')) {
      const match = message.match(/明天(\d+)[点时]/);
      if (match) {
        const hour = parseInt(match[1]);
        const tomorrow = new Date();
        tomorrow.setDate(tomorrow.getDate() + 1);
        tomorrow.setHours(hour, 0, 0, 0);
        time = tomorrow.toISOString();
      }
      content = content.replace(/明天\d+[点时]/, '').replace(/明天/, '');
    }
    // 今天下午 3 点
    else if (message.includes('今天')) {
      const match = message.match(/今天(\d+)[点时]/);
      if (match) {
        const hour = parseInt(match[1]);
        const today = new Date();
        today.setHours(hour, 0, 0, 0);
        time = today.toISOString();
      }
      content = content.replace(/今天\d+[点时]/, '').replace(/今天/, '');
    }
    // 3 点（今天）
    else {
      const match = message.match(/(\d+)[点时]/);
      if (match) {
        const hour = parseInt(match[1]);
        const today = new Date();
        today.setHours(hour, 0, 0, 0);
        time = today.toISOString();
      }
      content = content.replace(/\d+[点时]/, '');
    }
    
    // 清理内容
    content = content
      .replace(/提醒|闹钟|定时|通知/g, '')
      .trim();
    
    return { time, content };
  },
  
  // 保存提醒
  async saveReminder(userId, reminder) {
    const storagePath = path.join(__dirname, 'data');
    if (!fs.existsSync(storagePath)) {
      fs.mkdirSync(storagePath, { recursive: true });
    }
    
    const filePath = path.join(storagePath, `${userId}.json`);
    let reminders = [];
    
    if (fs.existsSync(filePath)) {
      reminders = JSON.parse(fs.readFileSync(filePath, 'utf-8'));
    }
    
    const id = Date.now().toString();
    reminders.push({ id, ...reminder, createdAt: new Date().toISOString() });
    
    fs.writeFileSync(filePath, JSON.stringify(reminders, null, 2));
    
    const timeStr = new Date(reminder.time).toLocaleString('zh-CN');
    return `✅ 已设置提醒：${reminder.content}\n⏰ 时间：${timeStr}`;
  },
  
  // 查看提醒
  listReminders(userId) {
    const filePath = path.join(__dirname, 'data', `${userId}.json`);
    if (!fs.existsSync(filePath)) {
      return '你还没有设置任何提醒';
    }
    
    const reminders = JSON.parse(fs.readFileSync(filePath, 'utf-8'));
    if (reminders.length === 0) {
      return '你还没有设置任何提醒';
    }
    
    const list = reminders.map(r => {
      const time = new Date(r.time).toLocaleString('zh-CN');
      return `${r.id}. ${r.content} - ${time}`;
    }).join('\n');
    
    return `📋 你的提醒列表：\n\n${list}\n\n回复"删除 ID"可取消提醒`;
  },
  
  // 删除提醒
  deleteReminder(userId, id) {
    const filePath = path.join(__dirname, 'data', `${userId}.json`);
    if (!fs.existsSync(filePath)) {
      return '没有找到提醒';
    }
    
    let reminders = JSON.parse(fs.readFileSync(filePath, 'utf-8'));
    const index = reminders.findIndex(r => r.id === id);
    
    if (index === -1) {
      return '找不到这个提醒';
    }
    
    const deleted = reminders.splice(index, 1)[0];
    fs.writeFileSync(filePath, JSON.stringify(reminders, null, 2));
    
    return `✅ 已删除提醒：${deleted.content}`;
  },
  
  // 提取 ID
  extractId(message) {
    const match = message.match(/(\d+)/);
    return match ? match[1] : null;
  },
  
  // 帮助信息
  getHelp() {
    return `📝 提醒助手使用指南：

设置提醒：
- "提醒我明天早上 9 点开会"
- "提醒我今天下午 3 点交材料"
- "提醒我 2 小时后打电话"

查看提醒：
- "查看我的提醒"

删除提醒：
- "删除 123456"
`;
  }
};
```

---

## 🎯 技能 3：搜索助手

```javascript
// skills/search/index.js
const https = require('https');

module.exports = {
  name: 'search',
  description: '搜索助手',
  version: '1.0.0',
  
  keywords: ['搜索', '查一下', '找找', '搜搜'],
  
  async handle(context) {
    const query = this.extractQuery(context.message);
    
    if (!query) {
      return '请告诉我你想搜索什么？';
    }
    
    // 这里调用真实搜索 API
    const results = await this.search(query);
    
    return this.formatResults(query, results);
  },
  
  extractQuery(message) {
    return message
      .replace(/搜索|查一下|找找|搜搜/g, '')
      .trim();
  },
  
  async search(query) {
    // 模拟搜索结果
    // 实际项目中调用 Google/Bing/DuckDuckGo API
    
    await new Promise(r => setTimeout(r, 500));
    
    return [
      { title: `${query} - 官方文档`, url: `https://example.com/docs/${query}`, snippet: '官方文档介绍...' },
      { title: `${query} 教程`, url: `https://example.com/tutorial/${query}`, snippet: '入门教程...' },
      { title: `${query} 最佳实践`, url: `https://example.com/best/${query}`, snippet: '最佳实践指南...' },
    ];
  },
  
  formatResults(query, results) {
    const list = results.map((r, i) => 
      `${i + 1}. [${r.title}](${r.url})\n   ${r.snippet}`
    ).join('\n\n');
    
    return `🔍 搜索"${query}"的结果：\n\n${list}`;
  }
};
```

---

## 🎯 技能 4：文件处理器

```javascript
// skills/file-processor/index.js
const fs = require('fs');
const path = require('path');

module.exports = {
  name: 'file-processor',
  description: '文件处理助手',
  version: '1.0.0',
  
  keywords: ['文件', '整理', '处理', '转换'],
  
  async handle(context) {
    const message = context.message;
    const attachments = context.attachments || [];
    
    // 1. 批量重命名
    if (message.includes('重命名') || message.includes('改名')) {
      return await this.batchRename(context);
    }
    
    // 2. 文件整理
    if (message.includes('整理')) {
      return await this.organizeFiles(context);
    }
    
    // 3. 文件转换
    if (message.includes('转换') || message.includes('转')) {
      return await this.convertFiles(context);
    }
    
    // 4. 上传文件处理
    if (attachments.length > 0) {
      return await this.processAttachments(attachments);
    }
    
    return `📁 文件处理助手：

我可以帮你：
- 批量重命名文件
- 整理文件（按类型/日期）
- 转换文件格式
- 处理上传的文件

直接告诉我你要做什么！`;
  },
  
  // 批量重命名
  async batchRename(context) {
    // 提取目标目录和命名规则
    const message = context.message;
    
    // 示例：把 *.txt 改成 *.md
    const from = message.match(/(\.\w+)/)?.[1] || '.txt';
    const to = message.match(/改成(\.\w+)/)?.[1] || '.md';
    
    return `📝 批量重命名示例：
从 ${from} → ${to}

请提供具体目录路径`;
  },
  
  // 整理文件
  async organizeFiles(context) {
    return `📂 文件整理：

我可以按以下方式整理：
- 按文件类型（图片/文档/代码）
- 按日期（今天/昨天/更早）
- 按大小（大文件/小文件）

请告诉我整理哪个目录？`;
  },
  
  // 转换文件
  async convertFiles(context) {
    return `🔄 文件转换：

支持格式转换：
- Markdown ↔ HTML
- JSON ↔ YAML
- 图片格式互转

请告诉我转换哪个文件？`;
  },
  
  // 处理附件
  async processAttachments(attachments) {
    const results = [];
    
    for (const file of attachments) {
      results.push(`✅ ${file.name} (${this.formatSize(file.size)})`);
    }
    
    return `📎 已接收 ${attachments.length} 个文件：\n\n${results.join('\n')}\n\n请告诉我如何处理这些文件？`;
  },
  
  formatSize(bytes) {
    if (bytes < 1024) return bytes + ' B';
    if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
    return (bytes / 1024 / 1024).toFixed(1) + ' MB';
  }
};
```

---

## 🧪 测试所有技能

```bash
# 重启加载新技能
openclaw restart

# 测试天气
"北京天气怎么样"

# 测试提醒
"提醒我明天早上 9 点开会"
"查看我的提醒"

# 测试搜索
"搜索 OpenClaw 教程"

# 测试文件
"帮我整理下载文件夹"
```

---

## 🎉 这节目标达成

✅ 掌握技能开发模式  
✅ 做出 4 个实用技能  
✅ 学会技能调试  
✅ 能独立开发新技能  

---

## ➡️ 下节学什么

- [08. 高级 Skill 开发](./08-advanced-skill.md) — 多模态、外部 API、长对话
- [09. 记忆系统进阶](./09-memory-advanced.md) — 向量存储、语义搜索
