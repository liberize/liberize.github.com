---
layout: post
title: "同时使用 GitHub 和 GitCafe 托管博客"
keywords: ["GitHub", "GitCafe", "博客"]
description: "同时使用 GitHub 和 GitCafe 托管博客的一种方法"
category: "tech"
tags: ["github", "gitcafe", "博客"]
---
{% include JB/setup %}

在 V2EX 上看到一个[帖子](http://v2ex.com/t/106559)，里面提到将博客同时托管在 GitHub 和 GitCafe 上，国外访客解析到 GitHub，国内访客解析到 GitCafe。禁不住 GitCafe 速度的诱惑，于是也动手在 GitCafe 上创建了一个博客镜像，并通过修改 git 设置实现了同步提交。

## 一、将仓库拷贝到 GitCafe

在 GitCafe 上创建一个和用户名相同的仓库，如我的是 liberize。

在账户设置中添加公钥，可以使用之前 GitHub 的公钥，只需要复制 ~/.ssh/id_rsa.pub 的内容。

打开 .git/config，修改远程仓库，将 origin 改为 github，并添加 gitcafe：

```ini
[remote "github"]
    fetch = +refs/heads/*:refs/remotes/github/*
    url = git@github.com:liberize/liberize.github.com.git
[remote "gitcafe"]
    fetch = +refs/heads/*:refs/remotes/gitcafe/*
    url = git@gitcafe.com:liberize/liberize.git
```

之后将仓库 push 到 GitCafe 上（必须使用 gitcafe-pages 分支）：

```
git push -u gitcafe master:gitcafe-pages
```

GitCafe 上在“项目管理”中找到“自定义域名”，添加要绑定的域名，比如我是 liberize.me。

## 二、实现同步提交

因为 GitHub 和 GitCafe 的 Pages 使用不同的分支，所以无法在 remote 里添加两个 url：

```ini
[remote "all"]
    url = git@github.com:liberize/liberize.github.com.git
    url = git@gitcafe.com:liberize/liberize.git
```

然后执行 `git push all master` 同时 push。

可以在 .git/config 中添加一个 alias 来实现：

```ini
[alias]
    publish = !sh -c \"git push github master && git push gitcafe master:gitcafe-pages\"
```

当需要 push 的时候，只需执行 `git publish` 就可以了。
