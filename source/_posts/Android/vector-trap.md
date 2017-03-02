---
title: VectorDrawble踩过的坑
date: 2017-03-02 21:00:00
category: [Android]
tags: [Android]
toc: true
description:
---

## 前言
Android 从5.0（代号L）开始支持矢量图，心想，这个好哇，drawable/mipmap图片资源终于可以瘦身了！后来还特地翻墙在YouTube上观看了Google IO大会上在Android Studio中创建Vector drawable的视频。本着匆用新版本的原则。一直对它处于了解阶段。并未在项目中实践。前一阵子，实践了一把，结果差点被坑死。下面列举一下本人亲自踩过的坑。

<!-- more -->

## fillColor无法在低于API21的版本使用引用颜色
在项目中，使用定义了`colorPrimary`作为主题首选色，比如蓝色。现引用一个Vector，要求颜色与主题首选色一致。第一时间想到的就是，在Vector中作如下定义：

```xml
<path
        android:fillColor="@color/colorPrimary"
        android:pathData="..."/>
```
然而在5.0以下的设备中运行时发现，图标为黑色，`1`fillColo`设置无效。Google之后，说应该使用Tint来对vector着色，好吧，于是写了一个util方法进行处理。
``` java
public static Drawable tintDrawable(Drawable drawable, int color) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        drawable.setTint(color);
        return drawable;
    }
    else {
        final Drawable wrappedDrawable = DrawableCompat.wrap(drawable);
        DrawableCompat.setTint(wrappedDrawable, color);
        return wrappedDrawable;
    }
}
```

## TextView compound drawable无法着色
我有一个带Compound Drawable的`TextView`
``` xml
<TextView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:drawableLeft="@drawable/ic_back"
    android:gravity="center"
    android:minHeight="40dp"
    android:minWidth="40dp"
    android:paddingRight="5dp"
    android:text="@string/common_back"
    android:textColor="@color/white"
    android:textSize="16sp">
</TextView>
```
ic_back是VectorDrawble，我想设置图片颜色与文字一致，都为白色。本以为加上`android:drawableTint="#FFF"`就行了，然而发现并不起作用。于是尝试将`TextView`换成`AppcompatTextView`，竟然还是不起作用，debug了一把，发现`AppcompatTextHelper`无法生成`TintInfo`
```java
    protected static TintInfo createTintInfo(Context context,
            AppCompatDrawableManager drawableManager, int drawableId) {
        final ColorStateList tintList = drawableManager.getTintList(context, drawableId);
        if (tintList != null) {
            final TintInfo tintInfo = new TintInfo();
            tintInfo.mHasTintList = true;
            tintInfo.mTintList = tintList;
            return tintInfo;
        }
        return null;
    }
```
我不明白`ImageView`的`android:tint`可以生效，而`TextView`不可以，又不想将`TextView`拆分为`ImageView`和`TextView`。所以呢，没办法，我只能复制一份ic_back.xml，然后修改fillColor为white之后另存为ic_back_white.xml。

## VectorDrawble无法在selector中使用
这里其实有两个问问题。
为了提高用户体验，上面的`TextView`点击颜色会变换，我想让图片也跟着一起变色。我本是这么定义的
```xml ic_back_selector.xml
<selector
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item android:state_pressed="true">
        <bitmap android:tint="#f00"
                android:src="@drawable/ic_back"
                />
    </item>
    <item>
        <bitmap android:src="@drawable/ic_back"
            android:tint="#00f"/>
    </item>
</selector>
```
首先在5.0之前的系统上运行，发现tint根本不起作用。所以我又定义了一个ic_back_pressed.xml。
然后在5.0上的系统运行时，直接crash了。根据`Caused by: android.content.res.Resources$NotFoundException: File res/drawable/ic_back_selector.xml from drawable resource ID #0x7f02005d`错误，stackOverflow上又找到了答案，原来gradle中需要配置android.defaultConfig.vectorDrawables.useSupportLibrary = true。然而并没有卵用，gradle sync之后，还是crash。不过Android Studio有新的错误提示，意思是要使用app:srcCompat代替android:src，按照这个提示，又修改了一次。结果又是crash，提示`Caused by: org.xmlpull.v1.XmlPullParserException: Binary XML file line #6: <bitmap> requires a valid src attribute`。好吧，我认输了。drawable下面已经有ic_back.xml，ic_back_pressed.xml，ic_back_white.xml，ic_back_selector.xml这么多vectorDrawable了，而且我想要的效果还不能实现。这坑就不填了。
