---
title: Linux SSH 服务配置笔记
date: 2025-10-25 00:00:00
tags:
  - 工具
categories:
  - 实践
cover: images/delivery.jpeg
excerpt: 记录 Linux 服务器 SSH 服务配置过程
---

## 前言
最近为一台Linux电脑配置了SSH服务，顺便记录下相关知识和配置过程。

## 知识
SSH是安全外壳协议（**S**ecure **Sh**ell）的简称，是一种在不安全网络上用于安全远程登录和其他安全网络服务的协议，主要包括从下至上的三个组件：传输层协议、用户认证协议、连接协议。OpenSSH是最常用的一种对SSH的开源实现，提供了服务端后台程序和客户端工具。

## 开启SSH服务
在服务器上安装OpenSSH并开启SSH服务：
```bash
# 安装openssh服务端
sudo apt update
sudo apt install openssh-server

# 开启ssh服务并确认服务状态
sudo service ssh start
sudo service ssh status
```
有时可能还要在Ubuntu图形化界面的系统设置中确认一下“允许SSH远程连接”之类的选项是否打开。

此时SSH服务已经在**默认端口22**开启，可以在客户端使用 `ssh username@IP_addr` 并输入密码登录，其中IP地址通过服务器的 `ifconfig` 查看。

## 限定SSH密钥认证
服务器在开启SSH服务后，存在被网络中其他电脑暴力破解密码登录的风险。最有效的防护方法就是关闭密码登录，使用SSH密钥登录。

在受信任的客户端上生成密钥：
```bash
# 用ed25519算法生成密钥
ssh-keygen -t ed25519 -C "your_email@example.com"

# 如果系统不支持ed25519，也可以用rsa算法，生成4096位密钥
# ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
此时，`~/.ssh/` 目录下会分别出现一个公钥文件 `id_ed25519.pub` 和一个私钥文件 `id_ed25519`，将公钥文件中的文本复制到服务器的 `~/.ssh/authorized_keys` 文件中。为了确保 `~/.ssh/` 目录不被其他用户访问，还可以设置一下权限：
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
确认用户的公钥都已经存放好以后，在服务器上编辑配置文件 `/etc/ssh/sshd_config`，禁用密码登录并开启密钥登录：
```bash
PasswordAuthentication no
PubkeyAuthentication yes
```
然后应用修改：
```bash
# 重新加载systemd配置
sudo systemctl daemon-reload

# 重启socket服务
sudo systemctl restart ssh.socket

# 检查socket状态
sudo systemctl status ssh.socket
```
此时只有受信任的客户端可以登录服务器，且不需要手动输入密码。

## 修改SSH服务端口
修改SSH服务的默认端口可以躲避绝大多数来自其他电脑的自动化扫描。

在服务器上编辑 `/etc/ssh/sshd_config`：
```bash
# 设置端口号，可以选一个1024到65535之间不常用的端口
Port 25252
```
应用修改后，还需要配置防火墙。假设防火墙已启用且默认拒绝所有入站连接，此时需要打开新端口：
```bash
# 允许新端口的入站连接并加载设置
sudo ufw allow 25252/tcp
sudo ufw reload
```
最后验证一下端口是否修改成功：
```bash
# 检查新端口是否被监听
sudo ss -tlnp | grep :25252
```
从此，客户端登录指令需要加上端口号参数，即 `ssh username@IP_addr -p 25252`。