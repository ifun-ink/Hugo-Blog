---
title: rime
subtitle:
date: 2024-12-20T21:41:18+08:00
slug: 5831dd
tags:
  - Linux
  - 时区
categories:
  - 系统管理
collections:
  - Linux基础
---


官网：[https://rime.im/](https://rime.im/)

安装说明：[https://github.com/rime/home/wiki/RimeWithIBus](https://github.com/rime/home/wiki/RimeWithIBus)

#### 安装

Archlinux

```
pacman -S ibus-rime
```

Debian

```
sudo apt install ibus-rime
```

Gentoo

```
emerge ibus-rime
```

Ubuntu

Ubuntu 12.10及以上版本

```
sudo apt-get install ibus-rime
```

Fedora 22+

```
sudo dnf install ibus-rime
```

#### 开始使用

重新启动IBus

```
ibus-daemon -drx
```

打开设置在IBus首选项（IBus设置）中，将“中文-Rime”添加到输入法列表中

```
ibus-setup
```

在系统设-键盘设置中添加Chinese(Rime).

若不能正常使用建议注销重新登录系统。

按快捷键：`Ctrl+~` 设置中文简体


