title: 解决Mac无法写入U盘问题
tags: 采坑日记
categories: 采坑日记
keywords: U盘
comments: false
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

# 前言
​   新手使用MacBook Pro时，会发现Mac系统下只能读取U盘，但不能写入。其实这个问题是因为，Mac OS系统硬盘格式为HFS， Windows 的硬盘格式为 NTFS，二者互不兼容。那么有没有解决的办法呢？

网上的资料一般都安装第三方软件，如 **NTFS for MAC** 等，但一般都是收费的。或者，格式化U盘，将U盘磁盘格式设定为 **FAT** 或 **exFAT**，但个人不提倡。

现在提供一种方法进行开启 Mac OS 读写 NTFS格式 U盘的功能。经过测试！

# 实现步骤
**1**、打开 终端，输入：
```
diskutil list
```
该命令用于列出系统下的各个磁盘信息，找到你要处理的U盘名称，如名称为：**Chloneda**。

**2**、在终端中，输入：
```
sudo vim /etc/fstab
```
然后输入电脑密码（没有密码的不用输），输入电脑密码后，加入以下内容，进行配置：
```
LABEL=U盘名称 none ntfs rw,auto,nobrowse
```
如下图：
![vim_fstab](/uploads/vim_fstab.png)

注意：如果你的U盘只有一个，只需添加一个即可，不能有空行！其次，如果你的U盘含有空格，如 **Chloneda X**，U盘名称中的空格用\040代替，即命令应该写成：
```
LABEL=Chloneda\040X none ntfs rw,auto,nobrowse
```

参数说明：
- U盘名称：建议不要有中文。
- ntfs rw： 表示把这个分区挂载为可读写的 ntfs格式。
- nobrowse：这个代表了在 finder 里不显示这个分区，这个选项非常重要，如果不打开的话挂载会失败。
完成后，按 **esc** 键退出编辑模式，并按  **:wq!** 命令，然后回车进行保存。

**3**、创建快捷方式
终端中输入以下内容，创建桌面快捷方式。
```
sudo ln -s /Volumes/U盘名称 ~/Desktop/U盘名称
```
因为刚刚创建的分区是不会在 Finder 里不显示的,创建桌面快捷方式，方便以后再次访问U盘（可将 **快捷方式** 拖拽至 Finder 的侧边栏喔）。

**4**、拔掉U盘，重新插入，可见正常显示，正常读写。

# 补充

1、如果不可以写入U盘，请重启一下电脑。
2、如果要恢复之前样子，请输入命令  **sudo vim /etc/fstab** 重新编辑，把写入的 **LABEL** 一行删除，重新保存即可。



------
