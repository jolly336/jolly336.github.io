---
title: Android 在线源码阅读
date: 2019-07-06 10:19:07
tags: Android
categories: Android
---

本文讲解如何在线阅读 Android 源码，避免本地下载源码的麻烦。作为一名开发工程师，在日常开发中有必要了解自己开发中使用的某一个模块，某一个设计机制，甚至某一个 API，它的底层是如何实现的。才能做到知其然，知其所以然。还是那句话 *Read the fucking source code*。。。

<!-- more -->

## Android 源码在线阅读网址

1. http://androidxref.com
2. http://www.grepcode.com/
3. [Android SDK Search 插件](https://chrome.google.com/webstore/detail/android-sdk-search/hgcbffeicehlpmgmnhnkjbjoldkfhoin?hl=zh-CN)

### Android SDK Search 插件

Chrome 浏览器安装插件，再次搜索 Android API 时，后面会多出来 `(view source)`，点击可以查看源码，其实是链接到 Google 的 Android 源码的 Git。

<img src="http://wanghaoxun.com/img/Android_SDK_Search.jpg" width="80%">
<img src="http://wanghaoxun.com/img/Android_View_Source.jpg" width="80%"/>
<img src="http://wanghaoxun.com/img/Google_Android_Source.jpg" width="80%"/>

### androidxref.com

首页：
<img src="http://wanghaoxun.com/img/androidxref_home.jpg" width="80%">

详情页：
<img src="http://wanghaoxun.com/img/androidxref_detail.jpg" width="80%">

* Full Search: 进行全文搜索，会匹配所有的单词、字符串、标识符以及数字等，例如在frameworks 下通过 Full Search 搜索”mediacodec“，点击”search“，会显示所有包含mediacodec字符(忽略大小写)的结果，即使是注释也会显示出来，如下图，点击对应的链接会打开包含mediacodec所在的文件夹
* Definition：搜索符号定义相关的代码，例如搜索 ondraw 函数的定义
* Symbol：搜索符号，例如可以搜索类中的成员变量等，下图显示了通过 Symbol 搜索FEATURE_NO_TITLE的结果
* File Path：搜索源码文件名中包含给定字符串的文件，例如想要搜索文件名包含mediacodec的源码文件，则可以在 File Path 中填入 mediacodec 进行搜索
* History：这个几乎没有用，用处肯定也不大

> Definition 和 File path 混合搜索：start、mediacodec.cpp，例如搜索mediacodec.cpp中的start函数

## Refs

* [在线看Android系统源码，那些相见恨晚的几种方案](https://blog.csdn.net/hejjunlin/article/details/53454514)


