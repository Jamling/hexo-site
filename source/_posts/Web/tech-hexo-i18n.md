---
title: Hexo多语言站点解决方案
date: 2016-06-02 20:42:28
category: [Web]
tags: Hexo
toc: true
---
Hexo是一个静态博客应用，所以在多语言站点支持上，并不是很好。虽然Hexo支持多语言，但是要生成多语言站点，还是有一些困难的。基于Hexo的一些特性，我给出了几种解决方案

<!-- more -->

## 方案一
生成的多语言站点单独放入独立的语言目录中，如public/en是英文站点，public/fr是法文站点，而public/则是默认语言站点。然后源文件目录，对于不同的语言的内容，放入对应的语言目录中，如source/en则放置英文站点下的源文件，同理，source/fr放置的是法语站点源目录。

通过修改_config.yml language顺序和public_dir，来生成不同语言。如生成默认语言
```yaml
language: [zh, en]
public_dir: public
```
生成英文站点
```yaml
language: [en, zh]
public_dir: public/en
```
但是有以下问题：
- 会生成类似public/en/en的多余文件
- 修改源文件，不好同步，如修改了默认语言下的某个源文件，不能直接copy到en/目录中

## 方案二
针对方案一的缺点，在方案一的基础上，加入title2 front-matter，来控制冗余与修改同步的问题。比如_post，在不同的语言中，title是不会自动翻译的。引入title2就是为了解决title的多语言支持。但是仍然无法解决生成public/en/en这样的问题。

但是实际应用中发现，这两种方案还存在其它的一些问题，比如语言显示顺序，语言切换后之类的问题。

## 方案三
利用hexo的i18n_dir作文章，编写generator插件来生成多语言。这便是[hexo-generator-i18n](http://github.com/jamling/hexo-generator-github)的雏形。通过添加路由，对page和post自动生成一个带语言前缀的路由。规则如下：
- 对于不存在多语言的源文件，添加一个多语言路由，输出的内容同默认语言
- 如果存在多语言的源文件，则只修改源目录的语言即可
如源目录下的donate/index.md，默认的page generator会生成donate/index.html，那么通过编写一个page-i18n的generator再生成一个en/donate/index.html。如果源目录中存在en/donate/index.md，那么只修改此页面的语言即可，不用再生成。

通过测试发现，多语言页面都可以正确生成，但是存在以下问题
- 像首页，归档页这些界面，切换语言时，仍然会出现404

## 方案四
为解决方案三中的问题，特地研究了hexo的生成器，发现像首页，归档页都是动态生成的。所以，添加国际化的范围，将动态生成的页面，也按方案三的page页面一样，再次动态生成多语言页面。

对于每个生成器，动态注册一个-i18n的生成器，专门用于处理首页，归档页等页面。
此时，发现所有的页面都支持多语言了。而且在hexo s模式下运行，语言想换就换。真是激动极了！然而在非默认语言下，点击链接的时候，发现仍然跳转到了默认语言！

OMG！难道之前的工作都白费了吗？

## 方案五
针对链接，特地重写了辅助函数url_for_lang(path)，根据当前路径判断是否带语言，如果带语言，则链接地址也带上语言前缀，插件中使用url_for(path)的地方全部替换为url_for_lang(path)。终于~全部搞定！

