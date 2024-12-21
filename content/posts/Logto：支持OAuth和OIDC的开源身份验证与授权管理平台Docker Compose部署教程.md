---
title:  Docker-Compose部署Logto，一个开源的单点登录服务
subtitle:
date: 2024-12-11T21:41:18+08:00
slug: zpX6f6Uo
tags:
  - Linux
  - 单点登录
  - docker应用
categories:
  - 应用部署
---

介绍:
Logto 是一个开源的身份验证和授权管理平台，旨在简化用户认证、用户管理和权限控制的流程。它提供了与多种身份认证方式的集成，包括 OAuth 2.0、OpenID Connect、SAML、LDAP 等，能够快速搭建用户登录、单点登录（SSO）以及多因素认证（MFA）的系统。

本文使用docker-compose部署logto，并且使用nginx-proxy-manager做反向代理。

#### 使用docker-compose部署logto

创建项目目录：

```bash
mkdir logto
cd logto
touch docker-compose.yml
```

编辑：`docker-compose.yml`

```yml
services:
  logto:
    image: svhd/logto
    container_name: logto
    #ports:
    #  - 3001:3001 # API端口
    #  - 3002:3002 # 控制台端口
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      TRUST_PROXY_HEADER: "1"
      CASE_SENSITIVE_USERNAME: "false" # 禁用用户名的大小写敏感
      DB_URL: postgresql://logto:password@logto_db:5432/logto_db
      ENDPOINT: https://logto-api.azfum.com
      ADMIN_ENDPOINT: https://logto-admin.azfum.com
    entrypoint: ["sh", "-c", "npm run cli db seed -- --swe && npm start"]
    networks:
      - nginx_network

  postgres:
    image: postgres:17-alpine
    container_name: logto_db
    environment:
      POSTGRES_USER: logto #数据库用户名
      POSTGRES_PASSWORD: password #数据库密码
      POSTGRES_DB: logto_db #数据库名
    volumes:
      - ./data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - nginx_network

networks:
  nginx_network:
    external: true
```

上面的配置使用了一个外部网络：nginx_network，使用nginx-proxy-manager做代理不需要对外暴露端口，所以把端口注释掉了。
关于nginx-proxy-manager的使用可以看这篇文章：https://www.azfum.com/archives/zSA7KJ8k

如果你不用nginx-proxy-manager，可以把nginx_network相关配置删掉，比如下面这样：

```yml
services:
  logto:
    image: svhd/logto
    container_name: logto
    ports:
      - 3001:3001 # API端口
      - 3002:3002 # 控制台端口
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      TRUST_PROXY_HEADER: "1"
      CASE_SENSITIVE_USERNAME: "false" # 禁用用户名的大小写敏感
      DB_URL: postgresql://logto:password@logto_db:5432/logto_db
      ENDPOINT: https://logto-api.azfum.com
      ADMIN_ENDPOINT: https://logto-admin.azfum.com
    entrypoint: ["sh", "-c", "npm run cli db seed -- --swe && npm start"]

  postgres:
    image: postgres:17-alpine
    container_name: logto_db
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: logto #数据库用户名
      POSTGRES_PASSWORD: password #数据库密码
      POSTGRES_DB: logto_db #数据库名
    volumes:
      - ./data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
```

启动服务：

```
docker compose up -d
```

查看运行日志

```
docker ps
docker compose logs logto
```

#### 使用nginx-proxy-manager反向代理配置域名

![1971A2A4-8F0D-4A77-8874-8CE0F105CAED.png](https://img.idev.ink/2024/12/11/1971A2A4-8F0D-4A77-8874-8CE0F105CAED.png)

1.控制台代理：
![A659C36A-D5D6-4BD8-ABEC-CF3136659929.png](https://img.idev.ink/2024/12/11/A659C36A-D5D6-4BD8-ABEC-CF3136659929.png)
![22428FB0-B6E7-4FF1-A055-463DD5902F3D.png](https://img.idev.ink/2024/12/11/22428FB0-B6E7-4FF1-A055-463DD5902F3D.png)
Forward Hostname / IP* ：填写上面指定的容器名称：`container_name: logto`

2.API代理

![F93E092F-070D-4441-9D50-289DBBA5C535.png](https://img.idev.ink/2024/12/11/F93E092F-070D-4441-9D50-289DBBA5C535.png)

![E3AA3A85-8A22-4945-BDF7-511562A26403.png](https://img.idev.ink/2024/12/11/E3AA3A85-8A22-4945-BDF7-511562A26403.png)

这样就完成了，如下图所示：
![C198DB58-CFCC-48C2-9A44-D0A2D051667E.png](https://img.idev.ink/2024/12/11/C198DB58-CFCC-48C2-9A44-D0A2D051667E.png)

进入控制台：https://logto-admin.azfum.com

![CCB53D34-168A-4953-AB9F-101861DE3C5A.png](https://img.idev.ink/2024/12/11/CCB53D34-168A-4953-AB9F-101861DE3C5A.png)
创建账号密码后登录：

![0D47F5D8-0871-453D-9DCA-65F44C3D2D24.png](https://img.idev.ink/2024/12/11/0D47F5D8-0871-453D-9DCA-65F44C3D2D24.png)

到此部署完成。

Logto的详细使用说明文档：https://docs.logto.io/zh-CN/logto-oss/deployment-and-configuration

