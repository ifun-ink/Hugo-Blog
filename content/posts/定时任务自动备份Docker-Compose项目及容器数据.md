---
title:  Docker-Compose自动备份
subtitle:
date: 2024-12-20T21:41:18+08:00
slug: zimYReKR
tags:
  - Docker-Compose
  - 自动备份
categories:
  - 系统管理
---

本文介绍使用脚本来自动备份 Docker Compose 项目中的容器数据，并设置定时任务定期执行该脚本。

## 前置说明

### 适用条件

本教程适用于 Docker Compose 配置文件和容器数据都映射到当前项目目录的情况。也就是整个配置文件和数据文件都位于同一个目录下。

例如，以PostgreSQL 为例，容器数据映射到当前项目目录下的 `data` 文件夹中。以下是一个示例配置：

### 示例 Docker Compose 配置

`docker-compose.yml` 文件内容：

```
services:
  postgres:
    image: postgres:16.6
    container_name: postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: 75FL59d5
      POSTGRES_DB: sqldb
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
```

### 项目目录结构

在这个例子中，我的 Docker Compose 项目目录结构如下：

```
[azfum@localhost PostgreSQL]$ tree
.
├── data
└── docker-compose.yml
```

上面所有的配置文件和容器数据都在当前一个目录下。

---

## 编写备份脚本

### 脚本内容

创建脚本：
存放在任意位置都行：

```
vim backup.sh
```

填入下面内容，按注释说明修改：

```bash
#!/bin/bash

# 此处按需修改！！
# 定义需要备份的docker-compose项目目录（使用绝对路径）
PROJECTS=(
    "/home/azfum/docker/PostgreSQL"
    "/home/azfum/docker/Traefik"
    #"需要备份的项目路径依次添加进来，每行一个，注意格式"
    #"如果有公共的数据库容器，一定要把数据库放到最后一个执行"
)

# 此处按需修改！！
# 定义备份存储位置（使用绝对路径）
BACKUP_DIR="/home/azfum/docker_backups"

# 备份保留的次数（保留最近的 N 次备份）
BACKUP_COUNT=5

# ----------- 下面内容不要动！！---------#
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
# 锁文件路径
LOCK_FILE="/tmp/backup_and_restart.lock"

# 创建备份目录（如果不存在）
mkdir -p "$BACKUP_DIR"

# 检查是否已有脚本实例在运行
if [ -e "$LOCK_FILE" ]; then
    echo "另一个脚本实例正在运行（锁文件：$LOCK_FILE），退出..."
    exit 1
fi

# 创建锁文件并设置清理机制
trap 'rm -f "$LOCK_FILE"; exit' INT TERM EXIT
touch "$LOCK_FILE"

# 记录脚本启动时间
START_TIME=$(date +%s)
echo "脚本开始执行..."

# 初始化任务结果列表
TASK_RESULTS=()

# 遍历配置的项目目录
for PROJECT_PATH in "${PROJECTS[@]}"; do
    PROJECT_NAME=$(basename "$PROJECT_PATH")  # 获取项目目录的名称
    CONTAINER_STOPPED=false

    echo "===================="
    echo "正在处理项目：$PROJECT_NAME"

    if [ -d "$PROJECT_PATH" ]; then
        # 进入项目目录
        cd "$PROJECT_PATH" || { echo "无法进入目录 $PROJECT_PATH，跳过..."; TASK_RESULTS+=("$PROJECT_NAME 失败"); continue; }

        # 检查容器是否在运行
        CONTAINER_STATUS=$(docker compose ps -q)
        if [ -z "$CONTAINER_STATUS" ]; then
            echo "容器 $PROJECT_NAME 当前未在运行，直接进行备份操作..."
        else
            echo "容器 $PROJECT_NAME 已在运行，准备停止容器..."
            # 停止容器
            docker compose down || { echo "停止容器失败，跳过项目 $PROJECT_NAME..."; TASK_RESULTS+=("$PROJECT_NAME 失败"); continue; }
            CONTAINER_STOPPED=true
            # 等待停止完成
            echo "等待 5 秒确保容器完全停止..."
            sleep 5
        fi

        # 备份项目目录
        echo "正在备份项目目录..."
        BACKUP_FILE="$BACKUP_DIR/${PROJECT_NAME}_${TIMESTAMP}.tar.gz"
        if ! tar -czf "$BACKUP_FILE" -C "$(dirname "$PROJECT_PATH")" "$PROJECT_NAME"; then
            echo "备份项目 $PROJECT_NAME 时发生错误，跳过该项目..."
            # 如果容器之前被停止，重新启动容器
            if [ "$CONTAINER_STOPPED" = true ]; then
                echo "重新启动容器 $PROJECT_NAME ..."
                docker compose up -d || { echo "启动容器失败，请手动检查 $PROJECT_NAME"; TASK_RESULTS+=("$PROJECT_NAME 失败"); continue; }
            fi
            TASK_RESULTS+=("$PROJECT_NAME 失败")
            continue
        fi
        echo "项目 $PROJECT_NAME 已备份至 $BACKUP_FILE"

        # 启动容器
        if [ "$CONTAINER_STOPPED" = true ]; then
            echo "重新启动容器 $PROJECT_NAME ..."
            docker compose up -d || { echo "启动容器失败，请手动检查 $PROJECT_NAME"; TASK_RESULTS+=("$PROJECT_NAME 失败"); continue; }
            # 等待容器完全启动
            echo "等待 5 秒确保服务稳定运行..."
            sleep 5
        fi

        TASK_RESULTS+=("$PROJECT_NAME 成功")
        echo "$PROJECT_NAME 成功"
    else
        echo "项目目录 $PROJECT_PATH 不存在，跳过..."
        TASK_RESULTS+=("$PROJECT_NAME 失败")
    fi
done

# 删除多余的备份文件，只保留最新的 $BACKUP_COUNT 次备份
echo "保留最近的 $BACKUP_COUNT 次备份，删除其余的备份文件..."
BACKUP_FILES=$(find "$BACKUP_DIR" -type f -name "*.tar.gz" | sort -r)
EXTRA_FILES=$(echo "$BACKUP_FILES" | tail -n +$((BACKUP_COUNT+1)))
if [ -n "$EXTRA_FILES" ]; then
    echo "删除以下多余的备份文件："
    echo "$EXTRA_FILES"
    echo "$EXTRA_FILES" | xargs rm -f
else
    echo "没有多余的备份文件需要删除。"
fi

echo "过期备份文件删除完成。"

# 删除锁文件
rm -f "$LOCK_FILE"

# 记录脚本结束时间
END_TIME=$(date +%s)
EXECUTION_TIME=$((END_TIME - START_TIME))

# 输出脚本执行总结
echo "===================="
echo "脚本执行总结"
echo "--------------------"
for result in "${TASK_RESULTS[@]}"; do
    echo "$result"
done
echo "脚本执行结束时间: $(date)"
echo "总耗时: $EXECUTION_TIME 秒"
```

