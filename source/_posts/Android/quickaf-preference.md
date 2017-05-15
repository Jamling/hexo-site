---
title: QuickAF Preference介绍
date: 2016-08-12 10:16:41
tags: [Android, QuickAF]
category: [Android]
toc: true
---

## 简介

首先上个图

{% asset_img preference.png %}

Preference控件的目的在于替换Android自带的Preference，在[QuickAF]中，Preference被设计为控件，可以在layout/Activity中随意使用，不必像自带的Preference，需要配合res/xml/xxx_preference、PreferenceFragment/PreferenceActivity使用。

<!-- more -->

## 配置
先来看看控件属性
``` xml
    <declare-styleable name="Preference">
        <attr name="android:key"/>
        <attr name="android:title"/>
        <attr name="android:summary"/>
        <attr name="android:persistent"/>
        <attr name="android:layout"/>
        <attr name="android:icon"/>
        <attr name="android:drawableRight"/>
        <attr name="android:gravity"/>
        <attr name="android:background"/>
    </declare-styleable>

```

各属性含义如下：

- `android:key`：持久化时，写入shared preference中的key名称
- `android:title`：主标题，一般是左侧的文字
- `android:summary`：副标题，一般是右则的文字或主标题下方的文字
- `android:persistent`：是否持久化，如果为true，则会将checkbox的值写入shared preference
*目前仅支持checkbox写入*
- `android:layout`：允许添加一个自定义的layout到当前控件中。
- `android:icon`：设置主标题左侧的图标
- `android:drawableRight`：设置副标题右侧的图标，比如一个〉右箭头
- `android:gravity`：设置主标题的对齐方式
- `android:background`：设置背景，比如list_selector。

## 使用示例

### 布局

```xml sample_activity_preference.xml https://github.com/Jamling/QuickAF/blob/master/sample/src/main/res/layout/sample_activity_preference.xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:divider="@drawable/divider_linear"
    android:orientation="vertical"
    android:showDividers="middle|end" >

    <cn.ieclipse.af.view.Preference
        android:id="@+id/s01"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout="@layout/preference_item"
        android:summary=""
        android:title="普通的"/>

    <cn.ieclipse.af.view.Preference
        android:id="@+id/s02"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:icon="@android:drawable/ic_menu_help"
        android:layout="@layout/preference_item"
        android:summary=""
        android:title="带图标的" />

    <cn.ieclipse.af.view.Preference
        android:id="@+id/s03"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:icon="@android:drawable/ic_menu_edit"
        android:layout="@layout/preference_item"
        android:summary="有描述的"
        android:title="带图标的" />

    <cn.ieclipse.af.view.Preference
        android:id="@+id/s04"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:drawableRight="@drawable/preference_arrow"
        android:icon="@android:drawable/ic_menu_share"
        android:layout="@layout/preference_item"
        android:summary="有描述的"
        android:title="带图标, 带箭头的" />

    <cn.ieclipse.af.view.Preference
        android:id="@+id/s05"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:drawableRight="@drawable/preference_arrow"
        android:icon="@android:drawable/ic_menu_view"
        android:layout="@layout/preference_chk_item"
        android:persistent="true"
        android:summary="有描述的"
        android:title="带图标，带Checkbox的" />
</LinearLayout>
```

### Java代码

```java
Preference p = view.findViewById(R.id.s01);
p.setOnClickListener(new View.OnClickListener() {
    public void onClick(View v) {
        // TODO something.
    }
});
```

## 关于

[QuickAF]是一个Android平台上的app快速开发框架，欢迎读者在github star或fork。本人写作水平有限，欢迎广大读者指正，如有问题，可与我直接联系或在我的官方博客中给出评论。

## 参考
QuickAF: https://github.com/Jamling/QuickAF

[QuickAF]: https://github.com/Jamling/QuickAF
