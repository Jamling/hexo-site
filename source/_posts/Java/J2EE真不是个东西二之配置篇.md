---
title: J2EE真不是个东西二之配置篇
date: 2007-8-24 20:24:00
category: [Java]
tags: [J2EE]
toc: true
---

J2EE的配置真不是个东西！它也确实不是个什么东西！Java的诞生让我想到的lichking的诞生，有了lichking，于是就有了荒芜之地。J2EE既然这是片荒芜之地的产物，那么要建立一个企业级的应用-一支强大的UD军队，非得要有黑暗城堡-JDK不可。

说到JDK的配置我就一肚子的火，记得刚学习Java的时候，写了书上的一个叫做你好世界的程序，然后在CMD下面

```bash
javac HelloWord.java
```

结果。。。

我大骂Java真不是个东西。后来才知道，这个你好世界的程序要拿到java的bin目录下面，才能编译，才能
```sh 
Java HelloWord
```
才能看见世界！后来老师说在CMD下面敲个

```bash
set classpath=.;
```
就不用拿到BIN目录下面去了，这句话又长，还没等我拿笔记下来，它就不跟我玩了。于是我每次上实习课就要咒这个classpath。我想我还有C++呢，这个Java真不是东西，它不跟我玩，我也不跟它玩下去了，于是，这个学生时代的Java就这么混过去了。

时隔两年，毕业设计是J2EE方向，郁闷啊！我的候选课题都没选上！于是无奈的再次面对事实。
 
搭建第一个JSP环境，我用了整整一天的时间，最终的结果也就是能在一个test.jsp里面看到了

```jsp
out.println(“JSP, you are a shit !”)
```
的结果。

J2EE真不是个东西！这个时候也只==看见了荒芜之地正在建造的光环。 

## JDK的配置
1. 设置环境变量
JAVA_HOME == Java Installed Directory（jdk_ver: ver为JDK的版本号）；
2. 同理设置classpath=.;%java_home%/lib/dt.jar;%java_home%/lib/tools.jar;  
   重要的两个JAR：dt.jar & tools.jar；
3. 设置path=%java_home%/bin 
好了，进入CMD javac 回车，java回车，看见的是help 而不是Bad command就行了，连System.out.println(“Hello,World”)的世界都可以不用去测试了。

## Tomcat配置
Tomcat这只猫的配置是这样的，
1. 解压（注意不要出现重复的路径）也好，安装也好，安装（解压）到一个目录下面。这个目录称之为catalina_home（某个版本以前叫tomcat_home，这名字蛮好的，有魅力，看一遍就记住了）。
2. 设置环境变量，catalina_home ,catalina_base= C:/apache-tomcat5.5.17。3，在地址栏内输入http://localhost:8080/
看到那只猫的话，那就要恭喜你了。
 
        
## JSP的配置
1. classpath追加;%catalina_home%/common/lib/jsp-api.jar; %catalina_home%/common/lib/jasper-runtime.jar; %catalina_home%/common/lib/servlet-api.jar（使用Servlet才用到）;建议这些在配置Tomcat的时候一起配置好。

2. web.xml的配置，从webapps/ROOT/WEB-INF下面拷贝web.xml放入自己的JSP WEB-INF目录中，打开它，把从<web-app> 到</web-app>之间的内容全删掉就OK了。好了，自己试着写第一个JSP文件吧，如果你英文功底很好的话就写个out.println(“hello,World”)。如果你是相当的热爱我们的祖国的话，那么请你在文件的第一行加入下面此句（个人建议，要热爱祖国！）
```htmlbars
<%@ page contentType=”text/html;charset=GBK” language=”java” %>。
```
如果以上配置都通过的话，那么是否该为自己庆祝一下呢？如果只是做个毕业设计或者玩玩的话，那么，是该休息一会了。毕竟荒芜之地上能造的也都造了，也可以带着侍僧和小鬼去失落的神庙狩门口的311了。但是战争不是这么快就能结束的，观众也不希望就这样结束，而且你很快也会发现，做掉了311就无事可干了。于是很快就有命令传来，于是诅咒将再一次开启。
