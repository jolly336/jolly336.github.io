---
title: 性能优化之Matrix
date: 2019-10-14 11:14:01
tags: Android
categories: [Android, 性能优化]
---

本文讲解微信团队开源的 Android 性能优化检测工具-[Matrix](https://github.com/Tencent/matrix) 中 TracePlugin 原理，分析检测慢函数、卡顿的过程 。

<!-- more -->

## 原理

### Trace Plugin 代码框架

Trace Canary通过字节码插桩的方式在编译期预埋了方法进入、方法退出的埋点。运行期，慢函数检测、FPS检测、卡顿检测、启动检测使用这些埋点信息排查具体哪个函数导致的异常。

#### 编译期

函数埋点思路：

代理编译期间的任务 transformClassesWithDexTask，将全局 class 文件作为输入，利用 ASM 工具，修改字节码的方式，在编译期修改所有 class 文件中的函数字节码，对所有函数前后进行打点插桩。

<img src="matrix_tracecanary_arch.jpg" title="matrix-tracecanary总框架"  width="80%" height="50%" />

（图示1）matrix tracecanary总框架

<img src="matrix_compile_flow.jpg" title="matrix-编译期方法代码插装分析"  width="80%" height="50%" />

（图示2）matrix 编译期方法代码插装分析

代码插桩的整体流程如上图。在打包过程中，hook生成Dex的Task任务，添加方法插桩的逻辑。hook点是在Proguard之后，Class已经被混淆了，所以需要考虑类混淆的问题。

插桩代码逻辑大致分为三步：

* hook原有的Task，执行自己的MatrixTraceTransform，并在最后执行原逻辑
* 在方法插桩之前先要读取ClassMapping文件，获取混淆前方法、混淆后方法的映射关系并存储在MappingCollector中。
* 之后遍历所有Dir、Jar中的Class文件，实际代码执行的时候遍历了两次。

第一次遍历Class，获取所有待插桩的Method信息，并将信息输出到methodMap文件中；
第二次遍历Class，利用ASM执行Method插桩逻辑。

#### 运行期

基于编译期函数插装的逻辑，在运行期，检测到某个方法异常时，会上报一个 methodId，后端通过下图的 methodId 到 method name 的映射关系，追查到有问题的方法。

![matrix-methodmapping文件.jpg](matrix_methodmapping.jpg)

（图示3）matrix methodmapping文件

### API

* FrameTracer

计算掉帧率，生成 json 报告上报

* UIThreadMonitor 

ui 主线程监控 回调 doFrame(focusedActivityName, long frameCostMs) 每一帧总耗时，供 FrameTracer 来计算掉帧率（frameCostMs/frameIntervalMs 16.6667 + 1），

* Choregrapher.getInstance()

监控相邻两次 Vsync 事件通知的时间差

* LooperMonitor 

implements MessageQueue.IdleHandler 监控空闲事件

### FPS 帧率检测

clicfg_matrix_trace_fps_time_slice 表示检测总时长，由IDynamicConfig#getInt() 设置即可


### 慢函数

* 原理：

上部分讲述了编译器，会在每个方法的执行前后添加 `AppMethodBeat.i(int methodId)`和`AppMethodBeat.o(int methodId)`的方法调用，methodId 是在编译期生成的，在运行期是一个写死的常量。通过编译期的这个操作，就能感知到具体每个方法的进入、退出操作。

```java
   /**
     * hook method when it's called in.
     *
     * @param methodId
     */
    public static void i(int methodId) {

       ...

        if (Thread.currentThread().getId() == sMainThread.getId()) {
            if (assertIn) {
                android.util.Log.e(TAG, "ERROR!!! AppMethodBeat.i Recursive calls!!!");
                return;
            }
            assertIn = true;
            if (sIndex < Constants.BUFFER_SIZE) {
                mergeData(methodId, sIndex, true);
            } else {
                sIndex = -1;
            }
            ++sIndex;
            assertIn = false;
        }
    }

    /**
     * hook method when it's called out.
     *
     * @param methodId
     */
    public static void o(int methodId) {

        ...
        
        if (Thread.currentThread().getId() == sMainThread.getId()) {
            if (sIndex < Constants.BUFFER_SIZE) {
                mergeData(methodId, sIndex, false);
            } else {
                sIndex = -1;
            }
            ++sIndex;
        }
    }

```

* 检测过程

代码统计了当应用处于前台时，在主线程执行方法的进入、退出，这些信息存储在 AppMethodBeat 的 sBuffer「数组 long[100 * 10000]」 中。当主线程有疑似慢函数存在时，读取 Buffer 的数据，分析可能的慢函数，并上报 json 数据到后端，后端将 methodId 转换为具体的方法声明。

* 发生场景
    1. 掉帧场景
    2. 类似 ANR 长时间主线程阻塞 UI 绘制的场景

其中，掉帧场景，内部 FrameBeat 类实现了 Choreographer.FrameCallback，可以感知每一帧的绘制时间，通过前后两帧的时间差判断是否有慢函数发生；

主线程长时间阻塞 UI 绘制的场景，LazyScheduler 内有一个 HandlerThread，调用 LazyScheduler.setup 方法向 HandlerThread 的 MQ 发送一个延时5s的消息，若没有发生类似 ANR 的场景，在每一帧的 doFrame 回调中取消这个消息，同时发送一个新的延时 5s 的消息（正常情况下消息是得不到执行的），若发生类似 ANR 的情况，doFrame 没有被回调，这个延时 5s 的消息得到执行，将回调到 onTimeExpire 方法。

* 生成映射文件

目前生成函数堆栈映射文件在工程的 `app/build/outputs/mapping/debug/methodMapping.txt`，如下格式

* 分析结果

<img src="xrk_matrix_analyse_result.png" title="慢函数示例"  width="30%" height="50%" />

```java

123570,1,com.google.zxing.client.activity.MipcaActivityCapture onWindowFocusChanged (Z)V
...

* 第一个数字表示分配方法的Id，-1表示插桩为activity加入的onWindowFocusChanged方法。其他方法从1开始计数
* 第二个数字表示方法权限修饰符，常见的值为ACC_PUBLIC = 1; ACC_PRIVATE = 2;ACC_PROTECTED = 4; ACC_STATIC = 8等等。1即表示public方法
* 第三个参数标识类名 包名+类名；方法名 onWindowFocusChanged；参数及返回值类型Z表示参数为boolean类型，V表示返回值为空

```

### 卡顿

* 原因

主线程执行繁重的UI绘制、大量的计算或IO等耗时操作

* 方案

业内主要框架的主要思想是，监控主线程执行耗时，当超过阈值时，dump出当前主线程的执行堆栈，通过堆栈分析找到卡顿原因。

* 监控原理
    
    1. 依赖主线程 Looper，监控每次 dispatchMessage 的执行耗时。（BlockCanary）
    2. 依赖 Choreographer 模块，监控相邻两次 Vsync 事件通知的时间差。（ArgusAPM、LogMonitor）

* 生成结果

<img src="xrk_matrix_fps_result.png" title="帧率检测"  width="30%" height="50%" />

```java 
* scene 表示场景，使用 Activity + Fragment 类名作为唯一标志
* dropLevel 是表示掉帧情况-掉帧次数，衡量帧率掉帧的水平
* dropSum 总共掉帧的帧数
* fps 表示当前帧率

掉帧计算 final int droppedCount = (int) ((frameNanos - lastFrameNanos) / REFRESH_RATE_MS);

```

* 常见卡顿场景

    * 布局嵌套层次太深，可以使用 merge、viewStub、include 来优化
    * onDraw() 里面循环创建了大量临时变量，频繁触发 GC
    * 主线程等待优先级子线程问题（锁同步问题）
    * 主线程执行耗时操作，阻塞主线程执行（同步读写文件，DB 操作）

## Q&A

* 如何监听app 是否退到后台？

通过 Application.ActivityLifecycleCallbacks 接口向全局 app 注册监听，当有 onActivityStarted(activity) 时，主动标记前台为 true，当 onActivityStopped(activity) 时，先判断当前 activity 堆栈里是否有被 paused 的页面，如果没有，则表示已退到后台。

* FPS 检测时，如何判断当前是在 drawing？

* stack 慢函数堆栈如何分析？

可以参考 [Matrix issue #104 stack 如何分析](https://github.com/Tencent/matrix/issues/104)，或者 [hotfix/0.4.x sample-android](https://github.com/Tencent/matrix/blob/hotfix%2F0.4.x/samples/sample-android/app/src/main/java/sample/tencent/matrix/issue/IssuesListActivity.java)

## 参考

WeChat-Matrix

* [Tencent/matrix](https://github.com/Tencent/matrix)
* [Matrxi Release Versions](https://github.com/Tencent/matrix/releases)
* [Matrix for Android 文档](https://github.com/Tencent/matrix#matrix_android_cn)
* [Matrix Android TraceCanary
](https://github.com/Tencent/matrix/wiki/Matrix-Android-TraceCanary)
* [堆栈字段说明](https://github.com/Tencent/matrix/wiki/Matrix-Android--data-format)
* [慢函数的堆栈如何解读，现在是一堆数字](https://github.com/Tencent/matrix/issues/57)
* [matrix/home wiki](https://github.com/Tencent/matrix/wiki#%E5%B8%A7%E7%8E%87)
* [腾讯Bugly/广研Android卡顿监控系统](http://www.10tiao.com/html/330/201801/2653579565/1.html)
* [Github/QPM-聚美性能检测](https://github.com/ZhuoKeTeam/QPM?from=timeline)
* [(分析到位！) Matrix源码分析————Trace Canary](https://www.qingtingip.com/h_227199.html)

Matrix Stack

* [Matrix-Issue/stack 如何分析](https://github.com/Tencent/matrix/issues/104)
* [matrix的耗时方法日志解析-jar 工具](https://github.com/qhy2013/matrix_evil_method_stack_parser)
* 

其它 APM

* [Github/QPM](https://github.com/ZhuoKeTeam/QPM?from=timeline)

Choreographer

* [Android Choreographer 源码分析](https://www.jianshu.com/p/996bca12eb1d)

悬浮窗

* [FloatWindow 悬浮窗](https://github.com/duqian291902259/Android-FloatWindow)

进程优先级

* [Android adb 查看进程优先级](https://blog.csdn.net/QQxiaoqiang1573/article/details/79988752)
* [/proc/stat解析](http://gityuan.com/2017/08/12/proc_stat/)
* [花式读取Android CPU使用率](http://www.voidcn.com/article/p-oesjkqaq-bnv.html)
* [Android 查看进程ID（PID）比较进程优先级](https://blog.csdn.net/Zz110753/article/details/70048811)

Android是如何管理内存的？

* [管理应用的内存](http://hukai.me/android-training-course-in-chinese/performance/memory.html)
* [Android漫游记(1)---内存映射镜像(memory maps)](https://blog.csdn.net/lifeshow/article/details/29174457)
* [Android性能优化之内存篇](http://hukai.me/android-performance-memory/)

Android Davik vs Java JVM？

* [一篇文章告诉你Dalvik 和JVM的区别](https://juejin.im/post/59b7fa8cf265da066d3323bb)
* [Java JVM 中 堆，栈，方法区 详解](https://blog.csdn.net/zhangqiluGrubby/article/details/59110906)

性能实践

* [Andriod性能优化之列表卡顿——以“简书”APP为例](https://www.jianshu.com/p/336362b23c30)
* [Activity界面显示全解析](https://blog.csdn.net/zhaokaiqiang1992/article/details/49681321)
* [Android官方翻译-Android性能优化](http://hukai.me/android-training-course-in-chinese/performance/memory.html)
* [Android UI性能优化 检测应用中的UI卡顿](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650822205&idx=1&sn=6b8e78bc1d71eb79a199667cf132acf7&chksm=80b782a3b7c00bb5c12437556fca68136c75409855e9252e395b545621319edf23959942b67c&mpshare=1)
* [Android应用性能优化实践](https://mp.weixin.qq.com/s?__biz=MzA4MzEwOTkyMQ==&mid=405927192&idx=1&sn=71db45156530a20136b88851df3ef3e1&scene=4#wechat_redirect)
* [性能优化汇总](https://mp.weixin.qq.com/s?__biz=MzAxNjI3MDkzOQ==&mid=203692693&idx=1&sn=ecf0166abdb22d28463d4facd5de473b&scene=23&srcid=0215Ia7vlFqGw2lWtgZbcWBy#rd)

Gradle

* [深入理解 Android（一）：Gradle 详解](https://www.infoq.cn/article/android-in-depth-gradle/?utm_source=infoq&utm_campaign=user_page&utm_medium=link)

