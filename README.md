# 个人博客系统

一个现代化的个人博客系统，支持文章发布、管理、分类和评论等功能。

## 项目概述

本项目是一个全栈个人博客应用，分为前台展示和后台管理两部分。前台负责展示博客内容，后台负责内容管理。

### 功能特点

- 📝 文章发布与管理
- 🏷️ 分类与标签系统
- 🔍 全文搜索功能
- 💬 评论系统
- 📊 数据统计与分析
- 🌓 深色模式支持
- 📱 响应式设计，适配移动设备

## 技术栈

### 前端
- React 18
- TypeScript 5.x
- Next.js 14 (App Router)
- Tailwind CSS
- React Query
- React Hook Form

### 后端
- Node.js
- Express.js
- TypeScript
- MongoDB
- Mongoose ODM

### 开发工具
- ESLint
- Prettier
- Jest
- React Testing Library
- Docker
- GitHub Actions

## 项目结构

```
personal-blog-test/
├── client/                 # 前端代码
│   ├── app/                # Next.js App Router
│   ├── components/         # React组件
│   ├── lib/                # 工具函数
│   ├── public/             # 静态资源
│   └── styles/             # 样式文件
├── server/                 # 后端代码
│   ├── controllers/        # 控制器
│   ├── models/             # 数据模型
│   ├── routes/             # API路由
│   ├── services/           # 业务逻辑
│   └── utils/              # 工具函数
├── .github/                # GitHub配置
├── docs/                   # 项目文档
└── scripts/                # 脚本工具
```

## 开发指南

### 环境要求
- Node.js 18+
- MongoDB 6.0+
- npm 9+ 或 yarn 1.22+

### 安装与运行

1. 克隆仓库
```bash
git clone https://github.com/Zanedname/personal-blog-test.git
cd personal-blog-test
```

2. 安装依赖
```bash
# 安装前端依赖
cd client
npm install

# 安装后端依赖
cd ../server
npm install
```

3. 配置环境变量
```bash
# 在server目录下创建.env文件
cp .env.example .env
# 编辑.env文件，填入必要的配置
```

4. 启动开发服务器
```bash
# 启动后端服务
cd server
npm run dev

# 启动前端服务
cd ../client
npm run dev
```

5. 访问应用
- 前台: http://localhost:3000
- 后台: http://localhost:3000/admin
- API: http://localhost:4000/api

## 部署指南

详细的部署指南请参考 [docs/deployment.md](./docs/deployment.md)。

## 贡献指南

欢迎贡献代码，请参考 [CONTRIBUTING.md](./CONTRIBUTING.md)。

## 许可证

本项目采用 MIT 许可证，详情请参阅 [LICENSE](./LICENSE) 文件。