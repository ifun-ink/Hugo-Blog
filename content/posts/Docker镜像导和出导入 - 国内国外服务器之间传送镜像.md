---
title:  Docker镜像导和出导入
subtitle: 国内国外服务器之间传送镜像 
date: 2024-12-25T21:41:18+08:00
slug: qhrYeiAw
tags:
  - Linux
  - docker镜像导出导入
categories:
  - 系统管理
---

本文介绍如何在 VPS A 上拉取 Docker 镜像，并将其传输到 VPS B 的详细步骤。

---

### 第一步：准备工作

在开始操作之前，确保以下条件已满足：

1. **拥有两台 VPS：** VPS A 和 VPS B，均可通过 SSH 访问。
2. **安装 Docker 和 Docker Compose：** 在 VPS A 和 VPS B 上安装 Docker 和 Docker Compose，用于后续容器操作。

---

### 第二步：拉取 Docker 镜像

在 VPS A 上拉取所需的 Docker 镜像：

1. **拉取镜像：** 使用 `docker pull` 从 Docker Hub 或其他镜像仓库拉取。例如：

   ```bash
   docker pull jc21/nginx-proxy-manager:latest
   ```

   此处 `jc21/nginx-proxy-manager:latest` 是镜像的名称和标签。
2. **确认镜像拉取成功：** 使用以下命令查看拉取的镜像：

   ```bash
   docker images
   ```

---

### 第三步：保存和传输镜像文件

#### 3.1 保存镜像为文件

在 VPS A 上，使用 `docker save` 命令将镜像保存为 `.tar` 文件：

```bash
docker save -o image.tar jc21/nginx-proxy-manager:latest
```

这里的 `image.tar` 是保存的文件名，可自行定义。

#### 3.2 传输镜像文件

**方法一：使用密钥传输**

```bash
scp -i /path/to/private_key /local/path/image.tar user@VPS_B_IP:/remote/path/
```

* `/path/to/private_key`：SSH 私钥路径。
* `/local/path/image.tar`：VPS A 上镜像文件路径。
* `user@VPS_B_IP`：VPS B 的用户名和 IP 地址。
* `/remote/path/`：VPS B 上存放镜像文件的路径。

**方法二：使用密码传输**

```bash
scp /local/path/image.tar user@VPS_B_IP:/remote/path/
```

执行命令后，系统会提示输入密码。

---

### 第四步：导入和使用镜像

在 VPS B 上操作：

1. **加载镜像：** 使用 `docker load` 导入镜像文件：

   ```bash
   docker load -i image.tar
   ```
2. **确认加载成功：** 使用以下命令验证镜像是否导入：

   ```bash
   docker images
   ```
3. **部署应用程序：** 编辑或创建 `docker-compose.yml` 文件，指定镜像和服务配置：

   ```yaml
   version: '3'
   services:
     myservice:
       image: jc21/nginx-proxy-manager:latest
       # 其他配置项
   ```

   启动服务：

   ```bash
   docker-compose up -d
   ```

---

以上步骤完成后，镜像传输及部署即告完成。
