---
layout: post
title: "Shell 变量扩展"
keywords: ["Shell", "变量扩展"]
description: "Shell 变量扩展简介"
category: "tech"
tags: ["shell"]
---
{% include JB/setup %}

在做shell批处理程序时候，经常会涉及到字符串相关操作。有很多命令语句，如：awk,sed都可以做字符串各种操作。 其实shell内置一系列操作符号，可以达到类似效果，大家知道，使用内部操作符会省略启动外部程序等时间，因此速度会非常的快。

### 一、判断读取字符串值

表达式           | 含义
--------------- | ----------------------------------------------------
${var} 	        | 变量var的值, 与$var相同
${var-DEFAULT} 	| 如果var没有被声明, 那么就返回DEFAULT
${var:-DEFAULT} | 如果var没有被声明, 或者其值为空, 那么就返回DEFAULT
${var=DEFAULT} 	| 如果var没有被声明, 那么就将DEFAULT赋给它并返回其值
${var:=DEFAULT} | 如果var没有被声明, 或者其值为空, 那么就将DEFAULT赋给它并返回其值
${var+OTHER} 	| 如果var声明了, 那么返回OTHER, 否则就返回空字符串
${var:+OTHER} 	| 如果var被设置了, 那么返回OTHER, 否则就返回空字符串
${var?ERR_MSG} 	| 如果var没被声明, 那么就打印 var: ERR_MSG 并异常终止
${var:?ERR_MSG} | 如果var没被设置, 那么就打印 var: ERR_MSG 并异常终止
${!varprefix*} 	| 返回之前已设置的以varprefix开头的变量名列表，以$IFS分隔，类似$*
${!varprefix@} 	| 返回之前已设置的以varprefix开头的变量名列表，与$IFS无关，类似$@

一个简单的例子：

```
$ echo ${abc-'ok'}
ok
$ echo $abc

$ echo ${abc='ok'}
ok
$ echo $abc
ok
```

### 二、字符串操作

表达式 	                             | 含义
------------------------------------ | ---------------------------------------------------------------------
${#string} 	                         | $string的长度
${string:position} 	                 | 在$string中, 从位置$position开始提取子串
${string:position:length} 	         | 在$string中, 从位置$position开始提取长度为$length的子串
${string#substring} 	             | 从变量$string的开头, 删除最短匹配$substring的子串
${string##substring} 	             | 从变量$string的开头, 删除最长匹配$substring的子串
${string%substring} 	             | 从变量$string的结尾, 删除最短匹配$substring的子串
${string%%substring} 	             | 从变量$string的结尾, 删除最长匹配$substring的子串
${string/substring/replacement} 	 | 使用$replacement, 来代替第一个匹配的$substring
${string//substring/replacement} 	 | 使用$replacement, 代替所有匹配的$substring
${string/#substring/replacement} 	 | 如果$string的前缀匹配$substring, 那么就用$replacement来代替匹配到的$substring
${string/%substring/replacement} 	 | 如果$string的后缀匹配$substring, 那么就用$replacement来代替匹配到的$substring


#### 1.长度

```
$ test='I love china'
$ echo ${#test}
12
```

#### 2.截取字串

```
$ test='I love china'
$ echo ${test:5}    
e china
$ echo ${test:5:10}
e china
```

#### 3.字符串删除

```
$ test='c:/windows/boot.ini'
$ echo ${test#/}
c:/windows/boot.ini
$ echo ${test#*/}
windows/boot.ini
$ echo ${test##*/}
boot.ini
$ echo ${test%/*}
c:/windows
$ echo ${test%%/*}
c:
```

_注意_：`${test##*/}`,`${test%/*}` 分别是得到文件名，或者目录地址最简单方法。

#### 4.字符串替换

```
$ test='c:/windows/boot.ini'
$ echo ${test/\//\\}
c:\windows/boot.ini
$ echo ${test//\//\\}
c:\windows\boot.ini
```

### 三、性能比较

在shell中，通过awk,sed,expr 等都可以实现，字符串上述操作。下面我们进行性能比较。

```
$ test='c:/windows/boot.ini'                      
$ time for i in $(seq 10000);do a=${#test};done;           

real    0m0.173s
user    0m0.139s
sys     0m0.004s

$ time for i in $(seq 10000);do a=$(expr length $test);done;      

real    0m9.734s
user    0m1.628s
```

速度相差上百倍，调用外部命令处理，与内置操作符性能相差非常大。在shell编程中，尽量用内置操作符或者函数完成。使用awk,sed类似会出现这样结果。
