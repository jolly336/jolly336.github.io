---
title: Glide4.x在Android 8.0手机上加载图片崩溃问题
date: 2018-07-15 17:44:08
tags: Android
categories: Android
---

最近自己负责的项目在发布了新版本后，Fabric 收到了一些关于 ImageView onDraw 的问题，而且都是在 Android Oreo 8.0 (API 26)上出现的，分析异常日志可以清晰发现都是 bitmap 解码出错，关键日志异常：**java.lang.IllegalStateException: Software rendering doesn't support hardware bitmaps**。同时新版本向日葵把图片加载库[Glide](https://github.com/bumptech/glide)从 3.7.1 版本更新到了 4.7.1，App 内部图片展示和操作代码都没有做更改。

<img src="http://wanghaoxun.com/glide_logo.png">

<!-- more -->

## 分析

既然 App 代码没有做更改，那么问题就很有可能出在了 Glide 加载器和 Android 8.0系统。所以有必要了解这2者在新版本有哪些更新。

### 不同Android版本时Bitmap内存模型

Bitmap内存优化[^Bitmap内存优化]
[^Bitmap内存优化]: https://juejin.im/entry/5ad0213f6fb9a028df2306cd

我们知道Android系统中，一个进程的内存可以简单分为Java内存和native内存两部分，而Bitmap对象占用的内存，有Bitmap对象内存和像素数据内存两部分，在不同的Android系统版本中，其所存放的位置也有变化。[Android Developers](https://link.juejin.im/?target=https%3A%2F%2Flink.jianshu.com%3Ft%3Dhttps%253A%252F%252Fdeveloper.android.com%252Ftopic%252Fperformance%252Fgraphics%252Fmanage-memory.html)上列举了从API 8 到API 26之间的分配方式：


| API级别           | API 10- | API 11 ~ API 25 | API 26+|
| :---              | :--: | :--: |   :---    |
| Bitmap对象存放    | Java heap | Java heap | Java heap   |
| 像素(pixel data)数据存放    | native heap | Java heap    | native heap    |

最新的Android 8.0之后，谷歌又把像素存放的位置，从java heap该回到了native heap，API 11的那次改动，是源于native的内存释放不及时，会导致OOM，因此才将像素数据保存到java heap，从而保证Bitmap对象释放时，能够同时把像素数据内存也释放掉。至于为什么在8.0上改变了Bitmap像素数据的存放方式，我猜想和8.0中的GC算法调整有关系。GC算法的优化，使得Bitmap占用的大内存区域，在GC后也能够比较快速的回收、压缩，重新利用。

### Bitmap.Config.HARDWARE

Android 8.0 Bitmap.Config配置枚举新增了HARDWARE[^Bitmap.Config]，来配置图片像素数据存储在GPU里，去掉了内存存储的相同一份数据，来节省内存开销，加速UI渲染。如果要使用hardware，在bitmap decode时，设置`option.inPreferredConfig = Bitmap.Config.HARDWARE;`即可。如果要获取存储到显存上的bitmap，需要使用`PixelCopy`类把bitmap数据拷贝到内存中。PixelCopy类有几个静态方法，传递一个surface或window，指定一个bitmap，就可以把surface或window上的bitmap拷贝到你指定的bitmap上了。

[^Bitmap.Config]: https://developer.android.com/reference/android/graphics/Bitmap.Config

### Glide 4.x

Glide4.+版本开始，适配了8.0系统的变化，解码默认是禁用hardware，如果外面设置了其它值，会打开hardware开关使用系统新特性。

## 解决

由于应用内对于图片显示没有严格的要求，所以使用`RGB_565`每像素4个字节加载图片来节省内存。在**Android 8.0**系统官方对bitmap的存储又进行了调整，图片像素(pixel data)数据存放到了**native heap**，如果图片只用作展示，不进行缩放、matrix处理的时候，图片数据只会在GPU存储一份用来渲染UI，否则图片需要在运存上也存在一份数据，来进行后续其它操作。

Glide全局配置类代码（线上）：

```java
@GlideModule
public class GlideService extends AppGlideModule {

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        ViewTarget.setTagId(R.id.glide_tag_id);
        builder.setDefaultRequestOptions(RequestOptions.formatOf(DecodeFormat.PREFER_RGB_565));
    }
}

```

Glide关于DecodeFormat文档解释

```java

public enum DecodeFormat {

  PREFER_ARGB_8888,

  /**
   * Identical to {@link #PREFER_ARGB_8888} but prevents Glide from using {@link
   * android.graphics.Bitmap.Config#HARDWARE} on Android O+.
   *
   * @deprecated If you must disable hardware bitmaps, set
   * {@link com.bumptech.glide.load.resource.bitmap.Downsampler#ALLOW_HARDWARE_CONFIG} to false
   * instead.
   */
  @Deprecated
  PREFER_ARGB_8888_DISALLOW_HARDWARE,

  /**
   * Bitmaps decoded from image formats that support and/or use alpha (some types of PNGs, GIFs etc)
   * should return {@link android.graphics.Bitmap.Config#ARGB_8888} for
   * {@link android.graphics.Bitmap#getConfig()}. Bitmaps decoded from formats that don't support or
   * use alpha should return {@link android.graphics.Bitmap.Config#RGB_565} for
   * {@link android.graphics.Bitmap#getConfig()}.
   *
   * <p>On Android O+, this format will will use ARGB_8888 only when it's not possible to use
   * {@link android.graphics.Bitmap.Config#HARDWARE}.
   */
  PREFER_RGB_565;

  /**
   * The default value for DecodeFormat.
   */
  public static final DecodeFormat DEFAULT = PREFER_ARGB_8888_DISALLOW_HARDWARE;
}
```

可以看到Glide 4.7.1上默认使用了`PREFER_ARGB_8888_DISALLOW_HARDWARE`，来禁用硬件位图，阻止Glide使用Android O+ 上`android.graphics.Bitmap.Config＃HARDWARE`。
而应用设置了`PREFER_RGB_565`，从而打开了硬件位图存储的开关，只要应用内有读取变换`bitmap`的操作都有奔溃的问题。基于这个问题，目前做了disable hardware配置，使用Glide默认配置来解决图片解码问题。后续计划分出来只加载和变化的图片，使用不用的配置策略，兼容系统新API。

PS：为单个请求设置禁止hardware，可进行如下设置。

```java
RequestOptions options = new RequestOptions().disallowHardwareConfig();
Glide.with(fragment)
  .load(url)
  .apply(options)
  .into(imageView);
```

## 总结

* 新版本Glide 4.x适配了Android O+系统，做了bitmap加载优化
* Android O+ 系统修改了图片像素数据存储位置，从java heap改到native heap
* Android O+ 系统新增了`Bitmap.Config.HARDWARE`选项，hardware标志不是关于质量，而是关于像素存储位置（graphics memory），这样看来hardware更像个黑客
* 如果使用了hardware功能，像从java层面获取bitmap byte数据，可以使用`PixelCopy`类来赋值获取，友情提示：从surface拷贝数据到内存很缓慢哦 :)
* 如果你还想知道更多关于hardware的解释，请移步这里[Hardware Bitmaps](https://bumptech.github.io/glide/doc/hardwarebitmaps.html)

## 更多关于Glide

本人创建了一个github repo，包含了Glide3.x和4.x版本的入门和高级使用演示。如果你想知道更多关于Glide的使用，可以参考[GlideTest](https://github.com/haoxunwang/GlideTest)项目。备注：master分支是3.7.1版本演示，4.7.1分支是4.7.1版本演示。

## 参考

* [Bitmap.Config SDK Docs](https://developer.android.com/reference/android/graphics/Bitmap.Config)
* [Android中Bitmap内存优化](https://juejin.im/entry/5ad0213f6fb9a028df2306cd)
* [Bitmap.Config.HARDWARE vs Bitmap.Config.RGB_565](https://stackoverflow.com/questions/45511017/bitmap-config-hardware-vs-bitmap-config-rgb-565)
* [Glide高级演示项目-GlideTest](https://github.com/haoxunwang/GlideTest)









