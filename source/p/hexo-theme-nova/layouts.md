title: Layouts
layout: project
title2: hexo-theme-nova.userguide.layouts
---
The theme provided three layouts to demonstrate the page.

 1. `post` for blog
 2. `project` for github project
 3. `page` for other pages

## Summary ##
### post
Similar to most hexo theme, nova has index, archive, widgets layout. The difference is nova rewrite archive list helpers and provided two paginator helpers.

### project
Project layout is aimed to demonstrate the github projects info. For project layout, a `gh` front-matter is nessary in your page.

The projects sidebar is configurated in <var>_data</var>/<var>projects.yml</var>

### page

The default layout of other pages.

A `type` front-matter is used to mark special pages.

- categories: Categories page
- donates: Donate list page.

### other
- comment: Page comment, default is uyan comment
- share: Jiathis share component
- analytics: Baidu analytics component
- donate: page donate component

## Layout tree
<pre>
layout
	post
		index.swig // home
		article.swig // post article detail
		archive.swig // archive/tag/category 
		widget_xxx.swig // search, tags, categories, recently... widget_xxx.swig // search, tags, categories, recently...widget_xxx.swig // search, tags, categories, recently...
	project
		projects.swig // project home, list repos on github
		releases.swig // list repo releases on github
		contents.swig // display repo contents on github
		sidebar.swig // project navigator side bar
	portail
		header.swig // html head, header
		footer.swig // html footer
		comment.swig // page comment
		toc.swig    // toc suffix
	page
		categories.swig // all categories
		donates.swig // donate records and rank
		article.swig // the article 
	index.swig // index layout dispatcher, link to post/index.
	post.swig // post layout dispatcher.
	page.swig // page layout dispatcher.
	project.swig // project layout dispatcher.
</pre>

### post 

The index page is a special article list page, listing all post with excerpt in main container.

The archive, category, tag page regards as archive list page, listing article archive list in main container.

The article page display the post detail content in main container

The `layout/post.swig` dispatche layout to `layout/post/xxx.swig` :

```htmlbars
<div class="container container-fluid">
<div class="row">
  <div class="{{theme.layout.index.main}}">
    {% if is_home() %}
    {{ partial('post/index') }}
    {% elseif is_archive() || is_category() || is_tag() %}
    {{ partial('post/archive') }}
    {% elseif is_post() %}
    {% set show_toc = theme.toc.post && page.toc %}
    {{ partial('post/article', {post: page}) }}
    {% endif %}
  </div>
  <!-- aside -->
  <div class="{{theme.layout.index.widgets}}">
    <aside>
      {%- if page_toc() %}
      {{ partial('./partial/toc', {style: theme.layout.index.widgets}) }}
      {%- endif %}
    </aside>
  </div>
</div>
</div>
```
The `layout/index.swig` is a special layout for home page.

### page

For common page, display content in main container, and TOC in right aside.

For special page, dispatcher to special layout.

`layout/page.swig` dispatcher:

```htmlbars
{%- if page.type === 'categories' %}
{{ partial('page/categories', {}) }}
{%- elseif page.type === 'donates' %}
{{ partial('page/donates', {}) }}
{%- else %}
<div class="container container-fluid">
  <div class="row">
    <div class="{{theme.layout.page.main}}">
      {{ partial('./page/article') }}
    </div>
    <div class="{{theme.layout.page.toc}}">
      {{ partial('./partial/toc') }}
    </div>
  </div><!--  end row -->
</div>
{%- endif %}

```

### project

Projects page, listing repos on github

Other page, display content in main container, TOC in right aside and nagivation bar in left aside.

`layout/project.swig` dispatcher:

```js
{% if page.gh %}
  {% set gh = gh_opts() %}
  {% if gh.type === 'get_contents' %} 
    {% set page.content = gh_contents() %}
    {{ partial('project/contents', {} )}}
  {% elseif gh.type === 'get_repos' %}
    {{ partial('project/projects', {} )}}
  {% elseif gh.type === 'get_releases' %}
    {{ partial('project/releases', {} )}}
  {% endif %}
{% else %}
  {{ partial('project/contents', {} )}}
{% endif %}
```

# Plugin
There are many [plugins](https://hexo.io/plugins) of [hexo], it's easy to write a plugin under [hexo].
Just write a <var>.js</var> under <var>script</var> in your theme.

here is a sample (<var>script</var>/<var>helpers.js</var>) to write a helper plugin to return page title.
```js
// return page title.
hexo.extend.helper.register('page_title', function(){
  var p = this.page;
  var ret = '';
  if (p.title2) { // if has a title2 in front-matter, i18n title2 value as title
    ret = this.i18n(p.title2);
  }
  else if (p.title){ // use title value as title 
    ret = p.title;
  }
  return ret;
});
```

And another sample of display categories in post:
```js
// insert category of post
hexo.extend.helper.register('post_cates', function(post){
  var cats = post.categories;
  var _self = this;
  var ret = '';
  if (cats == null || cats.length == 0) {
      return ret;
  }
  ret += '<span class="glyphicon glyphicon-folder-close" aria-hidden="true"></span>&nbsp;' + _self.__('category.label') + '';
  ret += '<ol class="breadcrumb category">';
  cats.forEach(function(item){
    ret += '<li><a class="" href="' + _self.url_for_lang(item.path) + '">' + item.name + '</a></li>';
  });
  ret += '</ol>';
  return ret;
});
```
Use in layout/post.swig

```htmlbars
          <footer class="post-item-footer">
            {{ post_cates(post) }} 
            {{ post_tags(post) }}
          </footer>
```
Will output:
<pre>
    <footer class="post-item-footer">
            <span class="glyphicon glyphicon-folder-close" aria-hidden="true"></span>&nbsp;分类<ol class="breadcrumb category"><li><a class="" href="/categories/软件技术/">软件技术</a></li><li><a class="" href="/categories/软件技术/Web/">Web</a></li></ol> 
            <span class="glyphicon glyphicon-tags" aria-hidden="true"></span>&nbsp;标签<ol class="breadcrumb tag"><li><a class="" href="/tags/Hexo/">Hexo</a></li><li><a class="" href="/tags/Node-js/">Node.js</a></li></ol>
    </footer>
</pre>

The [Nova] rewrite lost of helper of [hexo] to simplify the style. Please visit [Helpers](http://ieclipse.cn/p/hexo-theme-nova/helpers.html) for more informations.