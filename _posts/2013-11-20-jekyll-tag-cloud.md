---
layout: post
title: "Jekyll 不用 JS 生成标签云"
keywords: ["Jekyll", "标签云"]
description: "一种使用纯 Liquid 语法、不用 JavaScript 生成标签云的方法"
category: "tech"
tags: ["jekyll", "标签云"]
---
{% include JB/setup %}
{% raw %}

标签云是网站中的内容标签的视觉化描述。透过标签云，网站访客可以很直观地看到某个标签下的文章数目多少。对于动态博客，可以在后台查询数据库，生成标签云；对于静态博客，一般需要使用 JavaScript。使用 js 生成标签云的基本思路是：首先生成携带文章数信息的标签列表，然后通过 js 计算并修改每个标签的样式，比如颜色、大小等等。事实上，这些工作可以在生成静态网页时完成，无需使用 js。不使用 js 的原因是：静态网站要实现动态网站相同的功能，基本都需要将后台 php 等做的工作转成前台 js 来实现，这会导致在网页中使用过多 js 而带来一些不良影响。Jekyll 使用 Liquid 模板引擎来处理模板，而 Liquid 本身有一些简单的计算功能，下面就介绍如何使用 Liquid 来生成标签云。如果你对 Liquid 不太熟悉，可以在[这儿](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers)了解一下。

## 一、统计每个标签下的文章数，得到最大最小值

此处先介绍一下 `site.tags` 变量。 `site.tags` 是一个数组，里面每一项对应一个标签，每一项本身又是一个数组，第一个元素是标签名，第二个元素是一个具有该标签的文章的数组。通过遍历 `site.tags` 就找到每个标签下文章数的最大最小值。

```
{% assign first = site.tags.first %}
{% assign max = first[1].size %}
{% assign min = max %}
{% for tag in site.tags offset:1 %}
  {% if tag[1].size > max %}
    {% assign max = tag[1].size %}
  {% elsif tag[1].size < min %}
    {% assign min = tag[1].size %}
  {% endif %}
{% endfor %}
{% assign diff = max | minus: min %}
```

## 二、计算每个标签的样式

此处，将字体大小范围设成 9pt ~ 18pt，字体大小可以取 9.5pt 等，因此总共分为 18 个区间；字体颜色范围设为 #999 ~ #000，为简单起见，只使用 #xxx 这种颜色（即颜色全是黑白的），因此总共分为 9 个区间。文章数最少的标签样式将是 `font-size: 9pt; color: #999;`，文章数最多的标签样式将是 `font-size: 18pt; color: #000;`，这样文章数越多字体越大、颜色越深，看起来越醒目。

需要注意的是，Liquid 只支持整数运算，因此需要做一些转换。另外，此处考虑的比较细致，采取四舍五入的方式进行近似，因此更为精确。

```
{% for tag in site.tags %}
  {% assign temp = tag[1].size | minus: min | times: 36 | divided_by: diff %}
  {% assign base = temp | divided_by: 4 %}
  {% assign remain = temp | modulo: 4 %}
  {% if remain == 0 %}
    {% assign size = base | plus: 9 %}
  {% elsif remain == 1 or remain == 2 %}
    {% assign size = base | plus: 9 | append: '.5' %}
  {% else %}
    {% assign size = base | plus: 10 %}
  {% endif %}
  {% if remain == 0 or remain == 1 %}
    {% assign color = 9 | minus: base %}
  {% else %}
    {% assign color = 8 | minus: base %}
  {% endif %}
  <a href="{{ site.JB.tags_path }}#{{ tag[0] }}-ref" style="font-size: {{ size }}pt; color: #{{ color }}{{ color }}{{ color }};">{{ tag[0] }}</a>
{% endfor %}
```

实际效果请看本博客左侧边栏。

{% endraw %}
