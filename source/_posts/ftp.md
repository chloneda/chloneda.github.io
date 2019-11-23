---
title: Ubuntu中FTP安装配置及基本概念
tags: FTP
categories: FTP
keywords: FTP
comments: false
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

# 安装
用apt-get工具安装vsftpd
```
$ sudo apt-get install vsftpd 
```
检查FTP端口是否已经打开
```
$ netstat -tnl　或　ps -ef | grep ftp
```
检查FTP服务是否开启
```
$ service vsftpd status
```
如果FTP服务已经开启，则会显示如下信息，由**Active**关键词可知FTP服务正在运行
![ftpStatus](/uploads/ftp_Status.png)
FTP启动、停止、重启、查看状态的三种方式。
```
$ service vsftpd start|stop|restart|status
$ systemctl start|stop|restart|status vsftpd
$ /etc/init.d/vsftpd start|stop|restart|status|reload
```
**reload**为重新加载配置文件


# 配置
修改FTP配置文件
```
$ sudo vi /etc/vsftpd.conf
```
## FTP主要配置
主要配置说明：
```
# 设置登录FTP欢迎信息
ftpd_banner=Welcome to CHL FTP service. 

# 基本配置1
listen=YES               # 服务器监听
local_enable=YES         # 是否允许本地用户访问
write_enable=YES         # 是否允许上传文件，不开启会报 550 permission denied
anonymous_enable=NO      # 匿名访问允许，默认不要开启
anon_upload_enable=YES   # 匿名上传允许，默认是NO
anon_mkdir_write_enable=YES  # 匿名创建文件夹允许 

# 基本配置2
local_umask=022	　　# FTP上本地的文件权限，默认是077。此时umask为022，则目录为777-022=755，文件为666-022=644。
dirmessage_enable=YES     # 进入文件夹允许 
connect_from_port_20=YES   # 启用20号端口作为数据传送的端口 
data_connection_timeout=120	# 设置数据连接超时时间

# 日志配置
utf8_filesystem=YES	# vsftpd使用utf8文件系统
use_localtime=YES
xferlog_enable=YES         # 激活上传和下传的日志 
xferlog_file=/var/log/vsftpd.log	# 设定系统维护记录FTP服务器上传和下载情况的日志文件
xferlog_std_format=YES     # 使用标准的日志格式 

# 自定义
local_root=/share/vsftpd	# 设置自定义的ftp根目录的位置

# 读写权限
allow_writeable_chroot=YES	# 解决"500 OOPS: vsftpd: refusing to run with writable root inside chroot()" 问题
write_enable=YES	# 允许向FTP服务器写入权限
chown_uploads=YES	# 设定是否允许改变上传文件的属主，与下面一个设定项配合使用
chown_username=whoever	# 设置想要改变的上传文件的属主，可设为ftp

ascii_upload_enable=YES	　# 允许服务器以ASCII方式传输数据,但引起"SIZE /big/file"方式的DoS攻击
ascii_download_enable=YES
deny_email_enable=YES	　# 黑名单设置。如果很讨厌某些email address，可以取消他的登录权限
banned_email_file=/etc/vsftpd.banned_emails

# FTP限制最大连接数和传输速率，进行资源控制，避免负担过大而运行异常
max_client=50	# FTP服务器的所有客户端最大连接数不超过50个
max_per_ip=5	# 同一IP地址的FTP客户机与FTP服务器建立的最大连接数不超过5个
local_max_rate=100000	# FTP服务器的本地用户最大传输速率设置为100KB/s.
anon_max_rate=50000	# FTP服务器的匿名用户最大传输速率设置为50KB/s.

# 权限设置
#是否启动userlist为禁止模式，YES表示在userlist中的用户禁止登录ftp（黑名单），NO表示黑名单失效
userlist_deny=NO
userlist_enable=NO	# 是否启动限制用户的名单为允许模式，上面的YES限制了所有用户，可以用这个名单作为白名单，作为例外允许访问ftp根目录以外
userlist_file=/etc/vsftpd.user_list

# 在默认配置下，本地用户登入FTP后可以使用cd命令切换到其他目录，这样会对系统带来安全隐患,可配置如下
chroot_list_enable=YES     # 设置是否启用chroot_list_file配置项指定的用户列表文件。默认值为NO。
chroot_local_user=YES      # 用于指定用户列表文件中的用户是否允许切换到上级目录。默认值为NO。
chroot_list_file=/etc/vsftpd.chroot_list	# 禁用名单，用于指定用户列表，该文件用于控制哪些用户可以切换到home目录的上级目录。
```

