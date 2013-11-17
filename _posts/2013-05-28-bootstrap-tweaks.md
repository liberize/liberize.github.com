---
layout: post
title: "Bootstrap 简单定制"
keywords: ["Bootstrap", "定制", "搜索框", "代码高亮", "评论"]
description: "Bootstrap 简单定制，添加搜索、代码高亮、评论与分析系统等"
category: web
tags: [jekyll, bootstrap, tweak]
---
{% include JB/setup %}
{% raw %}

### 一、增加搜索框

#### 1、GCSE

向jekyll中增加搜索功能有多种方式，此处采用Google自定义搜索来实现全站搜索。

在/assets/themes/twitter/searchbox/下新建searchbox.css：

```css
.navbar-search div{position:relative;margin-right:20px;}
.navbar-inner .navbar-search .search-query:focus,
.navbar-inner .navbar-search .search-query.focused,
.navbar-inner .navbar-search .search-query{padding-right:28px;}
.navbar-search button.icon-search{position:absolute;right:10px;top:7px;border:none;background-color:transparent;}
.search-hidden{width:0px;overflow:hidden;height:0px;}
```

打开/_includes/themes/twitter/default.html，在head部分添加以下内容：

```html
<!-- search with gcse -->
<link href="{{ ASSET_PATH }}/searchbox/searchbox.css" rel="stylesheet">
```

body部分，添加`{% include tweaks/searchbox %}`：

```html
<div class="navbar">
  <div class="navbar-inner">
    <div class="container-narrow">
      <a class="brand" href="{{ HOME_PATH }}">{{ site.title }}</a>
      <ul class="nav">
        {% assign pages_list = site.pages %}
        {% assign group = 'navigation' %}
        {% include JB/pages_list %}
      </ul>
    </div>
    {% include tweaks/searchbox %}
  </div>
</div>
```

/_includes/tweaks/searchbox的内容：

