---
title: nginx 配置访问 dist
date: 2024-07-16 +0800
categories: [工作日志, 指南]
tags: [nginx]
author: <yurongku>  
mermaid: true
---

default.conf
```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       8080;
        server_name  localhost;

        # 静态文件的根目录
        root C:/code/zhenjiang/dist;
        index index.html;

        # 静态文件处理
        location / {
            try_files $uri $uri/ /index.html;
        }

        # 代理 API 请求
        location /zlm/ {
            proxy_pass http://121.225.97.170:8081/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_buffering off;
            proxy_request_buffering off;
        }
    }
server {
        listen       8081;
        server_name  localhost;

        # 静态文件的根目录
        root C:/code/zhangjiagang/dist;
        index index.html;

        # 静态文件处理
        location / {
            try_files $uri $uri/ /index.html =404;
        }

        # 代理 API 请求（如果需要）
        location /zlm/ {
            proxy_pass http://localhost/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            proxy_buffering off;
            proxy_request_buffering off;
        }
    }
}

```