**通过搭配能实现以下几种效果**

- 当**chroot_list_enable=YES，chroot_local_user=YES时**，在/etc/vsftpd.chroot_list文件中列出的用户，可以切换到其他目录；未在文件中列出的用户，不能切换到其他目录。

- 当**chroot_list_enable=YES，chroot_local_user=NO**时，在/etc/vsftpd.chroot_list文件中列出的用户，不能切换到其他目录；未在文件中列出的用户，可以切换到其他目录。

- 当**chroot_list_enable=NO，chroot_local_user=YES**时，所有的用户均不能切换到其他目录。

- 当**chroot_list_enable=NO，chroot_local_user=NO**时，所有的用户均可以切换到其他目录。 
 
配置正确后可**浏览器**或**终端**输入以下信息访问
```
ftp://服务器IP	# 浏览器方式访问

$ ftp 服务器IP	# 终端方式访问
```
也可以通过浏览器这样访问。
```
ftp://用户名:密码@IP/具体FTP路径	# 如:ftp://vsftpd:vsftpd@192.167.2.20/chl
```

## 修改默认端口

默认FTP服务器端口号是21，出于安全目的，有时需修改默认端口号，编辑**/etc/vsftpd.conf**文件。
```
listen_port=6666
```
重新指定了FTP服务器的端口号，需重启服务使配置生效,并利用终端访问。
```
$ /etc/init.d/vsftpd restart
$ ftp 服务器IP 6666

```
**注：端口号需正确，否则连接失败。**

## 设置FTP目录
创建FTP根目录，需与配置文件一致。
```
$ mkdir -p /share/vsftpd

```
## 创建FTP用户
```
$ sudo useradd -g vsftpd -d /share/vsftpd -m test 
```
命令参数说明：
- g：用户所在的组 
- d：指定FTP目录 
- m：不建立默认家目录

设置FTP用户密码
```
$ sudo passwd vsftpd 
```
编辑/etc/vsftpd.chroot_list文件，将vsftpd的帐户名添加进去，保存退出,并重启FTP服务。

# 卸载
当我们不需要FTP时，可以卸载FTP并删除FTP用户。
```
$ sudo apt-get remove --purge vsftpd	# purge 选项表示彻底删除改软件和相关文件
```
删除FTP用户
```
$ sudo userdel vsftpd
```

# 常用命令
## 路径切换
FTP可以定位服务器与本地硬盘的路径。其中使用 **lcd** 命令切换宿主机本地路径，命令如下：
```
$ lcd 目录名	# 进入宿主机目录
```
而用 **cd** 命令切换远程服务器的路径,命令如下:
```
$ cd 目录名	# 进入FTP服务器目录

```

说到这里，得说说 ! 命令的作用，在FTP中!会执行宿主机shell命令，如:
```
$ !cmd [args]	# 在宿主机中执行交互shell，exit回到FTP环境,例：
$ !dir	或	!ls
```
或如果不加!,显示FTP服务器当前目录内容,如：
```
$ dir	或	ls
```

**此外**，ftp命令支持"含有空格"的文件夹/文件名，即在引用时加上双引号""。

## 下载文件
- get:一次只下载一个文件。
- mget:一次可以下载多个文件，而且支持通配符。

## 上传文件
- send: 上传一个文件。
- put：上传一个文件。
- mput: 上传多个文件。

## 其他命令
其实FTP命令的核心就是善用　**help** 或 **?**　查看具体命令的含义，例如：
```
ftp> help	或	?
```

可利用 **? [cmd]** 或 **help [cmd]** 查看具体命令含义，如图。
![ftpHelp](/uploads/ftp_Help.png)

有时侯我们会对多个文件进行操作，此时需要对每一个文件都选择y/n，挺麻烦的！可用prompt命令关掉交互方式。
```
prompt off	# 关闭
prompt on	# 打开
```

# 其他信息
[FTP 数字代码的意义](https://wenku.baidu.com/view/ed191363af1ffc4ffe47ac2c.html)

**参考资料**
[vsftpd最详细的配置文件](https://www.cnblogs.com/9426yu/p/4835443.html)



---
