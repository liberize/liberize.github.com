---
layout: post
title: "Shell sudo 的若干问题"
keywords: ["Shell", "sudo"]
description: "Shell sudo 的若干问题"
category: "tech"
tags: ["shell"]
---
{% include JB/setup %}

### 1、'sudo echo x > file' 时 'Permission denied'

	sudo echo 268435456 > /proc/sys/kernel/shmmax
	bash: /proc/sys/kernel/shmmax: Permission denied

这时 bash 拒绝这么做，说是权限不够。这是因为重定向符号 “>” 和 “>>” 也是 bash 的命令。sudo 只是让 echo 命令具有了 root 权限，但是没有让 “>” 和 “>>” 命令也具有root 权限，所以 bash 会认为这两个命令都没有写入信息的权限。

解决这一问题的途径有两种。第一种是*利用 “`sh -c`” 命令*，它可以让 bash 将一个字串作为完整的命令来执行，这样就可以将 sudo 的影响范围扩展到整条命令。具体用法如下：

	sudo sh -c ‘echo 268435456 > /proc/sys/kernel/shmmax’

另一种方法是*利用管道和 tee 命令*，该命令可以从标准输入中读入信息并将其写入标准输出或文件中，具体用法如下：

	echo "268435456" | sudo tee -a test.txt

注意，tee命令的作用是读取标准输入并同时写入到标准输出和文件。tee 命令的 “-a” 选项的作用等同于 “>>” 命令，如果去除该选项，那么 tee 命令的作用就等同于 “>” 命令。

### 2、cd时的错误

	sudo cd /var/log/audit
	sudo: cd: command not found

cd是shell内置的,不是普通的命令,所以不能通过sudo运行。如果确实需要运行cd,可以先输入`sudo -s`，变成root@hostname再cd

### 3、sudo时找不到命令

sudo有时候会出现找不到命令，而明明PATH路径下包含该命令，让人疑惑。其实出现这种情况的原因，主要是因为当 sudo以管理权限执行命令的时候，linux将PATH环境变量进行了重置，当然这主要是因为系统安全的考虑，但却使得sudo搜索的路径不是我们想要的PATH变量的路径，当然就找不到我们想要的命令了。两种方法解决该问题：

首先，都要打开sudo的配置文件：`sudo gedit /etc/sudoers`

1）*可以使用 `secure_path` 指令修改 sudoers 中默认的 PATH为你想要的路径*。这个指令指定当用户执行 sudo 命令时在什么地方寻找二进制代码和命令。这个选项的目的显然是要限制用户运行 sudo 命令的范围，这是一种好做法。

2）*将`Defaults env_reset`改成`Defaults !env_reset`取消掉对PATH变量的重置*，然后在.bashrc中最后添加alias sudo='sudo env PATH=$PATH'，这样sudo执行命令时所搜寻的路径就是系统的PATH变量中的路径，如想添加其他变量也是类似。
