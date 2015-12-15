---
layout: post
title: "多说实时邮件提醒"
keywords: ["多说", "邮件", "提醒", "通知"]
description: "当有人在文章下评论时给博主发送实时邮件提醒"
category: "tech"
tags: ["多说"]
---
{% include JB/setup %}

用多说有一段时间了，也发现了不少问题，比如加载速度越来越慢，服务不稳定，然而最不能忍受的是，当有人在文章下发表评论时，博主居然得不到任何通知，只能隔一段时间去多说的后台检查一下。Google 了一下，看到几篇文章里面说，导入一个用户，并指定 `data-author-key` 为用户 id，就可以有每日提醒了。但是我照着做了一下，在后台也可以看见导入的用户，然后仍然没有收到任何邮件。

后来发现了另外一种方法可以发送实时邮件提醒，基本原理就是不断轮询多说获取实时评论的 API，发现了新评论就自动发邮件到指定地址。看了下作者在 GitHub 上的[代码](https://github.com/clinyong/duoshuo)，本来准备直接拿来用，随后发现了一些问题，于是我重写了一下脚本。

## 一、多说实时评论 API

API 地址为 `http://api.duoshuo.com/log/list.json`。请到官网查看[详细文档](http://dev.duoshuo.com/docs/50037b11b66af78d0c000009)。

注意有个 `limit` 参数，也就是每次只能得到 1-200 条评论，如果评论总数超过 200 条，用这个接口也只能得到最近的 200 条，所以原作者那种根据返回的评论条数来判断有没有新评论的方法是不合理的（而且他没有加 `order=desc`，得到的其实永远是最旧的 50 条）。

注意到返回数据里面有个 `log_id`，这个 id 可以认为是评论的序号，它是递增的，因此我们只需要每次取最近的 50 条，然后往前找上一次查询时得到的最新的 `log_id`，这中间的就都是新评论了。 

另外，多说的接口返回的数据其实是操作日志，所以删除评论的日志也在里面，我们只关心 `action` 为 `create` 的日志。

由于返回数据里面只有 `thread_key`，我的 `thread_key` 是文章的标题，文章多的时候还要根据标题找一下，很不方便。看了下多说的其它接口，有一个获取文章评论的接口可以根据 `thread_key` 得到文章 url：`http://api.duoshuo.com/threads/listPosts.json`，于是把这个接口也用上了。

```python
last_log_id = None

def get_response(base_url, params):
    url = '{}?{}'.format(base_url, urllib.urlencode(params))
    try:
        data = urllib.urlopen(url).read()
        resp = convert(json.loads(data))
        if resp['code'] != 0:
            return None
        return resp
    except Exception, e:
        print str(e)
        return None

def check(duoshuo):
    global last_log_id
    log_data = get_response('http://api.duoshuo.com/log/list.json', {
        'short_name': duoshuo['short_name'],
        'secret': duoshuo['secret'],
        'order': 'desc'
    })
    if log_data is None:
        return None

    comments = []
    if last_log_id:
        for log in log_data['response']:
            if log['log_id'] == last_log_id:
                break
            if log['action'] == 'create' and log['meta']['author_name'] != duoshuo['short_name']:
                thread_data = get_response('http://api.duoshuo.com/threads/listPosts.json', {
                    'short_name': duoshuo['short_name'],
                    'thread_key': log['meta']['thread_key']
                })
                comments.append({
                    'title': log['meta']['thread_key'],
                    'url': thread_data['thread']['url'] if thread_data else '',
                    'name': log['meta']['author_name'],
                    'email': log['meta']['author_email'],
                    'content': log['meta']['message'],
                    'time': log['meta']['created_at'].split('+')[0].replace('T', ' ')
                })
    last_log_id = log_data['response'][0]['log_id']
    return comments
```

## 二、发送邮件提醒

发邮件仍然是用的 smtplib，用起来很方便。

由于加上了文章链接和用户的 email 地址，邮件内容不再使用纯文本，改用简单的 html，将前面 check 得到的结果套入模板，渲染成 html：

```python
content_tmpl = '''
<html>
<head><meta http-equiv=Content-Type content="text/html;charset=utf-8"><title>{title}</title></head>
<body>{body}</body>
</html>
'''
comment_tmpl = '''
<p><a href="mailto:{email}">{name}</a> 于 {time} 在文章《<a href="{url}">{title}</a>》下发表了评论：</p>
<p>{content}</p>
'''

def render_content(comments):
    global content_tmpl, comment_tmpl
    comments = [comment_tmpl.format(**comment) for comment in comments]
    content = content_tmpl.format(title='多说', body=''.join(comments))
    return content

def send_email(email, content):
    msg = MIMEText(content, _subtype='html', _charset='utf-8')
    msg['Subject'] = '多说评论通知'
    msg['From'] = email['from']
    msg['To'] = email['to']
    msg['X-Mailer'] = 'Microsoft Outlook 14.0'
    try:
        server = smtplib.SMTP()
        server.connect(email['host'])
        server.login(email['name'], email['password'])
        server.sendmail(email['from'], [email['to']], msg.as_string())
        server.close()
        return True
    except Exception, e:
        print str(e)
        return False
```

最后发出去的邮件是这样的：

![收到的多说邮件提醒]({{ IMAGE_PATH }}/duoshuo-mail-notification.png)

完整代码见 [Gist](https://gist.github.com/liberize/c270eaff222047df9246)。

## 三、脚本的运行

使用 crontab 定期执行，以一分钟执行一次为例，编辑 crontab 添加：

```
* * * * * python /path/to/duoshuo.py
```
