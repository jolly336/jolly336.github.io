---
title: Mac 外接硬盘安装操作系统的那些事
date: 2019-11-25 00:06:21
tags: Mac
categories: Mac
---

本文主要讲解如何使用外接移动硬盘来安装操作系统（MacOs 和 win10），博主使用的移动硬盘为三星 T5。

<!-- more -->

## 固态硬盘安装mac

方法一：（需要两块移动硬盘）

首先，您需要把数据+系统，通过“时间机器”备份到外置磁盘A中，然后使用“时间机器”恢复到外置磁盘B中

方法二：（只需一块移动硬盘）

* 首先把Mac系统制作到外置SSD中
* 然后把数据导入到外置SSD中的Mac系统内

1. 外置SSD抹盘，macOS扩展日志式+GUID分区图方案
2. 进入Mac恢复分区，在“macOS实用工具”中，点击“重新安装macOS”
3. 最后选择安装到外置SSD
4. 完成操作后，开机时按住option不松手，选择外置SSD进入系统
5. 最后拷贝转移原来Mac电脑中的数据
6. 到家后，把SSD插入家里的Mac电脑中，按住option，选择外置SSD启动进入系统，完成

## 固态硬盘安装 win10

准备：

* windows 10 专业版 64 位 iso 镜像文件
* WTG 辅助工具 

![WTG 安装界面](wtg_setup.jpg)

![安装过程](installation_process.jpg)

[小容量Mac用户必看——苹果电脑外接硬盘安装windows教程](https://post.smzdm.com/p/736280/)

Win10 激活工具

[最简单的WIN10激活工具（永久激活）](https://blog.csdn.net/weixin_45313590/article/details/93883300)

## 其他

mac 格式化

* [移动硬盘无法被格式化为APFS格式 只有macOS扩展日志式 怎么办 急！！！在线等！！！](https://www.macx.cn/thread-2217347-1-1.html)
* [如何恢复已分区的移动硬盘，分区成为Mac os 日志式](https://bbs.feng.com/read-htm-tid-11885702.html)

## 参考

* [给Macbook Pro更换固态硬盘并转移系统的最简单办法](https://www.cnblogs.com/guogangj/p/4051894.html)
* [用外置 SSD 拯救你的老 Mac](https://zhuanlan.zhihu.com/p/30528652)


