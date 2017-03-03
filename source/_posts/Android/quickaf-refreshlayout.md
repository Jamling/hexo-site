---
title: QuickAF中的下拉刷新
date: 2017-03-03 12:16:41
tags: [Android, QuickAF]
category: [Android]
toc: true
---

{% asset_img RefreshLayout.png %}

[QuickAF]使用`RefreshLayout`控件进行下拉刷新和上拉加载，通过在布局中定义`app:ptr_content`和`app:ptr_empty`或api中设置内容layout和错误view。
`RefreshLayout`的特点如下：
- 支持任意Layout的下拉刷新和上拉加载，默认支持`VScrollView`,`RecyclerView`, `ListView`, `GridView`上拉/下拉，还可以通过registerDetector来支持更多的Layout;
- 支持自定义EmptyView，一个EmptyView包含loading, empty, error三个子view
- 支持empty view的下拉刷新
- 支持FooterView，FooterView一般在Adapter中定义

<!-- more -->

## 使用示例
请参考[QuickAF]中的`BaseListFragment.java`

```java
public abstract class BaseListFragment<T> extends BaseFragment implements RefreshLayout.OnRefreshListener {
    protected RefreshLayout mRefreshLayout;
    protected RefreshRecyclerHelper mRefreshHelper;
    protected RecyclerView mListView;
    protected AfRecyclerAdapter<T> mAdapter;

    @Override
    protected int getContentLayout() {
        return R.layout.base_refresh_recycler;
    }

    @Override
    protected void initContentView(View view) {
        super.initContentView(view);

        mRefreshLayout = (RefreshLayout) view.findViewById(R.id.refresh);
        mRefreshLayout.setOnRefreshListener(this);
        mRefreshLayout.setMode(RefreshLayout.REFRESH_MODE_BOTH);
        mListView = (RecyclerView) mRefreshLayout.findViewById(R.id.rv);

        mRefreshHelper = generateRefreshHelper();
        mAdapter = generateAdapter();
        mRefreshHelper.setAdapter(mAdapter);
    }

    @Override
    public void onRefresh() {
        load(false);
    }

    @Override
    public void onLoadMore() {
        load(false);
    }

    protected void load(boolean needCache) {
    }

    protected RefreshRecyclerHelper generateRefreshHelper() {
        AppRefreshRecyclerHelper helper = new AppRefreshRecyclerHelper(mRefreshLayout);
        return helper;
    }

    protected abstract AfRecyclerAdapter<T> generateAdapter();
}
```

布局
```xml base_refresh_recycler.xml
<?xml version="1.0" encoding="utf-8"?>
<cn.ieclipse.af.view.refresh.RefreshLayout
    android:id="@+id/refresh"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:ptr_content="@layout/base_rv"
    app:ptr_empty="@layout/ptr_empty_view">

</cn.ieclipse.af.view.refresh.RefreshLayout>
```
ptr_content布局
```xml base_rv.xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/rv"
    style="@style/base_rv">

</android.support.v7.widget.RecyclerView>
```

## 关于

[QuickAF]是一个Android平台上的app快速开发框架，欢迎读者在github star或fork。本人写作水平有限，欢迎广大读者指正，如有问题，可与我直接联系或在我的官方博客中给出评论。

## 参考
QuickAF: https://github.com/Jamling/QuickAF

[QuickAF]: https://github.com/Jamling/QuickAF
