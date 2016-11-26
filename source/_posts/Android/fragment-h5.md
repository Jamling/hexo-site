---
title: Fragment加载WebView
date: 2016-11-26 12:00:00
category: [Android]
tags: [Android]
toc: false
description: 在Fragment中使用WebView网页的问题
---

做Android 5年多了，第一次在Fragment中使用WebView，先将Activity中的代码复制过来改改，没有想到，竟然无法正常显示，而是打开手机浏览器渲染WebView，返回后，Fragment一片空白。这还能忍受？Google一把，说是要继承WebViewFragment，但是一旦继承，就破坏了我的框架结构。然后看了下WebViewFragment的源码，并不复杂。所以就将相关的源码合并到现有的fragment中，本以为可以妥妥地解决问题，结果还是那样。最后在stackoverflow上解决了终极解决办法，那就是需要设置特殊的WebViewClient。

```java
    private class H5WebViewClient extends WebViewClient {
        @Override
        public boolean shouldOverrideUrlLoading(WebView webView, String s) {
            webView.loadUrl(s);
            //  一定要return true，不然就会使用浏览器打开
            return true;
        }
    }
```

<!-- more -->
