---
title: Linux 文件权限管理：用户、用户组与读写权限设置详解 
subtitle:
date: 2024-07-12T21:41:18+08:00
slug: l0nXIkOv
tags:
  - Linux
  - 权限管理
categories:
  - 系统管理
---


### Linux 文件权限管理：用户、用户组与读写权限设置详解

本文结合实例详细讲解在Linux系统中如何查看、修改和管理文件权限。

---

### **1. Linux 文件权限基础概念**

在 Linux 中，每个文件和目录都有 **所有者 (Owner)**、**用户组 (Group)** 和 **其他用户 (Others)** 的权限。权限分为三种：

- **读 (r)**：可以查看文件内容或列出目录内容。
- **写 (w)**：可以修改文件内容或在目录中创建、删除文件。
- **执行 (x)**：可以运行文件（例如脚本）或进入目录。

#### **查看权限**

使用 `ls -l` 查看文件或目录的权限：

```bash
ls -l
```

输出示例：

```
-rw-r--r-- 1 user group  1024 Dec  9 10:00 file.txt
```

权限的结构分为以下几个部分：

1. 第一列的 `-rw-r--r--` 表示权限。
   - 第一个字符：文件类型，`-` 代表普通文件，`d` 代表目录。
   - 后面每三位分别表示 **所有者**、**用户组** 和 **其他用户** 的权限。
     - `r`：读权限
     - `w`：写权限
     - `x`：执行权限
2. `user` 是文件的所有者，`group` 是所属的用户组。

---

### **2. 修改文件权限**

#### **1. 使用 `chmod` 修改权限**

`chmod` 命令用于改变文件或目录的权限。

##### **符号模式**

通过符号表示修改权限：

- `u`：所有者 (user)
- `g`：用户组 (group)
- `o`：其他用户 (others)
- `a`：所有用户 (all)

**示例：**

1. 给所有者添加执行权限：
   ```bash
   chmod u+x file.txt
   ```
2. 移除用户组的写权限：
   ```bash
   chmod g-w file.txt
   ```
3. 将所有用户的权限设置为仅可读：
   ```bash
   chmod a=r file.txt
   ```

##### **数字模式**

每种权限用数字表示：

- 读 (r) = 4
- 写 (w) = 2
- 执行 (x) = 1
- 无权限 = 0

三种权限的总和表示该用户的权限。例如：

- `7` (4+2+1)：读、写、执行
- `5` (4+1)：读、执行
- `6` (4+2)：读、写

使用数字设置权限：

```bash
chmod 754 file.txt
```

解释：

- `7`：所有者拥有读、写、执行权限
- `5`：用户组拥有读、执行权限
- `4`：其他用户只有读权限

---

### **3. 修改文件的所有者和用户组**

#### **1. 使用 `chown` 修改所有者**

`chown` 命令用于更改文件的所有者：

```bash
chown newuser file.txt
```

将 `file.txt` 的所有者改为 `newuser`。

#### **2. 使用 `chgrp` 修改用户组**

`chgrp` 命令用于更改文件的用户组：

```bash
chgrp newgroup file.txt
```

将 `file.txt` 的用户组改为 `newgroup`。

#### **3. 同时修改所有者和用户组**

`chown` 也可以同时修改所有者和用户组：

```bash
chown newuser:newgroup file.txt
```

---

### **4. 文件权限的高级管理**

#### **1. 默认权限：`umask`**

`umask` 决定了新创建文件或目录的默认权限。

**查看当前 umask：**

```bash
umask
```

输出示例：

```
0022
```

**解释：**

- 新文件的默认权限是 `666`，新目录的默认权限是 `777`。
- `umask` 会从默认权限中减去指定的值。因此，`umask 0022` 的文件权限为 `644`，目录权限为 `755`。

**修改 umask：**

```bash
umask 0027
```

新文件的权限将为 `640`，新目录的权限将为 `750`。

#### **2. 设置文件的特殊权限**

**a. SUID (Set User ID)**

- 如果文件设置了 SUID 位，执行该文件时将以文件所有者的身份运行。
- 设置 SUID：
  ```bash
  chmod u+s file
  ```

**b. SGID (Set Group ID)**

- 如果目录设置了 SGID 位，目录中创建的文件将继承目录的用户组。
- 设置 SGID：
  ```bash
  chmod g+s dir
  ```

**c. Sticky Bit**

- 如果目录设置了 Sticky Bit，只有文件的所有者或管理员可以删除目录中的文件。
- 设置 Sticky Bit：
  ```bash
  chmod +t dir
  ```

**查看特殊权限：**
使用 `ls -l` 查看，`s` 表示 SUID 或 SGID，`t` 表示 Sticky Bit：

```
drwxrwsr-x  2 user group 4096 Dec  9 10:00 shared_dir
```

---

### **5. 实战案例：结合操作**

#### **案例1：设置共享目录**

1. 创建一个共享目录：
   ```bash
   mkdir shared_dir
   ```
2. 设置用户组为 `developers`：
   ```bash
   chgrp developers shared_dir
   ```
3. 允许用户组有写权限，并设置 SGID 位：
   ```bash
   chmod 2775 shared_dir
   ```

#### **案例2：限制文件访问**

1. 创建一个敏感文件：
   ```bash
   touch secret.txt
   ```
2. 将文件权限设置为仅所有者可读写：
   ```bash
   chmod 600 secret.txt
   ```

#### **案例3：批量修改文件所有者**

假设所有文件需要更改所有者为 `admin`：

```bash
chown admin file1 file2 file3
```

---

### **6. 总结**

Linux 文件权限管理通过 `chmod`、`chown` 和 `chgrp` 等命令实现灵活控制，结合 `umask` 和特殊权限 (SUID、SGID、Sticky Bit)，可以满足各种复杂的安全需求。以下是关键命令的速览：

- **查看权限**：`ls -l`
- **修改权限**：`chmod`
- **修改所有者**：`chown`
- **修改用户组**：`chgrp`
- **设置默认权限**：`umask`


