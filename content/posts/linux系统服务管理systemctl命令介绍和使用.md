---
title:  linux系统服务管理systemctl命令介绍和使用 
subtitle:
date: 2023-10-05T21:41:18+08:00
slug: 11UMia90
tags:
  - Linux
  - systemctl
categories:
  - 系统管理
collections:
  - Linux基础
---

systemd是linux的系统和服务管理器,他可以自动化和简化Linux 系统的管理和维护，包括启动、停止和管理后台服务。Systemd 可以管理所有系统资源，不同的资源统称为 Unit（单元/单位）。Unit 是 Systemd 管理服务的基本单元，可以认为每个服务就是一个 Unit，并使用一个 Unit 文件定义。在 Unit 文件中需要包含相应服务的描述、属性以及需要运行服务的命令。

而`systemctl`则是 Systemd 的主要命令，用于管理系统。

要获取所有的服务单元包括启用的和禁用的，使用下面命令：

```
systemctl list-units --type service
```

> 每行输出从左到右包含以下几列：
> 
> - `UNIT`-服务单元的名称。
> - `LOAD` -有关单元文件是否已加载到内存中的信息。
> - `ACTIVE` -高级单元文件激活状态，可以是active活动，reloading重新加载，inactive非活动，failed失败，activating激活，deactivating停用。
> - `SUB` -低级单元文件激活状态。 该字段的值取决于单元类型。 例如，类型服务可以处于以下状态之一：dead死亡，exited退出，failed失败，inactive不活动或running正在运行。
> - `DESCRIPTION` -单位文件的简短说明。

默认情况下，该命令仅列出已加载的活动单元。 要同时查看已加载但无效的单元，需要`--all`选项：

```
systemctl list-units --type service --all
```

如果要查看所有已安装的单元文件，而不仅仅是加载的文件，请使用：

```
systemctl list-unit-files
```

查看正在运行的服务单元

```
systemctl list-units --type=service --state=running
```

查看非活动的服务单元

```
systemctl list-units --all --type=service --state=inactive
```

查看系统启动时自动运行的服务

```
systemctl list-unit-files --type=service --state=enabled
```

查看启动或停止失败的服务

```
systemctl --type=service --state=failed
```

#### systemctl的一些常用命令总结

首先模糊查询出服务的准确名称

```
systemctl | grep <service-name>
```

有了服务名称之后即可对服务进行操作：

启动服务

```
systemctl start <service-name>
```

停止服务

```
systemctl stop <service-name>
```

重启服务

```
systemctl restart <service-name>
```

重新加载服务配置

```
systemctl reload <service-name>
```

设置服务开机自启动

```
systemctl disable <service-name>
```

取消服务开机自启动

```
systemctl enable <service-name>


systemctl enable --now <service-name>
```

查看指定服务的状态

```
systemctl status <service-name>


#冻结指定服务
systemctl mask <service-name>

#启用服务
systemctl bashunmask <service-name>
```

如果我们要在 shell 脚本中检查服务是否处于活动（active）状态，可以使用 is-active 子命令，0 表示已激活（active）。

```
systemctl is-active <service-name>
```

类似的，如果要检查服务是否已启用（设置为自启动），可以使用 is-enabled 子命令，0 表示已启用：

```
systemctl is-enabled <service-name>
```


