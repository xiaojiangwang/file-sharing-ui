# 文件分享平台 (File Sharing Platform)

一个基于 Vue 3 + Vite + Ant Design Vue 构建的现代化文件分享平台前端应用。支持文件上传、文本分享、密码保护等功能，提供简洁美观的用户界面。

## ✨ 功能特性

### 📁 文件上传
- 🚀 支持拖拽上传和点击上传
- 📦 支持单个或批量文件上传
- 🔒 可选密码保护功能
- 📝 支持添加备注信息
- 📊 实时上传进度显示
- 💾 文件列表管理（查看、下载、删除）
- 📏 文件大小限制和格式化显示

### 📝 文本分享
- ✍️ 在线文本内容上传
- 🔐 文本密码保护
- 📋 一键复制文本内容
- 🗂️ 文本历史记录管理
- 💬 支持备注信息

### 🛡️ 安全特性
- 🔑 密码保护机制
- 🔒 安全的文件访问控制
- ✅ 文件上传前验证

### 🎨 用户体验
- 📱 响应式设计，支持多设备访问
- 🎯 直观的标签页切换
- ⚡ 流畅的交互动画
- 🔔 友好的错误提示和状态反馈

## 🛠️ 技术栈

- **前端框架**: [Vue 3](https://vuejs.org/) - 渐进式 JavaScript 框架
- **构建工具**: [Vite](https://vitejs.dev/) - 下一代前端构建工具
- **UI 组件库**: [Ant Design Vue](https://antdv.com/) - 企业级 UI 设计语言
- **图标库**: [@ant-design/icons-vue](https://github.com/ant-design/ant-design-icons) - Ant Design 图标库
- **HTTP 客户端**: [Axios](https://axios-http.com/) - Promise 基的 HTTP 库
- **开发语言**: JavaScript (ES6+)
- **样式**: CSS3 + Ant Design Vue 样式系统

## 📋 系统要求

- Node.js >= 16.0.0
- npm >= 8.0.0 或 yarn >= 1.22.0

## 🚀 快速开始

### 1. 克隆项目

```bash
git clone <repository-url>
cd file-sharing-front
```

### 2. 安装依赖

```bash
npm install
# 或
yarn install
```

### 3. 启动开发服务器

```bash
npm run dev
# 或
yarn dev
```

开发服务器将在 `http://localhost:5173` 启动

### 4. 构建生产版本

```bash
npm run build
# 或
yarn build
```

### 5. 预览生产构建

```bash
npm run preview
# 或
yarn preview
```

## 📁 项目结构

```
file-sharing-front/
├── public/                 # 静态资源
│   ├── cloud-upload.svg   # 上传图标
│   └── vite.svg           # Vite 图标
├── src/                   # 源代码
│   ├── assets/            # 资源文件
│   ├── components/        # Vue 组件
│   │   ├── FileUpload.vue # 文件上传主组件
│   │   └── HelloWorld.vue # 示例组件
│   ├── App.vue           # 根组件
│   ├── main.js           # 应用入口
│   └── style.css         # 全局样式
├── index.html            # HTML 模板
├── package.json          # 项目配置
├── vite.config.js        # Vite 配置
└── README.md            # 项目文档
```

## ⚙️ 配置说明

### API 代理配置

项目配置了 API 代理，将 `/api` 路径的请求代理到后端服务器：

```javascript
// vite.config.js
server: {
  proxy: {
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true
    }
  }
}
```

### 环境变量

可以通过环境变量配置不同环境的 API 地址：

```bash
# .env.development
VITE_API_BASE_URL=http://localhost:8080

# .env.production
VITE_API_BASE_URL=https://your-api-domain.com
```

## 🔧 开发指南

### 组件开发

主要组件位于 `src/components/` 目录：

- `FileUpload.vue`: 核心文件上传组件，包含文件上传和文本分享功能
- 组件使用 Vue 3 Composition API 和 `<script setup>` 语法

### 样式规范

- 使用 Ant Design Vue 的设计系统
- 遵循 BEM 命名规范
- 响应式设计优先

### API 接口

项目与后端 API 交互，主要接口包括：

- `POST /api/files/upload` - 文件上传
- `GET /api/files/:id` - 文件下载
- `DELETE /api/files/:id` - 文件删除
- `POST /api/texts/upload` - 文本上传

## 🤝 贡献指南

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开 Pull Request

## 📝 开发规范

- 遵循 Vue 3 最佳实践
- 使用 ESLint 进行代码检查
- 组件命名使用 PascalCase
- 文件命名使用 kebab-case
- 提交信息遵循 [Conventional Commits](https://conventionalcommits.org/)

## 🐛 问题反馈

如果您发现任何问题或有改进建议，请：

1. 查看 [Issues](../../issues) 确认问题是否已存在
2. 创建新的 Issue 并详细描述问题
3. 提供复现步骤和环境信息

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情

## 🙏 致谢

- [Vue.js](https://vuejs.org/) - 优秀的前端框架
- [Ant Design Vue](https://antdv.com/) - 精美的 UI 组件库
- [Vite](https://vitejs.dev/) - 快速的构建工具

## 📞 联系方式

- 项目维护者: [Your Name]
- 邮箱: [your.email@example.com]
- 项目主页: [Project Homepage]

---

⭐ 如果这个项目对您有帮助，请给我们一个 Star！
