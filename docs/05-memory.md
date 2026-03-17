# 05. 记忆系统 — 让 AI 记住你的偏好

> **💡 Motto**: "一次告诉 AI，下次它就记得"

这节教程的目标：让 AI 记住你的名字、偏好、习惯，下次不用再重复说。

---

## 🧩 这节学什么

| 技能 | 预计时间 | 难度 |
|------|---------|------|
| 记住用户名字 | 5 分钟 | ⭐ |
| 记住项目偏好 | 5 分钟 | ⭐ |
| 每日工作日志 | 10 分钟 | ⭐⭐ |

---

## 🎯 目标：做一个"记得住"的 AI 助手

学完这节，AI 会记住：
- 你叫什么名字
- 你喜欢什么编程语言
- 你上次做到哪了

---

## 🚀 第一步：启用记忆系统

```yaml
# ~/.openclaw/config.yaml
memory:
  enabled: true
  type: local  # local / sqlite
```

重启服务：
```bash
openclaw restart
```

---

## 🎯 场景 1：记住用户名字

**你说：** "以后叫我杜博士"

**AI 记住后，下次：**
- 你说："你好"
- AI 回复："你好，杜博士！"

### 快速实现

只需对 AI 说：
```
记住我的名字是杜博士
```

AI 会自动保存到记忆，下次就能识别你。

---

## 🎯 场景 2：记住项目偏好

**你说：** "我更喜欢 Python"

**AI 记住后：**
- 你说："帮我写个 API"
- AI 会用 Python 写

### 快速实现

```
记住我偏好使用 Python
记住我的项目都用 src 目录
记住我使用 pnpm 作为包管理器
```

---

## 🎯 场景 3：记住工作进度

**你说：** "我在做一个博客系统，先做到创建用户模型这里"

**AI 记住后，下次：**
- 你说："继续之前的博客项目"
- AI 回复："好的，继续博客系统，上次你做到创建用户模型..."

### 快速实现

```
记住我现在在做博客项目，已经完成了用户模型
```

---

## 🎯 场景 4：每日工作日志

AI 可以每天自动记录你做了什么。

### 查看日志

```
今天我做了什么？
```

### 添加日志

```
记录：今天完成了用户认证功能
```

---

## 💻 手动使用记忆 API

如果需要更精细的控制，可以使用记忆 API：

### 存储数据

```javascript
// 在技能中存储
await context.memory.persist.set('user:name', '杜博士');
await context.memory.persist.set('user:pref:language', 'python');
await context.memory.persist.set('project:blog:status', '用户模型完成');
```

### 读取数据

```javascript
const name = await context.memory.persist.get('user:name');
const lang = await context.memory.persist.get('user:pref:language');
```

### 读取所有记忆

```javascript
const all = await context.memory.persist.getAll('user:*');
// 返回 { 'user:name': '杜博士', 'user:pref:language': 'python' }
```

### 删除记忆

```javascript
await context.memory.persist.delete('user:name');
```

---

## 🗂️ 记忆存储位置

```
~/.openclaw/
├── memory/
│   ├── user/           # 用户记忆
│   │   └── default/
│   │       ├── name.txt
│   │       ├── preferences.json
│   │       └── project-blog.json
│   ├── daily/         # 每日日志
│   │   └── 2026-03-17.md
│   └── sessions/     # 会话历史
```

---

## 💡 实用记忆指令模板

| 场景 | 指令 |
|------|------|
| 记住名字 | 记住我叫 XXX |
| 记住偏好 | 记住我偏好 XXX |
| 记住项目 | 记住我项目是 XXX，做到 XXX 了 |
| 查看记忆 | 我的记忆里有什么？ |
| 继续工作 | 继续之前的 XXX 项目 |
| 忘记某事 | 忘记我 XXX 偏好 |

---

## ❓ 常见问题

### Q: 记忆在哪查看？
```bash
# 查看所有记忆
ls ~/.openclaw/memory/user/default/

# 查看具体记忆
cat ~/.openclaw/memory/user/default/name.txt
```

### Q: 记忆会丢失吗？
- 默认存在本地，不会丢失
- 可配置 SQLite 存储

### Q: 如何清空记忆？
```bash
rm -rf ~/.openclaw/memory/user/*
```

---

## 🎉 这节目标达成

✅ 会让 AI 记住名字  
✅ 会让 AI 记住偏好  
✅ 会让 AI 记住工作进度  
✅ 会使用记忆 API  

---

## ➡️ 下节学什么

- [06. 工作流自动化](./06-workflow.md) — 告别重复任务
- [07. Skill 开发实战](./07-skill.md) — 更复杂的技能开发
