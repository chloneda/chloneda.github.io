---
title: SSH自动断开后重连的解决方案
tags: Linux
categories: Linux
keywords: ssh
comments: false
date: 2019-09-23 14:38:47
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

# 问题场景
终端连接远程 SSH 服务，经常会出现长时间无操作后就自动断开，或者无响应，无法再通过键盘输入，只能强行断开重连。

那么有没有办法保持 SSH 连接不断开，或者断开连接后自动重连呢？有的！

# 解决方法
**方案一：客户端发送心跳**

Linux / Unix 下，编辑 ssh 配置文件：
```
vim /etc/ssh/ssh_config
```

在文件中添加以下内容：
```
ServerAliveInterval 20
ServerAliveCountMax 999
```

- ServerAliveInterval：表示每隔多少秒，从客户端向服务器端发送一次心跳（alive 检测）。
- ServerAliveCountMax：表示服务端多少次心跳无响应之后，客户端才会认为与服务器的 SSH 连接已经断开，然后断开连接。

上述配置则表示：每隔20秒，向服务器发出一次心跳。若超过999次请求都没有发送成功，则会主动断开与服务器端的连接。

**方案二：服务器端发送心跳**

在服务器端中，编辑 ssh 配置文件：
```
sudo vim /etc/ssh/sshd_config
```

在文件中添加以下内容：
```
ClientAliveInterval 60
ClientAliveCountMax 3
```
- ClientAliveInterval：表示每隔多少秒，从服务器端向客户端发送一次心跳。
- ClientAliveInterval：表示客户端多少次心跳无响应之后，服务端才会认为客户端已经断开连接，然后断开连接。

上述配置则表示：每隔60秒，服务器向客户端发出一次心跳。若客户端超过3次请求未响应，则会从服务器端断开与客户端的连接。

所以，总共允许无响应的时间是 60 * 3 = 180 秒以内。

其实，依赖 ssh 客户端定时发送心跳，putty、SecureCRT、XShell 工具也有这个功能。

完！