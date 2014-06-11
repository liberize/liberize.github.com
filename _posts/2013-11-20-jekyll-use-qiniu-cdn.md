---
layout: post
title: "Jekyll 使用七牛云存储"
keywords: ["Jekyll", "七牛", "CDN"]
description: "Jekyll 博客使用七牛云存储的方法"
category: "tech"
tags: ["jekyll", "cdn"]
---
{% include JB/setup %}
{% raw %}

使用 Github Page 的博客由于服务器在国外，访问速度不佳，可以使用国内的 CDN 服务存储图片、js/css 等静态文件来为网站提速。事实上，因为整个网站都是静态的，可以把所有文件都放在 CDN 服务器上，但是这样就无法使用 Git 了。如果使用 Bootstrap 框架，可以将 assets 文件夹和图像放在 CDN 服务器上，以兼顾两头。

目前国内的 CDN 提供商有七牛、又拍等，此处选择七牛。七牛按存储空间、流量、请求次数计费，提供一定的免费额度，对于小站来说基本够用了，而且还有一些优惠活动。如果还没有七牛账号，可以点[这里](http://portal.qiniu.com/signup?code=3lh1xdgm7zxaq)注册（这是我的邀请链接）。

假设网站资源文件都放在 /assets 目录下，图片放在 /images 目录下，要将这两个文件夹里的内容使用七牛加速，可以参考以下做法。

## 一、设置七牛

在七牛新建一个 bucket，命名为 liberize，设置为公开空间，则这个 bucket 里面的内容可以通过 `liberize.u.qiniudn.com` 访问。在"空间设置" - "基本设置" - "镜像存储"里面设置镜像源为博客地址 `http://liberize.me/`。设置了镜像存储以后，当访问 bucket 里面不存在的内容时会自动从镜像源获取。

## 二、修改_config.xml

在 _config.xml 文件中按如下设置。

```yaml
safe: false
cdn_url : http://liberize.u.qiniudn.com

JB :
  ASSET_PATH : false
  IMAGE_PATH : false
```

## 三、修改_includes/JB/setup

修改 `_includes/JB/setup` 里面 `{% if site.JB.ASSET_PATH %}` 语句块，增加对 `site.safe` 的判断，同时类比写出 IMAGE_PATH 部分。这么做的好处是本地测试时将使用本地资源，而提交到 GitHub 后，将自动使用 `site.cdn_url` 指定的 CDN 服务器上的资源。为什么会有这种效果呢？因为 GitHub 默认是禁止插件的，在运行 jekyll 时会加上 `--safe` 参数，这将会覆盖 _config.yml 中的 `safe: false` 设置。

```
{% if site.JB.ASSET_PATH %}
  {% assign ASSET_PATH = site.JB.ASSET_PATH %}
{% elsif site.safe %}
  {% capture ASSET_PATH %}{{ site.cdn_url }}/assets/themes/{{ page.theme.name }}{% endcapture %}
{% else %}
  {% capture ASSET_PATH %}{{ BASE_PATH }}/assets/themes/{{ page.theme.name }}{% endcapture %}
{% endif %}

{% if site.JB.IMAGE_PATH %}
  {% assign IMAGE_PATH = site.JB.IMAGE_PATH %}
{% elsif site.safe %}
  {% capture IMAGE_PATH %}{{ site.cdn_url }}/images{% endcapture %}
{% else %}
  {% capture IMAGE_PATH %}{{ BASE_PATH }}/images{% endcapture %}
{% endif %}
```

以后需要在文章中插入图片时，只需将图片放到 /images 文件夹，然后使用 `{{ IMAGE_PATH }}/图片文件名` 格式就可以了。由于设置了镜像存储，不需要手动上传，但是当更改了 /assets 或 /images 里面的文件时，应当删除七牛服务器上的对应资源，否则访问的仍然是旧的资源。

除了用来做图床、CDN 加速以外，七牛还有图片处理（缩放、旋转、打水印等）、防盗链等功能，请参考官方文档。

{% endraw %}
