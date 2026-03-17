# 12. 企业级部署 — Docker 与运维

> **💡 Motto**: "部署一次，稳跑一年"

这节教程的目标：学会用 Docker 部署 OpenClaw，掌握企业级安全配置、监控和运维。

---

## 🧩 这节学什么

| 技能 | 预计时间 | 难度 |
|------|---------|------|
| Docker 部署 | 10 分钟 | ⭐ |
| 安全配置 | 10 分钟 | ⭐⭐ |
| 监控与告警 | 10 分钟 | ⭐⭐ |
| 备份与恢复 | 10 分钟 | ⭐⭐ |

---

## 🎯 目标：部署一个稳定的企业级 AI 服务

学完这节，你将：
- 会用 Docker 部署
- 懂得配置安全
- 会设置监控告警
- 能做数据备份

---

## 🚀 第一步：Docker 部署（最简单）

### 为什么要用 Docker？

- 一键部署，不踩坑
- 环境一致，不出错
- 随时迁移，不担心

### 1 分钟部署

```bash
# 1. 安装 Docker Desktop (Windows/Mac)
# https://www.docker.com/products/docker-desktop/

# 2. 创建配置文件
mkdir openclaw-deploy && cd openclaw-deploy
```

### 创建 docker-compose.yml

```yaml
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    ports:
      - "8080:8080"
    environment:
      - OPENCLAW_MODEL=minimax
      - OPENCLAW_API_KEY=${OPENCLAW_API_KEY}
      - OPENCLAW_CHANNEL=feishu
      - FEISHU_APP_ID=${FEISHU_APP_ID}
      - FEISHU_APP_SECRET=${FEISHU_APP_SECRET}
    volumes:
      - ./data:/app/data
      - ./config:/app/config
    restart: unless-stopped

  # 可选：Redis 缓存
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  redis_data:
```

### 创建 .env 文件

```bash
# .env
OPENCLAW_API_KEY=your-api-key-here
FEISHU_APP_ID=your-feishu-app-id
FEISHU_APP_SECRET=your-feishu-app-secret
```

### 启动

```bash
# 启动服务
docker-compose up -d

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 访问

- 本地：http://localhost:8080
- 远程： http://你的IP:8080

---

## 🚀 第二步：安全配置

### 1. API 密钥保护

```yaml
# docker-compose.yml 中
environment:
  - OPENCLAW_API_KEY=${OPENCLAW_API_KEY}
```

**不要把密钥写死在代码里！** 用环境变量或 .env 文件。

### 2. 限制访问 IP

```yaml
# config.yaml
security:
  allowedIPs:
    - 127.0.0.1
    - 10.0.0.0/8    # 公司内网
    - 192.168.1.0/24  # 家庭网络
```

### 3. 禁用不需要的渠道

```yaml
channels:
  - type: feishu
    enabled: true
  - type: telegram
    enabled: false  # 关闭不需要的
```

### 4. 启用日志审计

```yaml
logging:
  level: info
  audit:
    enabled: true
    logFile: /app/data/audit.log
    
  # 记录所有 API 调用
  accessLog: true
```

---

## 🚀 第三步：监控与告警

### 1. 查看日志

```bash
# 查看实时日志
docker-compose logs -f

# 查看最近 100 行
docker-compose logs --tail 100

# 查看错误日志
docker-compose logs | grep ERROR
```

### 2. 设置告警（邮件/飞书通知）

```yaml
# config.yaml
alerts:
  enabled: true
  
  # 错误率告警
  rules:
    - name: high_error_rate
      condition: errors > 10
      interval: 5m
      message: "错误率过高，请检查！"
      
    - name: service_down
      condition: status == down
      message: "服务已宕机！"
  
  # 通知渠道
  channels:
    - type: email
      to: admin@example.com
    - type: webhook
      url: https://notify.example.com/alert
```

### 3. 监控指标

```bash
# 查看资源使用
docker stats

# 输出：
# CONTAINER     CPU%   MEM USAGE/LIMIT   NET I/O
# openclaw      2.5%   256MB/512MB       1.2MB/0.5MB
```

---

## 🚀 第四步：备份与恢复

### 1. 手动备份

```bash
# 备份配置
cp -r ~/.openclaw/config.yaml ./backup/

# 备份记忆数据
cp -r ~/.openclaw/memory ./backup/memory

# 备份技能
cp -r ~/.openclaw/skills ./backup/skills
```

### 2. 自动备份（每天凌晨 2 点）

```yaml
# docker-compose.yml 中添加
services:
  backup:
    image: openclaw/openclaw:latest
    volumes:
      - ./backup:/backup
    command: >
      sh -c "crond && tail -f /dev/null"
    environment:
      - BACKUP_ENABLED=true
      - BACKUP_SCHEDULE="0 2 * * *"
      - BACKUP_DEST=/backup
```

### 3. 恢复数据

```bash
# 停止服务
docker-compose down

# 恢复配置
cp ./backup/config.yaml ~/.openclaw/config.yaml

# 恢复记忆
cp -r ./backup/memory ~/.openclaw/memory

# 启动服务
docker-compose up -d
```

---

## 🚀 第五步：域名与 HTTPS

### 1. 购买域名

- 阿里云、腾讯云、Namecheap

### 2. 配置 DNS

```
A 记录 @ 你的服务器IP
```

### 3. 配置 Nginx + HTTPS（免费证书）

```nginx
# nginx.conf
server {
    listen 80;
    server_name your-domain.com;
    
    # 自动申请 SSL 证书
    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }
    
    location / {
        return 301 https://$server_name$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;
    
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 4. 使用 Nginx Proxy Manager（可视化）

```bash
# docker-compose.yml
services:
  nginx-proxy-manager:
    image: jc21/nginx-proxy-manager:latest
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    volumes:
      - ./nginx/data:/data
```

---

## 🧪 生产环境检查清单

| 检查项 | 状态 |
|--------|------|
| Docker 已安装 | ⬜ |
| .env 配置完成 | ⬜ |
| 防火墙端口开放（80, 443, 8080）| ⬜ |
| 日志正常 | ⬜ |
| 告警配置 | ⬜ |
| 备份已测试 | ⬜ |
| 域名解析正常 | ⬜ |
| HTTPS 生效 | ⬜ |

---

## 🎉 这节目标达成

✅ 会用 Docker 部署  
✅ 懂得安全配置  
✅ 会设置监控告警  
✅ 能做数据备份  

---

## ➡️ 下节学什么

- [13. 最佳实践](./13-best-practices.md) — 安全、性能、运维经验
