---
title:  docker部署beszel：一个轻量美观的服务器状态监控服务VPS探针
subtitle:
date: 2024-12-05T21:41:18+08:00
slug: hxWkpKuG
tags:
  - docker应用
  - VPS探针
categories:
  - 应用部署
collections:
  - 实用的docker项目
---


beszel是一个轻量级服务器监控中心，具有历史数据、docker 统计信息和警报等功能。
项目地址：[beszel](https://github.com/henrygd/beszel)

部署方式：前后端全部使用docker-compose

#### 前端搭建：

docker-compose.yml

```yml
services:
  beszel:
    image: 'henrygd/beszel'
    container_name: 'beszel'
    restart: unless-stopped
    ports:
      - '8090:8090'
    volumes:
      - ./beszel_data:/beszel_data
```

启动：

```
docker compose up -d
```

浏览器打开控制台,地址：IP:8090

![BF49866C-DB16-4D33-9E4D-39E039C7AE09.png](https://img.idev.ink/2024/12/05/BF49866C-DB16-4D33-9E4D-39E039C7AE09.png)

按照要求设置账号密码后登录。

反向代理设置域名可以参考这篇文章：[https://www.idev.ink/archives/zSA7KJ8k](https://www.idev.ink/archives/zSA7KJ8k)

#### 后端搭建

右上角点击添加客户端

![DE3BA822-A26E-4C17-9F1A-0A4DD2B4DDD5.png](https://img.idev.ink/2024/12/05/DE3BA822-A26E-4C17-9F1A-0A4DD2B4DDD5.png)

![505375F0-03CD-4505-9023-D8DEC11D94F4.png](https://img.idev.ink/2024/12/05/505375F0-03CD-4505-9023-D8DEC11D94F4.png)

名字：自定义
主机/IP：填IP地址。
端口：可自定义，防火墙要放行。

然后点击`复制 docker compose` 在单独的目录内保存为`docker-compose.yml`文件。

启动:

```
docker compose up -d
```

然后回到控制台点击`添加客户端 `图标变绿了，就意味着已经设置成功。

