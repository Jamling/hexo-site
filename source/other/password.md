---
title: 内容加密示例
date: 2016-01-01 17:43:47
toc: false
description: Nova主题加密网页示例
donate: false
password: 123
---

### 加密准备
在内容之前插入加密脚本，在内容之后插入解密脚本，示例代码如下：

``` htmlbars
{{ page_encrypt(page, {dom:'.page-body', src:'/js/encrypt.min.js'}) }}
<div class="page-body">
{%- block content %}
  {{ page.content }}
{%- endblock %}
</div>
{{ js('/js/decrypt.min.js') }}
```

### 加密选项
`v`：加密版本，目前为1，解密难度：低；所以一般懂前端的人要破解密码，不是一件难事，后面可能会升级加密算法。
`src`：加密脚本，默认为`/js/encrypt.min.js`，内容加密算法不在这里，估计有人不信，那自己看脚本吧。
`dom`：加密对象DOM元素的选择器，如果密码正确，对应的DOM节点会显示解密后的内容

### 解密算法
解密脚本与加密脚本是一一对应的。默认为`/js/decrypt.min.js`

### 密码
密码存放在源文件中，所以如果不想让别人看到你设置的密码，确保你的源文件不被别人看到即可，如将源代码仓库设置为私有

### 特别说明
本加密对`.md`的支持不是特别好，请慎重使用。因为它不像`.html`那样可以直接渲染，试想，如果对`.md`内容加密了，如何将markdown转为html呢？如果大家有什么好的想法，不妨留下您的宝贵意见。
