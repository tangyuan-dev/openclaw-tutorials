# 02. 用自然语言让 AI 帮你干活

> **💡 Motto**: "不需要会代码，只需要说人话"

这节教程的目标：学会用自然语言指挥 OpenClaw 完成各种实际任务。

---

## 🧩 这节学什么

| 技能 | 示例 |
|------|------|
| 📁 文件操作 | 创建、读取、修改文件 |
| 💻 代码执行 | 运行代码、安装包 |
| 🌐 网页浏览 | 搜索、提取网页内容 |
| 📝 内容创作 | 写文章、生成代码 |

---

## 🎯 场景 1：文件操作

### 场景 1.1：创建文件

**你说：**
```
创建一个 Python 文件 hello.py，输出 Hello World
```

**OpenClaw 做的：**
```python
# hello.py
print("Hello World")
```

---

### 场景 1.2：批量重命名

**你说：**
```
把当前文件夹所有 .txt 文件改成 .md
```

**OpenClaw 做的：**
- 列出所有 .txt 文件
- 逐个重命名

---

### 场景 1.3：读取并分析文件

**你说：**
```
读取 config.yaml，告诉我数据库配置是什么
```

**OpenClaw 做的：**
- 读取文件内容
- 提取数据库配置信息

---

## 🎯 场景 2：代码执行

### 场景 2.1：运行代码

**你说：**
```
运行 hello.py
```

**OpenClaw 做的：**
```bash
python hello.py
# 输出: Hello World
```

---

### 场景 2.2：安装依赖

**你说：**
```
安装 express 和 cors 这两个 npm 包
```

**OpenClaw 做的：**
```bash
npm install express cors
```

---

### 场景 2.3：调试代码

**你说：**
```
帮我看看下面这段代码有什么 bug：

function sum(arr) {
  return arr.reduce((a, b) => a + b);
}

console.log(sum([1, 2, "3"]));
```

**OpenClaw 做的：**
```
🔍 问题分析：
- 数组包含字符串 "3"
- "3" + 1 = "31"（字符串拼接）
- 正确结果应该是 6

✅ 修复：
function sum(arr) {
  return arr.reduce((a, b) => Number(a) + Number(b), 0);
}
```

---

## 🎯 场景 3：网页浏览

### 场景 3.1：搜索信息

**你说：**
```
搜索 OpenClaw 的最新版本
```

**OpenClaw 做的：**
- 打开搜索引擎
- 提取最新版本信息

---

### 场景 3.2：提取网页内容

**你说：**
```
打开 https://github.com/openclaw/openclaw，告诉我这个项目有多少 stars
```

**OpenClaw 做的：**
- 打开网页
- 提取 stars 数量

---

## 🎯 场景 4：内容创作

### 场景 4.1：写代码

**你说：**
```
帮我写一个 RESTful API，包含用户注册和登录接口，使用 Express
```

**OpenClaw 做的：**
```javascript
const express = require('express');
const app = express();
app.use(express.json());

// 用户注册
app.post('/api/register', (req, res) => {
  const { username, password } = req.body;
  // TODO: 保存到数据库
  res.json({ message: '注册成功' });
});

// 用户登录
app.post('/api/login', (req, res) => {
  const { username, password } = req.body;
  // TODO: 验证密码
  res.json({ token: 'xxx' });
});

app.listen(3000);
```

---

### 场景 4.2：写文章

**你说：**
```
写一篇 500 字的文章，介绍什么是 Vibe Coding
```

**OpenClaw 做的：**
- 生成结构化文章
- 包含标题、要点、总结

---

## 🛠️ 高阶技巧

### 1. 指定项目上下文

```
基于当前这个 Vue 项目，帮我添加用户认证功能
```

### 2. 分步执行

```
先创建数据库连接 -> 然后创建用户模型 -> 最后添加 API 路由
```

### 3. 引用具体文件

```
分析 src/app.js 这段代码，告诉我它的功能是什么
```

### 4. 复杂任务拆解

```
我需要一个图片处理工具，能：
1. 读取文件夹所有图片
2. 压缩到 50% 质量
3. 输出到 output 文件夹
```

---

## 📋 常用指令模板

| 场景 | 模板 | 示例 |
|------|------|------|
| 创建文件 | 创建 xxx 文件，内容是... | 创建 test.py，内容是 print("hi") |
| 读取文件 | 读取 xxx | 读取 config.json |
| 运行代码 | 运行 xxx | 运行 main.py |
| 安装包 | 安装 xxx | 安装 express |
| 写代码 | 帮我写一个 xxx | 写一个排序算法 |
| 搜索 | 搜索 xxx | 搜索 OpenClaw 文档 |

---

## 🎉 这节目标达成

✅ 会用自然语言指挥 AI  
✅ 会处理文件、运行代码、浏览网页  
✅ 能完成日常开发任务  

---

## ➡️ 下节学什么

- [03. 把 AI 接入微信/飞书/Telegram](./03-channels.md) — 让 AI 随时随地响应你
- [04. 自定义技能](./04-custom-skill.md) — 教 AI 更多技能
