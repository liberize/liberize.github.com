---
layout: post
title: "NS2 分裂对象模型"
keywords: ["NS2"]
description: "简单介绍 NS2 分裂对象模型"
category: "tech"
tags: ["ns2"]
---
{% include JB/setup %}


每实例化一个构件时，都会同时创建一个 Otcl 中的对象和一个对应的 C++ 对象，两个对象可以互操作。

## 一、Otcl 类和 C++ 类的对应关系

分裂对象模型中的 Otcl 类称为解释类， 对应的 C++ 类称为编译类。它们互为影像类。

TclObject 是所有编译类的基类，SplitObject 是所有解释类的基类。

## 二、创建和销毁 TclObject

new{} 创建解释对象和对应的影像对象。
首先创建出解释对象，然后执行该对象的初始化实例过程，init{}，并把用户提供的参数传给它。在基类 SplitObject 的初始化实例过程中，通过 create-shadow 方法创建该对象在 C++ 中的影像对象。然后执行影像对象的构造函数。

```tcl
set tcp1 [new Agent/TCP]
delete $tcp1
```

## 三、TclClass 类

TclClass 类用来登记 C++ 类和 Otcl 类的对应关系。它是个纯虚类。从这个基类继承出来的类提供两个功能：构造和编译类结构互为影像的解释类结构以及提供初始化新的 TclObject 的方法。

```c++
static class TcpClass : public TclClass {
public:
        TcpClass() : TclClass("Agent/TCP") { }
        TclObject* create(int, const char*const*) {
            return (new TcpAgent());
        }
} class_tcp;
```

## 四、变量绑定

C++ 成员变量可以和 Otcl 成员变量建立双向绑定，当任何一边变量值改变时对应另一边变量也会改变为相同的值。绑定的语法：

```
bind("Otcl变量名", &C++变量名);
```

```c++
TcpAgent::TcpAgent() : Agent(PT_TCP) // ...
{
	bind("t_seqno_", &t_seqno_);
	bind("rtt_", &t_rtt_);
	bind("srtt_", &t_srtt_);
	bind("rttvar_", &t_rttvar_);
	bind("backoff_", &t_backoff_);
	bind("dupacks_", &dupacks_);
    // ...
}
```

## 五、command 方法

Otcl 中的解释对象的实例过程 cmd{} 调用影像对象的方法 command()，将 cmd{} 的参数作为一个参数数组传给 command() 方法。

可以显式地调用 cmd{} 过程，将欲进行的操作指定为第一个参数，或者隐式调用，就像是存在一个和欲进行的操作同名的实例过程一样。

```c++
int TcpAgent::command(int argc, const char*const* argv)
{
	if (argc == 3) {
		if (strcmp(argv[1], "advance") == 0) {
			int newseq = atoi(argv[2]);
			if (newseq > maxseq_)
				advanceby(newseq - curseq_);
			else
				advanceby(maxseq_ - curseq_);
			return (TCL_OK);
		}
		if (strcmp(argv[1], "advanceby") == 0) {
			advanceby(atoi(argv[2]));
			return (TCL_OK);
		}
		// ...
	}
	return (Agent::command(argc, argv));
}
```

## 六、实际过程示例

1. 在 NS 开始时，命令行 `new Agent/TCP` 运行，在 new{} 的过程中，一个 Agent/TCP 的解释对象被创建了，并且调用了它的实例过程 init{}。
2. 因为 Agent/TCP 的父类先于其 init{} 调用，则基类 init{} 最先被调用。基类 SplitObject 的 create-shadow 方法负责创建影像对象。
3. 根据 TcpClass 登记的关联关系，class_tcp 的 create 函数被调用，创建 TcpAgent 对象，并返回其指针。
4. TcpAgent 的构造函数中绑定变量。
5. 当对 Agent/TCP 解释对象调用某个不存在的实例过程时，解释器将会调用实例过程 unknown{}，unknown{} 过程将会调用 cmd{} 过程来通过编译对象的 command() 函数执行这个操作。
