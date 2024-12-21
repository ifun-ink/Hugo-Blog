---
title: Linux服务器创建swap交换空间
subtitle:
date: 2024-12-20T21:41:18+08:00
slug: qw45LDLC
tags:
  - Linux
  - 交换空间
categories:
  - 系统管理
collections:
  - Linux基础
---


### 新建交换文件

#### 1. 检查是否已有交换空间

运行以下命令，若无输出，表示没有交换空间：

```bash
sudo swapon --show
```

#### 2. 检查硬盘空间

确保有足够空闲空间：

```bash
df -h
```

#### 3. 创建交换文件

创建 1GB 交换文件（可根据需要调整大小）：

```bash
sudo fallocate -l 1G /swapfile
```

验证文件已创建：

```bash
ls -lh /swapfile
```

示例输出：

```
-rw-r--r-- 1 root root 1.0G Aug 23 11:14 /swapfile
```

#### 4. 设置权限与标记

设置文件权限：

```bash
sudo chmod 600 /swapfile
```

标记为交换空间：

```bash
sudo mkswap /swapfile
```

#### 5. 启用交换文件

启用交换文件：

```bash
sudo swapon /swapfile
```

验证是否启用成功：

```bash
sudo swapon --show
```

示例输出：

```
NAME      TYPE  SIZE USED PRIO
/swapfile file 1024M   0B   -2
```

或检查内存状态：

```bash
free -h
```

示例输出：

```
total        used        free      shared  buff/cache   available
Mem:           976Mi        85Mi       612Mi       0.0Ki       279Mi       756Mi
Swap:          1.0Gi          0B       1.0Gi
```

---

### 配置交换空间

上述配置仅对当前会话有效。若需永久生效，请按以下步骤操作。

#### 1. 修改 `/etc/fstab`

备份文件：

```bash
sudo cp /etc/fstab /etc/fstab.bak
```

添加交换文件至配置：

```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

### 调整系统参数

#### 1. 修改 `swappiness`

`swappiness` 控制系统将数据从 RAM 移动到交换空间的频率：

- **0**：尽量使用物理内存，仅在内存不足时使用交换空间。
- **100**：积极使用交换空间。
  默认值为 **60**，建议服务器设为 **10**（或数据库服务器设为 0 或 1）。
  查看当前值：

```bash
cat /proc/sys/vm/swappiness
```

临时设置：

```bash
sudo sysctl vm.swappiness=10
```

#### 2. 修改 `vfs_cache_pressure`

该参数控制内核回收缓存的倾向。较低值可提高系统响应性，推荐设为 **50**：

```bash
sudo sysctl vm.vfs_cache_pressure=50
```

#### 3. 参数永久化

编辑配置文件：

```bash
sudo vim /etc/sysctl.conf
```

在末尾添加以下内容：

```
vm.swappiness=10
vm.vfs_cache_pressure=50
```

保存并退出。

---

### 验证配置

执行以下命令确认配置：

```bash
free -h
sudo swapon --show
```


