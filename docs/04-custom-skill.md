# 04. 自定义技能 — 教 AI 学会新技能

> **💡 Motto**: "教 AI 学会新技能，它就能帮你干更多活"

这节教程的目标：创建一个实用的技能（如天气查询），学会编写自己的技能。

---

## 🧩 这节学什么

| 技能 | 预计时间 | 难度 |
|------|---------|------|
| 创建打招呼技能 | 5 分钟 | ⭐ |
| 天气查询技能 | 10 分钟 | ⭐⭐ |
| 技能配置与调试 | 5 分钟 | ⭐ |

---

## 🎯 目标：做一个天气预报技能

学完这节，你将拥有一个可以说"北京天气怎么样"的 AI 助手。

---

## 🚀 第一步：创建技能结构

```bash
# 创建技能目录
mkdir -p ~/.openclaw/skills/weather

# 创建技能文件
touch ~/.openclaw/skills/weather/SKILL.md
touch ~/.openclaw/skills/weather/index.js
```

---

## 📝 第二步：编写技能定义（SKILL.md）

```markdown
# weather - 天气预报技能

## 触发条件
- 用户说 "天气"、"查询天气"、"...天气怎么样"
- 包含城市名：如"北京天气"、"上海天气怎么样"

## 功能
- 查询指定城市的天气
- 返回温度、天气状况、穿衣建议

## 示例
用户: 北京天气怎么样
回复: 🌤️ 北京今天：25°C，晴。适合穿短袖。

用户: 上海天气
回复: 🌧️ 上海今天：18°C，小雨。记得带伞。
```

---

## 💻 第三步：编写技能逻辑（index.js）

```javascript
// ~/.openclaw/skills/weather/index.js

module.exports = {
  name: 'weather',
  description: '查询天气预报',
  
  // 触发关键词
  keywords: ['天气', 'weather', '温度'],
  
  // 处理消息
  async handle(context) {
    const message = context.message;
    
    // 1. 提取城市名
    const city = this.extractCity(message);
    if (!city) {
      return '请告诉我你想查询哪个城市的天气？比如"北京天气怎么样"';
    }
    
    // 2. 调用天气 API（这里用免费接口示例）
    const weather = await this.getWeather(city);
    if (!weather) {
      return `抱歉，暂时无法获取 ${city} 的天气信息`;
    }
    
    // 3. 返回格式化结果
    return this.formatReply(weather);
  },
  
  // 提取城市名
  extractCity(message) {
    const cities = [
      '北京', '上海', '广州', '深圳', '杭州', '南京', 
      '成都', '重庆', '武汉', '西安', '苏州', '天津'
    ];
    
    for (const city of cities) {
      if (message.includes(city)) {
        return city;
      }
    }
    return null;
  },
  
  // 获取天气数据（模拟）
  async getWeather(city) {
    // 实际项目中这里调用真实 API
    // 如：https://www.tianqiapi.com/free/
    
    const mockData = {
      '北京': { temp: 25, condition: '晴', advice: '适合穿短袖' },
      '上海': { temp: 18, condition: '小雨', advice: '记得带伞' },
      '广州': { temp: 28, condition: '多云', advice: '天气舒适' },
      '深圳': { temp: 27, condition: '晴', advice: '适合户外活动' },
      '杭州': { temp: 22, condition: '阴', advice: '建议带外套' },
    };
    
    // 模拟网络延迟
    await new Promise(r => setTimeout(r, 300));
    
    return mockData[city] || null;
  },
  
  // 格式化回复
  formatReply(weather) {
    const emoji = {
      '晴': '☀️',
      '多云': '⛅',
      '阴': '☁️',
      '小雨': '🌧️',
      '中雨': '🌧️',
      '大雨': '⛈️',
    };
    
    const cond = weather.condition;
    return `${emoji[cond] || '🌤️'} ${city}今天：${weather.temp}°C，${weather.condition}。${weather.advice}！`;
  }
};
```

---

## 🧪 第四步：测试技能

```bash
# 重启 OpenClaw
openclaw restart
```

**测试指令：**
- "北京天气怎么样"
- "上海天气"
- "广州天气"

---

## 🎯 进阶：做一个"收藏夹"技能

这个技能可以记住用户喜欢的东西。

### 技能结构

```
~/.openclaw/skills/favorites/
├── SKILL.md
└── index.js
```

### SKILL.md

```markdown
# favorites - 收藏夹技能

## 触发条件
- 用户说 "收藏"、"记住"、"我喜欢"
- 用户说 "我的收藏"、"看看我收藏了什么"

## 功能
- 收藏用户喜欢的内容
- 查看收藏列表
```

### index.js

```javascript
// ~/.openclaw/skills/favorites/index.js

const fs = require('fs');
const path = require('path');

module.exports = {
  name: 'favorites',
  description: '收藏夹技能',
  
  keywords: ['收藏', '记住', '我喜欢', '我的收藏'],
  
  async handle(context) {
    const message = context.message;
    const userId = context.userId;
    const storagePath = path.join(__dirname, 'data', `${userId}.json`);
    
    // 1. 查看收藏
    if (message.includes('查看') || message.includes('我的收藏')) {
      const favorites = this.loadFavorites(storagePath);
      if (favorites.length === 0) {
        return '你还没有收藏任何内容，说"收藏 XXX"来添加。';
      }
      return '📚 你的收藏：\n' + favorites.map((f, i) => `${i+1}. ${f}`).join('\n');
    }
    
    // 2. 添加收藏
    const item = message.replace(/收藏|记住|我喜欢/, '').trim();
    if (item) {
      this.saveFavorite(storagePath, item);
      return `✅ 已收藏：${item}`;
    }
    
    return '请说"收藏 XXX"来添加收藏，或"查看我的收藏"来查看列表。';
  },
  
  loadFavorites(path) {
    try {
      if (!fs.existsSync(path)) return [];
      return JSON.parse(fs.readFileSync(path, 'utf-8'));
    } catch {
      return [];
    }
  },
  
  saveFavorite(path, item) {
    const dir = path.replace(/[^/\\]+$/, '');
    if (!fs.existsSync(dir)) fs.mkdirSync(dir, { recursive: true });
    
    const favorites = this.loadFavorites(path);
    favorites.push(item);
    fs.writeFileSync(path, JSON.stringify(favorites, null, 2));
  }
};
```

---

## 🔧 技能调试技巧

### 1. 查看技能日志

```bash
openclaw logs --tail 50
```

### 2. 调试模式

```yaml
# config.yaml
skills:
  weather:
    debug: true
```

### 3. 常用技能模板

```javascript
// 模板：带配置的技能
module.exports = {
  name: 'xxx',
  description: 'xxx',
  
  // 从 config 获取设置
  init(config) {
    this.apiKey = config.skills?.xxx?.apiKey;
  },
  
  async handle(context) {
    // 处理逻辑
  }
};
```

---

## 📦 官方技能市场

```bash
# 列出可用技能
openclaw skill list

# 安装技能
openclaw skill install weather
openclaw skill install github
```

---

## 🎉 这节目标达成

✅ 会创建自定义技能  
✅ 会编写 SKILL.md 和 index.js  
✅ 会调试技能  
✅ 做出了天气预报技能  

---

## ➡️ 下节学什么

- [05. 记忆系统](./05-memory.md) — 让 AI 记住你的偏好
- [06. 工作流自动化](./06-workflow.md) — 告别重复任务
