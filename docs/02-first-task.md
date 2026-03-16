# 02. 你的第一个任务

本指南将帮助你完成第一个 OpenClaw 任务。

## 基本使用

OpenClaw 是一个对话式 AI 助手，你可以用自然语言描述你想做的事情。

## 任务类型

### 📁 文件操作

```bash
# 创建文件
"帮我创建一个 hello.py 文件，输出 Hello World"

# 读取文件
"读取当前目录下的 config.yaml"

# 批量操作
"把当前文件夹的所有 .txt 文件重命名为 .md"
```

### 💻 代码执行

```bash
# 运行代码
"运行 main.py"

# 执行命令
"执行 git status 查看仓库状态"
"安装 express 包"
```

### 🌐 网页浏览

```bash
# 打开网页
"打开 https://github.com 并提取页面内容"

# 搜索信息
"搜索 OpenClaw 的最新版本"
```

### 📝 内容创作

```bash
# 写文章
"写一篇关于 Vibe Coding 的博客文章"

# 生成代码
"帮我写一个天气查询的 API"
```

## 对话示例

### 示例 1：创建项目

**你：** 帮我创建一个新的 Node.js 项目

**OpenClaw：**
```
好的，我来帮你创建 Node.js 项目...

✅ 已完成：
- 初始化 package.json
- 创建 index.js
- 安装依赖 express, cors

📁 项目结构：
.
├── package.json
├── index.js
└── node_modules/

下一步：输入 openclaw start 启动项目
```

### 示例 2：代码调试

**你：** 帮我看看这段代码有什么问题

```javascript
function sum(arr) {
  return arr.reduce((a, b) => a + b);
}

console.log(sum([1, 2, "3"]));
```

**OpenClaw：**
```
🔍 问题分析：

这段代码存在类型问题：
- 数组包含字符串 "3"
- 字符串 + 数字会导致拼接而非加法

⚠️ 输出结果：'12'（拼接）而非 6（加法）

✅ 修复建议：
function sum(arr) {
  return arr.reduce((a, b) => Number(a) + Number(b), 0);
}

或使用类型转换：
function sum(arr) {
  return arr.map(Number).reduce((a, b) => a + b, 0);
}
```

## 高级技巧

### 1. 指定上下文

```
"基于 /path/to/project 这个项目，帮我添加用户认证功能"
```

### 2. 分步执行

```
"先创建数据库连接，然后创建用户模型，最后添加 API 路由"
```

### 3. 引用文件

```
"分析 src/app.js 这段代码，告诉我它的功能"
```

## 下一步

- [03. 渠道配置](./03-channels.md) — 让 OpenClaw 通过更多渠道响应
- [04. 自定义技能](./04-custom-skill.md) — 创建你自己的技能
