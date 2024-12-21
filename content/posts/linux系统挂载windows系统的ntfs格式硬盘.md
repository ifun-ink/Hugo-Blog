---
title: linux系统挂载windows系统的ntfs格式硬盘 
subtitle:
date: 2024-07-03T21:41:18+08:00
slug: xWmyxlP6
tags:
  - Linux
  - 磁盘挂载
categories:
  - 系统管理
collections:
  - Linux基础
---

开始操作：

首先查看磁盘分区情况以及UUID

```
lsblk -f
```

输出如下：

```
nvme1n1                                                                    
└─nvme1n1p1 ntfs                  055412h513ghf67             

nvme0n1                                                                             
├─nvme0n1p1 vfat   FAT32          FFF6-92D1                             581.4M     3% /boot/efi
├─nvme0n1p2 ext4   1.0            c4f34556e0c-2d7a-4161-9fbf-537927f3b87e  659.8M    25% /boot
└─nvme0n1p3 btrfs        fedora   dc6f3a9435d-4cca-4431-bf49-6ae544510c6b  218.1G     5% /home
```

上面的 nvme1n1p1 分区就是需要挂载的ntfs格式的硬盘，我这里是m.2的固态。后面跟着的一串数字是分区的UUID，记录下来。

新建挂载点：

```
sudo mkdir -p /mnt/ssd-02
```

挂载分区：

```
sudo mount -o rw -t ntfs -U 055412h513ghf67 /mnt/ssd-02
```

UUID务必和上面查询的一致！

永久挂载：

上面挂载后系统重启即失效，使用下面方法永久挂载：

编辑配置文件：

```
sudo vim /etc/fstab
```

写入：

```
UUID=055412h513ghf67 /mnt/ssd-02 ntfs defaults 0 0
```

挂载完成！
