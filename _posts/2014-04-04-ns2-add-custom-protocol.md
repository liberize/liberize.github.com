---
layout: post
title: "NS2 添加自定义协议"
keywords: ["NS2", "协议"]
description: "在 NS2 中添加自定义协议的步骤"
category: "tech"
tags: ["ns2", "协议"]
---
{% include JB/setup %}

NS 模拟分两个层次：一个是基于 Otcl 编程的层次，利用 NS 已有的网络元素实现模拟；另一个是基于 C++ 和 Otcl 编程的层次，如果 NS 中没有所需的网络元素，就需要首先对 NS 扩展，添加你所需要的网络元素。本文将以 NS 中 已有的 Ping 协议为例，讲述第二个层次的具体操作步骤。

## 一、头文件

新建头文件 `ping.h`，首先在头文件中定义 ping 包头的数据结构：

```c++
struct hdr_ping {
	char ret;
	double send_time;
	double rcv_time;	// when ping arrived to receiver
	int seq;		// sequence number

	// Header access methods
	static int offset_; // required by PacketHeaderManager
	inline static int& offset() { return offset_; }
	inline static hdr_ping* access(const Packet* p) {
		return (hdr_ping*) p->access(offset_);
	}
};
```

然后定义类 PingAgent，作为 Agent 类的派生类：

```c++
class PingAgent : public Agent {
public:
	PingAgent();
 	int seq;	// a send sequence number like in real ping
	int oneway; 	// enable seq number and one-way delay printouts
	virtual int command(int argc, const char*const* argv);
	virtual void recv(Packet*, Handler*);
};
```

其中，command 方法在上一篇文章『[NS2 分裂对象模型](/post/ns2-split-object-model.html)』中有介绍，它定义了可在 Otcl 中使用的实例过程，
recv 方法定义了收到 packet 时的操作。除此之外，还有一个常用的方法 sendmsg，它定义了收到上层要发送的数据时的操作。以上三种方法都是虚函数。

## 二、实现文件

首先是定义 PingHeaderClass 并创建一个实例：

```c++
int hdr_ping::offset_;
static class PingHeaderClass : public PacketHeaderClass {
public:
	PingHeaderClass() : PacketHeaderClass("PacketHeader/Ping", 
					      sizeof(hdr_ping)) {
		bind_offset(&hdr_ping::offset_);
	}
} class_pinghdr;
```

然后，定义 PingClass 并创建一个实例，PingClass 派生自 TclClass，同样在『[NS2 分裂对象模型](/post/ns2-split-object-model.html)』中有介绍，它登记了 C++ 类和 Otcl 类的对应关系：

```c++
static class PingClass : public TclClass {
public:
	PingClass() : TclClass("Agent/Ping") {}
	TclObject* create(int, const char*const*) {
		return (new PingAgent());
	}
} class_ping;
```

然后，就是 PingAgent 的实现，协议的具体内容就体现在这里：

```c++
PingAgent::PingAgent() : Agent(PT_PING), seq(0), oneway(0)
{
	bind("packetSize_", &size_);
}

int PingAgent::command(int argc, const char*const* argv)
{
  if (argc == 2) {
    if (strcmp(argv[1], "send") == 0) {
      Packet* pkt = allocpkt();
      hdr_ping* hdr = hdr_ping::access(pkt);
      hdr->ret = 0;
      hdr->seq = seq++;
      hdr->send_time = Scheduler::instance().clock();
      send(pkt, 0);
      return (TCL_OK);
    }
    // ...
    else if (strcmp(argv[1], "oneway") == 0) {
      oneway=1;
      return (TCL_OK);
    }
  }
  return (Agent::command(argc, argv));
}


void PingAgent::recv(Packet* pkt, Handler*)
{
  // ...
  hdr_ping* hdr = hdr_ping::access(pkt);
  
  if (hdr->ret == 0) {
    double stime = hdr->send_time;
    int rcv_seq = hdr->seq;
    Packet::free(pkt);
    Packet* pktret = allocpkt();
    hdr_ping* hdrret = hdr_ping::access(pktret);
    hdrret->ret = 1;
    hdrret->send_time = stime;
    hdrret->rcv_time = Scheduler::instance().clock();
    hdrret->seq = rcv_seq;
    send(pktret, 0);
  } else {
    // ...
    Packet::free(pkt);
  }
}
```

## 三、要修改的地方

打开 common/packet.h，搜索 `PT_NTYPE`，找到：

```c++
static packet_t       PT_NTYPE = 73;
```

仿照格式在前面加上：

```c++
static const packet_t PT_PING = 44;
```

注意，如果是自己实现的新协议，应该将 PT_NTYPE 改为 74，然后将新协议 PT_XXX 的值设为 73，即要保证 PT_NTYPE 是最后一个。

然后继续找到:

```c++
name_[PT_NTYPE]= "undefined";
```
仿照格式在前面加上：

```c++
name_[PT_PING]="ping";
```

接着打开 tcl/lib/ns-default.tcl，添加：

```tcl
Agent/Ping set packetSize_ 64
```

这个文件设置了 Otcl 中实例变量的缺省值。

接着打开 tcl/lib/ns-packet.tcl，搜索 `set protolist`，在里面添加：

```c++
Ping 	# Ping
```

注意，这里面 # 前不需要加 ;。

最后，打开 Makefile，搜索 `OBJ_CC = \`，在底下添加：

```c++
ping.o \
```

如果你的协议实现目录结构比较复杂，应当在 `INCLUDES = \` 下面加上头文件所在目录，`LIB = \` 下面添加用到的库，在 `OBJ_CC = \` 下面添加所有 .cc 文件对应的目标文件，在 `OBJ_C = \` 下面添加所有 .c 文件对应的目标文件。如果 c++ 源文件使用的是 .cpp 后缀，可以仿照 .cc 添加一个 `OBJ_CPP = \`，并在其他 cc 出现的地方仿照格式添加 cpp。

完成以上步骤后，就可以开始编译 NS2 了。编译之前先修改源码的一个地方，不然无法顺利编译。

打开 indep-utils/webtrace-conv/dec/my-endian.h，将 `#ifndef` 和 `#define` 两个宏中的 `_ENDIAN_H_` 改为  `_MY_ENDIAN_H_`。

![修补代码]({{ IMAGE_PATH }}/ns2-patch-code-2.png)

改完以后，在 Cygwin 命令窗口中输入以下命令开始编译：

```bash
make depend
make
```

如果没有修改前面那处代码，编译过程中将会出现如下错误：

```
proxytrace2any.cc:112: error: `IsLittleEndian' undeclared (first use this
function)
proxytrace2any.cc:112: error: (Each undeclared identifier is reported only
once for each function it appears in.)
proxytrace2any.cc:120: error: `ToOtherEndian' undeclared (first use this
function)
```

## 四、编写 Otcl 测试脚本

关于测试脚本的编写，建议多看一些示例。此处不详述。
