---
title:  忘记nginx-proxy-manager密码如何找回密码恢复数据 
subtitle:
date: 2022-03-20T21:41:18+08:00
slug: 82QEBjap
tags:
  - Nginx
  - NginxProxyManager
categories:
  - 系统管理
---

*Nginx Proxy Manager 默认使用 SQLite 数据库。*

**具体原理**：
Nginx Proxy Manager支持多个管理员账号，不同账号数据不通，但是任意管理员可修改另一个管理员的密码。
由此我们就可以通过操作数据库新建一个管理员账号，用这个新的管理员账号去直接修改旧管理员账号的密码。以此达到重置找回Nginx Proxy Manager账号密码的目的。

---

### 第一步：进入容器并重置数据库

1. 获取容器名：
   ```bash
   sudo docker ps
   ```

假设容器名为 `nginx-proxy-manager-app-1`。

2. 进入容器：
   
   ```bash
   sudo docker exec -it nginx-proxy-manager-app-1 sh
   ```
3. 安装 SQLite（如未安装）：
   
   ```bash
   apt update && apt install sqlite3 -y
   ```
4. 进入数据库：
   
   ```bash
   sqlite3 /data/database.sqlite
   ```
5. 重置用户状态，将所有用户标记为删除：
   
   ```sql
   UPDATE user SET is_deleted=1;
   
   .exit
   ```
6. 退出容器：
   
   ```bash
   exit
   ```
7. 重启容器：
   
   ```bash
   sudo docker restart nginx-proxy-manager-app-1
   ```

---

### 第二步：登录默认账户

1. 重启后，打开控制台页面。
2. 使用系统默认账户登录：
   
   - **账号**：`admin@example.com`
   - **密码**：`changeme`
3. 登录后修改账户名称和密码，务必设置一个与原账号不同的名称以防覆盖原数据。

---

### 第三步：还原原账号

1. 此时新账号创建完成，接着重新启用所有其他账号（分别输入下面命令）：
   
   ```bash
   sudo docker exec -it nginx-proxy-manager-app-1 sh
   
   sqlite3 /data/database.sqlite
   
   UPDATE user SET is_deleted=0;
   
   .exit
   
   exit
   ```
2. 然后重启容器：
   
   ```bash
   sudo docker restart nginx-proxy-manager-app-1
   ```

---

### 第四步：修改原账号密码

1. 刷新页面，原账号会重新出现。
2. 点击账号管理页面，修改老账号的密码。
3. 之后使用原账号登录即可。

---

### 注意事项

- 重置过程中，新创建的账号数据与原账号数据不互通。
- 确保新创建的账号名称不同于原账号，以防覆盖数据。

