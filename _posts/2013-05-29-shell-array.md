---
layout: post
title: "Shell 数组"
keywords: ["Shell", "数组"]
description: "Shell 数组使用简介"
category: "tech"
tags: ["shell"]
---
{% include JB/setup %}

shell中的数组使用方法：

```bash
$ arr=(123 34 3 5)  
$ echo $arr           # 默认获取第一个元素  
123  
$ echo ${arr[1]}      # 通过下标访问  
34  
$ echo ${arr[@]}      # 访问整个数组 ，@或者* 获取整个数组  
123 34 3 5  
$ echo ${#arr[@]}     # 获取数组的长度（最大下标） ，#获取长度 数组中是最后一个下标  
3  
$ echo ${#arr[3]}     # 获取字符串长度  
1  
$ echo ${arr[@]:1:2}  # 切片方式获取一部分数组内容  
34 3  
$ echo ${arr[@]:2}    # 从第二个元素开始  
3 5  
$ echo ${arr[@]::2}   # 到第二个元素  
123 34  
$ arr[0]=1            # 可以直接赋值修改  
$ arr[4]=6            # 不存在自动添加  
$ echo ${arr[@]}  
1 34 3 5 6  
```

array 的模拟操作：

```bash
push:  
array=("${array[@]}" $new_element)  
  
pop:  
array=(${array[@]:0:$((${#array[@]}-1))})  
  
shift:  
array=(${array[@]:1})  
  
unshift:  
array=($new_element "${array[@]}")  
  
function del_array {  
	local i  
	for (( i = 0 ; i < ${#array[@]} ; i++ ))  
	do  
		if [ "$1" = "${array[$i]}" ] ;then  
			break  
		fi  
	done  
	del_array_index $i  
}  
  
function del_array_index {  
	array=(${array[@]:0:$1} ${array[@]:$(($1 + 1))})  
}  
```
