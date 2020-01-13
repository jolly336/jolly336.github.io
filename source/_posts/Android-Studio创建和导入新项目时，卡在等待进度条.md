---
title: Android Studio创建和导入新项目时，卡在等待进度条..
date: 2018-03-24 14:38:08
tags: Android Studio
categories: Android Studio
---

本文将讲解使用Android Studio新建或导入外部基于gradle构建的Android项目时，卡在进度条的问题。

<img src="http://wanghaoxun.com/androidstudio_loading.png" width="60%" height="30%">

<!-- more -->

当我们使用AS工具新建或者导入外部基于gradle构建的安卓项目时，会一直卡在进度条那里，因为没有详细的log信息，不知道as在干了些神马。其实，as工具是在检测和下载项目要使用的对应gradle版本，一般要下载大于100M左右的zip文件，如果没有翻墙，那将很浪费我们时间。所以我们可以手动去下载对应gradle工具包，来跳过此次漫长的等待，毕竟我们的时间很有限的。

##  一、结束正在等待的AS进程

* Mac打开monitor活动管理器，找到Android studio，强制退出即可
* window打开资源管理器，关闭AS进程

## 二、查看项目要使用gradle工具的版本号

找到我们的项目，进入`gradle->wrapper`目录，找到`gradle-wrapper.properties`，打开查看最后一行distrubutionUrl代表的版本号，例如`distributionUrl=https\://services.gradle.org/distributions/gradle-3.3-all.zip`，就说明是gradle-3.3-all.zip

## 三、去gradle官网下载刚才查看的gradle版本包

[gradle工具包下载地址](https://services.gradle.org/distributions/)
下载刚才gradle3.3的包，如图

![gradle3.3](http://wanghaoxun.com/gradle3.3%E4%B8%8B%E8%BD%BD.png)

## 四、替换gradle

* 进入` ~/.gradle/wrapper/dists/gradle-3.3-all`目录，看到一个很长随机名字的目录像ac27o8rbd0ic8ih41or9l32mv，里面有两个文件`gradle-4.0-all.zip.lck`和`gradle-4.0-all.zip.part`
* 修改`gradle-4.0-all.zip.part`文件后缀名为`gradle-4.0-all.zip.ok`，来欺骗Android studio已经下载好了
* 把刚才下载的gradle-3.3-all.zip包解压到这个目录里，保留压缩包文件

![gradle directory](http://omdtn071e.bkt.clouddn.com/gradle3.3%E7%9B%AE%E5%BD%95.png)

## 五、重新导入项目

最后重新导入项目，就可以跳过进度条，很快编译起来

## 总结
> 原理

AS工具创建项目时，回去检查~/.gradle/wrapper/dists/gradle-x.x-all目录是否有临时文件夹，如果没有，就会去gradle官网下载gradle-x.x-all.zip到临时文件夹。所以上面的步骤就是跳过此次检查和下载。





