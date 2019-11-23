---
title: Docker容器Centos不能使用systemctl命令问题
tags: 采坑日记
categories: 采坑日记
keywords: 坑
comments: false
---
**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

最近使用Docker搭建Centos容器时遇到这样的问题：Centos系统的不能使用systemctl命令!

**具体场景**
使用 systemctl 或 service 命令重启服务时。
```
systemctl restart snmpd.service 
```

会报无权限的错误：
```
Failed to get D-Bus connection: Operation not permitted；
```

这是docker中centos7的bug，官网上也提到了这个问题，并给出了 [解决办法](https://github.com/docker-library/docs/tree/master/centos#dockerfile-for-systemd-base-image)，但有点复杂。我们可以通过以下方法解决！

**首先**，使用docker构建centos容器加上 **privileged** 参数，即在docker run命令是要加上 --privileged=true，该参数在docker容器运行时，让系统拥有真正的root权限。

**其次**，在启动容器时，在docker run 命令最后，加上/usr/sbin/init，最终命令为：
```
docker run -v /tmp/:/tmp --privileged --cap-add SYS_ADMIN -e container=docker -it --name=centos -d --restart=always centos /usr/sbin/init
```
参数说明：
- -v /tmp/:/tmp：挂载宿主机的一个目录，冒号":"前面的目录是宿主机目录，后面的目录是容器内目录。
- --privileged： 指定容器是否是特权容器。

- --cap-add SYS_ADMIN： 添加系统的权限，不然系统很多功能都用不了的。

- -e container=docker：设置容器的类型。

- -it： 启动互动模式。

- /usr/sbin/init：初始容器里的CENTOS，用于启动dbus-daemon。

**最后**，如果想查看Docker更多内容，请查看[Docker](https://docs.docker.com/)官网文档。
