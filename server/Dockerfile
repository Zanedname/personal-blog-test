FROM node:18-alpine

WORKDIR /app

# 复制package.json和package-lock.json
COPY package*.json ./

# 安装依赖
RUN npm ci --production

# 复制源代码
COPY . .

# 构建TypeScript
RUN npm run build

# 创建上传和日志目录
RUN mkdir -p uploads logs

# 创建非root用户
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# 设置目录权限
RUN chown -R appuser:appgroup /app/uploads /app/logs

# 切换到非root用户
USER appuser

EXPOSE 4000

CMD ["node", "dist/index.js"]