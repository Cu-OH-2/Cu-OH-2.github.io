---
title: NAS 挂载笔记
date: 2025-08-05 00:00:00
tags:
  - NAS
categories:
  - 实践
cover: images/delivery.jpeg
excerpt: 记录 NAS 挂载过程
---

## 前言
实验室最近购入了新的NAS服务器（绿联DXP8800）。本人之前对NAS较为陌生，遂对相关知识和挂载流程进行记录。

## 知识
NAS（Network Attached Storage）是一种专门用于数据存储和共享的网络设备，可以挂载到不同的文件系统中，提供RAID、用户权限管理等功能。以实验室的NAS为例：
- 装有8个硬盘。其中3个为JBOD模式，用作共享空间，剩下5个支持RAID 5，用作个人空间。
- 通过SMB协议连接用户文件系统，支持Windows/MacOS/Linux，基于用户/用户组管理读写权限。
  > 其中Linux系统挂载使用的工具虽然名为“mount.cifs”，但该工具已支持SMB2/3，CIFS协议已被现代场景淘汰。
- 品牌一般会提供对应的电脑/手机客户端，可以在上面进行个人空间的管理。

## 在实验室服务器上挂载NAS（Linux）
### 1. 切换有效组
切换有效组为 `users`，保证有访问共享空间的权限：
```bash
newgrp users
```

### 2. 挂载
在本地创建一个挂载点：
```bash
sudo mkdir /home/lxy/nas
```

然后进行挂载：
```bash
sudo mount -t cifs //**.**.*.***/personal_folder /home/lxy/nas -o username='xylin',password='*********',iocharset=utf8,uid=$(id -u),gid=$(id -g),dir_mode=0700,file_mode=0600
```
其中：
- `**.**.*.***` 是NAS服务器的内网IP。
- `personal_folder` 是NAS管理员创建的目录。
- `username` 和 `password` 是NAS管理员给每个人分配的账号密码。
- `dir_mode=0700` 指仅属主可读/写/进入目录。`0700` 是遵循Linux权限模式的八进制数字，后三位分别代表属主权限、属组权限、其他用户权限。
- `file_mode=0600` 指仅属主可读/写文件。
- `uid=$(id -u)` 将文件属主绑定到当前用户UID。
- `gid=$(id -g)` 将文件属组绑定到当前用户GID。

### 3. 检查挂载结果
```bash
df -h # 查看挂载情况
ls /home/lxy/nas  # 检查文件访问
touch /home/lxy/nas/test.txt  # 测试写入权限
```
### 4. 再次挂载
如果服务器重启了，需要重新挂载。

## 在个人电脑上挂载NAS（MacOS）
右键“访达” - 连接服务器 - 输入 `**.**.*.***` - 输入用户名和密码 - 选择 `personal_folder`

## 在个人电脑上挂载NAS（Windows）
右键“此电脑” - 映射网络驱动器 - 输入 `\\**.**.*.***\personal_folder` - 输入用户名和密码