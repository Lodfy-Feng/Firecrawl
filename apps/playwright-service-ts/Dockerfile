FROM node:18-slim

## 设置工作目录
WORKDIR /usr/src/app

# 复制依赖配置文件
COPY package*.json ./

# 安装依赖（走代理）
RUN npm install

# 复制项目文件
COPY . .

# 设置 Playwright 安装路径
ENV PLAYWRIGHT_BROWSERS_PATH=/tmp/.cache

# 安装 Playwright 浏览器及依赖
RUN DEBUG=pw:browser-download npx playwright install --with-deps
# 构建项目
RUN npm run build

# 设置端口参数
ARG PORT
ENV PORT=${PORT}

# 暴露端口
EXPOSE ${PORT}

# 启动服务
CMD [ "npm", "start" ]
