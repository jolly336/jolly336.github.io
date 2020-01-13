
---
title: Android图片压缩
date: 2017-02-23 21:49:15
tags: Android
categories: Android
---

> 本文讲解了Android Compress image的方式，主要从**图片的存在形式**、**检测图片大小的方法**、**压缩方式**、**数据对比**等方面阐述了压缩方案。

<!-- more -->


## 图片的存在形式
1. **文件形式**（即以二进制形式存在于硬盘上）
2. **流的形式** （即以二进制形式存在于内存中）
3. **Bitmap形式**

>  **区别：**文件形式和流的形式对图片体积大小没有影响，也就是说，如果图片在手机SD卡上的大小是100KB，那么通过流的形式读到内存中，也是占用100KB的大小，注意是流的形式，不是bitmap的形式；当图片以Bitmap的形式存在时，其占用的内存会瞬间变大，曾经试过把500KB左右的图片加载到内存中，以Bitmap形式存在时，内存占用近10MB，当然这个增大的倍数不是固定的。

## 检测图片大小的方法
1. **文件形式** file.length( )
2. **流的形式** 将图片文件读到内存输入流中，看它的byte数
3. **Bitmap** bitmap.getByteCount( )

## 压缩方式

### 质量压缩

`bitmap.compress(CompressFormat,quality,outputstream)`
1. 第一个参数是个枚举类CompressFormat，主要有JPEG，PNG，WEBP三种类型，一般使用JPEG；
2. 第二个参数是压缩质量，取值0-100之间，100代表质量最高，0代表最低；
3. 第三个参数是输出流，可把图片字节流保存成指定文件。
使用实例：

```java
    public static void compressBmpToFile(Bitmap bmp, File file) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        int options = 80;// 压缩质量从80开始 
        bmp.compress(Bitmap.CompressFormat.JPEG, options, baos);
        while (baos.toByteArray().length / 1024 > 100) { // 如果大于100KB时，压缩质量-10；  
            baos.reset();
            options -= 10;
            bmp.compress(Bitmap.CompressFormat.JPEG, options, baos);
        }
        try {
            FileOutputStream fos = new FileOutputStream(file);
            fos.write(baos.toByteArray());
            fos.flush();
            fos.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
**方法说明：**该方法是压缩图片的质量，*但并不会减少图片的像素*，比如，图片是800KB 1008x756像素的，经该方法压缩后，File形式的图片是在100KB左右，以方便上传服务器，但是BitmapFactory.decodeFile( )到内存中变成bitmap时，它的像素值仍然是1008x756，计算图片像素的方法是bitmap.getWidth( )和bitmap.getHeight( )，图片是由像素组成的，每个像素又包含什么呢？熟悉PS的人都知道，图片是有色相，明度和饱和度构成的。

该方法也是Google官方文档解释说，它会让图片重新构造，但是有可能图片的位深（既色深）和每个像素的透明度会变化，JPEG only supports opaque(不透明)，也就是说jpeg格式压缩后，原来图片中透明的元素将会消失，所以这种格式很可能造成失真。 

既然它是改变了图片的显示质量，达到了对File形式的图片进行压缩，图片的像素没有改变的话，重新读取经过压缩file为bitmap时，它占用的内存并不会减少。

因为：bitmap.getByteCount( )是计算它的像素所占用的内存，请看官方解释：Returns the number of bytes used to store this bitmap's pixels.

### 像素压缩

**特点：**通过设置采样率，减少图片的像素，即图片从File形式变成Bitmap形式

先看一个方法: 该方法是对内存中的Bitmap进行质量上的压缩, 由上面的理论可以得出该方法是无效的, 而且也是没有必要的,因为你已经将它读到内存中了,再压缩多此一举, 尽管在获取系统相册图片时,某些手机会直接返回一个Bitmap,但是这种情况下, 返回的Bitmap都是经过压缩的, 它不可能直接返回一个原声的Bitmap形式的图片, 后果可想而知。

```java
    private Bitmap compressBmpFromBmp(Bitmap image) {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        int options = 100;
        image.compress(Bitmap.CompressFormat.JPEG, 100, baos);
        while (baos.toByteArray().length / 1024 > 100) {
            baos.reset();
            options -= 10;
            image.compress(Bitmap.CompressFormat.JPEG, options, baos);
        }
        ByteArrayInputStream isBm = new ByteArrayInputStream(baos.toByteArray());
        Bitmap bitmap = BitmapFactory.decodeStream(isBm, null, null);
        return bitmap;
    }

