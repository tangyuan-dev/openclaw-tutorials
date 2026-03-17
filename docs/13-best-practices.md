# 13. 最佳实践 — 安全、性能、运维经验

> **💡 Motto**: "少踩坑、多睡觉"

这节教程的目标：汇总开发、运维、安全、性能的最佳实践，让你少走弯路。

---

## 🧩 这节学什么

| 模块 | 预计时间 | 难度 |
|------|---------|------|
| 开发最佳实践 | 10 分钟 | ⭐ |
| 运维最佳实践 | 10 分钟 | ⭐⭐ |
| 安全最佳实践 | 10 分钟 | ⭐⭐ |
| 性能最佳实践 | 10 分钟 | ⭐⭐⭐ |

---

## 🚀 开发最佳实践

### 原则 1：单一职责

```javascript
// ❌ 一个 Skill 干太多事
module.exports = {
  name: 'all-in-one',
  handle(context) {
    // 天气 + 新闻 + 提醒 + ... 太乱了
  }
};

// ✅ 一个 Skill 只做一件事
module.exports = {
  name: 'weather',
  handle(context) { /* 只处理天气 */ }
};

module.exports = {
  name: 'reminder',
  handle(context) { /* 只处理提醒 */ }
};
```

### 原则 2：清晰的触发条件

```javascript
// ❌ 模糊匹配
match(context) {
  return true;  // 匹配所有消息
}

// ✅ 精确匹配
match(context) {
  const msg = context.message.toLowerCase();
  return msg.includes('天气');
}
```

### 原则 3：完善的错误处理

```javascript
module.exports = {
  async handle(context) {
    try {
      const result = await this.process(context);
      return result;
    } catch (error) {
      console.error('错误:', error);
      return '抱歉出错了，请稍后再试';
    }
  }
};
```

### 原则 4：日志记录

```javascript
module.exports = {
  async handle(context) {
    console.info('Skill 调用:', {
      skill: this.name,
      user: context.userId,
      time: new Date().toISOString()
    });
    
    const result = await this.process(context);
    
    return result;
  }
};
```

---

## 🚀 运维最佳实践

### 1. 配置管理

```bash
# ✅ .env 文件（不提交到 Git）
OPENAI_API_KEY=sk-xxx
TELEGRAM_BOT_TOKEN=xxx
```

```yaml
# config.yaml
model:
  provider: openai
  apiKey: ${OPENAI_API_KEY}  # 引用环境变量
```

**⚠️ 永远不要把密钥写死在代码里！**

### 2. 关键指标监控

| 指标 | 正常 | 告警 |
|------|------|------|
| 错误率 | < 1% | > 5% |
| 响应时间 | < 3s | > 10s |
| CPU | < 70% | > 90% |
| 内存 | < 80% | > 95% |

### 3. 日告警模板

```yaml
alerts:
  - name: high_error_rate
    condition: errors > 10
    message: "⚠️ 错误率过高，请检查！"
    
  - name: service_down
    condition: status == down  
    message: "🚨 服务已宕机！"
```

---

## 🚀 安全最佳实践

### 1. 输入验证

```javascript
module.exports = {
  async handle(context) {
    // 长度限制
    if (context.message?.length > 2000) {
      return '输入内容过长，请简化';
    }
    
    // 敏感词过滤
    const sensitive = ['密码', 'token', '密钥', 'password'];
    if (sensitive.some(w => context.message.includes(w))) {
      return '检测到敏感信息，已拦截';
    }
    
    return await this.process(context);
  }
};
```

### 2. 速率限制

```yaml
# config.yaml
rateLimit:
  enabled: true
  limits:
    - endpoint: /api/chat
      requests: 100
      window: 60  # 每分钟
```

### 3. IP 白名单

```yaml
security:
  allowedIPs:
    - 127.0.0.1
    - 10.0.0.0/8    # 公司内网
    - 192.168.1.0/24
```

---

## 🚀 性能最佳实践

### 1. 缓存策略

```javascript
const cache = new Map();

module.exports = {
  async handle(context) {
    const key = JSON.stringify(context.message);
    
    // 命中缓存
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    // 计算结果
    const result = await this.process(context);
    
    // 缓存 5 分钟
    cache.set(key, result);
    setTimeout(() => cache.delete(key), 300000);
    
    return result;
  }
};
```

### 2. 并发控制

```javascript
// 最多 10 个并发请求
const queue = [];
let running = 0;

async function addTask(fn) {
  return new Promise((resolve, reject) => {
    queue.push({ fn, resolve, reject });
    process();
  });
}

async function process() {
  if (running >= 10 || queue.length === 0) return;
  
  running++;
  const task = queue.shift();
  
  try {
    task.resolve(await task.fn());
  } catch (e) {
    task.reject(e);
  } finally {
    running--;
    process();
  }
}
```

### 3. 连接复用

```javascript
// 全局 HTTP 客户端
const http = require('axios').create({
  timeout: 10000
});

module.exports = {
  async callAPI(url) {
    return await http.get(url);  // 复用连接
  }
};
```

---

## 🐛 常见问题排查

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 响应慢 | LLM API 慢 | 增加超时、优化 Prompt |
| 内存高 | 缓存太大 | 限制缓存大小 |
| 错误多 | API 不稳定 | 增加重试 |
| 队列满 | 请求太多 | 限流、扩容 |

### 调试命令

```bash
# 查看日志
openclaw logs --tail 100

# 查看状态
openclaw status

# 健康检查
openclaw health

# 测试技能
openclaw skill test weather --input "北京天气"
```

---

## ✅ 快速检查清单

### 上线前检查

- [ ] 错误处理已加
- [ ] 日志已加
- [ ] 敏感信息用环境变量
- [ ] 速率限制已配
- [ ] 监控告警已配
- [ ] 备份已测试

### 运维检查

- [ ] 每周看一次错误日志
- [ ] 每月检查一次容量
- [ ] 定期更新依赖
- [ ] 定期测试备份恢复

---

## 🎉 教程完成！

恭喜你学完了整个 OpenClaw 教程！

### 你现在会的

| 技能 | 掌握程度 |
|------|---------|
| 安装配置 | ✅ 从 0 到 1 |
| 自然语言编程 | ✅ 用人话指挥 AI |
| 自定义技能 | ✅ 独立开发 |
| 记忆系统 | ✅ 让 AI 记住一切 |
| 自动化工作流 | ✅ 告别重复任务 |
| 产品思维 | ✅ 能做出产品 |
| 部署运维 | ✅ 企业级部署 |

### 下一步

1. **动手做** — 做一个自己的 AI 助手
2. **分享** — 把教程分享给朋友
3. **贡献** — 给 OpenClaw 提 Issue/PR
4. **探索** — 学习更多 AI 知识

---

**🎯 记住：AI 不是替代你，而是放大你。**

开始行动吧！
