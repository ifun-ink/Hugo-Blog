---
title: 使用Rclone同步文件到Cloudflare R2 
subtitle:
date: 2024-11-28T21:41:18+08:00
slug: E8ul24sN
tags:
  - rclone
  - Cloudflare R2
categories:
  - 系统管理
---

Rclone 是一个功能强大的命令行工具，用于管理和同步文件到各种云存储服务。本文将介绍如何安装、配置和使用 Rclone，将文件同步和备份到 Cloudflare R2 存储服务。

---

## 一、安装 Rclone

1. **安装 Rclone：**
   以下以 Debian 系统为例安装 Rclone：
   
   ```sh
   sudo apt install rclone
   ```
2. **验证安装：**
   执行以下命令检查安装是否成功：
   
   ```sh
   rclone version
   ```

---

## 二、配置 Rclone 连接 Cloudflare R2

在开始配置前，需要先登录到 Cloudflare，创建 API 令牌并获取必要的访问密钥和端点信息。

### 1. 创建 API 令牌和存储桶信息

1. **登录 Cloudflare 管理界面：**
   
   - 进入 **R2 存储** 页面。
   - 创建一个 API 令牌，确保拥有“读写”权限，并指定相关的存储桶。
2. **保存关键信息：**
   
   - **Access Key ID** 和 **Secret Access Key**（创建完成后仅显示一次）。
   - **存储桶端点地址**：从 S3 API 设置中获取（去掉路径部分）。
     - 示例：`https://3f33d3f3g3cx33fds3f31f.r2.cloudflarestorage.com/`

### 2. 配置 Rclone

这里直接使用编辑配置文件的方式来配置，比较简单直接。

1. **编辑 Rclone 配置文件：**
   配置文件路径：
   
   - Linux/macOS: `~/.config/rclone/rclone.conf`
   - Windows: `C:\Users\<YourUsername>\.config\rclone\rclone.conf`
2. **添加 Cloudflare R2 配置：**
   在配置文件中添加以下内容，替换占位值为实际信息：
   
   ```ini
   [r2]
   type = s3
   provider = Cloudflare
   access_key_id = <your_access_key_id>
   secret_access_key = <your_secret_access_key>
   endpoint = <your_endpoint>
   ```
   
   示例：
   
   ```ini
   [r2]
   type = s3
   provider = Cloudflare
   access_key_id = a61111d64a7cec3594rrrr46931995a39
   secret_access_key = 967180217f1ea1e777274b079c5821136a29112f2786f8b32547356b00de563
   endpoint = https://3f33d3f3g3cx33fds3f31f.r2.cloudflarestorage.com/
   ```

---

## 三、使用 Rclone 同步和备份文件

### 1. 文件同步

#### (1) **同步本地文件到 R2 存储桶：**

```sh
rclone sync /path/to/local/folder r2:<bucket_name>/path/in/r2 -P
```

`-P`: 显示进度

示例：

```sh
rclone sync /home/user/documents r2:mybucket/documents -P
```

#### (2) **从 R2 存储桶同步文件到本地：**

```sh
rclone sync r2:<bucket_name>/path/in/r2 /path/to/local/folder
```

示例：

```sh
rclone sync r2:mybucket/documents /home/user/documents -P
```

### 2. 文件备份

在文件同步中，如果目标文件被修改或删除，会自动同步到另一端。为避免数据丢失，可以使用 Rclone 的 **备份** 功能。

#### (1) **备份本地文件到 R2：**

使用 `copy` 命令，仅将新增或修改的文件上传到存储桶：

```sh
rclone copy /path/to/local/folder r2:<bucket_name>/path/in/r2
```

示例：

```sh
rclone copy /home/user/documents r2:mybucket/documents-backup -P
```

#### (2) **从 R2 备份文件到本地：**

类似地，仅下载新增或修改的文件：

```sh
rclone copy r2:<bucket_name>/path/in/r2 /path/to/local/folder
```

示例：

```sh
rclone copy r2:mybucket/documents-backup /home/user/documents -P
```

### 3. 其他常用操作

- **列出 R2 存储桶中的文件：**
  
  ```sh
  rclone ls r2:<bucket_name>/path/in/r2
  ```
  
  示例：
  
  ```sh
  rclone ls r2:mybucket/documents
  ```
- **检查同步情况（不执行实际操作）：**
  
  ```sh
  rclone sync --dry-run /path/to/local/folder r2:<bucket_name>/path/in/r2
  ```

---

---

通过以上步骤，您可以高效地使用 Rclone 在 Cloudflare R2 上完成文件的同步与备份操作，同时保障数据的安全性和可用性。

## 四、完整操作流程示例

1. **创建 R2 存储桶：**
   在 Cloudflare R2 创建 `mybucket` 存储桶。
2. **获取并保存信息：**
   
   - `Access Key ID`: `a61111d64a7cec359467c66931995a39`
   - `Secret Access Key`: `967180217f1ea1e21c5f49b079c5821136a29112f2786f8b32547356b00de563`
   - `Endpoint`: `https://3f33d3f3g3cx33fds3f31f.r2.cloudflarestorage.com/`
3. **配置 Rclone：**
   编辑 `rclone.conf` 文件，添加对应的存储配置。
4. **同步与备份：**
   
   - 同步文件：
     ```sh
     rclone sync /home/user/documents r2:mybucket/documents -P
     ```
   - 备份文件：
     ```sh
     rclone copy /home/user/documents r2:mybucket/documents-backup -P
     ```
5. **验证：**
   列出文件以确认操作：
   
   ```sh
   rclone ls r2:mybucket/documents
   ```