```html
<!-- display custom searchbox -->
<div>
  <script type="text/javascript">
    (function() {
      var cx = 'XXXXX:YYYY';
      var gcse = document.createElement('script');
      gcse.type = 'text/javascript';
      gcse.async = true;
      gcse.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') +
          '//www.google.com/cse/cse.js?cx=' + cx;
      var s = document.getElementsByTagName('script')[0];
      s.parentNode.insertBefore(gcse, s);
    })();
  </script>
  <div class="search-hidden">
    <gcse:search></gcse:search>
  </div>
  <form id="searchbox_XXXXX:YYYY" action="" class="navbar-search pull-right">
    <div>
      <input value="XXXXX:YYYY" name="cx" type="hidden"/>
      <input value="FORID:11" name="cof" type="hidden"/>
      <button class="icon-search"></button>
      <input id="q" name="q" type="text" class="search-query span2" placeholder="Search"/>
    </div>
  </form>
</div>
```
*注意*：此处cx中的"`XXXXX:YYYY`"应当替换为你的gcse id，若没有，请到[google官网](http://www.google.com/cse/)申请一个。

至此就基本完成了。当然你也可以把上面这些文件的内容都写入html，但为了保持结构清晰，此处还是将他们分开。

#### 2、其它

其它添加搜索的办法请参考：

* [Create Simple Search box](http://truongtx.me/2012/12/28/jekyll-create-simple-search-box/)
* [Search for your Jekyll Site](http://pradeepnayak.in/technology/2012/06/20/search-for-your-jekyll-site/)
* ...

### 二、设置代码高亮

#### 1、介绍

在jekyll中使用代码高亮同样有很多方式。以下从`代码转换为html`和`语法高亮显示`两方面介绍。

##### 1）转换方式及相应语法

将代码转换为html的方式很多，语法也各不相同。例如，可以使用liquid模板引擎：

	{% highlight ruby %}
		require 'redcarpet'
		markdown = Redcarpet.new("Hello World!")
		puts markdown.to_html
	{% endhighlight %}

也可以使用markdown转换器来实现，不同markdown转换器支持的语法格式也可能不同。

事实上，由于liquid语法的冗长以及在markdown中插入liquid代码带来的不协调，很多人更喜欢采用markdown本身的语法格式。

例如下面是`GFM(GitHub Flavored Markdown)`的`fenced code block`语法：


	```ruby
	require 'redcarpet'
	markdown = Redcarpet.new("Hello World!")
	puts markdown.to_html
	``` 

GFM语法是对markdown的扩充，需要markdown转换器支持。有关GFM的更多内容，请看[这里](https://help.github.com/articles/github-flavored-markdown)。

这里顺便提一下markdown转换器的有关内容。常见的转换器有：

* `maruku` (默认，纯ruby实现，速度较慢，语法灵活，支持若干扩展特性)
* `rdiscount` (C语言实现，速度最快，但基本上只支持标准markdown语法)
* `kramdown` (纯ruby实现，速度比maruku快好几倍，常与coderay一起使用，支持非常丰富的扩展语法)
* `redcarpet` (基于ruby库Sundown实现，速度较快，支持较为丰富的扩展语法，方便自定义)

*注意*：以上比较不一定完全准确，仅供参考。

由于我比较喜欢`fenced code block`语法，上面列举的4个中貌似只有redcarpet支持，所以此处采用redcarpet。

##### 2）语法高亮显示引擎

常见的语法高亮引擎有：

* `pygments` (python实现，支持多达百种语言，支持多种主题，速度较慢)
* `coderay` (ruby实现，速度非常快，提供的功能特性有限，支持的语言种类也不多)

除此之外，还可以使用javascript来渲染，比如`highlight.js`。

`highlight.js`目前支持50多种语言和20多种主题，而且可以自动检测语言。

#### 2、设置

此处介绍使用redcarpet+pygments和redcarpet+highlight.js两种设置。

##### 1）设置redcarpet

首先，安装redcarpet：

```
sudo gem install redcarpet
```

然后，在/_config.yml中设置：

```yaml
markdown: redcarpet
redcarpet:
  extensions: ["no_intra_emphasis", "fenced_code_blocks", "autolink", "strikethrough", "superscript", "with_toc_data"]
```

##### 2）设置pygments

首先，安装pygments(Ubuntu):

```
sudo apt-get install python-pygments
```

然后，在/_config.yml中设置：

```yaml
pygments: true
```

在jekyll目录下执行：

```bash
mkdir -p ./assets/themes/twitter/pygments/styles
cd ./assets/themes/twitter/pygments/styles
pygmentize -S default -f html > default.css
```

可以把`default`换成其他主题，如`vs`、`emacs`(推荐)等。关于主题，请看这个[例子](http://pygments.org/demo/82675/)。

最后，打开/_includes/themes/twitter/default.html，在head部分添加以下内容：

```html
{% if site.pygments %}
  <!-- highlight with pygments -->
  <link href="{{ ASSET_PATH }}/pygments/styles/emacs.css" rel="stylesheet">
{% endif %}
```

##### 3）设置highlight.js
	
可以直接打开/_includes/themes/twitter/default.html，在head部分添加以下内容即可：

```html
<!-- highlight with highlight.js -->
<link rel="stylesheet" href="http://yandex.st/highlightjs/7.3/styles/default.min.css">
<script src="http://yandex.st/highlightjs/7.3/highlight.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

也可以采用我的办法：

首先，下载[highlight.js](http://softwaremaniacs.org/soft/highlight/en/download/)，解压到/assets/themes/twitter/highlightjs。

然后，在打开/_includes/themes/twitter/default.html，在head部分添加以下内容：

```html
{% if site.highlightjs and page.langs != null %}
  <!-- highlight with highlight.js -->
  <link href="{{ ASSET_PATH }}/highlightjs/styles/vs.css" rel="stylesheet">
  <script src="{{ ASSET_PATH }}/highlightjs/highlight.pack.js"></script>
  {% if page.langs == empty %}
    <script>hljs.initHighlightingOnLoad();</script>
  {% else %}
    <script>hljs.initHighlightingOnLoad({{ page.langs | join:"','" | prepend:"'" | append:"'" }});</script>
  {% endif %}
{% endif %}
```
此处演示的是vs主题，可以自己更换。

最后，在/_config.yml中，添加：

```yaml
highlightjs: true
```

这么做的好处是：

* 可以在/_config.yml中设置`highlightjs: false`来关闭highlightjs.
* 可以在每个post的YAML头部中添加类似`langs: [bash, python]`的项来指定用到的语言，这可以加快highlight.js的加载速度。


### 三、设置评论、分析系统

#### 1、bootstrap

##### 1）评论系统

我用的是disqus。

由于bootstrap已经内置了评论功能，我们只需做简单的设置即可。

打开`/_config.yml`，设置"comments"下的"provider"为"disqus"，然后在"disqus"下设置"short_name"即可。就像我的：

```yaml
comments :
  provider : disqus
  disqus :
    short_name : liberize
```

如果你还没有disqus账号，请到[官网](http://disqus.com/)注册，你将得到一个short_name，填入即可。

设置`provider: false`关闭全局评论功能，或者在page/post的YAML头部设置`comments: false`关闭该page/post的评论功能。

##### 2）分析系统

我用的是google analytics.

同样的，我们作如下设置：

/_config.yml：

```yaml
analytics :
  provider : google
  google : 
  tracking_id : 'UA-123-12'
```

如果还没有tracking_id，请到[官网](http://www.google.com/analytics/)申请。

设置`provider: false`关闭全局分析功能，或者在page/post的YAML头部设置`analytics: false`关闭该page/post的分析功能。

#### 2、其它

如果，你用的不是bootstrap，请参照disqus和google analytics的提示添加相应的代码到你的网站模板中。


### 四、添加分享按钮

#### 1、方法

##### 1）国内网站
	
方法很简单，到[JiaThis](http://www.jiathis.com/index2)、[bShare](http://www.bshare.cn/)、[passit](http://www.passit.cn/)或[百度分享](http://share.baidu.com)上遵照提示定制样式，复制相应的代码到你的post模板中。

post模板位置在/_includes/themes/twitter/post.html。
	
##### 2）google+

到[google官网](https://developers.google.com/+/web/+1button/)遵照提示定制google+按钮，复制相应的代码到你的post模板中。
	
##### 3）twitter

到[twitter官网](https://dev.twitter.com/docs/tweet-button)遵照提示定制tweet按钮，复制相应的代码到你的post模板中。

#### 2、代码

以下是我的代码：

post.html：

```html
<div class="row-fluid post-full">
  <div class="span12">
    <div class="date">
      <span>{{ page.date | date_to_long_string }}</span>
    </div>
    {% include tweaks/share %}
    <div class="content" style="clear:both;">
      {{ content }}
    </div>
</div>
```

/_includes/tweaks/share：

```html
<!-- share buttons -->
<!-- google plus button -->
<div class="g-plusone pull-left" data-size="medium" data-annotation="inline" data-width="300"></div>
<script type="text/javascript">
  window.___gcfg = {lang: 'zh-CN'};
  (function() {
    var po = document.createElement('script'); po.type = 'text/javascript'; po.async = true;
    po.src = 'https://apis.google.com/js/plusone.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(po, s);
  })();
</script>
<!-- other buttons -->
<div class="jiathis_style pull-right">
	<a class="jiathis_button_qzone"></a>
	<a class="jiathis_button_tsina"></a>
	<a class="jiathis_button_tqq"></a>
	<a class="jiathis_button_weixin"></a>
	<a class="jiathis_button_renren"></a>
	<a class="jiathis_button_xiaoyou"></a>
	<a href="http://www.jiathis.com/share" class="jiathis jiathis_txt jtico jtico_jiathis" target="_blank"></a>
	<a class="jiathis_counter_style"></a>
</div>
<script type="text/javascript" src="http://v3.jiathis.com/code/jia.js?uid=1367154719992159" charset="utf-8"></script>
```

### 五、更改主题

#### 1、主题

请看官网提供的[主题](http://themes.jekyllbootstrap.com/)。

主题安装及更换方法：

```bash
cd /path/to/your/jekyll/
# install via git
rake theme:install git="https://github.com/jekyllbootstrap/theme-the-program.git"
# or by name
rake theme:install name="the-program"
# switch themes
rake theme:switch name="the-program"
```

#### 2、样式

你也可以替换twitter中的bootstrap.min.css：

到[此处](http://bootswatch.com/)查看效果并下载css。


### 六、参考网址

* [jekyll官网](http://jekyllrb.com/)
* [bootstrap官网](http://jekyllbootstrap.com/)

{% endraw %}
