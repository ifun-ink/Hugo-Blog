---
title: netcup重装系统图文教程 netcup vps重置系统 
subtitle:
date: 2024-12-01T21:41:18+08:00
slug: 78sRc5lF
tags:
  - Linux
  - VPS
  - netcup
categories:
  - 资源分享
---

首先登录SCP控制台：https://www.servercontrolpanel.de/

按照下面图片说明依次操作:

![DC3560CE-C009-44EA-BA74-7237DA211F1A.png](https://img.idev.ink/2024/12/01/DC3560CE-C009-44EA-BA74-7237DA211F1A.png)

![2ED4CF1B-6318-4308-9CF6-F21D3B18C67C.png](https://img.idev.ink/2024/12/01/2ED4CF1B-6318-4308-9CF6-F21D3B18C67C.png)

![5604CE71-15E5-41E0-95F5-5A71089BE676.png](https://img.idev.ink/2024/12/01/5604CE71-15E5-41E0-95F5-5A71089BE676.png)

Partition layout：硬盘分区方案,选one big ...。
Hostname :主机名，任意设置
Locale：选en-us
Define a User：可选项，创建自定义用户。
SSH Key：可选项，这个需要提前上传公钥，才能选择。上传路径右上角Options -> SSH Keys
其他选项如图默认即可

![E62A8C73-F1D5-468A-A1C1-779E97C51851.png](https://img.idev.ink/2024/12/01/E62A8C73-F1D5-468A-A1C1-779E97C51851.png)

send e-mail to me：一定要勾上，系统安装完成后会给你发一封带密码的邮件
SCP login password：输入SCP密码

![3AB95831-D5ED-46D5-A312-9E00FD490FA6.png](https://img.idev.ink/2024/12/08/3AB95831-D5ED-46D5-A312-9E00FD490FA6.png)

最后点击reinstall，重装完成后收到邮件后即可登录系统。

