---
layout: post
title: "Jekyll 侧边栏滚动效果"
keywords: ["jekyll", "侧边栏", "滚动", "固定"]
description: "滚动到某个位置后保持侧边栏小工具固定的实现方法"
category: "tech"
tags: ["jekyll", "特效"]
---
{% include JB/setup %}
{% raw %}

这里要实现的效果是滚动到某个位置后保持侧边栏小工具位置固定，实际效果请看本博客侧边栏。其实这跟 jekyll 没什么关系，但因为同属博客折腾心得，所以姑且放在一块儿。之所以想到这个，是因为之前玩 WordPress 时在我爱水煮鱼的博客中见到过，感觉效果还不错，就想加到自己的博客里。

## 一、添加容器

定义一个 id 为 `fixed-container` 的 div，将小工具代码都放在里面，比如我放了 3 个小工具在里面。

```html
<div id="fixed-container">
  <section>
    {% include widgets/repos_list %}
  </section>
  <section>
    {% include widgets/contacts_list %}
  </section>
  <section>
    {% include widgets/links_list %}
  </section>
</div>
```

## 二、编写js代码

添加以下 js 代码，响应窗口的滚动事件。滚动到 `fixed-container` 顶端时，将其的样式设为 `position: fixed; top: 0;`，让它位置固定，否则设为 `position: static;` 让它恢复原样。

```javascript
$(document).ready(function() {
  var position = $('#fixed-container').offset();
  $(window).scroll(function() {
    if($(window).scrollTop() > position.top) {
      $('#fixed-container').css('position','fixed').css('top','0');
    } else {
      $('#fixed-container').css('position','static');
    }
  });
});
```

{% endraw %}
