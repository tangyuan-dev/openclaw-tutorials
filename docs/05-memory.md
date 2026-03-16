# 05. 记忆系统

本文介绍 OpenClaw 的记忆系统，让 AI 能够跨会话保持上下文。

## 什么是记忆系统？

OpenClaw 的记忆系统让 AI 能够：
- 记住用户的偏好和习惯
- 跨会话保持上下文
- 持续学习用户的工作方式

## 记忆类型

### 1. 短期记忆（当前会话）

```javascript
// 在技能中访问短期记忆
module.exports = {
  handle(context) {
    // 获取当前会话的上下文
    const session = context.memory.session;
    // 存储临时信息
    session.temp = { key: 'value' };
  }
};
```

### 2. 长期记忆（持久存储）

```javascript
// 存储长期信息
module.exports = {
  async handle(context) {
    // 存储用户偏好
    await context.memory.persist.set('user:preferences', {
      theme: 'dark',
      language: 'zh-CN'
    });
    
    // 读取用户偏好
    const prefs = await context.memory.persist.get('user:preferences');
  }
};
```

### 3. 每日笔记

```markdown
# 2026-03-16 工作日志

## 完成事项
- 创建 GitHub 仓库
- 完善 README

## 待办
- 推广内容
```

## 配置记忆系统

```yaml
# config.yaml
memory:
  enabled: true
  type: local  # local / sqlite
  
  local:
    path: ~/.openclaw/memory
    
  sqlite:
    path: ~/.openclaw/memory.db
```

## 使用场景

### 场景 1：记住用户偏好

```javascript
// 用户说 "以后叫我张三"
module.exports = {
  async handle(context) {
    const name = context.message.extractName(); // 提取"张三"
    await context.memory.persist.set('user:name', name);
    return `好的，以后叫你 ${name}！`;
  }
};

// 下次用户说 "你好"
module.exports = {
  async handle(context) {
    const name = await context.memory.persist.get('user:name') || '朋友';
    return `你好，${name}！`;
  }
};
```

### 场景 2：记住项目上下文

```javascript
// 用户在不同会话中继续之前的工作
module.exports = {
  async handle(context) {
    // 检查是否有未完成的任务
    const task = await context.memory.persist.get('project:currentTask');
    if (task) {
      return `继续之前的工作：${task.description}`;
    }
  }
};
```

## 下一步

- [06. 工作流自动化](./06-workflow.md) — 创建自动化工作流
