---
title:  使用Docker-Compose部署Minio
subtitle: 搭建私有S3对象存储服务
date: 2024-11-27T21:41:18+08:00
slug: K31cB1xq
tags:
  - docker
  - minio
  - S3
categories:
  - 系统管理
  - 应用部署
---

本文介绍使用docker-compose结合traefik部署minio， 搭建属于自己的私有S3对象存储服务。
minio可以搭配piclist picgo做图床。

配置文件：`docker-compose.yml`

```yml
services:
  minio:
    image: minio/minio:RELEASE.2024-11-07T00-52-20Z
    container_name: minio1
    user: "1001:1001" #使用普通用户启动，若注释掉则以root用户启动。改成你当前用户ID
    environment:
      TZ: Asia/Shanghai
      MINIO_ROOT_USER: admin #登录账号
      MINIO_ROOT_PASSWORD: KGH2RYI17EZRTDFZGQ91 #登录密码
      MINIO_API_ROOT_ACCESS: on #根用户API访问权开关

      MINIO_COMPRESSION_ENABLE: off  #压缩
      MINIO_COMPRESSION_ALLOW_ENCRYPTION: off #加密压缩，不建议开启！

      MINIO_BROWSER: on #启用web控制台
      MINIO_BROWSER_LOGIN_ANIMATION: off  #是否启用控制台登录动画
      MINIO_BROWSER_REDIRECT: false #Web浏览器的请求是否自动重定向到控制台地址

    volumes:
      - "./minio/data:/data" #数据持久化
    command: server /data --address ":9000" --console-address ":9001"
    restart: unless-stopped

    #----下面traefik配置项，很容易明白，如果不使用traefik，把下面全部删掉就好------#

    labels:
      - "traefik.enable=true"  # 启用 Traefik 路由
      #API
      - "traefik.http.routers.minio-api.service=minio-api"
      - "traefik.http.routers.minio-api.rule=Host(`api.example.com`)" # API域名 
      - "traefik.http.routers.minio-api.entrypoints=websecure"
      - "traefik.http.routers.minio-api.tls=true"  # 启用 TLS
      - "traefik.http.services.minio-api.loadbalancer.server.port=9000"  # API默认端口9000

      #console
      - "traefik.http.routers.minio-console.service=minio-console"
      - "traefik.http.routers.minio-console.rule=Host(`minio.example.com`)"  # 控制台域名
      - "traefik.http.routers.minio-console.entrypoints=websecure"
      - "traefik.http.routers.minio-console.tls=true"  # 启用 TLS
      - "traefik.http.services.minio-console.loadbalancer.server.port=9001"  #控制台默认端口9001

    networks:
      - traefik_network  #加入网络：traefik_network

networks:
  traefik_network:
    external: true
```

**需要修改内容：`user: "1001:1001"` ,终端输入命令 `id`，会显示你当前用户ID和组ID**

**域名部分和用户密码按需修改**

**上面使用的是普通用户启动，最好手动创建数据目录，以避免权限问题;**

```bash
mkdir -p minio/data
```

启动：

```bash
docker compose up -d
```

使用下面命令查看启动状态：

```
docker ps

docker compose logs minio
```

浏览器打开控制台：minio.example.com 登录

登录后先创建一个存储桶：只需要设个名字，其他不动。

![CAC2B440-E7B6-4569-81DE-070FD52CEED8.webp](https://img.idev.ink/2024/12/02/CAC2B440-E7B6-4569-81DE-070FD52CEED8.webp)

![983BA8EE-9F5F-4865-BCCE-CEFFE6A6A35B.webp](https://img.idev.ink/2024/12/02/983BA8EE-9F5F-4865-BCCE-CEFFE6A6A35B.webp)

![F3BF8F2F-E834-454E-9455-CCFF16AA26AA.webp](https://img.idev.ink/2024/12/02/F3BF8F2F-E834-454E-9455-CCFF16AA26AA.webp)

然后把桶设权限为公开

![3AD40E69-DD8E-4905-AF6B-D3320F92E800.webp](https://img.idev.ink/2024/12/02/3AD40E69-DD8E-4905-AF6B-D3320F92E800.webp)

上传一张图片：

![3A734858-0C15-4A11-8F62-316782E82622.webp](https://img.idev.ink/2024/12/02/3A734858-0C15-4A11-8F62-316782E82622.webp)

复制图片路径：

![00092644-FE97-413F-BDA9-D2B56B91EB30.webp](https://img.idev.ink/2024/12/02/00092644-FE97-413F-BDA9-D2B56B91EB30.webp)

浏览器输入访问地址：API域名+文件路径

`https://api.example.com/test/sky.jpg`

就可以访问到图片了。

##### 一些注意点：

API域名尽量不要套CDN，套了CDN需要做一些设置，否则的话API请求会有异常，甚至不能使用API上传文件。

