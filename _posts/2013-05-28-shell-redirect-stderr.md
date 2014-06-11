---
layout: post
title: "Shell 重定向标准错误流"
keywords: ["Shell", "重定向", "标准错误"]
description: "Shell 重定向标准错误流简介"
category: "tech"
tags: ["shell"]
---
{% include JB/setup %}

### 文件描述符

文件描述符是程序发送输出和获取输入的地方。当执行一个程序时，运行该程序的进程打开了3个文件描述符，分别是：0(_标准输入_)、1(_标准输出_)和2(_标准错误输出_)。重定向输出符号(`>`)是`1>`的简写，它通知shell重定向标准输出。类似地，`<`是`0<`的简写，表示重定向标准输入。符号`2>`将重定向标准错误输出。

下面的示例演示了如何将标准输出和标准错误输出重定向到不同的文件和相同的文件。当运行cat时，如果所带参数中的某个文件不存在，而另一个文件存在，那么cat将发送一条错误消息到标准错误输出，同时还将已存在的那个文件复制一份到标准输出。除非重定向，否则两条消息都将出现在屏幕上。

    $ cat y
	This is y.
	$ cat x
	cat: x: No such file or directory

	$ cat x y
	cat: x: No such file or directory
	This is y.

将命令的标准输出重定向时，发送到标准错误输出的输出将不受影响，仍然出现在屏幕上。

    $ cat x y > hold
	cat: x: No such file or directory
	$ cat hold
	This is y.

类似地，当使用管道发送标准输出时，标准错误输出也不会受到影响。下面的示例将cat的标准输出通过管道发送给tr(在本例中，这个程序将小写字母转换为大写字母)。cat发送到标准错误输出的文本并没有转换，这是因为它直接发送到屏幕，并没有经过这个管道。

    $ cat x y | tr "[a-z]" "[A-Z]"
	cat: x: No such file or directory
	THIS IS Y.

下面的示例将标准输出和标准错误输出重定向到不同的文件中。符号2>告诉shell将标准错误输出(文件描述符为2)重定向到的具体位置。1>告诉shell将标准输出(文件描述符为1)重定向到的具体位置。可以使用>代替`1>`。

    $ cat x y 1> holdl 2> ho1d2
	$ cat holdl
	This "is y.
	$ cat ho1d2
	cat: x: No such file or directory

### 复制文件描述符

在下一个示例中，`1>`将标准输出重定向到文件hold。然后，`2>&1`声明文件描述符2为文件描述符1的副本。这样做的结果是，标准输出和标准错误输出均被重定向到文件hold中。

    $ cat x y 1> hold 2>&1
	$ cat hold
	cat: x: No such file or directory
	This is y.

在这个示例中，`1>hold`放在了`2>&1`的前面。如果将它们的顺序颠倒的话，在标准输出重定向到文件hold之前，标准错误输出就已经复制了标准输出的一个副本。这样一来，就只有标准输出被重定向到文件hold。

在下面的示例中，文件描述符2是文件描述符1的副本，通过一个到tr命令的管道将输出发送到文件描述符1。

    $ cat x y 2>&1 | tr "[a-z]" "[A-Z]"
	CAT: X: NO SUCH FILE OR DIRECTORY
	THIS IS Y.

发送错误到标准错误输出  还可以使用1>&2将命令的标准输出重定向到标准错误输出。shell脚本中经常使用这项技术将echo的输出发送到标准错误输出。在下面的脚本中，第1个echo命令的标准输出被重定向到标准错误输出：

	$ echo This is an error message. 1>&2
	$ echo This is not an error message.

在脚本中，还可以使用内置命令exec创建另外的文件描述符，并重定向shell脚本的标准输入、标准输出和标准错误输出。

### 重定向操作符

* `<filename`   
将标准输入重定向为文件filename

* `>filename`   
除非文件filename已存在并且设置了noclobber标记，否则标准输出将被重定向到文件filename；如果文件filename不存在且没有设置noclobber标记，那么重定向操作将创建该文件

* `>|filename`   
即使文件filename存在且设置了noclobber标记，仍将标准输出重定向到该文件

* `>>filename`   
除非文件filename已存在并且设置了noclobber标记，否则标准输出将被重定向到文件filename，并将内容添加到原文件的末尾；如果没有设置noclobber标记，并且文件filename不存在，那么将创建该文件

* `<&m`   
从文件描述符m复制标准输入

* `[n]>&m`   
从文件描述符m复制标准输出或者文件描述符n(如果命令中指定了n)

* `[n]<&-`   
关闭标准输入或者文件描述符n(如果指定了n)

* `[n]>&-`   
关闭标准输出或者文件描述符n(如果指定了n)
