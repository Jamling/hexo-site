---
title: Gradle Compile Exclude
date: 2016-11-29 12:00:00
category: [Android]
tags: [Android,Gradle]
toc: false
description: Gradle Compile Exclude
---

Android兼容库从24.2.0开始，对support-v4做了分库处理，原有的support-v4库拆分成为了support-compat、support-core-ui、support-fragment等库，本着稳定的原则，一直到25.0.0才开始在项目中升级support-v4库，将原有的support-v4替换为support-core-ui（其实，项目中只用到了ViewPager，SwipeRefreshLayout而已）以减少apk体积。然而问题来了，编译时出错，提示存在重复的类库。为保证support库版本一致，我还统一定义并使用了各模块的support版本变量，使用`gradle dependencies`查看依赖的时候，发现有个第三方库依赖于于support-v4:23.0.0。结果导致编译不通过。google了一把，发现gradle complie竟然还可以传参数。来exclude某些库。

<!-- more -->

特记录如下：
```gradle
    compile("com.timehop.stickyheadersrecyclerview:library:0.4.3") {
        exclude group: 'com.android.support'//, module: 'support-v4'
    }
    compile ('com.google.android:flexbox:0.1.2') {
        exclude group: 'com.android.support'
    }
```

附：不想频繁更新不稳定library的方法

```gradle
repositories {
    flatDir { dirs 'libs' }
}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.3.0'
    // compile 'com.nostra13.universalimageloader:universal-image-loader:1.9.5'
    // compile 'cn.ieclipse.af:af-library:1.0.0'
    compile(name: 'af-library-release', ext: 'aar')
    compile 'com.android.volley:volley:1.0.0'
    compile 'com.google.code.gson:gson:2.3'
}
```
缺点就是library的依赖库，必须手动在app模块中配置