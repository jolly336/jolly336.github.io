---
title: 解决Android手机无法查看data文件夹
date: 2015-04-04 20:23:16
tags: [Android]
categories: [Android, Tools]
---

Android DDMS 连接真机（已ROOT)，用 file explore 看不到 data/data 文件夹，问题在于 data 文件夹没有权限，用 360 手机助手或豌豆荚也是看不见的。 有以下两种解决方法：

<!-- more -->

* 方法一：使用 adb shell命令

**注：Android 下的 shell 是不完整的，不能用 -R 参数，即使su 到 root 帐号也执行不了。**效果如下： 

![adb_shell_fail](adb_shell_fail.png)

所以只能一个一个文件夹去设置权限。打开 cmd 控制台，依次执行以下命令：

```
adb shell
su
chmod 777 /data
chmod 777 /data/data
```
效果显示如下：

![adb_shell_success](adb_shell_success.png)
 
* 方法二：用RE文件管理器：

在手机上安装 RE文件管理器，将 RE文件管理器授予 root 权限，然后将 data 和 data/data 文件夹设置成 777 权限。效果如下：

![RE文件管理器1](re_permission.png)

![RE文件管理器2](re_warning.png)

![RE文件管理器3](re_chmod_777.png)
 
下面来打开 DDMS中 的 file explore 看一下。执行菜单栏Windows---Show view---Others：

![eclipse_toolbar](eclipse_toolbar.png)

弹出对话框，选中 Android 目录下的 File Explorer：

![File Explorer](eclipse_show_view.png)

这样就可以在 data/data 下看到自己安装的 Android 程序了：

![data_dir_success](eclipse_file_explorer.png)

 


