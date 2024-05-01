---
layout: post
title:  "Linux/CentOS 硬盘剩余空间分区"
date:   2024-05-01 12:17:00 +0800
categories: Linux CentOS
permalink: /archivers/LinuxHardDiskPartition
---

# 背景

公司新分配的服务器硬盘有部分空间未初始化未分区，需要对其进行分区并挂载。

## 操作步骤

1. 查看硬盘信息 `fdisk -l`  

2. 创建新分区 `fdisk /dev/vda`
    - 输入n添加新分区
    - 输入p选择分区类型（默认回车就是p）
    - 选择分区号，一般1-4，默认的就行
    - 选择起始扇区，默认的就行
    - 输入t
    - 输入分区类型，输入8e表示虚拟逻辑卷分区，后期硬盘分区空间不足可以在线扩容
    - 输入命令w，重写分区表  

3. 查看分区信息 `fdisk -l`

4. 初始化硬盘为Linux LVM
    - `partprobe /dev/sda`
    - `pvcreate /dev/vda3`  

5. 创建卷组及逻辑卷并挂载
    - 创建卷组 `vgcreate data_vg /dev/vda3`
    - 创建逻辑卷 `lvcreate -l +100%FREE -n data_lv data_vg`
    - 查看硬盘情况 `df -hl`
    - 格式化 `mkfs.ext4 /dev/data_vg/data_lv`
    - 创建挂载目录 `mkdir /root/data`
    - 挂载 `mount /dev/data_vg/data_lv /root/data`  

6. 设置重启后自动挂载
    - 编辑配置文件 `vim /etc/fstab`
    - 添加新行 `/dev/data_vg/data_lv /root/data ext4 defaults 0 0`
    - 保存文件并退出即可
