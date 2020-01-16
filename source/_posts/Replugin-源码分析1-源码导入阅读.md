---
title: Replugin 源码分析1-源码导入阅读
date: 2020-01-14 11:45:40
categories: [Android]
tags: [Android, 插件化]
---

## 前言

本文会讲解如何导入阅读 Android 热门插件化项目 [RePlugin](https://github.com/Qihoo360/RePlugin)，这个项目由 360 手机卫士客户端团队开发。号称只用一处 Hook 点（ClassLoader）来完成，详细信息请移步 [Replugin wiki](https://github.com/Qihoo360/RePlugin/wiki) 查看。

<!-- more -->

## RePlugin 工程

[Replugin](https://github.com/Qihoo360/RePlugin) 源码 clone 下来，目录结构如下：

<img desc="Replugin 源码目录结构" src="replugin_source_dir.jpg" width="30%"/>

RePlugin 框架本身是基于 Android Studio 开发的，主要包括两种类型的库：

* Android Library
* Gradle 插件

每种库又分为宿主和插件，最后 RePlugin 包括五部分：

* replugin-host-gradle （宿主 gradle 插件，生成清单占坑文件、自定义属性文件等）
* replugin-host-lib （Replugin 核心 Java 工程）
* rplugin-plugin-gradle（插件 Gradle 插件，动态生修改 activity 继承、provider 重定向等）
* replugin-plugin-lib（通过 Java 反射调用主程序中 replugin-host-gradle 相关接口，并提供「双向通信」的能力）
* replugin-sample、replugin-sample-extra （Replugin 使用演示工程，包含宿主和插件的演示）

## 导入 Android Studio

* 源码导入

Replugin 源码工程中 host & plugin 的 gradle 插件是个 Android library，library 项目是个 Android Application，不能一次性导入到一个工程看查看源码，所以我们需要新建个 AS 空项目，把上面介绍的工程通过 `import module`的方式导入进来，导入进来就可以方便插件它的源码了。

如图，replugin_source_reading 是我创建的一个空项目，依次点击 toolbar  -> File -> New -> Import Module 把 replugin-host-gradle、replugin-host-library、replugin-plugin-gradle、replugin-plugin-library 四个 module 导入进来，到这里就把 Replugin 源码环境搭建起来了。

<img desc="源码库导入 AS" src="replugin_source_reading.jpg" width="60%"/>

* sample demo 导入

    1. replugin-sample：包括 host、plugin 演示工程，都是 Android Application 的，所以只能分别导入成一个个 Android 工程来运行
    2. replugin-sample-extra：包括 fresco host & plugin 演示工程，展示宿主和插件公共一份代码的能力，和 replugin-sample 一样导入就行
    
如图，导入实例工程和其它 Android App 一样，选择你要导入的 demo 工程导入即可。

<img desc="Replugin host demo" src="replugin_host_demo.jpg" width="50%" height="50%"/>
<img desc="Replugin plugin demo" src="replugin_plugin_demo.jpg" width="50%" height="50%" />


## 参考

* [Github/Replugin](https://github.com/Qihoo360/RePlugin)
* [RePlugin阅读源码环境搭建](https://www.jianshu.com/p/2244bab4b2d5)

