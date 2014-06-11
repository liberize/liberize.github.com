---
layout: post
title: "Git 入门"
keywords: ["Git"]
description: "Git 入门简介"
category: "tech"
tags: ["git"]
---
{% include JB/setup %}

### Git主要优势及安装

git，一个非常强大的版本管理工具。Github则是一个基于Git的日益流行的开源项目托管库。Git与svn的最大区别是，它的使用流程不需要联机，可以先将对代码的修改，评论，保存在本机。等上网之后，再实时推送过去。同时它创建分支与合并分支更容易，推送速度也更快，配合Github提交需求也更容易。

git的入门，稍微有点麻烦，需要在本机创建一个ssh的钥匙，其他的则海空天空了。

### Git全局设置

下载并安装Git

	git config --global user.name "Your Name"
	git config --global user.email youremail@email.com

### 将Git项目与Github建立联系

	mkdir yourgithubproject
	cd yourgithubproject
	git init
	touch README
	git add README
	git commit -m 'first commit'
	git remote add origin git@github.com:yourgithubname/yourgithubproject.git
	git push origin master

### 导入现有的Git仓库

	cd existing_git_repo
	git remote add origin git@github.com:yourgithubname/yourgithubproject.git
	git push origin master

### git最主要的命令

	git --help

The most commonly used git commands are:

	add        Add file contents to the index  
	bisect     Find by binary search the change that introduced a bug  
	branch     List, create, or delete branches  
	checkout   Checkout a branch or paths to the working tree  
	clone      Clone a repository into a new directory  
	commit     Record changes to the repository  
	diff       Show changes between commits, commit and working tree, etc  
	fetch      Download objects and refs from another repository  
	grep       Print lines matching a pattern  
	init       Create an empty git repository or reinitialize an existing one  
	log        Show commit logs  
	merge      Join two or more development histories together  
	mv         Move or rename a file, a directory, or a symlink  
	pull       Fetch from and merge with another repository or a local branch  
	push       Update remote refs along with associated objects  
	rebase     Forward-port local commits to the updated upstream head  
	reset      Reset current HEAD to the specified state  
	rm         Remove files from the working tree and from the index  
	show       Show various types of objects  
	status     Show the working tree status  
	tag        Create, list, delete or verify a tag object signed with GPG  

### 第一次提交的时候

	git push origin master

### 日常提交常用命令

	git add .
	git commit -m "some files"
	git push origin master

附：

* [git 简易指南，木有高深内容](http://www.bootcss.com/p/git-guide/)