```

再看一个方法:

```java
    private Bitmap compressImageFromFile(String srcPath) {
        BitmapFactory.Options newOpts = new BitmapFactory.Options();
        newOpts.inJustDecodeBounds = true;//只读边,不读内容  
        Bitmap bitmap = BitmapFactory.decodeFile(srcPath, newOpts);

        newOpts.inJustDecodeBounds = false;
        int w = newOpts.outWidth;
        int h = newOpts.outHeight;
        float hh = 800f;//  
        float ww = 480f;//  
        int be = 1;
        if (w > h && w > ww) {
            be = (int) (newOpts.outWidth / ww);
        } else if (w < h && h > hh) {
            be = (int) (newOpts.outHeight / hh);
        }
        if (be <= 0)
            be = 1;
        newOpts.inSampleSize = be;//设置采样率  

        newOpts.inPreferredConfig = Config.ARGB_8888;//该模式是默认的,可不设  
        newOpts.inPurgeable = true;// 同时设置才会有效  
        newOpts.inInputShareable = true;//。当系统内存不够时候图片自动被回收  

        bitmap = BitmapFactory.decodeFile(srcPath, newOpts);
        // return compressBmpFromBmp(bitmap);//原来的方法调用了这个方法企图进行二次压缩  
        // 其实是无效的,大家尽管尝试  
        return bitmap;
    }

```

###  压缩终极方案，修改libjpeg图像库，使用c库来保存bitmap到file

* [[Android算法] android图片压缩终极解决方案](http://www.see-source.com/blog/300000022/684.html)
* [为什么Android的图片质量会比iPhone的差？](http://www.cnblogs.com/MaxIE/p/3951294.html)
* [github demo](https://github.com/bither/bither-android-lib)

> 以上是参考文章和修改libjpeg图像库的demo，本人亲自试了下，没有像介绍的那样可靠，也有可能不同ROM是否已经修复了此bug？暂时还不得而知，需要在此验证。

### 数据对比

**默认使用的位图配置是ARGB_8888**，图片加载到内存大小3456x4608x4字节约等于60.75MB，直接加载会OOM，需要进行长宽压缩 + 像素质量压缩。
> 原图大小 5.21MB 3456x4608px，缩小4时，quality = 80，864x1152px

|   origin  |   jpegTrue    |   jpegFalse   |   sampleSize  |   quality | 
|   :--:        |   :--:            |   :--:            |   :--:                |   :--:        | 
|145.75KB|      -           |   -               |   4                   | 80            |       
|               |   141.63KB |  -               |   4                   | 80            |  
|               |                   |  144.79KB  |  4                   | 80            |

| origin            |    jpegTrue |     jpegFalse  | sampleSize | quality | 
|   :--:            |   :--:            |       :--:         |  :--:                |   :--:      | 
| 706.34KB  |   -               |       -           |   4                   | 100        |      
|                  |    629.34KB |          -           |   4                   | 100        |  
|                  |                     |      661.51KB |  4                   | 100        |


| origin            |    jpegTrue |     jpegFalse  | sampleSize | quality   | 
| :--:              | :--:              |       :--:         |  :--:                |   :--:        | 
| 8.71MB        |   -               |       -           |   1                   | 100       |       
|                  |    7.13MB  |       -           |   1                   | 100       |  
|                  |                    |   7.62MB  |   1                   | 100       |

| origin            |    jpegTrue |     jpegFalse  | sampleSize | quality  | 
| :--:              |       :--:          |         :--:         |  :--:                |   :--:      | 
| 1.43MB        |       -           |       -           |   1                   | 80          |     
|                  |    1.34MB  |       -           |   1                   | 80          |  
|                  |                    |   1.41MB  |   1                   | 80          |


-------

> Transform a bitmap into an input stream 

```java
    // Your Bitmap.
    Bitmap bitmap = XXX;  

    int byteSize = bitmap.getRowBytes() * bitmap.getHeight();
    //int byteSize = bitmap.getByteCount();
    ByteBuffer byteBuffer = ByteBuffer.allocate(byteSize);
    bitmap.copyPixelsToBuffer(byteBuffer);  

    // Get the byteArray.
    byte[] byteArray = byteBuffer.array();

    // Get the ByteArrayInputStream.
    ByteArrayInputStream bs = new ByteArrayInputStream(byteArray);
```


