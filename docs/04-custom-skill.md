# 04. 自定义技能

本文介绍如何创建自定义技能，扩展 OpenClaw 的能力。

## 什么是技能？

技能 (Skill) 是 OpenClaw 的扩展模块，可以：
- 添加新的命令
- 集成第三方 API
- 自动化特定工作流

## 技能结构

```
skills/
└── my-skill/
    ├── SKILL.md      # 技能定义
    ├── index.js       # 入口文件
    └── package.json   # 依赖（可选）
```

## 创建第一个技能

### 1. 创建技能目录

```bash
mkdir -p skills/hello-world
```

### 2. 编写技能定义

```markdown
# hello-world

## 触发条件
- 用户说 "hello" 或 "你好"
- 用户说 "who are you" 或 "你是谁"

## 功能
- 打招呼并自我介绍

## 示例
用户: hello
回复: 👋 你好！我是你的 AI 助手 OpenClaw！

用户: 你是谁
回复: 我是 OpenClaw，一个本地运行的 AI 助手...
```

### 3. 编写执行逻辑

```javascript
// index.js
module.exports = {
  name: 'hello-world',
  description: '打招呼技能',
  
  async handle(context) {
    const message = context.message.toLowerCase();
    
    if (message.includes('hello') || message.includes('你好')) {
      return '👋 你好！我是 OpenClaw，你的 AI 助手！';
    }
    
    if (message.includes('你是谁') || message.includes('who are you')) {
      return '我是 OpenClaw，一个本地运行的 AI 助手。\n\n我可以帮你：\n- 写代码\n- 查资料\n- 自动化任务\n- 等等...';
    }
    
    // 返回 null 表示不处理
    return null;
  }
};
```

### 4. 注册技能

技能目录放入 `~/.openclaw/skills/` 后会自动加载。

## 技能开发进阶

### 使用外部 API

```javascript
const axios = require('axios');

module.exports = {
  name: 'weather',
  description: '查询天气',
  
  async handle(context) {
    const message = context.message;
    const city = this.extractCity(message);
    
    if (!city) {
      return '请告诉我你想查询哪个城市的天气？';
    }
    
    const weather = await axios.get(
      `https://api.weather.com.cn?city=${city}`
    );
    
    return `🌤️ ${city}天气：${weather.data.temp}°C，${weather.data.condition}`;
  },
  
  extractCity(message) {
    // 提取城市名
    const match = message.match(/天气.*?(\w+)/);
    return match ? match[1] : null;
  }
};
```

### 添加配置项

```yaml
# config.yaml
skills:
  weather:
    apiKey: ${WEATHER_API_KEY}
    defaultCity: 北京
```

```javascript
// 在技能中获取配置
module.exports = {
  handle(context) {
    const config = context.config.skills.weather;
    // 使用 config.apiKey, config.defaultCity
  }
};
```

## 技能市场

OpenClaw 提供官方技能市场：

```bash
# 列出可用技能
openclaw skill list

# 安装技能
openclaw skill install weather

# 卸载技能
openclaw skill uninstall weather
```

## 下一步

- [05. 记忆系统](./05-memory.md) — 让 OpenClaw 记住上下文
- [06. 工作流自动化](./06-workflow.md) — 创建自动化工作流
