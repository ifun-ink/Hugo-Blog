---
title:  阿里云对象存储OSS使用rclone上传、下载与同步教程 
subtitle:
date: 2024-12-20T21:41:18+08:00
slug: 3MK8Sk0m
tags:
  - 阿里云OSS
  - rclone
categories:
  - 系统管理
---

下面是一个关于如何配置和使用 `rclone` 操作阿里云对象存储（OSS）的详细教程，包括上传、下载、列出文件等常见操作。此教程会引导你如何配置 `rclone` 与阿里云 OSS 配合使用。

---

### 安装 `rclone`

查看 [rclone 官网](https://rclone.org/downloads/) 按步骤进行安装。

### 一 , 获取阿里云OSS API

管理页面：https://ram.console.aliyun.com/users
点击创建用户

![94B7DF48-CF02-49DF-8827-36613161C352.png](https://img.idev.ink/2024/12/16/94B7DF48-CF02-49DF-8827-36613161C352.png)

名称自定义，其他选择如图所示：
![085053FA-83DD-43E9-B0F0-69ECA1B5F7FC.png](https://img.idev.ink/2024/12/16/085053FA-83DD-43E9-B0F0-69ECA1B5F7FC.png)

把密钥ID和值复制下来备用：
![FB4846FC-622E-49B5-B1F1-46911ABD8834.png](https://img.idev.ink/2024/12/16/FB4846FC-622E-49B5-B1F1-46911ABD8834.png)

依次点击：权限，新增授权。授权主体为刚创建的用户，授权策略选择`AliyunOSSFullAccess`
![91656D81-7637-4A7A-97A6-0EB2A1519A2C.png](https://img.idev.ink/2024/12/16/91656D81-7637-4A7A-97A6-0EB2A1519A2C.png)

---

### 二、配置 `rclone` 连接阿里云 OSS

1. **启动 `rclone` 配置向导**：
   在终端中输入以下命令开始配置：
   
   ```
   rclone config
   ```
2. **创建新配置**：
   
   - 输入 `n` 以创建一个新的配置。
   - 设置远程名称为 `aliyun`（你可以随意命名）。
3. **选择存储类型**：
   输入 `4` 选择 `s3` 类型，因阿里云 OSS 是兼容 S3 的。
4. **配置阿里云 OSS 相关参数**：
   根据你提供的配置填写以下信息：
   
   - **`access_key_id`**：你的阿里云 OSS Access Key ID。
   - **`secret_access_key`**：你的阿里云 OSS Access Key Secret。
   - **`endpoint`**：阿里云 OSS 的服务端点，比如 `oss-cn-shanghai.aliyuncs.com`。
   - **`storage_class`**：设置存储类型，标准： `STANDARD`。

5.**或者直接手动配置**

rclone 配置文件路径如下：

Windows: `%APPDATA%/rclone/rclone.conf`
Linux/macOS: `~/.config/rclone/rclone.conf`

配置示例：

```
[aliyun]
type = s3
provider = Alibaba
access_key_id = LTAI5frgr6mR4hrt3dJvr
secret_access_key = LxrwegVu4w0VcgtyrmVuhGdgrl2B
endpoint = oss-cn-shanghai.aliyuncs.com
storage_class = STANDARD
```

---

### 三、常见操作

配置完成后，你就可以使用 `rclone` 操作阿里云 OSS 了。

#### 1. **列出远程 OSS 存储中的文件**

使用以下命令列出远程 OSS 中的文件和目录：

```
rclone ls aliyun:bucket_name
```

其中，`bucket_name` 是你在 OSS 中创建的存储桶名称。

#### 2. **上传文件到远程 OSS**

要将本地文件上传到阿里云 OSS，可以使用 `rclone copy` 或 `rclone sync` 命令。

- 上传单个文件：
  
  ```
  rclone copy /path/to/local/file aliyun:bucket_name/path/in/oss -P
  ```
  
  `-P`: 显示进度
- 上传整个目录：

```
rclone copy /path/to/local/directory aliyun:bucket_name/path/in/oss --recursive  -P
```

#### 3. **下载文件到本地**

要将远程 OSS 中的文件下载到本地，可以使用以下命令：

- 下载单个文件：
  
  ```
  rclone copy aliyun:bucket_name/path/in/oss /path/to/local/file -P
  ```
- 下载整个目录：
  
  ```
  rclone copy aliyun:bucket_name/path/in/oss /path/to/local/directory -P
  ```

#### 4. **同步本地目录和远程 OSS**

如果你希望将本地目录与远程 OSS 目录同步，使用 `rclone sync` 命令。此命令会确保目标位置和源位置的内容完全一致，删除远程没有的文件。

- 同步本地目录到远程 OSS：
  
  ```
  rclone sync /path/to/local/directory aliyun:bucket_name/path/in/oss -P
  ```
- 同步远程 OSS 到本地：
  
  ```
  rclone sync aliyun:bucket_name/path/in/oss /path/to/local/directory -P
  ```

#### 5. **删除远程文件或目录**

要删除远程 OSS 中的文件或目录，可以使用 `rclone delete` 命令。

- 删除文件：
  
  ```
  rclone delete aliyun:bucket_name/path/in/oss/file -P
  ```
- 删除目录：
  
  ```
  rclone purge aliyun:bucket_name/path/in/oss/directory -P
  ```

#### 6. **查看远程 OSS 存储桶中的文件结构**

使用 `rclone ls` 或 `rclone lsd` 来查看远程存储桶中的文件和目录结构：

- 列出文件：
  
  ```
  rclone ls aliyun:bucket_name
  ```
- 列出目录：
  
  ```
  rclone lsd aliyun:bucket_name
  ```

#### 7. **获取远程文件的详细信息**

如果你想查看某个远程文件的详细信息，可以使用 `rclone lsl` 命令：

```
rclone lsl aliyun:bucket_name/path/in/oss/file
```

---

### 四、其他常用命令

- **查看所有远程配置**：
  
  ```
  rclone config show
  ```
- **测试远程连接是否正常**：
  
  ```
  rclone check aliyun:bucket_name/path/in/oss /path/to/local/directory
  ```
- **查看上传/下载进度**：
  使用 `-P` 参数可以显示进度：
  
  ```
  rclone copy /path/to/local/file aliyun:bucket_name/path/in/oss -P
  ```

