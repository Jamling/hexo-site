---
title: 使用Travis-CI构建Android应用
date: 2016-09-06 21:00:00
category: [Android]
tags: [Android, CI]
toc: true
description: 使用Travis-CI构建github上的Android应用
---

## 简介
注册Travis-CI都N长时间了，但是一直没有使用它来构建。今天先使用它构建了两个小的node.js项目，还算顺序，然后使用它来构建[QuickAF]，没有想到竟然是一条如此艰辛的路！特地将构建过程记录如下。

<!-- more -->

## 配置
参考官方文档[https://docs.travis-ci.com/user/languages/android/]的配置，稍做改动就提交构建了。还以为能像node.js那样顺风顺水，没有想到竟然是一次又一次的失败。

## gradle版本

```
FAILURE: Build failed with an exception.

* Where:

Build file '/home/travis/build/Jamling/QuickAF/library/build.gradle' line: 12

* What went wrong:

A problem occurred evaluating project ':af-library'.

> Failed to apply plugin [id 'com.android.library']

   > Minimum supported Gradle version is 2.14.1.  Current version is 2.2.1. If using the gradle wrapper, try editing the distributionUrl in /home/travis/build/Jamling/QuickAF/gradle/wrapper/gradle-wrapper.properties to gradle-2.14.1-all.zip

* Try:

Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

Total time: 42.348 secs
```
根据以上日志，travis-ci上默认的gradle版本比较旧，所以需要自己下载gradle，并配置环境。

```yaml
before_install:
  # Gradle
  - wget http://services.gradle.org/distributions/gradle-2.14.1-bin.zip
  - unzip -n gradle-2.14.1-bin.zip
  - export GRADLE_HOME=$PWD/gradle-2.14.1
  - export PATH=$GRADLE_HOME/bin:$PATH
```

***我有gradle/wrapper/gradle-wrapper.properties文件，而且里面的版本就是2.14.1，为毛不起作用啊！！！***
***unzip 要使用-n参数，不然，目录存在，会一直卡死在unzip***

如果不想每次下载gradle，可以在root project的build.gradle中手动设置gradle版本
```gradle
task wrapper(type: Wrapper) {
    gradleVersion = '3.3'
}
```
然后执行的命令使用`./gradle app:assembleRelease`

但接着又发现`./gradlew: Permission denied` gradlew因为权限问题而导致执行失败。

解决办法如下：参考http://stackoverflow.com/questions/33820638/travis-yml-gradlew-permission-denied
```bash
git update-index --chmod=+x gradlew
git commit -m "permission access for travis"
git ls-tree HEAD
```
如果gradlew权限为100755表示成功给它加上了执行权限

## SDK设置
sdk的配置一定要对应好。不然编译会失败

## jdk设置
好像build tool 24以上的版本需要使用jdk8

## 总结
遇到各种错误，编译了21次，终于通过了。

## 参考
QuickAF: https://github.com/Jamling/QuickAF
https://docs.travis-ci.com/user/languages/android/

[QuickAF]: https://github.com/Jamling/QuickAF
[https://docs.travis-ci.com/user/languages/android/]: https://docs.travis-ci.com/user/languages/android/

