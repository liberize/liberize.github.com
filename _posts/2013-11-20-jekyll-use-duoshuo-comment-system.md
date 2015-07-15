---
layout: post
title: "Jekyll 使用多说评论系统"
keywords: ["Jekyll", "多说", "评论系统"]
description: "Jekyll 博客使用多说评论系统的方法"
category: "tech"
tags: ["jekyll", "评论"]
---
{% include JB/setup %}
{% raw %}

静态博客要实现评论功能必须依赖第三方评论系统。使用 Bootstrap 框架的 Jekyll 博客默认的评论系统是 Disqus，然而 Disqus 在国内访问速度和稳定性并不理想，而且无法和国内的各种社交网站耦合，因此 Disquz 并不适合面向国内用户的网站。在国内也有诸多社会化评论系统，如多说、友言、灯鹭、评论啦等，这些基本都是 Disqus 的本地化版本，功能和 Disqus 非常相似。据观察，目前最受欢迎、评价最高的应该是多说了，本博客使用的评论系统就是多说。

多说官网提供了很多插件供不同类型的网站使用，对静态博客可以使用通用代码。

## 一、添加多说

添加多说非常简单，只需要加入一段 js 代码，然后在需要显示评论框的地方插入一个 div 标签即可。除了评论框以外，多说还支持显示评论次数、最新评论、最近访客或热评文章，使用方法基本相同。

此处仅演示评论框的使用。可以直接将下面一段代码加入到你的 post 模板中，注意设置 `short_name`。忍不住吐槽一句，多说的 js 代码跟 Disqus 的简直如出一辙，赤果果的抄袭呀。

```html
<div id="ds-thread" class="ds-thread" data-url="{{ page.url }}" data-title="{{ page.title }}" data-thread-key="{{ page.title }}"></div>
<script type="text/javascript">
var duoshuoQuery = {short_name: '{{ site.JB.comments.duoshuo.short_name }}'};
(function() {
    var ds = document.createElement('script');
    ds.type = 'text/javascript';ds.async = true;
    ds.src = 'http://static.duoshuo.com/embed.js';
    ds.charset = 'UTF-8';
    (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(ds);
})();
</script>
```

如果你使用的是 Bootstrap 框架，这里推荐另一种做法。

首先将以上代码保存为 duoshuo，放到 `_includes/JB/comments-providers` 文件夹下。然后修改 `_includes/JB/comments` 文件，加入多说的判断语句，如下所示，

```
{% case site.JB.comments.provider %}
{% when "duoshuo" %}
  {% include JB/comments-providers/duoshuo %}
{% when "disqus" %}
  {% include JB/comments-providers/disqus %}
...
{% endcase %}
```

最后在 `_config.yml` 中的 `comments` 下面添加多说，设置 `short_name`，并将 `provider` 改为 `duoshuo`。

```yaml
comments :
  provider : duoshuo
  duoshuo :
    short_name : liberize
  disqus :
      short_name : liberize
  # ...
```

这样就将多说无缝地融入了 Bootstrap 框架，以后若要修改为其他评论系统，直接修改 `provider` 就可以了。

## 二、修改多说样式

最近比较流行圆形头像，咱也来试试。在你的 css 文件中添加以下代码，不仅可以让头像变成圆形，还可以让头像在鼠标放上去时进行360度旋转哦。

```css
#ds-reset .ds-avatar img {
  width: 54px !important;
  height: 54px !important;
  -webkit-border-radius: 27px !important;
  -moz-border-radius: 27px !important;
  border-radius: 27px !important;
  -webkit-transition: -webkit-transform 0.4s ease-out;
  -moz-transition: -moz-transform 0.4s ease-out;
  transition: transform 0.4s ease-out;
}
#ds-reset .ds-avatar img:hover {
  -webkit-transform: rotateZ(360deg);
  -moz-transform: rotateZ(360deg);
  transform: rotateZ(360deg);
}
```

不喜欢评论框底下显示“xx正在使用多说”等字样？咱把它隐藏掉！

```css
#ds-reset .ds-powered-by {
  display: none;
}
```

{% endraw %}
