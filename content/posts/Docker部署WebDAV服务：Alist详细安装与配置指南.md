---
title:  使用Docker搭建WebDAV服务：用户权限配置与目录管理全指南 
subtitle:
date: 2024-12-13T21:41:18+08:00
slug: mY1FL6vb
tags:
  - Linux
  - docker
  - WebDAV
categories:
  - 应用部署
---

本文介绍使用Docker部署Alist,并且演示了如何挂载本地磁盘。

#### 安装

直接给出启动配置文件
`docker-compose.yml`:

```yml
services:
    alist:
        image: 'xhofe/alist:latest-ffmpeg' #这里使用的是内置 ffmpeg 版镜像
        container_name: alist
        volumes:
            - './data/etc:/opt/alist/data' #应用目录
            - './data/drive:/opt/alist/drive' #主数据目录，上传的文件都存放在这里
            - './data/drive/documents:/opt/alist/drive/documents' #存文档
            - './data/drive/music:/opt/alist/drive/music'   #存音乐
            - './data/drive/movie:/opt/alist/drive/movie'   #存电影
            - './data/cache:/opt/alist/cache'   #缩略图
            - './data/trash:/opt/alist/trash'   #回收站
        ports:
            - '5244:5244'
        environment:
            - PUID=1000
            - PGID=1000
            - UMASK=022
            - TZ=Asia/Shanghai
        restart: unless-stopped
```

上面我自定义了很多目录 ，其中主要的也就是两个：

```
- './data/etc:/opt/alist/data' #应用目录
- './data/drive:/opt/alist/drive' #主数据目录，上传的文件都存放在这里
```

其他都是可选的。

环境变量：

```
- PUID=1000
- PGID=1000
```

Linux环境下输入命令`id` 会显示当前用户的ID，填进去。

根据上面的配置，开始创建需要的目录：
在项目目录内执行命令，创建所需目录：

```
mkdir -p ./data/etc ./data/drive ./data/drive/documents ./data/drive/music ./data/drive/movie ./data/cache ./data/trash
```

然后启动服务：

```
docker compose up -d
```

启动成功后查看日志：

```
docker ps 
docker compose logs -f <容器名或ID>
```

![E7A10C5C-36B4-40A9-A7D6-0B0536AF424E.png](https://img.idev.ink/2024/12/02/E7A10C5C-36B4-40A9-A7D6-0B0536AF424E.png)
如图所示记下用户名和密码，用户名是：admin

### 配置

浏览器打开控制台：`http://localhost:5244/` 输入账号密码登录

来添加存储

![B2F37FB4-CD42-447C-902A-901E26ECA9DE.png](https://img.idev.ink/2024/12/02/B2F37FB4-CD42-447C-902A-901E26ECA9DE.png)

![3387BFA6-6167-44E2-A225-FE6B1FFDBA67.png](https://img.idev.ink/2024/12/02/3387BFA6-6167-44E2-A225-FE6B1FFDBA67.png)

驱动选择本地存储

![CAF4AC53-772D-4CB9-99EF-DB807EE19F77.png](https://img.idev.ink/2024/12/02/CAF4AC53-772D-4CB9-99EF-DB807EE19F77.png)

挂载路径：这里是目录的显示名，可以自定义。这里以/movie 为例，创建一个存放电影的目录。

![43398BE7-65BC-4398-A703-83D87A21AE51.png](https://img.idev.ink/2024/12/02/43398BE7-65BC-4398-A703-83D87A21AE51.png)

根文件夹路径：容器内部的路径，已经挂载到了本地。
缩略图：存放图片视频缩略图的目录
回收站：如果不设置，文件删除就不能恢复了。

点击添加，完成。

![8E586D25-CAB9-402A-84FF-733781F7F5F3.png](https://img.idev.ink/2024/12/02/8E586D25-CAB9-402A-84FF-733781F7F5F3.png)

![CB61B6E2-F1FC-4BB1-B539-7C78E1C1DB6E.png](https://img.idev.ink/2024/12/02/CB61B6E2-F1FC-4BB1-B539-7C78E1C1DB6E.png)

#### 使用

局域网其他设备怎么使用呢。
首先先使用防火墙放行端口：5244

然后任何支持WebDAV的播放器或文件浏览器，添加网络存储，类型选择WebDAV。

地址输入设备局域网IP，例如：192.168.10.188/dav/
端口：5244
输入账号密码就可以挂载。建议单独创建一个账号使用。

当然也可以直接使用浏览器，可以在线播放。

