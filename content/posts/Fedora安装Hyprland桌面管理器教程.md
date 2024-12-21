---
title:  Fedora安装Hyprland桌面管理器教程 
subtitle:
date: 2024-12-08T21:41:18+08:00
slug: uPZOf9iv
tags:
  - Hyprland
  - 平铺桌面管理器
  - Fedora
categories:
  - 系统管理
---

### Fedora Server 安装 Hyprland 图形化桌面环境完整教程

Hyprland 是一款基于 Wayland 的动态窗口管理器，支持高度定制化和特效。

本教程在Fedora Server系统上构建基于 Hyprland 的现代桌面环境。

---

#### 一、安装准备

##### 1. **更新系统**

在开始之前，确保系统已经更新到最新版本：

```bash
sudo dnf upgrade -y
```

---

#### 二、安装 Hyprland 主环境

##### 1. **添加 Hyprland 软件库**

首先添加 Hyprland 的 Copr 仓库：

```bash
sudo dnf copr enable solopasha/hyprland
```

##### 2. **安装 Hyprland**

运行以下命令安装 Hyprland 主程序：

```bash
sudo dnf install hyprland -y
```

---

#### 三、安装图形环境必备软件

为了使桌面环境完整运行，需要安装以下组件：

##### 1. **系统组件**

* ​**Udiskie**​：自动挂载 U 盘和外接设备。
* ​**Hypridle**​：用于空闲控制，支持自动锁屏等功能。
* ​**Hyprlock**​：轻量级锁屏程序。
* ​**Hyprpaper**​：壁纸管理器。

```bash
sudo dnf install udiskie hypridle hyprlock hyprpaper -y
```

##### 2. **桌面工具**

* ​**Mako**​：简单轻量的通知管理器。
* ​**Waybar**​：支持模块化配置的状态栏。
* ​**Wlogout**​：用于退出、重启或锁屏的窗口工具。
* ​**Thunar**​：轻量级文件管理器。

```bash
sudo dnf install mako waybar wlogout thunar -y
```

##### 3. **剪贴板与截图工具**

* ​**Cliphist**​：剪贴板历史管理工具。
* ​**Wl-clipboard**​：Wayland 剪贴板工具。
* ​**Hyprshot**​：截图工具，支持 Wayland。

```bash
sudo dnf install cliphist wl-clipboard hyprshot -y
```

##### 4. **终端与启动器**

* ​**Alacritty**​：现代化的 GPU 加速终端模拟器。
* ​**Wofi**​：Wayland 下的应用启动器。

```bash
sudo dnf install alacritty wofi -y
```

##### 5. **权限管理**

Hyprpolkitagent：为 Wayland 提供 Polkit 权限支持。

```
sudo dnf install hyprpolkitagent -y
```

---

#### 四、功能增强软件安装

##### 1. **网络和蓝牙管理**

* ​**Network Manager Applet**​：网络管理工具，支持 GUI 操作。
* ​**Blueman**​：蓝牙设备管理工具。

```bash
sudo dnf install network-manager-applet blueman -y
```

##### 2. **通知管理**

* ​**SwayNotificationCenter**​：功能更强的通知中心。

安装步骤：

```bash
sudo dnf copr enable erikreider/SwayNotificationCenter
sudo dnf install SwayNotificationCenter -y
```

##### 3. **输入法支持**

* ​**Fcitx5**​：输入法框架。
* ​**Fcitx5-rime**​：Rime 输入法支持。

```bash
sudo dnf install fcitx5 fcitx5-rime -y
```

##### 4. **用户目录管理**

安装并生成标准用户目录（如 `文档`、`下载` 等）：

```bash
sudo dnf install xdg-user-dirs -y
xdg-user-dirs-update
```

---

#### 五、配置启动项

在 Hyprland 的配置文件中（ `~/.config/hypr/hyprland.conf`），添加以下内容：

```plaintext
exec-once = swaync
exec-once = udiskie & hypridle & hyprpaper & waybar & hyprlock
exec-once = fcitx5 --replace -d
exec-once = systemctl --user start hyprpolkitagent
exec-once = wl-paste --type text --watch cliphist store
exec-once = wl-paste --type image --watch cliphist store
```

**配置说明：**

* ​`udiskie`：自动挂载外接存储设备。
* ​`hypridle`​：空闲管理器，支持锁屏或节能操作。
* ​`swaync`：通知管理器。
* ​`hyprpaper`：壁纸管理器。
* ​`waybar`：状态栏。
* ​`fcitx5`​：输入法框架。
* ​`wl-paste`：剪贴板历史管理。

---

#### 六、桌面环境运行

完成安装和配置后，通过以下命令启动 Hyprland 图形化桌面：

```bash
Hyprland
```

或者将其配置为默认的登录会话。

---

以上是一个基础的Hyprland的桌面环境，后续还有很多优化项目设置需要做，之后再更新。

