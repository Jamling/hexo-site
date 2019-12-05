---
title: Git submodule简介
date: 2016-06-02 16:42:28
category: [其它]
tags: git
---
## 修改历史
2016-07-29 修改部分内容

如果项目很大，分为众多模块，并且需要对每个模块单独进行版本控制的时候，引入git submodule是一个非常不错的选择。尤其是对一些sdk+plugin的项目尤为适用。

<!-- more -->

比如本站是使用hexo搭建的博客，其中使用的主题与插件，都是在迁移博客时同步开发的，但是这主题与插件，同样可以应用到其它的博客站点，所以，不能把源文件、主题和插件纳入同一个代码仓库进行版本控制，为方便主题与插件的版本控制，它们在github上都建立了单独的代码仓库。对于博客源文件、主题及插件的修改，只能提交到其对应的git仓库。

为方便理解，将我的博客工程结构列举如下：
<pre>
hexo-site #整个工程，对应 hexo-site 仓库
    source #博客源文件
    themes # 主题目录，在hexo-site仓库.gitignore中
        nova  # 主题，对应hexo-theme-nova仓库
    node_modules # 插件目录，在hexo-site仓库.gitignore中
        hexo-generator-github #插件1，对应hexo-generator-github仓库
        hexo-generator-i18n #插件2，对应hexo-generator-i18n仓库
</pre>

其中hexo-site对应Jamling/hexo-site.git，themes/nova目录对应Jamling/hexo-theme-nova.git，node_modules/下的github和i18n插件对应的也是单独的git仓库。

为了保持博客源文件的干净，在.gitignore文件中，把themes目录和node_modules目录都加入了。所以，主题与插件的修改，都不会影响到hexo-site仓库。

在hexo-site目录下，通过`git submodule add`命令添加主题及插件子模块

```bash
git submodule add -f git@github.com:Jamling/hexo-theme-nova themes/nova
git submodule add -f git@github.com:Jamling/hexo-generator-github node_modules/hexo-generator-github
git submodule add -f git@github.com:Jamling/hexo-generator-i18n node_modules/hexo-generator-i18n
git submodule add -f git@github.com:Jamling/hexo-generator-index2 node_modules/hexo-generator-index2
git submodule add -f git@github.com:Jamling/hexo-filter-highlight node_modules/hexo-filter-highlight
```

加-f选项，是因为themes目录和node_modules目录加入了.gitignore，`git submodule add`第一个参数为子模块的git仓库地址，第二个参数为子模块的名字。子模块成功添加之后，会在hexo-site目录下创建一个<var>.gitmodules</var>文件。内容如下：

```
[submodule "themes/nova"]
	path = themes/nova
	url = gh:Jamling/hexo-theme-nova
[submodule "node_modules/hexo-generator-github"]
	path = node_modules/hexo-generator-github
	url = gh:Jamling/hexo-generator-github
[submodule "node_modules/hexo-generator-i18n"]
	path = node_modules/hexo-generator-i18n
	url = gh:Jamling/hexo-generator-i18n
```

如果对git submodule命令不熟，可以使用`git submodule add --help`查看帮助，也可以通过直接修改<var>.gitmodules</var>文件来添加。

注：将子模块加入<var>.gitignore</var>的好处是主仓库中不会出现子模块需要更新的提示，进一步将主仓库与子模块独立开来，但是缺点就是，如果要更新子模块，须单独cd到子模块目录，手动`git pull`更新。

子模块添加成功之后，主仓库与子模块的操作都是独立的。执行`git status`等操作，只会看到当前仓库的变更，跟其它的仓库（模块）没有任何关系。


