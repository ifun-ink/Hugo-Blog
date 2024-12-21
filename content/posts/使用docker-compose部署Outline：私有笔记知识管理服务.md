---
title:  使用docker-compose部署Outline 
subtitle: 私有笔记软件知识管理服务
date: 2024-11-20T21:41:18+08:00
slug: F8xuaB5U
tags:
  - Linux
  - docker
  - outkine
categories:
  - 应用部署
---

### 部署 Outline 项目的教程

介绍：个人或团队知识库，美观、实时协作、功能丰富且兼容 markdown。媲美obsidian和notion的知识管理，笔记应用。可以自己用，也可以团队使用。可以给你家人，同学，同事一起使用。和其他用户可独立使用，也可以共享使用。多语言，完全支持中文。

项目地址：https://github.com/outline/outline

效果图：
![70A5F8B3-1561-4CE3-BA8F-7EF23702D30C.png](https://imgr.idev.ink/2024/12/11/70A5F8B3-1561-4CE3-BA8F-7EF23702D30C.png)
![68113248-B430-497B-9D60-048C90ED22A3.png](https://imgr.idev.ink/2024/12/11/68113248-B430-497B-9D60-048C90ED22A3.png)

### 开始部署：

#### 先决条件

1. 确保已安装 Docker 和 Docker Compose。
2. 创建项目目录，包含 `docker-compose.yml` 和 `docker.env` 文件。

#### 步骤

1. **创建目录结构**：
   
   ```bash
   mkdir -p ~/outline
   cd ~/outline
   ```
2. **保存配置文件**：
   
   - 将提供的 `docker-compose.yml` 内容保存为 `docker-compose.yml`。
   - 将提供的环境变量内容保存为 `docker.env`。
3. **启动服务**：
   在项目目录中运行以下命令启动所有服务：
   
   ```bash
   docker compose up -d
   ```
4. **检查服务状态**：
   使用以下命令确认所有服务都在运行：
   
   ```bash
   docker compose ps
   ```
5. **访问应用**：
   在浏览器中访问 `http://localhost:3000` 或配置的公共 URL。

反向代理设置域名可以参考这篇文章：[https://www.idev.ink/archives/zSA7KJ8k](https://www.idev.ink/archives/zSA7KJ8k)
6. **日志监控**（可选）：
如需查看服务日志，可以使用：

```bash
docker compose logs -f outline
```

后续问题：

如果在写入文件时出现权限错误：不能上传图片，不能修改用户头像。使用下面命令解决：

```
sudo chown 1001 ./data/outline
```

#### 配置文件:

`docker-compose.yml` :

```
#docker-compose.yml
services:
  outline:
    image: docker.getoutline.com/outlinewiki/outline:latest
    env_file: ./docker.env
    volumes:
      - ./data/outline:/var/lib/outline/data  # 将本地 data/outline 目录映射到容器内的 /var/lib/outline/data
    depends_on:
      - redis
      - postgres
    ports:
      - "3000:3000"


  redis:
    image: redis
    env_file: ./docker.env
    volumes:
      - ./data/redis/redis.conf:/redis.conf  # 将本地 data/redis 目录映射到容器内的 /redis.conf
    command: ["redis-server", "/redis.conf"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 30s
      retries: 3
    ports:
      - "6379:6379"

  postgres:
    image: postgres
    env_file: ./docker.env
    ports:
      - "5432:5432"
    volumes:
      - ./data/postgres:/var/lib/postgresql/data  # 将本地 data/postgres 目录映射到容器内的 /var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-d", "outline", "-U", "root"]
      interval: 30s
      timeout: 20s
      retries: 3
    environment:
      POSTGRES_USER: 'root'
      POSTGRES_PASSWORD: 'Asdr3HfCC'
      POSTGRES_DB: 'outline'
```

`docker.env` ：

```
#docker.env
# 使用命令：'openssl rand -hex 32'生成的随机密钥，用于加密和安全目的
SECRET_KEY=91a2d80fde9504c5ee6b6141d546f122

# 使用命令：'openssl rand -hex 32'生成的唯一随机密钥，通常用于其他安全目的
UTILS_SECRET=02670ace00ea669cf12b7131a8936555

# 连接到本地数据库（确保使用正确的用户名和密码）
DATABASE_URL=postgres://root:Asdr3HfCC@postgres:5432/outline

PGSSLMODE=disable

# Redis 连接 URL，包含主机和端口
REDIS_URL=redis://redis:6379

# 应用程序的公开 URL，应该是一个完全合格的域名
URL=https://docs.XXX.com

# 应用程序监听的端口
PORT=3000

# 指定使用什么存储系统。可能的值是 "s3" 或 "local"。
# 对于 "local"，头像图片和文档附件将保存在本地磁盘上。
FILE_STORAGE=local
FILE_STORAGE_LOCAL_ROOT_DIR=/var/lib/outline/data

# 允许上传的附件最大大小（以字节为单位）
FILE_STORAGE_UPLOAD_MAX_SIZE=262144000

# 最大文档导入大小，通常应该小于文件上传最大值
FILE_STORAGE_IMPORT_MAX_SIZE=10485760

# 可选项
# 日志级别，选择 error, warn, info, http, verbose, debug, silly
LOG_LEVEL=info

# 是否启用速率限制，默认是 true
RATE_LIMITER_ENABLED=true

# 速率限制的默认参数
RATE_LIMITER_REQUESTS=1000  # 每个窗口允许的请求数
RATE_LIMITER_DURATION_WINDOW=60  # 窗口时间（秒）

# –––––––––––––– 身份验证 ––––––––––––––

# 第三方登录凭据，至少需要 Google、Slack 或 Microsoft 中的一个，否则将没有登录选项。 
#推荐使用stack，SMTP，OIDC，这几个比较简单

# 要配置 Slack 身份验证，您需要在以下地址创建一个应用程序：
# => https://api.slack.com/apps
#
# 配置客户端 ID 时，在“OAuth 和权限”下添加重定向 URL：
# https://<URL>/auth/slack.callback
SLACK_CLIENT_ID=get_a_key_from_slack
SLACK_CLIENT_SECRET=get_the_secret_of_above_key

#SMTP
# 示例配置：
# SMTP_HOST=smtp.mailgun.org
# SMTP_PORT=587
# SMTP_SECURE=false
# SMTP_USERNAME=docs@mg.XX.com
# SMTP_PASSWORD=08fbddc1158ce8f556d869991-d010bdaf-92e1af2d
# SMTP_FROM_EMAIL=docs@mg.XX.com

#OIDC （推荐使用logto,zitadel,这两个都可以免费用，也可以自己托管部署到自己的服务器上）
# 示例配置：
# OIDC_CLIENT_ID=235681356992155525
# OIDC_CLIENT_SECRET=FJVCg638Krzo4pEbOdsBsgEfdgsC7wYkl4UubUGcXPua0j8bA
# OIDC_AUTH_URI=https://oauth.xx.com/oauth/v2/authorize
# OIDC_TOKEN_URI=https://oauth.xx.com/oauth/v2/token
# OIDC_USERINFO_URI=https://oauth.xx.com/oidc/v1/userinfo

#Google 身份验证 - 这个需要开通谷歌云，不推荐
# 要配置 Google 身份验证，您需要在以下地址创建 OAuth 客户端 ID：
# => https://console.cloud.google.com/apis/credentials
#
# 配置客户端 ID 时，添加授权重定向 URI：
# https://<URL>/auth/google.callback
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=

#Microsoft/Azure 身份验证 -- 这个需要开通微软云或者office365才能用，不推荐
# 要配置 Microsoft/Azure 身份验证，您需要创建一个 OAuth 客户端。有关设置 Azure 应用的详细信息，请参见
# => https://wiki.generaloutline.com/share/dfa77e56-d4d2-4b51-8ff8-84ea6608faa4
AZURE_CLIENT_ID=
AZURE_CLIENT_SECRET=
AZURE_RESOURCE_APP_ID=
```

