---
layout: post
title: "Jekyll 博客基于 Feed 实现静态搜索"
keywords: ["Jekyll", "Feed", "静态", "搜索"]
description: "一种比较理想的基于 Feed 的静态博客搜索的实现方式"
category: "web"
tags: ["jekyll", "搜索"]
---
{% include JB/setup %}

折腾 jekyll 有一段时间了，也积累了一点经验，趁还没忘赶紧记录下来，分享给大家。今天先来说说静态搜索的实现。由于静态博客没有后台，目前站内搜索大多使用 Google 自定义搜索，缺点是 Google 在国内访问速度慢且不稳定，最近百度也推出了站内搜索，据说可以搜到没被收录的内容，可是目前还处于内测中，小站长根本没机会。在网上转了一圈，发现有几篇文章提到先生成一个含有标题、日期、链接等信息的文件 `search.xml`，然后用 js 搜索这个文件的方法，但是这种方法功能太过简单。换英文搜了一下，发现有人提出了一个类似的但功能更完善的方法 —— 搜索 RSS Feed 或 Atom Feed。这种方法好处在于不用生成额外的文件，而且可以通过 Feed 中的内容摘要来搜索文章内容。本博客目前使用的就是这种方法，还有人专门建了一个网站为静态博客提供这种服务。

## 一、定义表单

因为表单不是真的要提交，只需要一个输入框就可以了，还可以加上一个按钮。

```html
<form id="search-form">
  <input id="query" type="text" placeholder="正则搜索"></input>
  <button type="submit"><i class="icon-search"></i></button>
</form>
```

## 二、定义相关div

因为使用 ajax 获取 feed 的 xml 文件，经过处理后将匹配的结果展现出来，所以需要定义两个 div，其中 `#loader` 显示一个动态图表示正在加载，`#main-content` 用来显示匹配的结果。

```html
<div id="loader"><img src="{{ ASSET_PATH }}/img/loading.gif" alt="请稍侯"></div>
<div id="main-content"></div>
```

## 二、编写js代码

代码不长，应该很容易看懂。大致思路是响应表单的提交事件，发出 ajax 请求，调用 `findEntries` 进行匹配。几个注意事项：

1. 由于使用了 jQuery，需要链接 jQuery 库文件。
2. 此处使用的是 Atom Feed，如果你用的是 RSS Feed，`findEntries` 需要做一些修改，因为两者格式不一样。
3. 此处使用正则表达式进行搜索，想搜索 `c++`，应该输入 `c\+\+`。如果不想使用正则表达式，请自行修改。

```javascript
$(document).ready(function() {
  var entries = null;

  function formatDate(date) {
    var monthNames = [ "January", "February", "March", "April", "May", "June",
      "July", "August", "September", "October", "November", "December" ];
    return date.getDate() + ' ' + monthNames[date.getMonth()] + ' ' + date.getFullYear();
  }

  function findEntries(q) {
    var matches = [];
    var rq = new RegExp(q, 'im');
    var rl = /^http:\/\/liberize\.me\/post\/(.+)\.html$/;
    for (var i = 0; i < entries.length; i++) {
      var entry = entries[i];
      var title = $(entry.getElementsByTagName('title')[0]).text();
      var link = $(entry.getElementsByTagName('link')[0]).attr('href');
      var title_en = rl.exec(link)[1].replace(/-/g, ' ');
      var content = $(entry.getElementsByTagName('content')[0]).text();          
      if (rq.test(title) || rq.test(title_en) || rq.test(content)) {
        var updated = formatDate(xmlDateToJavascriptDate($(entry.getElementsByTagName('updated')[0]).text()));
        matches.push({'title': title, 'link': link, 'date': updated, 'content': content});
      }
    }
    var html = '';
    for (var i = 0; i < matches.length; i++) {
      var match = matches[i]; 
      html += '<article class="nested">';
      html += '<header><h2><a href="' + match.link + '">' + htmlEscape(match.title) + '</a></h2></header>';
      html += '<section><p>' + htmlEscape(match.content) + '</p></section>';
      html += '<footer><p>更新日期：' + match.date + '</p></footer>';
      html += '</article>';
    }
    $('#main-content').html(html);
    $('#loader').hide();
    $('#main-content').show();
  }

  $('#search-form').submit(function() {
    var query = $('#query').val();
    $('#query').blur().attr('disabled', true);
    $('#main-content').hide();
    $('#loader').show();
    if (entries == null) {
      $.ajax({url: '/atom.xml?r=' + (Math.random() * 99999999999), dataType: 'xml', success: function(data) {
        entries = data.getElementsByTagName('entry');
        findEntries(query);
      }});
    } else {
      findEntries(query);
    }
    $('#query').blur().attr('disabled', false);
    return false;
  });

});
```

其中，用到的 htmlEscape 和 xmlDateToJavascriptDate 函数定义如下。

```javascript
function htmlEscape(s) {
  return String(s).replace(/[&<>"'\/]/g, function(s) {
    var entityMap = {
      "&": "&amp;",
      "<": "&lt;",
      ">": "&gt;",
      '"': '&quot;',
      "'": '&#39;',
      "/": '&#x2F;'
    };
    return entityMap[s];
  });
}

function xmlDateToJavascriptDate(xmlDate) {
  var re = /^([0-9]{4,})-([0-9]{2})-([0-9]{2})T([0-9]{2}):([0-9]{2}):([0-9]{2})(\.[0-9]+)?(Z|([+-])([0-9]{2}):([0-9]{2}))?$/;
  var match = xmlDate.match(re);
  if (!match)
    return null;

  var all = match[0];
  var year = match[1];  var month = match[2];  var day = match[3];
  var hour = match[4];  var minute = match[5]; var second = match[6];
  var milli = match[7]; 
  var z_or_offset = match[8];  var offset_sign = match[9]; 
  var offset_hour = match[10]; var offset_minute = match[11];

  if (offset_sign) {
    var direction = (offset_sign == "+" ? 1 : -1);
    hour =   parseInt(hour)   + parseInt(offset_hour)   * direction;
    minute = parseInt(minute) + parseInt(offset_minute) * direction;
  }
  month = parseInt(month) - 1;
  var utcDate = Date.UTC(year, month, day, hour, minute, second, (milli || 0));
  return new Date(utcDate);
}
```

第一次搜索因为需要取回 xml 会比较慢，后面每次搜索都非常快。另外，使用这种方法不建议将 feed 设为全文输出，因为速度会比较慢，尤其是在文章很多的情况下。

为简单起见，没有加入返回搜索前页面的功能，可以使用 `history.js` 来实现。
