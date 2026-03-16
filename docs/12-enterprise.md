# 12. 企业级部署指南

本文介绍如何将 OpenClaw 部署到企业环境。

## 企业需求

### 常见企业场景

| 场景 | 需求 |
|------|------|
| 内部客服 | 多员工、多渠道、安全 |
| 知识库 | 私有数据、向量化 |
| 流程自动化 | RPA、审批流 |
| 数据分析 | API 对接、报表 |

## 部署架构

### 单机部署

```
┌─────────────┐
│   OpenClaw │
├─────────────┤
│  Skills     │
│  Memory     │
│  Channels   │
└─────────────┘
```

### 集群部署

```
           ┌─────────────┐
           │   Nginx     │
           └──────┬──────┘
                  │
      ┌───────────┼───────────┐
      │           │           │
┌─────┴─────┐ ┌──┴────┐ ┌───┴────┐
│ OpenClaw 1│ │ OpenClaw 2│ │OpenClaw3│
│ (Worker)  │ │(Worker) │ │(Worker)│
└───────────┘ └────────┘ └────────┘
      │           │           │
      └───────────┼───────────┘
                  │
           ┌──────┴──────┐
           │   Redis     │
           │   (共享)     │
           └─────────────┘
```

## Docker 部署

### 1. 创建 Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

# 安装依赖
COPY package*.json ./
RUN npm install --production

# 复制应用
COPY . .

# 暴露端口
EXPOSE 8080

# 启动
CMD ["npm", "start"]
```

### 2. 创建 docker-compose.yml

```yaml
version: '3.8'

services:
  openclaw:
    build: .
    ports:
      - "8080:8080"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - DATABASE_URL=postgres://user:pass@db:5432/openclaw
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: openclaw
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 3. 启动

```bash
docker-compose up -d
```

## 安全配置

### 1. API 鉴权

```yaml
# config.yaml
security:
  apiKey:
    enabled: true
    keys:
      - key: "client-1"
        name: "客户A"
        rateLimit: 1000
      - key: "client-2"
        name: "客户B"
        rateLimit: 500
        
  jwt:
    enabled: true
    secret: ${JWT_SECRET}
    expiresIn: 24h
```

### 2. 敏感数据加密

```yaml
security:
  encryption:
    enabled: true
    algorithm: AES-256-GCM
    
  secrets:
    management: vault  # HashiCorp Vault
```

### 3. 网络隔离

```yaml
security:
  network:
    # 只允许内网访问
    allowedIPs:
      - 10.0.0.0/8
      - 172.16.0.0/12
      
    # 禁止外网访问敏感端口
    blockedPorts:
      - 22   # SSH
      - 3306 # MySQL
```

## 监控运维

### 1. 日志收集

```yaml
logging:
  level: info
  outputs:
    - type: file
      path: /var/log/openclaw/app.log
    - type: stdout
    - type: remote
      url: http://logs.example.com/ingest
```

### 2. 指标监控

```yaml
monitoring:
  prometheus:
    enabled: true
    port: 9090
    
  metrics:
    - requests_total
    - requests_duration
    - errors_total
    - active_connections
```

### 3. 告警配置

```yaml
alerts:
  email:
    enabled: true
    to: admin@example.com
    
  webhook:
    enabled: true
    url: https://hooks.example.com/alert
    
  rules:
    - name: high_error_rate
      condition: errors_rate > 0.05
      message: "错误率超过 5%"
      
    - name: high_latency
      condition: p95_latency > 1000
      message: "延迟超过 1 秒"
```

## 备份恢复

### 1. 数据备份

```bash
# 备份数据库
docker exec openclaw-db pg_dump -U user openclaw > backup.sql

# 备份配置
cp ~/.openclaw/config.yaml ./config.backup.yaml
```

### 2. 自动备份

```yaml
backup:
  enabled: true
  schedule: "0 2 * * *"  # 每天凌晨2点
  
  targets:
    - type: database
      destination: s3://backups/openclaw/db/
    - type: config
      destination: s3://backups/openclaw/config/
```

### 3. 恢复

```bash
# 恢复数据库
docker exec -i openclaw-db psql -U user openclaw < backup.sql

# 重启服务
docker-compose restart
```

## 性能优化

### 1. 缓存

```yaml
cache:
  enabled: true
  type: redis
  
  ttl:
    user_session: 3600
    api_response: 300
    llm_response: 600
```

### 2. 连接池

```yaml
database:
  pool:
    min: 5
    max: 20
    
redis:
  pool:
    min: 5
    max: 50
```

### 3. 水平扩展

```bash
# 增加 Worker
docker-compose up -d --scale openclaw=3
```

## 下一步

- [13. 最佳实践](./13-best-practices.md) — 开发与运维最佳实践
