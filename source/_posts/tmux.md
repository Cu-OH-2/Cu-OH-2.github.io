---
title: tmux 学习笔记
date: 2025-10-9 00:00:00
tags:
  - 工具
categories:
  - 学习
cover: images/brain.jpg
excerpt: tmux学习笔记
---
## 简介
tmux（**t**erminal **mu**ltiple**x**er）是一个终端复用工具，允许用户在单个终端中灵活地管理多个分离的终端，实现任务的并行以及不中断运行。

## 概念
tmux通过一个运行在后台的主进程来管理所有结构信息以及内部运行的进程，这个主进程称为“tmux服务器”。用户可以在一个终端中运行**tmux客户端**来连接**tmux服务器**，此时客户端会接管用户的终端并与tmux服务器通信。

tmux服务器管理内部终端的结构自上而下分为**会话**（session）、**窗口**（window）和**面板**（pane），它们之间的关系类似于浏览器、标签页和网页内容的关系。其中与每个内部终端一一对应的结构就是面板，即一个显示终端内容的矩形区域。每个窗口可以包含多个面板，每个会话又可以包含多个窗口，但tmux客户端同一时刻只能连接一个会话。会话和窗口都有各自的名字和索引用于标识。
 
## 使用
tmux的用户级配置文件为 `~/.tmux.conf`，在其中可以进行详细的设置，如修改快捷键、开启鼠标支持、设置窗口与状态栏样式等。tmux快捷键的默认前缀为 ctrl+b，也可以在配置文件中修改。
> tmux中的组合快捷键支持放开前缀键后再按下后续键。

### 会话管理
- 创建会话：`tmux new -s <name>`
- 列出会话：`tmux ls`
- 连接会话：`tmux at -t <name>`
- 删除会话：`tmux kill -t <name>`
- 重命名当前会话：`tmux rename <name>`
- 退出当前会话：`tmux det`（前缀键+d）

### 窗口管理
- 创建窗口：`tmux neww -n <name>`
- 选择指定编号窗口：`tmux selectw -t <num>`（前缀键+编号）
- 重命名当前窗口：`tmux renamew <name>`
- 删除当前窗口：`tmux killw`

### 窗格管理
- 关闭当前窗格：`tmux killp`（或直接使用 `exit` 关闭终端）
- 与上方窗格交换：`tmux swapp -U`

### 其他快捷键
- 列出所有快捷键：前缀键+问号
- 进入命令模式：前缀键+冒号
- 切换窗格：前缀键+方向键

## 文档
更多详细信息可以查询[tmux项目的官方wiki](https://github.com/tmux/tmux/wiki)