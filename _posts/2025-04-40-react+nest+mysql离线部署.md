---
title: react+nest+mysql使用docker-compose离线部署
date: 2025-04-30 +0800
categories: [工作日志, 指南]
tags: [docker]
author: <yurongku>  
mermaid: true
---

1. 准备镜像包：
   > node-18.18.0.tar\mysql.tar(8.0+)\nginx.tar

2. 加载镜像 
   > docker load -i nginx.tar

3. 目录结构：
   ```
    \admin\
    ├── docker-compose.yml
    ├── frontend\
    │   ├── dist\
    ├── backend\
    │   ├── node_modules\
    │   ├── package.json
    │   ├── dist\
    │   ├── Dockerfile
    ├── nginx
    │   ├── conf.d
    │       └── default.conf
    ├── mysql/
    │   └── Dockerfile
    │   └── initdb.d/          
    │       └── bvision_ai_master.sql
   ```

4. Dockerfile
  
  ```Dockerfile
  # backend
  FROM node:18.18.0

  WORKDIR /app

  COPY package.json ./
  COPY node_modules/ ./node_modules/ # 因为这边离线部署，所以直接复制node_modules,如果在线可以选择 npm run build --production
  COPY dist/ ./dist/

  CMD ["node", "dist/main.js"]

  # mysql
  FROM mysql:8.0
  # 复制自定义配置文件（可选）
  # COPY my.cnf /etc/mysql/conf.d/
  # 设置时区和字符集（或通过 my.cnf 配置）
  ENV TZ=Asia/Shanghai
  # RUN chmod 644 /etc/mysql/conf.d/my.cnf

  
  ```

5. nginx\default.conf
   
   ```
   server {
    listen 80;
    server_name localhost;

    # 前端静态文件
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # 后端API代理
    location /api {
        rewrite ^/api/(.*)$ /$1 break; 
        proxy_pass http://bvision_nest:3000;# 这边是写容器名
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
  } 
   ```

6. docker-compose.yml

```yml

services:
  mysql:
    build:
      context: ./mysql     
    container_name: bvision_mysql
    ports:
      - "3306:3306"
    volumes:
      - ./mysql/initdb.d:/docker-entrypoint-initdb.d
    environment:
      TZ: Asia/Shanghai  # 设置时区
      MYSQL_ROOT_PASSWORD: 123
      MYSQL_DATABASE: bvision_ai_master
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 10s
      retries: 5

  backend:
    # image: node:18.18.0
    container_name: bvision_nest
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_USERNAME=root
      - DB_PASSWORD=123
      - DB_DATABASE=bvision_ai_master
      - TZ=Asia/Shanghai
    volumes:
      - ./backend/dist:/app/dist             # 热更新 Nest 编译后的代码
      - ./backend/node_modules:/app/node_modules # 本地 node_modules
      - ./backend/package.json:/app/package.json # package.json 也映射方便查看版本
    depends_on:
      - mysql
    networks:
      - app-network

  nginx:
    image: nginx:latest
    container_name: bvision_nginx
    volumes:
      - ./nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf
      - ./frontend/dist:/usr/share/nginx/html
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

```

7. 启动
   > 先保证有三个镜像
   > docker-compose up -d --pull never
   > 会生成两个镜像，还有一个桥接网络下面有三个容器

其他指令：
- 单独跑某个Dockerfile
  
  ```sh
  docker build -t bvision_nest .
  docker run -it --rm bvision_nest
  docker run -d --name bvision_nest -p 3000:3000 --network jhdyh-admin_app-network bvision_nest # 这个是加入镜像的
  ```

- 查看网络里面的
  
  ```sh
  docker network ls # 查看所有网络
  docker network inspect jhdyh-admin_app-network
  ```

- 日志  
  
  ```sh
  docker logs bvision_nest
  ```

- 进入容器
  
  ```sh
  docker logs bvision_nest
  ```
