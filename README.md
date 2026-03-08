# 🐟 小鱼背单词 (Fishy Words)

一个可爱风格的背单词网页应用，让学习变得更有趣！✨

## 🌟 功能特点

- 📚 **卡片式学习** - 正面单词，背面释义，点击翻转
- 📊 **学习统计** - 追踪今日学习、掌握单词、错题本、连续打卡
- 🎯 **错题本** - 自动记录不认识的单词，方便复习
- 📈 **进度追踪** - 实时显示学习进度
- 💾 **本地存储** - 数据保存在浏览器，无需登录
- 📱 **响应式设计** - 手机电脑都能用

## 🚀 快速开始

### 安装依赖

```bash
npm install
```

### 开发模式

```bash
npm run dev
```

### 构建生产版本

```bash
npm run build
```

### 预览构建结果

```bash
npm run preview
```

## 📦 部署到 Vercel

1. 安装 Vercel CLI：
```bash
npm install -g vercel
```

2. 登录 Vercel：
```bash
vercel login
```

3. 部署：
```bash
vercel
```

或者在 [Vercel 官网](https://vercel.com) 导入此仓库自动部署！

## 🛠️ 技术栈

- **前端框架**: Vue 3 + Vite
- **样式**: 原生 CSS
- **数据存储**: LocalStorage
- **部署**: Vercel

## 📝 词库

内置 CET-4 示例词库（20 词），可以自行扩展 `src/data/words.js` 文件添加更多单词。

### 添加新单词格式

```javascript
{
  id: 21,
  word: "example",
  pronunciation: "/ɪɡˈzæmpl/",
  phonetic: "美 [ɪɡˈzæmpl] 英 [ɪɡˈzɑːmpl]",
  meanings: ["n. 例子；榜样"],
  pos: "n.",
  example: "This is a good example.",
  exampleCn: "这是一个好例子。",
  tags: ["CET-4", "高频"]
}
```

## 🎨 自定义

### 修改主题色

编辑 `src/App.vue` 中的 CSS 变量：

```css
body {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
```

### 修改应用名称

编辑 `index.html` 和 `src/App.vue` 中的标题。

## 📄 License

MIT License

---

Made with ❤️ by 小鱼助手 for 鱼鱼
