---
title: Android Studio那些错误的问题们
date: 2018-03-20 22:10:16
tags: Android Studio
categories: Android Studio
---

本片博客会记录关于Android开发工具Android Studio出错的那些问题，包括导入项目编译失败、时间过长，莫名其妙的歇菜等等。。。

<img src="http://opkjcw4sd.bkt.clouddn.com/AndroidStudioTool.jpg" width="60%" height="30%">

<!-- more -->

## 问题
1. 3facets cannot be loaded.You can mark them as ignored to suppress this error notification.
![as_error_1](http://opkjcw4sd.bkt.clouddn.com/as_error_1.png)
**解决：**
![as_error_1_a](http://opkjcw4sd.bkt.clouddn.com/as_error_1_a.png)
[Stack Overflow参考链接](https://stackoverflow.com/questions/20560746/in-android-studio-cannot-load-2-facets-unknown-facet-typeandroid-and-android-gr)

2. Unable to access Android SDK add-on list
![as_error_2](http://opkjcw4sd.bkt.clouddn.com/as_error_2.png)
**解决：**
![as_error_2_a](http://opkjcw4sd.bkt.clouddn.com/as_error_2_a.png)
[Stack Overflow参考链接](http://discussions.youdaxue.com/t/androidstudio-unable-to-access-android-sdk-add-on-list/14417)
> mac: application->Android Studio->Contents->bin->idea.properties

3. Error:Please select Android SDK
![as_error_3](http://opkjcw4sd.bkt.clouddn.com/as_error_3.png)
**解决：**
![as_error_3_a](http://opkjcw4sd.bkt.clouddn.com/as_error_3_a.png)
[Stack Overflow参考链接](https://stackoverflow.com/questions/34353220/android-studio-please-select-android-sdk)
> 这个问题很莫名其妙，明明project是有SDK的本地路径的，但是一直运行不了，最后在build.gradle文件里随便敲了一个空格或换行，在sync(同步)下工程就可以了。

## 总结
一直以来Android是大家公认的最糟糕的开发环境，不但体现在它繁多的API上，还有支持的IDE，比如之前的eclipse，到现在的Android Studio(基于IntelliJ IDEA)，时长会出现一些开发者抓耳挠腮的问题，所以遇到问题我们可以记录和总结一些常见的错误，方便以后快速处理，节约开发时间，不要想着就卸载，重启电脑，虽然有时候很有效，哈哈~相比别的工具，它们都有着非常丰富的第三方插件，很方便我们集成开发测试，确实方便不少，例如编译打包的构建工具Ant、Maven、Gradle（官方支持），有着方便的接口提供开发者配置使用，这点必须给个大大的赞！



