---
layout: post
title:  "Linux 挂载新硬盘"
date:   2023-01-09 09:41:00 +0800
categories: Linux
permalink: /archivers/LinuxHardDiskMount
---

# linux 挂载新硬盘

输入 lsblk-f 查看所有设备的挂载情况

将新添加的硬盘分区 输入命令fdisk /dev/sdb
输入 n 创建新分区
输入 p 创建主分区
输入 w 保存退出

分区创建好了之后 将分区格式化 格式化命令 mkfs -t ext4 /dev/sdb1

输入lsblk-f 查看分区和格式化结果

输入 vim /etc/fstab 修改这个文件的uuid

```
UUID=testse /mnt/sdb1 ext4 defaults 1 2
```

mount -a 重新加载配置