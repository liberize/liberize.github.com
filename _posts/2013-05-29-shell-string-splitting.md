---
layout: post
title: "Shell 字符串分割"
keywords: ["Shell", "字符串", "分割"]
description: "Shell 字符串分割方法简介"
category: "tech"
tags: ["shell"]
---
{% include JB/setup %}

```bash
a='hello,world,test'
```

#### 1、使用cut/awk分割字符串，取出其中一部分

```bash
echo $a | cut -d, -f1
echo $a | awk -F ',' '{print $1}'
```

awk的-F开关指定分隔符，多个分隔符应当写在[]中。

#### 2、使用IFS将字符串分割为数组

要将$a分割开，可以这样：

```bash
OLD_IFS="$IFS"
IFS=","
arr=($a)
IFS="$OLD_IFS"
for s in ${arr[@]}; do
    echo "$s"
done
```

#### 3、某些情况下使用字符替换也可以达到分割效果

使用外部命令tr或变量扩展格式${string//substr/replacement}来替换。

```bash
for s in ${a//,/ }; do
    echo "$s"
done
```
