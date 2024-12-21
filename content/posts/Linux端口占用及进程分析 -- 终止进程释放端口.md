---
title: Linux端口占用排查，终止进程释放端口 
subtitle:
date: 2024-12-09T21:41:18+08:00
slug: lLVFdEq7
tags:
  - Linux
  - 端口
categories:
  - 系统管理
collections:
  - Linux基础
---

Linux系统中端口冲突占用的事经常发生，如何排查解决呢？
本文介绍如何查看端口占用、分析进程，以及如何释放端口或终止相关进程。
同时介绍相关工具的使用： `ss`   `lsof`

---

### **1. 什么是端口及端口占用**

端口是网络通信的入口，每个端口对应一个唯一的端口号，用于区分不同的服务（如 HTTP 的 80 端口，SSH 的 22 端口）。当一个服务监听某个端口时，其他进程就无法使用该端口，因此端口冲突会导致服务启动失败。

---

### **2. 查看端口占用**

#### **1. 使用 `netstat` 查看端口占用**

`netstat` 是一个经典工具，可以查看网络连接和端口使用情况。

- 查看所有正在监听的端口：
  ```bash
  netstat -tuln
  ```

选项说明：

- `-t`：显示 TCP 连接
- `-u`：显示 UDP 连接
- `-l`：仅显示监听状态的端口
- `-n`：显示数字形式的地址和端口（避免解析为主机名）

输出示例：

```
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
```

说明：

- `0.0.0.0:80` 表示本地 80 端口正在监听（HTTP 服务）。
- `State` 为 `LISTEN`，表示正在等待连接。

#### **2. 使用 `ss` 替代 `netstat`**

`ss` 是现代 Linux 系统中更高效的工具，用于查看网络状态。

- 查看所有监听的端口：
  ```bash
  ss -tuln
  ```

#### **3. 查看特定端口的占用**

如果你知道某个端口号（如 80），可以直接过滤查看：

```bash
netstat -tuln | grep :80
```

或使用 `ss`：

```bash
ss -tuln | grep :80
```

---

### **3. 查找端口对应的进程**

#### **1. 使用 `lsof`**

`lsof` 是一个强大的工具，用于查看文件或端口的占用情况。

- 查看特定端口的占用：
  ```bash
  lsof -i :80
  ```
  
  输出示例：
  ```
  COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
  nginx     1234  root   6u   IPv4  12345      0t0  TCP *:80 (LISTEN)
  ```
  
  说明：
  - `COMMAND` 是占用端口的进程名称。
  - `PID` 是进程 ID。
  - `USER` 是运行该进程的用户。
  - `NAME` 显示了监听的协议和端口号。

#### **2. 使用 `netstat` 或 `ss` 查看进程**

`netstat` 和 `ss` 也可以显示端口对应的进程信息。

- 使用 `netstat`：
  
  ```bash
  netstat -tulnp
  ```
  
  选项 `-p` 会显示进程信息。
- 使用 `ss`：
  
  ```bash
  ss -tulnp
  ```

输出示例：

```
tcp    LISTEN   0    128    0.0.0.0:80   0.0.0.0:*   users:(("nginx",pid=1234,fd=6))
```

说明：

- `users:(("nginx",pid=1234,fd=6))` 表示端口 80 被 `nginx` 进程（PID 为 1234）占用。

#### **3. 使用 `fuser`**

`fuser` 是专门用于查看进程占用的工具。

- 查看特定端口占用的进程：
  ```bash
  fuser 80/tcp
  ```
  
  输出示例：
  ```
  80/tcp:  1234
  ```
  
  表示端口 80 被进程 ID 1234 占用。

---

### **4. 终止占用端口的进程**

#### **1. 使用 `kill` 命令**

- 查找到进程 ID 后，可以使用 `kill` 命令终止进程：
  ```bash
  kill -9 1234
  ```
  
  其中，`-9` 强制终止进程。

#### **2. 使用 `fuser` 直接释放端口**

`fuser` 可以直接杀死占用端口的进程：

```bash
fuser -k 80/tcp
```

---

### **5. 高级分析工具**

#### **1. 使用 `htop` 或 `top` 查看进程**

如果想通过交互式界面分析进程，可以使用 `htop` 或 `top`：

```bash
htop
```

在界面中按 `F3` 搜索进程名称或 PID。

#### **2. 使用 `tcpdump` 分析流量**

当端口被占用时，可能需要分析流量来源。`tcpdump` 是一个流量捕获工具：

```bash
tcpdump -i eth0 port 80
```

这会捕获通过网卡 `eth0` 上的端口 80 的数据包。

---

### **6. 实战案例：排查端口占用问题**

#### **案例1：服务无法启动，可能是端口被占用**

1. 查看服务日志，发现 80 端口已被占用。
2. 使用 `lsof` 检查端口：
   ```bash
   lsof -i :80
   ```
   
   输出：
   ```
   COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
   apache2   1234  root   6u   IPv4  12345      0t0  TCP *:80 (LISTEN)
   ```
3. 使用 `kill` 终止进程：
   ```bash
   kill -9 1234
   ```

#### **案例2：多个进程监听相同端口**

1. 使用 `ss` 检查所有监听端口：
   ```bash
   ss -tulnp
   ```
2. 发现多个进程绑定了端口（例如，`nginx` 和 `apache2` 同时尝试监听 443 端口）。
3. 修改其中一个服务的配置文件，将其绑定到其他端口（如 8443）。

---

### **7. 常见命令速查表**

| 功能                     | 命令                                       |
|--------------------------|--------------------------------------------|
| 查看监听端口             | `netstat -tuln` 或 `ss -tuln`             |
| 查看特定端口占用         | `lsof -i :port` 或 `fuser port/tcp`       |
| 显示端口对应进程         | `netstat -tulnp` 或 `ss -tulnp`           |
| 强制释放端口             | `fuser -k port/tcp`                       |
| 捕获流量分析             | `tcpdump -i interface port port_number`   |

---



通过以上命令和工具，可以快速定位并解决端口占用问题，保障服务正常运行。建议优先使用现代工具（如 `ss` 和 `lsof`）来代替较老的 `netstat`，并结合 `fuser` 等工具进行高效处理。


