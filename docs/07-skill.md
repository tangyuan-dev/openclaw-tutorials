# 07. Skill 开发实战

本文详细介绍 OpenClaw 技能（Skill）的开发方法，这是最常用的高频功能。

## 什么是 Skill？

Skill（技能）是 OpenClaw 的核心扩展机制，让你自定义 AI 的能力。

## Skill 目录结构

```
~/.openclaw/skills/
└── my-skill/
    ├── SKILL.md      # 技能定义（必选）
    ├── index.js      # 入口文件（必选）
    ├── config.yaml   # 配置文件（可选）
    └── package.json  # 依赖（可选）
```

## Skill 定义文件 (SKILL.md)

```markdown
# my-skill

## 触发关键词
- "天气"、"查询天气"
- "天气怎么样"

## 功能描述
查询指定城市的天气信息

## 使用示例
用户: 北京天气怎么样
回复: 🌤️ 北京今天晴，温度 15-25°C
```

## Skill 入口文件 (index.js)

```javascript
module.exports = {
  name: 'weather',
  description: '查询天气',
  
  // 触发条件：返回 true 表示处理这个请求
  match(context) {
    const msg = context.message.toLowerCase();
    return msg.includes('天气') || msg.includes('weather');
  },
  
  // 处理请求
  async handle(context) {
    // 从消息中提取城市
    const city = this.extractCity(context.message);
    
    if (!city) {
      return '请告诉我你想查询哪个城市的天气？';
    }
    
    // 调用天气 API（这里用模拟数据）
    const weather = await this.getWeather(city);
    
    return `🌤️ ${city}天气：${weather.temp}°C，${weather.condition}`;
  },
  
  // 提取城市名
  extractCity(message) {
    const cities = ['北京', '上海', '广州', '深圳', '杭州', '成都'];
    for (const city of cities) {
      if (message.includes(city)) return city;
    }
    return null;
  },
  
  // 获取天气（示例）
  async getWeather(city) {
    // 实际项目中调用天气 API
    return { temp: '20-28', condition: '晴' };
  }
};
```

## Skill 配置 (config.yaml)

```yaml
skills:
  weather:
    apiKey: ${WEATHER_API_KEY}
    defaultCity: 北京
    units: celsius
```

## 高频 Skill 模板

### 1. 提醒助手

```javascript
// skills/reminder/index.js
module.exports = {
  name: 'reminder',
  description: '定时提醒',
  
  match(context) {
    const msg = context.message;
    return msg.includes('提醒') || msg.includes('闹钟');
  },
  
  async handle(context) {
    const { time, content } = this.parseReminder(context.message);
    
    if (!time || !content) {
      return '请告诉我提醒时间和内容，例如："提醒我明天早上9点开会"';
    }
    
    // 设置提醒
    await context.scheduler.add({
      time,
      content,
      channel: context.channel
    });
    
    return `✅ 已设置提醒：${content}，时间：${time}`;
  },
  
  parseReminder(message) {
    // 解析时间和内容
    const timeMatch = message.match(/(\d+)[点时]/);
    const time = timeMatch ? timeMatch[1] : null;
    const content = message.replace(/提醒|闹钟/g, '').trim();
    return { time, content };
  }
};
```

### 2. 搜索助手

```javascript
// skills/search/index.js
module.exports = {
  name: 'search',
  description: '搜索信息',
  
  match(context) {
    const msg = context.message.toLowerCase();
    return msg.includes('搜索') || msg.includes('查一下') || msg.includes('找找');
  },
  
  async handle(context) {
    const query = this.extractQuery(context.message);
    
    if (!query) {
      return '请告诉我你想搜索什么？';
    }
    
    // 搜索
    const results = await this.search(query);
    
    return this.formatResults(results);
  },
  
  extractQuery(message) {
    return message
      .replace(/搜索|查一下|找找/g, '')
      .trim();
  },
  
  async search(query) {
    // 调用搜索 API
    return [
      { title: '结果1', url: 'https://example.com/1' },
      { title: '结果2', url: 'https://example.com/2' }
    ];
  },
  
  formatResults(results) {
    return results.map(r => `• [${r.title}](${r.url})`).join('\n');
  }
};
```

### 3. 文件处理

```javascript
// skills/file-processor/index.js
module.exports = {
  name: 'file-processor',
  description: '文件处理',
  
  match(context) {
    return context.message.includes('文件') || 
           context.message.includes('整理') ||
           context.attachments?.length > 0;
  },
  
  async handle(context) {
    const files = context.attachments || [];
    
    if (files.length === 0) {
      return '请上传要处理的文件';
    }
    
    // 处理每个文件
    const results = [];
    for (const file of files) {
      const result = await this.processFile(file);
      results.push(result);
    }
    
    return `✅ 已处理 ${results.length} 个文件：\n` + 
           results.map(r => `• ${r.name}: ${r.status}`).join('\n');
  },
  
  async processFile(file) {
    // 文件处理逻辑
    return { name: file.name, status: '完成' };
  }
};
```

## Skill 市场

OpenClaw 提供官方技能市场：

```bash
# 列出可用技能
openclaw skill list

# 安装技能
openclaw skill install weather
openclaw skill install reminder
openclaw skill install search

# 卸载技能
openclaw skill uninstall weather
```

## 最佳实践

1. **一个 Skill 只做一件事** — 保持简单
2. **提供清晰的触发词** — 让用户知道怎么用
3. **返回友好提示** — 无法处理时给用户反馈
4. **处理错误** — 网络失败等情况要优雅处理

## 下一步

- [08. 高级 Skill 开发](./08-advanced-skill.md) — 深入技能开发
