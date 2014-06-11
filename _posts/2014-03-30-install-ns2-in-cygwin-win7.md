---
layout: post
title: "Win 7 + Cygwin 1.7 + NS2 2.35 安装手记"
keywords: ["Win7", "Cygwin", "NS2"]
description: "在 Windows 7 下 Cygwin 环境下安装 NS2 过程记录"
category: "tech"
tags: ["win7", "cygwin", "ns2"]
---
{% include JB/setup %}

*主要参考：<http://blog.sina.com.cn/s/blog_579879b90100wz8x.html>*

NS 从 2.1b9 版本开始不再使用 VC++，改用 Cygwin/gcc。因此，要使用 NS2 必须首先搭建 Cygwin 环境。

Cygwin 从 1.7 版开始支持 Win7，而与之兼容的 NS2 则要达到最新的 2.35 版本。事实上，2.35 版本是在 2011 年更新的，它所依赖的一些软件包在最新的 Cygwin 中已经过时。

这里选用的版本是：Win7 + Cygwin 1.7.28 + NS2 2.35。

PS: 伸手党请直接到文章最后下载打包的 Cygwin。


## 一、安装 Cygwin


到[ Cygwin 官网](http://www.cygwin.com/)下载最新的安装程序。即使你是 64 位的 win7，也建议选择 x86 版本，因为某些源中 64 位的软件包可能不太全。

下载 setup-x86.exe 后运行，开始安装cygwin。选择软件源时，建议选择 ustc 的镜像（<http://mirrors.ustc.edu.cn/>），速度很不错，包也比较全。不要选择 163 的镜像，很多过时的包都没有。

![选择镜像]({{ IMAGE_PATH }}/cygwin-choose-mirror.png)

接着开始选择要安装的软件包，注意将下面的 "Hide obsolete packages" 前面的勾去掉，然后点一下右上角的 "View" 按钮。

![选择软件包]({{ IMAGE_PATH }}/cygwin-choose-packages.png)

在输入框中输入包名进行搜索，找到你要的包，如果 "New" 那一列显示的是 "Skip" 表示不安装，点一下 "Skip" 它会变成最新的版本号，表示将会安装该版本的包。

我们需要安装的包有：

```
gcc-core  gcc-g++  gcc4  gcc4-core  gcc4-g++  gawk  gnuplot  gzip  make  patch  perl  tar  w32api  diffutils
X-startup-scripts  xorg-x11-base  xorg-x11-bin  xorg-x11-bin-dlls  xorg-x11-bin-lndir  xorg-x11-devel
xorg-x11-etc  xorg-x11-fenc  xorg-x11-fnts  xorg-x11-libs-data  xorg-x11-xwin  libxt-devel  libXmu-devel
```

所有包都选中后，点击下一步、下一步开始安装。

如果以后想安装其他软件包，重新运行 setup-x86.exe 就可以了。

## 二、编译安装 NS2

到 sourceforge 上下载[ NS2 2.35 版本](http://sourceforge.net/projects/nsnam/files/allinone/ns-allinone-2.35/ns-allinone-2.35.tar.gz/download)，把下载的源码包放在 $HOME/ 目录下，双击桌面 Cygwin 图标，输入以下命令解压缩：

```bash
tar xfzv ns-allinone-2.35.tar.gz
```

解压缩后，开始编译安装。先修改源码的一个地方，不然无法顺利编译。

使用文本编辑器打开 `~/ns-allinone-2.35/ns-2.35/linkstate/ls.h`，在 137 行 `erase` 前面加上 `this->`。

![修补代码]({{ IMAGE_PATH }}/ns2-patch-code.png)

保存关闭后，继续使用以下命令安装 NS2:

```bash
cd ns-allinone-2.35
./install
```

如果没有修改前面那处代码，安装过程中将会出现如下错误：

```
linkstate/ls.h:137:58: 错误：‘erase’ was not declared in this scope...
```

安装完以后，遵照指示，将以下内容复制到 `~/.bashrc` 文件的末尾，设置环境变量：

```bash
export PATH="$PATH:$HOME/ns-allinone-2.35/bin:$HOME/ns-allinone-2.35/tcl8.5.10/unix:$HOME/ns-allinone-2.35/tk8.5.10/unix"
export LD_LIBRARY_PATH="$HOME/ns-allinone-2.35/otcl-1.14, $HOME/ns-allinone-2.35/lib"
export TCL_LIBRARY="$TCL_LIBRARY:$HOME/ns-allinone-2.35/tcl8.5.10/library"
```

## 三、后续工作

如果想验证 NS2 是否正确安装，在 Cygwin 命令窗口中执行以下命令: (相当耗时)

```bash
cd ns-allinone-2.35/ns-2.35
./validate
```

如果想验证 nam 是否可用，通过开始菜单 -> Cygwin-X -> XWin Server 打开 XWin Server，在弹出的窗口中执行以下命令：

```bash
cd ns-allinone-2.35/ns-2.35/tcl/ex
ns nam-example.tcl
```

![nam 示例]({{ IMAGE_PATH }}/nam-example.png)

如果想在 Cygwin 命令窗口中执行，需要首先运行 XWin Server，关掉窗口但不要退出，在 `~/.bashrc` 中加入 `export DISPLAY=:0.0`，然后打开 Cygwin 命令窗口，输入以上命令。

最后，附上包含 NS2 的完整的 Cygwin 包：<a href="http://pan.baidu.com/s/1pJud6GV" title="前往网盘下载"><button class="blue"><i class="icon-download-alt"></i> 百度网盘</button></a>
