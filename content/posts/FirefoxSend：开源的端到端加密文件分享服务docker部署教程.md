---
title:  docker部署FirefoxSend：开源的端到端加密文件分享服务
subtitle:
date: 2024-12-10T21:41:18+08:00
slug: Cqb3HqSg
tags:
  - Linux
  - 时区
categories:
  - 应用部署
---

类似于百度网盘阿里云网盘的文件分享服务。简单，私密，支持设置下载次数，访问密码和有效期。
最大文件支持2.5G，文件可以存在本地也可以存在S3对象存储上。
文件是端到端加密的，即使从服务器上也不能解密文件，非常安全。
项目地址：https://github.com/timvisee/send

#### 配置：

新建项目目录和数据目录：

```bash
mkdir -p send/uploads
cd send 
touch docker-compose.yml
```

目录结构图下：

```
send
├── docker-compose.yml
├── send-redis
└── uploads
```

`docker-compose.yml`：

```yml
services:
  send:
    image: 'registry.gitlab.com/timvisee/send:latest'
    restart: always
    ports:
      - '5679:1234' #端口映射
    volumes:
      - ./uploads:/uploads
    environment:
      - VIRTUAL_HOST=localhost:5679   #域名格式：send.example.com
      - VIRTUAL_PORT=1234

      # 应用程序配置
      - NODE_ENV=development # 开发环境或生产环境 "production/development" 
      - BASE_URL=http://localhost:5679  #外部访问地址，域名格式：https://send.example.com
      - PORT=1234
      - REDIS_HOST=redis # Redis 服务的主机名，默认为 redis

      # 本地文件存储相关配置
      - FILE_DIR=/uploads # 文件存储目录，默认为 /uploads

      # S3 对象存储相关配置（启用时需禁用 volume 和 FILE_DIR 变量）
      # - AWS_ACCESS_KEY_ID=******** # S3 的访问密钥 ID
      # - AWS_SECRET_ACCESS_KEY=******** # S3 的访问密钥
      # - S3_BUCKET=send # S3 的存储桶名称
      # - S3_ENDPOINT=s3.us-west-2.amazonaws.com # S3 的端点（如果不是 AWS，需要指定）
      # - S3_USE_PATH_STYLE_ENDPOINT=true # 是否强制使用路径风格的 URL

      # 上传和下载限制相关配置
      # - EXPIRE_TIMES_SECONDS=3600,86400,604800,2592000,31536000 # UI 中的过期时间选项（单位为秒）
      # - DEFAULT_EXPIRE_SECONDS=3600 # 默认的过期时间，单位为秒（默认为 86400）
      # - MAX_EXPIRE_SECONDS=31536000 # 最大过期时间，单位为秒（默认为 604800）
      # - DOWNLOAD_COUNTS=1,2,5,10,15,25,50,100,1000 # UI 中的下载次数选项
      # - MAX_DOWNLOADS=1000 # 最大下载次数（默认为 100）
      # - MAX_FILE_SIZE=2684354560 # 最大上传文件大小（默认为 2GB）

  redis:
    image: 'redis:alpine'
    restart: always
    volumes:
      - ./send-redis:/data
```

上面以本地部署为例，生产环境需要使用域名。反向代理设置域名可以参考这篇文章：[https://www.idev.ink/archives/zSA7KJ8k](https://www.idev.ink/archives/zSA7KJ8k)

启动服务

```
docker compose up -d
```

#### 使用

本地访问地址：`localhost:5679`

选择任意文件上传，可以设置有效期和密码，如下图所示：

![8E64E08A-A323-4BDB-A9D7-C559CA52198E.webp](https://img.idev.ink/2024/12/10/8E64E08A-A323-4BDB-A9D7-C559CA52198E.webp)

![1410DA2D-B5CC-45A5-90CC-4AB084D90D35.webp](https://img.idev.ink/2024/12/10/1410DA2D-B5CC-45A5-90CC-4AB084D90D35.webp)

![74A8EDF7-4312-435F-9BAA-D67C52A667EF.webp](https://img.idev.ink/2024/12/10/74A8EDF7-4312-435F-9BAA-D67C52A667EF.webp)

把生成的链接发给文件接收者即可。

