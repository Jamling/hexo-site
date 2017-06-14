---
title: Hexo博客双线部署
date: 2016-08-29 11:30:30
category: [Web]
tags: [Hexo]
toc: true
---

## 引言

鉴于github.io在国内访问速度较慢，而且github.io不会被百度收录，所以在将[Hexo]博客站点部署到国内的站点是一个不错的选择。之前，国内有一家与github类似的gitcafe公司，不过在今年的时候，gitcafe已经停止服务，用户数据全部迁移到[coding.net]了。而且[coding.net]也提供与github pages一样的pages服务，所以我选择[coding.net]作为国内的博客部署平台。

## 开启Coding Pages服务

在开启Coding Pages服务之前，你必须得有一个[coding.net]账户，然后创建一个（私有并且项目名称与您的用户名称一致的）项目。并开启（用户）Pages服务。

{% asset_img coding.net-console.png %}

如果未开启服务，则界面上有一个开启服务的按钮，点击开启服务。服务开启之后，可以绑定域名。Coding Pages默认使用https协议。如果不绑定域名，[coding.net]提供了一个默认的<var>yourname</var>.coding.me域名。

官方有详细文档，请点击[Coding Pages 帮助]查看

至于github pages服务，则不描述了，网上一搜一大把。

## DNS解析

双线部署的核心，其实是域名DNS的解析。我的域名是从万网购买的。它提供多路线解析。不知道其它的域名提供商是否提供此功能。

首先上我的域名解析规则

{% asset_img net.cn-console.png %}

www的CNAME有两条解析路线，默认（国内）路线指向<var>username</var>.coding.me；海外指向<var>username</var>.github.io。也即，在国内访问，其实是访问coding.net上的博客。而海外，则是访问github.io上的博客。那么如何验证访问的是海外还是国内路线呢？先不急，后面有介绍。

PS：
- 我的顶级域名解析到一个IP地址，其实是ping jamling.coding.me的IP地址，jamling.coding.me的IP地址是变动的。所以有时使用http://ieclipse.cn 访问的话，可能会出现无法连接到服务器而导致无法访问。解决的办法是将A记录指向<var>username</var>.coding.me。不过，DNS默认提供了@A记录解析，需要先删除掉才能添加成功（我的另一个域名把系统提供默认@记录解析删除掉了以实现http://domain.com 与http://www.domain.com 相同的访问效果）。
- 我的博客在github和coding.net，都是属于用户Pages，如果是项目Pages，请看我的dl.ieclipse.cn解析规则，CNAME国内解析到pages.coding.net。CNAME海外解析到<var>username</var>.github.io。

## 验证测试
验证测试的办法就是github.io上的内容与[coding.me]上的内容不一致，然后通过使用海外代理或翻墙的方式来访问相同的域名，看是否解析正常。浏览器不使用代理，VPN，访问的域名，就是coding.me。使用翻墙软件访问的就是github.io。然后看比较内容，看是否正常解析。还有一种方式就是注意看浏览器左下角的状态。比如输入域名后回车，状态栏有正在连接github.io，则说明访问的是海外路线。

## Hexo设置

github.io与coding.me都可以通过git来部署。

``` yaml
deploy:
  type: git
  repo: 
    coding: git@git.coding.net:Jamling/Jamling,coding-pages
    github: git@github.com:Jamling/jamling.github.com.git,master
```

repo可以写多个，格式为git仓库地址+英文逗号+分支名称。在国内，有时部署到github.io会失败，此时，[coding.me]上的博客是最新的，而github.io上的博客还是原来的，此时，正是验证你双线部署是否成功的绝佳时刻。

## 致谢
- **感谢[github.com]!**
- **感谢[coding.net]!** 

[github.com]和[coding.net]为大家免费提供的pages服务，希望大家不要滥用，更不要做违反相关法律法规的事，不然，哪一天迫于政府压力终止服务或被墙的时候，到哪里去找这么优质的免费空间呢？

## 参考
Coding Pages 帮助: https://coding.net/help/doc/pages/index.html
github pages自定义域名: https://help.github.com/articles/setting-up-a-custom-subdomain/
[github.com]: https://github.com
[coding.net]: https://coding.net
[Hexo]: https://hexo.io
[Nova]: http://github.com/Jamling/hexo-theme-nova
[Coding Pages 帮助]: https://coding.net/help/doc/pages/index.html
