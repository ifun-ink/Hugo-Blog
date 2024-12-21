---
title:  Docker PostgreSQL：创建用户和数据库操作指南 
subtitle:
date: 2024-12-16T21:41:18+08:00
slug: 3cRZLqBY
tags:
  - PostgreSQL
  - 数据库管理
  - docker PostgreSQL
categories:
  - 系统管理
---


本文将介绍如何在 Docker 容器中运行的 PostgreSQL 数据库中创建用户、创建数据库，并授予权限。
假设你的 `docker-compose.yml` 文件定义如下：

```yaml
services:
  postgres:
    image: postgres:16.6
    container_name: PostgreSQL
    ports:
      - "5432:5432"  # 映射容器内的 5432 端口到宿主机的 5432 端口
    environment:
      POSTGRES_USER: admin  # PostgreSQL 初始化时的用户名
      POSTGRES_PASSWORD: Z1075FL59d5  # PostgreSQL 初始化时的密码
      POSTGRES_DB: mydb  # 初始化时创建的数据库名
    volumes:
      - ./data/postgres:/var/lib/postgresql/data  # 持久化存储数据库数据
```

在上述文件中，`postgres` 服务会创建一个数据库容器，以下是服务的初始化配置：

* ​**数据库名**​：`mydb`
* ​**用户名**​：`admin`
* ​**用户密码**​：`Z1075FL59d5`

容器启动后，使用 `docker ps` 命令可以查看容器的运行状态：

```bash
ifun@localhost:~$ docker ps
```

输出结果示例：

```bash
CONTAINER ID   IMAGE            NAMES
50e960c03a98   postgres:16.6     PostgreSQL
```

这表明 PostgreSQL 容器已成功启动并命名为 ` PostgreSQL`。

### 1. 创建新用户

要在 PostgreSQL 中创建一个新用户，可以使用 `psql` 命令进入 PostgreSQL 容器，执行创建用户的 SQL 语句。假设我们要创建一个名为 `user1` 的用户，并为其设置密码。

```bash
docker exec -it PostgreSQL psql -U admin -d mydb -c "CREATE USER user1 WITH PASSWORD 'jJ3GINf9OCUK';"
```

* `docker exec -it PostgreSQL psql`: 在名为 `PostgreSQL` 的容器内执行 `psql` 命令。
* `-U admin`: 以数据库管理员 `admin` 用户的身份连接。
* `-d mydb`: 连接到数据库 `mydb`。
* `-c "CREATE USER user1 WITH PASSWORD 'jJ3GINf9OCUK';"`: 创建一个名为 `user1` 的新用户，并为其设置密码 `jJ3GINf9OCUK`。

### 2. 创建新数据库并授权给用户

接下来，我们将创建一个新数据库，并将其所有权限授予新创建的用户 `user1`。假设我们要创建一个名为 `db1` 的数据库。

#### 2.1 创建新数据库

```bash
docker exec -it PostgreSQL psql -U admin -d mydb -c "CREATE DATABASE db1 OWNER user1;"
```

* `CREATE DATABASE db1 OWNER user1;`：创建名为 `db1` 的数据库，并将其所有权授予 `user1`。

#### 2.2 授权用户对数据库的所有权限

```bash
docker exec -it PostgreSQL psql -U admin -d mydb -c "GRANT ALL PRIVILEGES ON DATABASE db1 TO user1;"
```

* `GRANT ALL PRIVILEGES ON DATABASE db1 TO user1;`：授予 `user1` 对数据库 `db1` 的所有权限（如 `SELECT`, `INSERT`, `UPDATE`, `DELETE` 等）。

### 3. 验证数据库和用户

#### 3.1 检查数据库是否创建成功

可以使用 `\l` 命令列出所有数据库，检查数据库 `db1` 是否创建成功。

```bash
docker exec -it PostgreSQL psql -U admin -d mydb -c "\l"
```

输出示例：

```bash
List of databases
      Name      |  Owner   | Encoding |   Collate   |   Ctype    |  Access privileges
----------------+----------+----------+-------------+------------+-------------------
 mydb           | admin    | UTF8     | en_US.utf8  | en_US.utf8 | 
 db1            | user1    | UTF8     | en_US.utf8  | en_US.utf8 | 
 template0      | admin    | UTF8     | en_US.utf8  | en_US.utf8 | =c/admin
 template1      | admin    | UTF8     | en_US.utf8  | en_US.utf8 | =c/admin
```

在输出中，应该能够看到新创建的数据库 `db1`。

#### 3.2 检查用户权限

可以使用 `\dt` 命令列出当前数据库中的所有表，验证 `user1` 是否可以访问数据库 `db1`。

```bash
docker exec -it PostgreSQL psql -U user1 -d db1 -c "\dt"
```

如果 `user1` 成功连接并查看到数据库中的表，说明权限配置正确。如果输出类似 `Did not find any relations`，则表示数据库 `db1` 中没有表，权限配置仍然有效。

完结！成功的在 Docker 容器中成功创建了新的 PostgreSQL 用户、数据库，并授予了相应的权限。

