---
title: github pages 配置
date: 2025-01-21 +0800
categories: [工作日志, 指南]
tags: [github]
author: <yurongku>  
mermaid: true
---

## github pages

[教程](https://pages.github.com/)

网络问题各种报错
config这样配
```
#Default gitHub user Self
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa.github
	Port 443

# gitee
Host gitee.com
    Port 22
    HostName gitee.com
    IdentityFile ~/.ssh/id_rsa.gitee
```

必须要开启代理服务器

> git config --global http.proxy http://127.0.0.1:7897  # 假设代理运行在本地 7897 端口
> 
> git config --global https.proxy http://127.0.0.1:7897  # 假设代理运行在本地 7897 端口