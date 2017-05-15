---
title: QuickAF标题栏介绍
date: 2016-08-03 21:00:00
category: [Android]
tags: [Android, QuickAF]
toc: true
description: QuickAF标题栏介绍,QuickAF使用一个IOS风格的TitleBar来实现标题栏
---

## 简介
在[QuickAF]框架中，标题栏不使用Android自带的ActionBar或ToolBar，而是使用一个IOS风格的TitleBar来实现。TitleBar的使用非常灵活，主要分为三块区域：
- 左侧区域，一般放置一个返回按钮，父类AfActivity已经默认点击为finish操作
- 中间区域，一般显示页面的标题，如“订单详情”
- 右侧区域，一般放置本页面的相关操作图标或文本按钮

<!-- more -->

## 初始化
一般来说，一个App的页面风格应该是统一的，所以一般在BaseActivity/BaseFragment中对TitleBar进行初始化。初始化方法一般是放在`initHeaderView()`中，如下所示：

```java
protected void initHeaderView() {
    mTitleLeftView = (TextView) View.inflate(this, R.layout.title_left_tv, null);
    mTitleTextView = (TextView) View.inflate(this, R.layout.title_middle_tv, null);

    mTitleBar.setLeft(mTitleLeftView);
    mTitleBar.setMiddle(mTitleTextView);

    int padding = AppUtils.dp2px(this, 8);
    mTitleBar.setPadding(padding, 0, padding, 0);
    if(!isOverlay()){
        mTitleBar.setBackgroundColor(getStatusBarColor());
        mTitleBar.setBottomDrawable(AppUtils.getColor(this, R.color.divider));
    }
    setOnClickListener(mTitleLeftView);
}
```

通常，BaseActivity/BaseFragment中已经对TitleBar初始化过了，所以子页面的初始化主要的就是控制TitleBar的右侧区域以及设置标题文字。设置标题文字调用`setTitle("titleName")`就可以了，下面主要描述对TitleBar右侧区域的初始化。

TitleBar提供了一个`addRightView()`的API来添加右侧菜单项。参数类型可以是layout resource或View对象。如此，用户可以自由控制菜单项的属性，比如设置图片，设置文字，添加onClick事件等。

- 右侧button为图片：
``` java
mTitleTextView.setText(R.string.house_title);
mMap = (ImageView) mTitleBar.addRightView(R.layout.layout_title_icon);
mMap.setImageResource(R.drawable.title_map);
mSearch = (ImageView) mTitleBar.addRightView(R.layout.layout_title_icon);
mSearch.setImageResource(R.drawable.title_search);
setOnClickListener(mMap, mSearch);
```

- 右侧button为文字：
```java
mManageTv = (TextView) mTitleBar.addRightView(R.layout.layout_title_text);
mManageTv.setText(isMyList() ? R.string.note_my_book_title_manage : R.string.note_other_book_title_change);
```

- 右侧button为图片加文字(比较少)
```java
mSubmit = (TextView) mTitleBar.addRightView(R.layout.layout_title_post_submit);
```

## 设置

### 对齐方式
TitleBar在垂直方向默认是居中对齐的，在水平方向，提供两种对齐方式：左对齐与居中对齐（默认）。可以通过`TitleBar#setGravity(int gravity)`方法设置TitleBar的对齐方式，一般来说只需要设置水平对齐即可

- `Gravity.CENTER_HORIZONTAL`，水平居中对齐（默认） ，TitleBar中间区域（标题）会居中显示
- `Gravity.LEFT`，水平左对齐，TitleBar中间区域（标题）会在左侧区域后面显示

不同的水平对齐方式，中间区域的显示空间是不一样子，如果不能满足需求，还可以通过`TitleBar.setConfig()`来设置TitleBar参数。

### 隐藏标题栏
TitleBar可以通过`View.setVisibility()`来控制显示或隐藏，如果是页面不需要标题栏，建议是在`initWindowFeature()`方法中通过`setShowTitleBar(false)`来设置。

### 悬浮标题栏
TitleBar支持设置其是否悬浮在页面上（不随页面滚动），可以在`initWindowFeature()`方法中通过`setOverlay(false)`来设置。

## PopupMenu
TitleBar内部未实现popup menu，如果需要使用下拉菜单，可以在特定的页面中结合PopupWindowUtils或自己实现一个下拉菜单

## 关于

[QuickAF]是一个Android平台上的app快速开发框架，欢迎读者在github star或fork。本人写作水平有限，欢迎广大读者指正，如有问题，可与我直接联系或在我的官方博客中给出评论。

## 参考
QuickAF: https://github.com/Jamling/QuickAF

[QuickAF]: https://github.com/Jamling/QuickAF
