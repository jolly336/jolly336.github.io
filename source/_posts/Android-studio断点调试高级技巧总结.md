---
title: Android studio断点调试高级技巧总结
date: 2017-03-06 10:47:59
tags: Android
categories: Android
---

> 本博客来自[Android studio断点调试高级技巧总结](http://www.jianshu.com/p/984e101fd73e)一文。

本文将总结几种在开发中比较重要的断点调试方法：**条件断点**、**异常断点**、**和日志断点**。

<!-- more -->

## 条件断点
假设我们有一个循环体，但是我们只想调试i==10的情况，此时可以采用条件断点来完成。

```java
int j = 0;
for (int i = 0; i < 1000; i++) {
    j++;
}
```
如果我们想观察i==500时，程序的执行情况，可以这样操作。首先在j++所在行设置一个断点

![条件断点_1](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_1.jpg)
此时 在断点上单击右键，输入i==500

![条件断点_2](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_2.jpg)

设置完毕啦，启动断点调试后，你就会发现，第一次进入端点时，i==500了。

![条件断点_3](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_3.jpg)

## 异常断点

这个功能个人感觉还是挺有用处的。先说适用场景，假设我们的App出现崩溃，但是因为一些原因log日志不好查看或者查看不到，我们知道可能是NullPointerException异常了，但是定位不到异常在哪里，此时我们可能会将自己怀疑可能出现空指针异常的地方都打上断点，这样做是非常低效的，有了异常断点功能，一步就可以搞定。
为了说明问题，我在37行人为的引入了空指针异常。

![异常断点_1](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_4.jpg)

打开Run->View BreakPoints或者直接ctrl+shift+F8,打开异常设置面板。

![异常断点_2](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_5.jpg)

![异常断点_3](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_6.jpg)

按上图操作成功的加入了空指针异常断点监控，此时直接Debug模式调试。

![异常断点_4](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_7.jpg)

程序直接在37行停下了，而且会打印出空指针的日志。注意37行我们并没有提前设置断点。是不是很神奇，直接定位到了bug。

## 日志断点
如果我们想要在断点调试的时候添加log日志，我们最常用的方式是直接加入一行log，重新编译，观察结果后，在去掉log，在重新编译，此时我们可以使用日志断点的功能。

![日志断点_1](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_8.jpg)

设置方法，首先设置普通断点，在断点上单击右键，将suspand属性设为false，在出现的对话框中勾选上Log evaluated expression，下面我们在第一个断点上想观察一下i的值，在第二个断点上打印下当前线程的名字。设置如下

![日志断点_2](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_9.jpg)

![日志断点_3](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_9.jpg)

![日志断点_4](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_10.jpg)

运行一把，看看效果：

![日志断点_5](http://omdtn071e.bkt.clouddn.com/2017-3-6_%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95_11.jpg)

android的断点调试功能就介绍到这里。

