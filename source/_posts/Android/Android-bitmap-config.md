---
title: Android bitmap config你理解对了吗？
date: 2017-06-14 17:50:00
category: [Android]
tags: [Android]
toc: true
description: Bitmap config ALPHA_8/ARGB_4444/ARGB_8888/RGB_565理解
---

在写本文之前，我还以为自己对android的bitmap很熟悉，直到自己亲手写代码实践，才发现自己错了很多年。真是汗颜啊！

<!-- more -->

## Bitmap Config
首先，根据Android API 25的文档简要说明一下Android的Bitmap.Config以下4个选项

- ALPHA_8: 每个像素占用1字节（8位），存储的是透明度信息。
- ARGB_4444: 每个像素占用2字节（4+4+4+4＝16位），ARGB分别占用4位，支持alpha通道。
 **注：从API 13开始不推荐使用，在android 4.4上面，设置的ARGB_4444会被系统使用ARGB_8888替换**
- ARGB_8888: 默认的选项，每像素占用4字节，ARGB分别占8位，支持1600万种颜色，质量最高，当然内存占用也高。
- RGB_565: 每像素占用2字节，RGB分别占5，6，5位。支持65535种颜色，不支持alpha。

| bitmap.config        | ALPHA_8          | ARGB_4444  | ARGB_8888| RGB_565|
| ------------- |:-------------:| -----:|----:|----:|
| bytes/pixel      | 1 byte | 2 byte | 4 byte | 2 byte|
| alpha channel      | 8 bit| 4 bit | 8 bit | not support|

## PNG 格式
其次，简要说一下png格式

- png 8: 支持不透明，索引色透明，alpha透明，最大支持256种颜色
- png 24: 不支持透明，支持1600万种颜色
- png 32: 支持透明，其它同png 24，支持1600万种颜色

## bitmap内存占用计算
第三，简要说一下bitmap占用的内存

Android中bitmap的内存占用是跟图片的尺寸（高和宽）相关。一张图片的内存占用大致的计算公式如下：

**占用内存 = 图像像素总和（width x height）再 x 每像素(bitmap config)占用的字节数**

*以下是通过代码准确计算*
```java
    public static int getSizeInBytes(@Nullable Bitmap bitmap) {
        if (bitmap == null) {
            return 0;
        }

        // There's a known issue in KitKat where getAllocationByteCount() can throw an NPE. This was
        // apparently fixed in MR1: http://bit.ly/1IvdRpd. So we do a version check here, and
        // catch any potential NPEs just to be safe.
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.KITKAT) {
            try {
                return bitmap.getAllocationByteCount();
            } catch (NullPointerException npe) {
                // Swallow exception and try fallbacks.
            }
        }

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB_MR1) {
            return bitmap.getByteCount();
        }

        // Estimate for earlier platforms.
        return bitmap.getWidth() * bitmap.getRowBytes();
    }
```

以一张1024*1024的图片为例，使用ARGB_8888，占用的内存为1024*1024*4=4M。像现在的手机摄像头动不动就是上千万像素，拍出来的照片如果按默认的ARGB_8888 config加载，则至少是几十M的内存占用。

Android的图片资源主要分两部分：

1. 一种是apk中自带的，多为png格式，由系统加载，支持缩放，代码中通过R.xxx引用，decode时使用的是默认的ARGB_8888选项，图像质量高；
2. 另一种是网络图片或本地图片，多为jpg格式，加载时一般使用第三方的图片加载库，为节省内存decode时多为RGB_565选项。

平时都是这么用，也没发现问题，优化内存占用时，一般也是从图片的尺寸方面入手。不过最近优化一个跟图片相关的功能，在图片尺寸无法缩放的条件下，只能通过更改bitmap config来降低内存的占用。然后意外的发现，导致颠覆了我的三观。为此我特地写了一个测试sample，代码详见[github](https://github.com/Jamling/BitmapConfig)，特地创建了一张背景色透明，图片内容为A（黑色50%透明度）R（红色）G（绿色）B（蓝色）图片，然后分别导出为：png8（alpha透明）、png24（不透明）、png32和jpeg（不透明）格式的图，分别使用ALPHA_8, ARGB_4444, ARGB_8888, RGB_565四种config加载图片，得到的实际结果如下（假设图像总像素为X）。

## 实践结果

### 运行截图
![screenshot][1]

### 结果统计

| bitmap.config | ALPHA_8 | ARGB_4444  | ARGB_8888| RGB_565|
| :-------------: |:-------------:|:-----:|:----:|:----:|
|![][2]png8   |**4**~~`1`~~ X|2 X|4 X|**4**~~`2`~~ X A ~~`不`~~ 透明|
|![][3]png24 |**4**~~`1`~~ X|2 X|4 X|**2** X|
|![][4]png32 |**4**~~`1`~~ X|2 X|4 X|**4**~~`2`~~ X A ~~`不`~~ 透明|
|![][5]jpeg    |**4**~~`1`~~ X|2 X|4 X|**2** X|

**请注意表格中带删除线的部分**

1. ALPHA_8：config占用的内存竟然和ARGB_8888一样，不是说每个像素占用1字节的么？
2. RGB_565：在png8和png32中，图片中的A都保持了50%的透明度，而且占用的内存也和ARGB_8888一样，不是说RGB_565不包含alpha么？不是说占用的内存是ARGB_8888的一半么？
3. ARGB_4444：在android 6.0上面，png8和png32看不见（全透明），png24和jpeg显示为一块黑色区域，在android 4.2上则显示正常。

带着上面的疑问，在网上进行了相关的搜索，也没有找到答案。好吧，我是懵了，不知道各位看客如何？附上github上的示例工程：https://github.com/Jamling/BitmapConfig

[1]: https://raw.githubusercontent.com/Jamling/BitmapConfig/master/screenshot.jpg
[2]: https://raw.githubusercontent.com/Jamling/BitmapConfig/master/app/src/main/res/mipmap-xhdpi/png8.png
[3]: https://raw.githubusercontent.com/Jamling/BitmapConfig/master/app/src/main/res/mipmap-xhdpi/png24.png
[4]: https://raw.githubusercontent.com/Jamling/BitmapConfig/master/app/src/main/res/mipmap-xhdpi/png32.png
[5]: https://raw.githubusercontent.com/Jamling/BitmapConfig/master/app/src/main/res/mipmap-xhdpi/jpeg_80.jpg
