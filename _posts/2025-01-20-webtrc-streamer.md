---
title: docker 离线部署 webrtc-streamer
date: 2025-01-20 +0800
categories: [工作日志, 指南]
tags: [webrtc, docker]
author: <yurongku>  
mermaid: true
---


### 加载镜像
```bash
docker load -i webrtc0116.tar
```

### 启动容器
#### 参数说明
- -itd

- -i：表示容器启动时保持标准输入流（stdin）打开，使你可以与容器交互。
- -t：分配一个伪终端，允许你通过命令行与容器交互。
- -d：表示容器在后台运行（detached mode），即不阻塞当前终端，容器将在后台运行。
- --restart=always
  - 这个选项指定容器应该在某些情况下自动重启：
  - 容器如果崩溃或停止，会自动重新启动。
  - Docker 重启时，容器也会自动启动。
  - 适合需要持续运行的应用（如 Web 服务）。
- --net=host
  - [如果不配置这个内网可能会有网络问题]
  - 这个选项指定容器使用宿主机的网络模式。
  - host 模式让容器共享宿主机的网络堆栈，而不是为容器分配独立的虚拟网络。换句话说，容器将直接绑定到宿主机的网络接口。
  - 在某些情况下，使用 host 网络模式可以提高性能，尤其是需要直接访问宿主机网络资源的应用，如实时视频流应用等。
- --name webrtc-streamer
  - 给容器命名为 webrtc-streamer，便于以后引用和管理容器。
  - 比如你以后可以通过 docker restart webrtc-streamer 来重启该容器。
  - mpromonet/webrtc-streamer
  - 这是 Docker 镜像的名称，表示你要运行的容器基于 mpromonet/webrtc-streamer 镜像。
  - 该镜像通常是用于 WebRTC 流媒体传输的服务应用。

```bash
docker run -itd --restart=always -net=host --name webrtc-streamer mpromonet/webrtc-streamer

```

### Docker 容器中复制文件到宿主机


```bash
docker cp webrtc-streamer:/app/config.json ./config.json

```


### 宿主机上的文件复制到 Docker 容器内部


```bash

docker cp ./config.json webrtc-streamer:/app/config.json

```


### 以后台模式运行一个新的 Docker 容器，并挂载宿主机上的文件。
- -d：以后台模式运行容器。
- --name webrtc-streamer：给容器命名为 webrtc-streamer。
- -v /path/to/config.json:/app/config.json：将宿主机上的配置文件 /path/to/config.json 挂载到容器中的 /app/config.json 路径。



```bash
docker run -d --name webrtc-streamer \ -v /path/to/config.json:/app/config.json \ mpromonet/webrtc-streamer

```


### 重启


```bash
docker restart webrtc-streamer

```



### 进容器内部并打开一个交互式的 shell（通常是 Bash）。
- -it：让你以交互模式进入容器，并分配一个终端。
- webrtc-streamer：指定要进入的容器名称。
- /bin/bash：在容器内运行 Bash shell。



```bash

docker exec -it webrtc-streamer /bin/bash
```

### 配置config.json

> subtype=0 辅码流 1 主码流

```json
{
    "urls":{              
        "waiting": {"video": "rtsp://admin:hasf@2024@10.24.213.141/cam/realmonitor?channel=2&subtype=0" },
		"waiting": {"video": "rtsp://admin:hasf@2024@10.24.213.141/cam/realmonitor?channel=2&subtype=0" },
		
    }
}
```