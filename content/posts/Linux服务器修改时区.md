---
title: Linux服务器修改时区 
subtitle:
date: 2023-05-17T21:41:18+08:00
slug: BXofQlvf
tags:
  - Linux
  - 时区
categories:
  - 系统管理
collections:
  - Linux基础
---

### 修改服务器时区为 Asia/Shanghai

本文演示如何将服务器的时区设置为 `Asia/Shanghai`。

---

#### 1. 查看当前系统时间和时区

使用 `timedatectl` 命令查看当前系统的时间和时区信息：

```bash
vzsu@Debian:~$ timedatectl
               Local time: Wed 2023-03-08 07:22:59 UTC
           Universal time: Wed 2023-03-08 07:22:59 UTC
                 RTC time: Wed 2023-03-08 07:23:00
                Time zone: UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: n/a
          RTC in local TZ: no
```

从输出可以看出，当前时区是 `UTC`。

---

#### 2. 设置时区为 Asia/Shanghai

使用以下命令更改系统时区：

```bash
sudo timedatectl set-timezone Asia/Shanghai
```

---

#### 3. 验证时区修改结果

再次使用 `timedatectl` 查看系统时区：

```bash
vnxi@Debian:~$ timedatectl
               Local time: Wed 2023-03-08 15:26:45 CST
           Universal time: Wed 2023-03-08 07:26:45 UTC
                 RTC time: Wed 2023-03-08 07:26:46
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: n/a
          RTC in local TZ: no
```

从输出可以看到：

* 时区已变更为 `Asia/Shanghai`，对应时间显示为 `CST, +0800`。
* 系统时间已同步调整。

---

### 总结

通过 `timedatectl` 命令可以快速查看和修改系统时区。修改完成后，建议再次检查是否符合预期，以确保系统时间正常运行。

