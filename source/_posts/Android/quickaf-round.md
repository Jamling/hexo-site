---
title: Android中实现圆角图片的几种姿势
date: 2017-05-18 10:00:00
tags: [Android, QuickAF]
category: [Android]
toc: true
---
 Android中实现圆角图片有多种姿势，不知你解锁了几种？
<!-- more -->

## 方法一：setXfermode法
此种方式就是再new一个相同尺寸的bitmap，然后使用paint.setXfermode(new PorterDuffXfermode(Mode.SRC_IN));先画圆角矩形，再画原始bitmap，然后就得到了一个圆角的bitmap了。
```java
public static Bitmap getRoundedCornerBitmap(Bitmap bitmap, float roundPx) {

    Bitmap output = Bitmap.createBitmap(bitmap.getWidth(), bitmap.getHeight(), Config.ARGB_8888);
    Canvas canvas = new Canvas(output);

    final int color = 0xff424242;
    final Paint paint = new Paint();
    final Rect rect = new Rect(0, 0, bitmap.getWidth(), bitmap.getHeight());
    final RectF rectF = new RectF(rect);

    paint.setAntiAlias(true);
    canvas.drawARGB(0, 0, 0, 0);
    paint.setColor(color);
    canvas.drawRoundRect(rectF, roundPx, roundPx, paint);

    paint.setXfermode(new PorterDuffXfermode(Mode.SRC_IN));
    canvas.drawBitmap(bitmap, rect, rect, paint);

    return output;
}
```
点评：
早期用得较多，占用bitmap双倍内存。

## 方法二：使用BitmapShader
此种方式是先将bitmap生成BitmapShader，然后将其绘制到canvas中, 部分关键代码如下，完整代码请参考[QuickAF]中的RoundImageView

```java
bitmapShader = new BitmapShader(bitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
paint.setShader(bitmapShader);
```

```java
@Override
public void draw(Canvas canvas) {
    Rect bounds = getBounds();
    canvas.drawRoundRect(fillRect, radius, radius, paint);
    if (mBorderWidth > 0) {
        if (mIsCircle) {
            canvas.drawCircle(bounds.width() / 2, bounds.height() / 2, radius, strokePaint);
        }
        else {
            canvas.drawRoundRect(fillRect, radius, radius, strokePaint);
        }
    }
}
```
点评：
占用内存较大，实现有点小复杂。

## 方法三：图片加载库
目前github上有许多流行的图片加载库，基于上都附带圆角图片功能，只需要稍微配置一下，即可轻松的实现想要的效果。其实在底层，无非也是使用上面的两种方式。比如[Android-Universal-Image-Loader](Android-Universal-Image-Loader) 早期的RoundedBitmapDisplayer使用setXfermode来实现，后来使用BitmapShader实现。
```java
DisplayImageOptions options = new DisplayImageOptions.Builder()
        .displayer(new RoundedBitmapDisplayer()) // display rounded bitmap
        .build();
```
再以比较另类的[fresco](https://github.com/facebook/fresco)为例，虽然底层是以C实现，不过在圆角处理上，仍然还是在Java层实现，用的方式还是BitmapShader。不过对于非bitmap的圆角实现，fresco是用Paint直接画的。附上fresco配置。
```xml
<com.facebook.drawee.view.SimpleDraweeView
  android:id="@+id/my_image_view"
  android:layout_width="20dp"
  android:layout_height="20dp"
  fresco:fadeDuration="300"
  fresco:actualImageScaleType="focusCrop"
  fresco:placeholderImage="@color/wait_color"
  fresco:placeholderImageScaleType="fitCenter"
  fresco:failureImage="@drawable/error"
  fresco:failureImageScaleType="centerInside"
  fresco:retryImage="@drawable/retrying"
  fresco:retryImageScaleType="centerCrop"
  fresco:progressBarImage="@drawable/progress_bar"
  fresco:progressBarImageScaleType="centerInside"
  fresco:progressBarAutoRotateInterval="1000"
  fresco:backgroundImage="@color/blue"
  fresco:overlayImage="@drawable/watermark"
  fresco:pressedStateOverlayImage="@color/red"
  fresco:roundAsCircle="false"
  fresco:roundedCornerRadius="1dp"
  fresco:roundTopLeft="true"
  fresco:roundTopRight="false"
  fresco:roundBottomLeft="false"
  fresco:roundBottomRight="true"
  fresco:roundWithOverlayColor="@color/corner_color"
  fresco:roundingBorderWidth="2dp"
  fresco:roundingBorderColor="@color/border_color"
/>
```
点评：
由框架实现，使用简单，稳定。

## 方法四：遮罩
此种方式还是使用setXfermode，不过与方法一不同的是：不对图片作任何更改，只在圆角之外再画一层与背景颜色相同的四个角来遮挡，在视觉上造成圆角图片的效果。关键代码如下：
```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    if (src != null && dst != null) {
        int w = getMeasuredWidth(), h = getMeasuredHeight();
        int sc = canvas.saveLayer(0, 0, w, h, null,
            Canvas.MATRIX_SAVE_FLAG | Canvas.CLIP_SAVE_FLAG | Canvas.HAS_ALPHA_LAYER_SAVE_FLAG
                | Canvas.FULL_COLOR_LAYER_SAVE_FLAG | Canvas.CLIP_TO_LAYER_SAVE_FLAG);
        canvas.drawBitmap(dst, 0, 0, paint); // 圆角矩形
        paint.setXfermode(mode); // new PorterDuffXfermode(PorterDuff.Mode.SRC_OUT);
        canvas.drawBitmap(src, 0, 0, paint); // 长方形
        paint.setXfermode(null);
        canvas.restoreToCount(sc);
    }
}
```
详细代码请参考[QuickAF]中的RoundMaskView

使用这种方式，圆角化的对象不限于ImageView，还可以是任意的layout哦，比如下面的示例
```xml
<FrameLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ImageView
            android:layout_width="match_parent"
            android:layout_height="150dp"
            android:src="@color/colorAccent"/>

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_gravity="bottom"
            android:background="@color/black_alpha_50"
            android:padding="12dp"
            android:text="I am text"/>
    </LinearLayout>

    <cn.ieclipse.af.view.RoundMaskView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:radius="10dp"
        app:af_borderColor="@color/white"
        app:af_borderWidth="1dp"/>
</FrameLayout>
```
配合FrameLayout，将LinearLayout实现了圆角，在视觉效果上，ImageView左上和右上圆角，TextView左下和右下圆角。

点评：
具有一定的局限性，不过不限于图片，所有的Layout都可以在视觉上实现圆角。

## 关于

[QuickAF]是一个Android平台上的app快速开发框架，欢迎读者在github star或fork。本人写作水平有限，欢迎广大读者指正，如有问题，可与我直接联系或在我的官方博客中给出评论。

## 参考
QuickAF: https://github.com/Jamling/QuickAF

[QuickAF]: https://github.com/Jamling/QuickAF
