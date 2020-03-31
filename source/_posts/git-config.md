---
title: git config全局和本地配置
tags: Git
categories: Git
keywords: Git
comments: false
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)



# 前言

程序开发人员在日常开发过程中，使用 **git** 可能会遇到这样一个情况：
- 公司项目使用公司账号提交git信息。
- 个人业余项目使用个人账户提交git信息。

也就是说，我们需要 **git** 根据不同项目使用不同的账号及邮箱提交相关记录，本文就是专门解决这个问题的。



# 本地设置

本地设置有两种方式，**命令方式** 和 **配置文件方式**，两种方式选择任意一种，都可以配置当前git项目提交git信息的账号及邮箱。



## 命令方式

对于本地 **git repo** 配置，先进入 **本地git repo** 目录，使用命令：

```bash
git config user.name "your-username"
git config user.email "your-email-address"
```



## 配置文件方式

对于本地 **git repo** 配置，先进入 **本地git项目** 目录，编辑 **.git/config** 文件，增加以下信息：

```bash
[user]
    name = your-username
    email = your-email-address
```

保存并退出。以上 **命令方式** 和 **配置文件方式** 两种方式，都可以在 **本地git repo** 目录下，通过以下命令查看本地项目的账号及邮箱是否更改成功。

```bash
git config user.name
```

对于很多个 **本地git repo** 来说，每个 **本地git repo** 都要设置自定义化git本地账号及邮箱，有时我们不想每个都设置，我们可以使用git的全局配置，所有 **本地git repo** 都是使用全局配置！



# 全局设置

全局设置有两种方式，**命令方式** 和 **配置文件方式**，两种方式选择任意一种，都可以配置全局git项目提交git信息的账号及邮箱。



## 命令方式

在git服务器中，任意非 **本地git repo** 中，使用以下命令配置全局配置：

```bash
git config --global user.name "your-username"
git config --global user.email "your-email-address"
```



## 配置文件方式

编辑 **〜/.gitconfig**，其内容与 **.git/config** 文件中的内容相同，即将 **[user]** 部分信息添加至 **〜/.gitconfig** 文件中，内容如下：

```bash
[user]
    name = your-username
    email = your-email-address
```

保存并退出。任意非 **本地git repo** 中，在使用以下命令查看是否配置成功

```bash
git config user.name
```

好了，这次git config全局及本地配置就到这里了！


------