脚本的执行过程：停止容器（确保数据统一）、备份项目文件、启动容器、清理多余的备份文件。
你只需要修改以下几个部分：

- **docker-compose 项目的路径**：指定要备份的 Docker Compose 项目路径。
- **备份存储目录**：指定备份文件的存储位置。
- **保留最近的 N 次备份**：指定要保留的备份文件数量。

### 设置脚本执行权限

将 `backup.sh` 保存到任意位置后，需要给脚本加上执行权限：

```
chmod +x ./backup.sh  # 使脚本具有执行权限
```

然后可以手动执行脚本测试其效果：

```
./backup.sh  # 执行脚本
```

---

## 设置定时任务定时执行

### 编辑定时任务

你可以通过 `cron` 设置定时任务，确保备份操作每天自动执行。

1. 输入命令：`crontab -e`，打开 `cron` 配置文件。
2. 添加以下内容，将 `0 2 * * *` 改为你希望执行备份的时间。这里设定为每天凌晨 2 点执行。

```
0 2 * * * /home/azfum/backup.sh >> /home/azfum/log/backup_docker_compose.log 2>&1
```

解释：

- `0 2 * * *`：表示每天的 **凌晨 2 点整** 执行。
- `/home/azfum/backup.sh`：脚本的路径。
- `>> /home/azfum/log/backup_docker_compose.log 2>&1`：将标准输出和标准错误输出都重定向到日志文件 `/home/azfum/log/backup_docker_compose.log`。

### 创建日志目录

确保你指定的日志文件路径存在：

```
mkdir -p /home/azfum/log  # 创建日志目录
```

#### 结合rclone 将备份文件上传到R2

再创建一个定时任务如下：

```
0 3 * * * rclone copy  /home/azfum/docker_backups cfr2:<bucket_name>/docker-compose-backups -P >> /home/azfum/log/rclone_backup.log 2>&1
```

rclone的使用说明参见这篇文章：https://www.azfum.com/archives/E8ul24sN

