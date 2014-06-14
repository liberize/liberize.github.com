---
layout: post
title: "我的 Conky 配置"
keywords: ["Conky", "配置"]
description: "分享我的 Conky 配置"
category: "tech"
tags: ["conky"]
---
{% include JB/setup %}

喜欢折腾的 Linux 党一般不会不知道 Conky 吧，它是一个轻量级的高度可配置的系统监视工具。说白了，就是桌面上的一块画布可以在上面任何位置输出任何文本（虽然不支持图像，但可以使用图标字体），并且可以调用内置的一些系统监视功能显示 CPU、内存、网络的使用状况。之前用过一段时间，后来重装系统的时候配置文件丢失了，这次又重新折腾了一回，基于 Ubuntu 中文论坛上的[帖子](http://forum.ubuntu.org.cn/viewtopic.php?t=313031)提供的模板修改而成。不多说，先上图：

![Conky 效果图]({{ IMAGE_PATH }}/conky-preview.png)

我的笔记本屏幕分辨率为1366x768，显卡为A卡（闭源驱动），CPU为双核四线程。

显卡部分主要用到`aticonfig --odgc`和`aticonfig --odgt`两个命令。

```
###############
# - AMD GPU - #
###############
${voffset 4}${font WenQuanYi Micro Hei:style=Bold:size=8}显卡 $stippled_hr${font}
${color0}${goto 2}${voffset 10}${font ConkyColorsLogos:size=20}r${font}${color}${goto 32}${voffset -18}型号: ${font Ubuntu:style=Bold:size=8}${color2}${execi 3600 aticonfig --odgt | grep "Default Adapter" | cut -c19-100}${color}${font}
${goto 32}核心: ${font Ubuntu:style=Bold:size=8}${color1}${execi 4 aticonfig --odgc | grep "Current Clocks" | awk -F ' ' '{print $4}'}MHz${color}${font}${goto 145}峰值: ${font Ubuntu:style=Bold:size=8}${color2}${execi 300 aticonfig --odgc | grep "Current Peak" | awk -F ' ' '{print $4}'}MHz${color}${font}
${goto 32}显存: ${font Ubuntu:style=Bold:size=8}${color1}${execi 4 aticonfig --odgc | grep "Current Clocks" | awk -F ' ' '{print $5}'}MHz${color}${font}${goto 145}峰值: ${font Ubuntu:style=Bold:size=8}${color2}${execi 300 aticonfig --odgc | grep "Current Peak" | awk -F ' ' '{print $5}'}MHz${color}${font}
${goto 32}温度: ${font Ubuntu:style=Bold:size=8}${color1}${execi 4 aticonfig --odgt | grep "Temperature" | awk -F ' ' '{print $5}'}ºC${color}${font}${goto 145}负载: ${font Ubuntu:style=Bold:size=8}${color1}${execi 4 aticonfig --odgc | grep "GPU load" | awk -F ' ' '{print $4}'}${color}${font}
```

天气部分利用 weather.com.cn 的 api 写了一个 Python 脚本，可以得到实时天气及未来几天的天气。用法为：`"~/.conkycolors/bin/weather" citycode sixdays|realtime`，如下所示。

```
##################
# - CN WEATHER - #
##################
# http://www.weather.com.cn/
${voffset -8}${font WenQuanYi Micro Hei:style=Bold:size=8}天气 $stippled_hr${font}
${if_gw}${execpi 300 ~/.conkycolors/bin/weather 101220101 realtime}
${execpi 1800 ~/.conkycolors/bin/weather 101220101 sixdays}
${else}${voffset 8}${color0}${font ConkyColors:size=15}q${font}${color}${voffset -8}${goto 32}天气信息不可用${voffset 14}${endif}
```

限于屏幕分辨率，修改了整体的高度和宽度，去掉了部分内容。还有一些改动的细节：修改了日历的显示，排列更整齐；修改了时钟样式，更美观；替换了获取农历日期的脚本 lunar，使用 Python 实现，效率更高。

所有文件打包下载：<a href="http://pan.baidu.com/s/1044a0" title="前往网盘下载"><button class="blue"><i class="icon-download-alt"></i> 百度网盘</button></a>

其中，除了配置文件外，还包含了全部字体、脚本以及 Conky 文档。
