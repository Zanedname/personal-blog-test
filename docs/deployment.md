# 部署指南

本文档提供了个人博客系统的部署说明，包括开发环境、测试环境和生产环境的部署方法。

## 环境要求

- Node.js 18+
- MongoDB 6.0+
- Docker (可选，用于容器化部署)
- Nginx (可选，用于反向代理)

## 本地开发环境部署

### 1. 克隆仓库

```bash
git clone https://github.com/Zanedname/personal-blog-test.git
cd personal-blog-test
```

### 2. 安装依赖

```bash
# 安装前端依赖
cd client
npm install

# 安装后端依赖
cd ../server
npm install
```

### 3. 配置环境变量

在server目录下创建.env文件：

```bash
cp .env.example .env
```

编辑.env文件，填入必要的配置：

```
# 服务器配置
PORT=4000
NODE_ENV=development

# 数据库配置
MONGODB_URI=mongodb://localhost:27017/personal-blog

# JWT配置
JWT_SECRET=your_jwt_secret_key
JWT_EXPIRES_IN=7d

# 跨域配置
CORS_ORIGIN=http://localhost:3000
```

在client目录下创建.env.local文件：

```
NEXT_PUBLIC_API_URL=http://localhost:4000/api
```

### 4. 启动开发服务器

```bash
# 启动后端服务
cd server
npm run dev

# 启动前端服务
cd ../client
npm run dev
```

### 5. 访问应用

- 前台: http://localhost:3000
- 后台: http://localhost:3000/admin
- API: http://localhost:4000/api

## 测试环境部署

### 使用Docker Compose

1. 确保安装了Docker和Docker Compose

2. 在项目根目录创建docker-compose.yml文件：

```yaml
version: '3'

services:
  mongodb:
    image: mongo:6.0
    container_name: blog-mongodb
    volumes:
      - mongodb_data:/data/db
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_DATABASE=personal-blog
    restart: unless-stopped

  server:
    build:
      context: ./server
      dockerfile: Dockerfile
    container_name: blog-server
    depends_on:
      - mongodb
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=production
      - PORT=4000
      - MONGODB_URI=mongodb://mongodb:27017/personal-blog
      - JWT_SECRET=your_jwt_secret_key
      - JWT_EXPIRES_IN=7d
      - CORS_ORIGIN=http://localhost:3000
    restart: unless-stopped

  client:
    build:
      context: ./client
      dockerfile: Dockerfile
    container_name: blog-client
    depends_on:
      - server
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:4000/api
    restart: unless-stopped

volumes:
  mongodb_data:
```

3. 在server目录创建Dockerfile：

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --production

COPY . .

RUN npm run build

EXPOSE 4000

CMD ["npm", "start"]
```

4. 在client目录创建Dockerfile：

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

5. 启动容器：

```bash
docker-compose up -d
```

## 生产环境部署

### 使用GitHub Actions和云服务

1. 在项目根目录创建.github/workflows/deploy.yml文件：

```yaml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies and build (Server)
      run: |
        cd server
        npm ci
        npm run build
        
    - name: Install dependencies and build (Client)
      run: |
        cd client
        npm ci
        npm run build
        
    - name: Deploy to server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          cd /path/to/deployment
          git pull
          cd server
          npm ci --production
          npm run build
          pm2 restart blog-server
          
          cd ../client
          npm ci --production
          npm run build
          pm2 restart blog-client
```

2. 在GitHub仓库设置中添加以下Secrets：
   - HOST: 服务器IP地址
   - USERNAME: SSH用户名
   - SSH_KEY: SSH私钥

### 使用Nginx作为反向代理

在服务器上配置Nginx：

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    
    location /api {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### SSL配置

使用Certbot安装SSL证书：

```bash
sudo apt-get update
sudo apt-get install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

## 数据库备份

设置定时备份MongoDB数据：

```bash
# 创建备份脚本
cat > backup.sh << 'EOF'
#!/bin/bash
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_DIR="/path/to/backups"
MONGO_DATABASE="personal-blog"

# 创建备份目录
mkdir -p $BACKUP_DIR

# 执行备份
mongodump --db $MONGO_DATABASE --out $BACKUP_DIR/$TIMESTAMP

# 删除7天前的备份
find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;
EOF

# 添加执行权限
chmod +x backup.sh

# 添加到crontab
(crontab -l 2>/dev/null; echo "0 2 * * * /path/to/backup.sh") | crontab -
```

## 监控

使用PM2进行应用监控：

```bash
# 安装PM2
npm install -g pm2

# 启动应用
cd /path/to/server
pm2 start npm --name "blog-server" -- start

cd /path/to/client
pm2 start npm --name "blog-client" -- start

# 设置开机自启
pm2 startup
pm2 save
```

## 故障排除

### 常见问题

1. **无法连接到数据库**
   - 检查MongoDB服务是否运行
   - 验证连接字符串是否正确
   - 确认网络设置允许连接

2. **API请求失败**
   - 检查CORS设置
   - 验证API URL配置
   - 查看服务器日志

3. **部署失败**
   - 检查GitHub Actions日志
   - 验证服务器SSH配置
   - 确认服务器上有足够的磁盘空间

### 日志查看

```bash
# 查看PM2日志
pm2 logs blog-server
pm2 logs blog-client

# 查看Nginx日志
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```