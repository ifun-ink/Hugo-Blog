---
title: Nginx可视化：Nginx Proxy Manager的部署和使用教程 
subtitle:
date: 2024-12-11T21:41:18+08:00
slug: zSA7KJ8k
tags:
  - Linux
  - NginxProxyManager
categories:
  - 系统管理
---


`nmcli` 是 **NetworkManager** 的命令行工具，用来管理Linux系统上的网络连接。内置在大多数Linux发行版中，在没有图形界面的服务器中可以很方便的连接和管理网络。

### **基本概念**

- **NetworkManager**：这是一个用于管理网络连接的后台服务，`nmcli` 是其命令行接口。它能够自动连接网络、管理有线/无线网络、处理 VPN 连接等。
- **Connection**：每个网络配置被称为连接。可以是有线（Ethernet）、无线（Wi-Fi）、移动数据等类型的连接。
- **Device**：网络设备，如网络接口（eth0, wlan0等）或其他网络适配器。

### **基本命令**

1. **查看网络设备状态**
   
   - 显示所有网络设备的状态，包括有线、无线接口。
     ```bash
     nmcli device status
     ```
2. **列出所有连接**
   
   - 显示当前的网络连接（包括已连接和未连接的连接）。
     ```bash
     nmcli connection show
     ```
3. **列出所有设备**
   
   - 查看系统中所有网络接口（有线、无线等）的状态。
     ```bash
     nmcli device show
     ```
4. **显示网络连接的详细信息**
   
   - 显示某个特定连接的详细信息。
     ```bash
     nmcli connection show <connection_name>
     ```

### **网络设备管理**

1. **启用或禁用网络设备**
   
   - 启用网络设备（例如 eth0）：
     ```bash
     sudo nmcli device connect eth0
     ```
   - 禁用网络设备：
     ```bash
     sudo nmcli device disconnect eth0
     ```
2. **启用/禁用 Wi-Fi**
   
   - 启用 Wi-Fi 适配器：
     ```bash
     sudo nmcli radio wifi on
     ```
   - 禁用 Wi-Fi 适配器：
     ```bash
     sudo nmcli radio wifi off
     ```
3. **查看 Wi-Fi 网络**
   
   - 显示周围的无线网络：
     ```bash
     nmcli device wifi list
     ```

### **连接管理**

1. **连接到 Wi-Fi 网络**
   
   - 连接到一个 Wi-Fi 网络：
     ```bash
     nmcli device wifi connect <SSID> password <password>
     ```
   - 其中 `<SSID>` 是无线网络的名称，`<password>` 是无线网络的密码。
2. **创建一个新的连接**
   
   - 创建并连接到新的无线网络（不需要手动创建配置文件）。
     ```bash
     nmcli connection add type wifi con-name <connection_name> ifname wlan0 ssid <SSID> wifi-sec.key-mgmt WPA-PSK wifi-sec.psk <password>
     ```
3. **断开当前连接**
   
   - 断开当前的连接：
     ```bash
     nmcli connection down <connection_name>
     ```
4. **删除连接**
   
   - 删除一个指定的连接配置：
     ```bash
     nmcli connection delete <connection_name>
     ```


### **网络共享**

1. **启用网络共享（桥接）**
   
   - 设置设备共享网络连接：
     ```bash
     nmcli connection modify <connection_name> ipv4.method shared
     ```
2. **禁用网络共享**
   
   - 禁用共享：
     ```bash
     nmcli connection modify <connection_name> ipv4.method manual
     ```

### **配置静态 IP 地址**

1. **设置静态 IP 地址**
   
   - 为连接设置静态 IP 地址：
     ```bash
     nmcli connection modify <connection_name> ipv4.addresses <IP_address>/<prefix_length>
     nmcli connection modify <connection_name> ipv4.gateway <gateway>
     nmcli connection modify <connection_name> ipv4.dns <dns_server>
     nmcli connection modify <connection_name> ipv4.method manual
     ```
2. **重置网络接口**
   
   - 在修改完连接配置后，重启网络接口使更改生效：
     ```bash
     nmcli connection down <connection_name>
     nmcli connection up <connection_name>
     ```

### **常见使用场景**

1. **连接 Wi-Fi 并获取 IP 地址**
   ```bash
   nmcli device wifi connect <SSID> password <password>
   ```

2. **查看设备的 IP 地址和状态**
   
   ```bash
   nmcli device show eth0
   ```
3. **禁用网络接口**
   
   ```bash
   sudo nmcli device disconnect eth0
   ```
4. **启用网络接口**
   
   ```bash
   sudo nmcli device connect eth0
   ```


