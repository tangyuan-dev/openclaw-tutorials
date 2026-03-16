# 13. 最佳实践

本文汇总 OpenClaw 开发与运维的最佳实践。

## 开发最佳实践

### 1. Skill 设计原则

#### 单一职责

```javascript
// ❌ 一个 Skill 做太多事情
module.exports = {
  name: 'all-in-one',
  handle(context) {
    // 处理天气
    // 处理新闻
    // 处理提醒
    // ...太多功能
  }
};

// ✅ 每个 Skill 只做一件事
module.exports = {
  name: 'weather',
  handle(context) {
    // 只处理天气
  }
};

module.exports = {
  name: 'reminder',
  handle(context) {
    // 只处理提醒
  }
};
```

#### 清晰的触发条件

```javascript
// ❌ 模糊的匹配
match(context) {
  return true; // 匹配所有消息
}

// ✅ 明确的匹配
match(context) {
  const msg = context.message.toLowerCase();
  return msg.includes('天气') || msg.includes('weather');
}
```

### 2. 错误处理

```javascript
module.exports = {
  async handle(context) {
    try {
      // 核心逻辑
      const result = await this.process(context);
      return result;
      
    } catch (error) {
      // 记录错误
      console.error('Skill error:', error);
      
      // 返回友好提示
      return '抱歉出错了，请稍后再试';
    }
  }
};
```

### 3. 日志记录

```javascript
module.exports = {
  async handle(context) {
    // 记录请求
    console.info('Skill called:', {
      skill: this.name,
      user: context.userId,
      message: context.message,
      time: new Date().toISOString()
    });
    
    const result = await this.process(context);
    
    // 记录响应
    console.info('Skill completed:', {
      skill: this.name,
      duration: Date.now() - startTime
    });
    
    return result;
  }
};
```

## 运维最佳实践

### 1. 配置管理

#### 环境变量

```bash
# .env 文件
OPENAI_API_KEY=sk-xxx
TELEGRAM_BOT_TOKEN=xxx
DATABASE_URL=postgres://xxx
```

```yaml
# config.yaml
model:
  provider: openai
  apiKey: ${OPENAI_API_KEY}
```

#### 敏感信息

```yaml
# ❌ 不要这样写
config:
  apiKey: "sk-xxx"  # 明文密码

# ✅ 使用环境变量
config:
  apiKey: ${OPENAI_API_KEY}
```

### 2. 监控告警

#### 关键指标

| 指标 | 告警阈值 |
|------|---------|
| 错误率 | > 1% |
| 响应延迟 P99 | > 5s |
| CPU 使用率 | > 80% |
| 内存使用率 | > 85% |

#### 告警模板

```yaml
alerts:
  - name: high_error_rate
    condition: errors / total_requests > 0.01
    severity: critical
    message: |
      🚨 错误率过高！
      当前错误率: {{ error_rate }}%
      请立即检查！
      
  - name: slow_response
    condition: p99_latency > 5000
    severity: warning
    message: |
      ⚠️ 响应较慢
      P99 延迟: {{ latency }}ms
```

### 3. 容量规划

#### 评估标准

```
用户数 → 请求量 → 资源需求

100 用户 → 1000 请求/天 → 1 CPU, 1GB 内存
1000 用户 → 10000 请求/天 → 2 CPU, 2GB 内存
10000 用户 → 100000 请求/天 → 4+ CPU, 4+GB 内存
```

#### 自动扩缩容

```yaml
autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 10
  
  metrics:
    - type: cpu
      target: 70%
    - type: memory
      target: 80%
    - type: requests
      target: 1000 per pod
```

## 安全最佳实践

### 1. 输入验证

```javascript
module.exports = {
  async handle(context) {
    // 验证输入
    if (!context.message || context.message.length > 1000) {
      return '输入内容过长，请简化';
    }
    
    // 过滤敏感词
    const sensitiveWords = ['password', 'token', '密钥'];
    for (const word of sensitiveWords) {
      if (context.message.toLowerCase().includes(word)) {
        return '检测到敏感信息，请勿提交';
      }
    }
    
    // 继续处理
    return await this.process(context);
  }
};
```

### 2. 速率限制

```yaml
rateLimit:
  enabled: true
  
  limits:
    - endpoint: "/api/chat"
      requests: 100
      window: 60  # 每分钟
      
    - endpoint: "/api/skill"
      requests: 1000
      window: 3600  # 每小时
```

### 3. 审计日志

```yaml
audit:
  enabled: true
  
  log:
    - user_login
    - api_call
    - config_change
    - sensitive_operation
    
  retention: 90 days
```

## 性能最佳实践

### 1. 缓存策略

```javascript
const cache = new Map();

module.exports = {
  async handle(context) {
    const cacheKey = this.getKey(context);
    
    // 命中缓存
    if (cache.has(cacheKey)) {
      return cache.get(cacheKey);
    }
    
    // 计算
    const result = await this.compute(context);
    
    // 存入缓存（5分钟过期）
    cache.set(cacheKey, result);
    setTimeout(() => cache.delete(cacheKey), 300000);
    
    return result;
  }
};
```

### 2. 并发控制

```javascript
// 使用信号量控制并发
const semaphore = new Semaphore(10); // 最多10个并发

module.exports = {
  async handle(context) {
    await semaphore.acquire();
    try {
      return await this.process(context);
    } finally {
      semaphore.release();
    }
  }
};
```

### 3. 连接复用

```javascript
// 全局 HTTP 客户端
const httpClient = axios.create({
  timeout: 10000,
  retries: 3
});

module.exports = {
  async callAPI(url) {
    return await httpClient.get(url);
  }
};
```

## 故障排查

### 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 响应慢 | LLM API 慢 | 增加超时、优化 Prompt |
| 内存高 | 缓存过大 | 限制缓存大小 |
| 错误多 | API 不稳定 | 增加重试机制 |
| 队列满 | 请求太多 | 限流、扩容 |

### 调试命令

```bash
# 查看日志
openclaw logs --tail 100

# 查看性能
openclaw status

# 测试 Skill
openclaw skill test weather --input "北京天气"

# 健康检查
openclaw health
```

## 总结

1. **开发** — 单一职责、清晰触发、错误处理
2. **运维** — 配置管理、监控告警、容量规划
3. **安全** — 输入验证、速率限制、审计日志
4. **性能** — 缓存策略、并发控制、连接复用
