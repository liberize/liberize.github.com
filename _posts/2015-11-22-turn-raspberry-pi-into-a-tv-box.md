---
layout: post
title: "将树莓派打造成电视盒子"
keywords: ["树莓派", "电视盒子", "kodi", "xbmc", "shairport"]
description: "通过安装 kodi，将树莓派上打造成电视盒子"
category: "tech"
tags: ["树莓派"]
---
{% include JB/setup %}

最近，光腚总局对盒子第三方应用下了禁令，天猫魔盒首当其冲，估计其他盒子不久也会自我阉割，树莓派 + kodi 做成的（伪）盒子自然不受影响。树莓派盒子其实介于盒子和 htpc 之间，一方面有盒子操作方便、界面友好的特点，另一方面又具有 htpc 的灵活性。这篇文章只适用于那些手上恰好有个树莓派，又想省下电视盒子的钱的人，只为了当盒子用而去买树莓派是非常不值的，因为树莓派盒子从硬件、软件、价格、体验等各个方面上都不及主流的电视盒子。

在使用 kodi 方案前，我考虑过另外一个方案：在树莓派上安装 android，做成真正的盒子。然而这个方案很快就放弃了，因为树莓派上目前并没有好用的 android 系统镜像。网上有人成功在树莓派上安装了 4.4、5.0 版本的 android，然而仅仅是可以跑，离实用还差很远，最主要的原因是不能使用 gpu 进行图形加速。kodi 方案其实也并不完美，在线播放功能主要依靠插件，而插件质量普遍比较低，甚至大部分都失效了，所以在线播放基本沦为鸡肋，好在 kodi 在管理本地资源方面做的还不错，可以说是一个美观、实用的媒体中心。

## 一、需要的硬件

电视机或显示器自然必不可少，如果显示器不带外放，还需要一个音箱或扬声器。

树莓派有两种音视频输出方式：HDMI 和 RCA。连接电视机/显示器有以下几种方案：

1. 一般的电视机和稍好点的显示器应该都有 HDMI 接口，直接与树莓派的 HDMI 接口相连即可。
2. 差一点的显示器可能只有 VGA 或 DVI，用转接线应该也可以。
3. 老式电视机可以通过 RCA 接口连接，树莓派将 RCA 插孔与 3.5mm 音频插孔合并了，与一般的 RCA 插头上四节顺序不同，需要转接一下。

我的显示器是 DELL U2414H，不带外放，所以还需要连接一个音箱。连接音箱有以下几种方案：

1. 直接用 3.5mm 插孔连接音箱 AUX 接口，零成本，使用内置 10 bit DAC，声音质量一般。实际测试，一直都有很大底噪，据说是因为供电不稳定，但是我试过 N 个电源适配器 + 带磁环、低电阻连接线，还试过换音频线、拔掉所有外设，都没有办法消掉。
2. 用 HDMI 音频分离器分离 HDMI 中的音频，价格略贵，需要两根 HDMI 线缆，需要额外供电，不方便。
3. 用蓝牙适配器连接蓝牙音箱，需要音箱支持蓝牙，而且 kodi 是否支持也未可知。
4. 用 USB 声卡连接音箱，价格便宜，音质尚可。实际测试，连接电脑是 OK 的，但是接树莓派仍然有噪声。

最后意外发现自己买的廉价蓝牙音箱竟然内置了 USB 声卡，直接 USB 连接树莓派，不仅免驱，而且也没有噪声，真是柳暗花明又一村。

## 二、Kodi

kodi 是一个强大、美观、跨平台、可扩展的媒体中心软件，一直享有盛誉。以前玩 linux 的时候，还叫 xbmc。树莓派上有一些内置 kodi 的操作系统，比如 OpenElec、OSMC(Raspbmc)，建议直接选择这类操作系统，省掉一些不必要的麻烦。因为之前做了很多自定义配置，不想重装系统，所以我选择了在 Raspbian 上安装 kodi。

### 1. 安装 kodi

安装过程主要参考[这里](http://michael.gorven.za.net/)。

添加额外软件源，编辑 `/etc/apt/sources.list.d/mene.list`，添加：

```
deb http://archive.mene.za.net/raspbian wheezy contrib
```

可以在后面加上 `unstable` 以使用最新的 kodi，缺点是可能不大稳定。

导入 key 以信任该软件源：

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key 5243CDED
```

然后，更新软件源，并安装 kodi：

```
sudo apt-get update
sudo apt-get install kodi
```

可以使用用户 pi 或 kodi 运行，以 pi 为例，确保 pi 在以下 group 内：

```
audio video input dialout plugdev tty
```

使用 `groups pi` 查看 pi 所属的 group，使用 `useradd -G <group> pi` 把 pi 加入到某个 group。

添加 udev 规则，编辑 `/etc/udev/rules.d/99-input.rules`，加入：

```
SUBSYSTEM=="input", GROUP="input", MODE="0660"
KERNEL=="tty[0-9]*", GROUP="tty", MODE="0660"
```

如果要开机启动，编辑 `/etc/default/kodi`，设置：

```
ENABLED=1
USER=pi
```

设置分配给 GPU 的 RAM 大小，最好不低于 128M，修改 `/boot/config.txt`：

```
gpu_mem=256
```

### 2. 插件及设置

首先，把界面改为中文。选择 SYSTEM 底下的 Settings，打开设置界面，选择 Appearance，然后选择 Skin，右侧 Fonts 改为 Arial Based，接着选择 International，右侧 Language 改为 Chinese (Simple)，稍等片刻，界面就变为中文了。

然后，可以更换你喜欢的主题。**更换主题前请将 Language 改为 English**，不然换了主题以后很有可能中文字符全都变成方块，找不到设置菜单。选择『皮肤』，右侧『皮肤』显示目前是默认的 Confluence，回车可以查看其他主题。吐槽一下，主题不支持预览也就算了，有些居然连图片都没有，也是醉了。除了 Confluence 以外，比较优秀又耗资源较少的首推 Amber。不过 Amber 似乎没有网格视图，可能会有少许 bug。换了『皮肤』以后，底下的『导航音效』也可以设为主题配套的音效。

![Kodi Amber 主题]({{ IMAGE_PATH }}/kodi-amber-theme.png)

接着，打开遥控功能。设置里面选择『服务』，然后选择『Web 服务器』，勾选『允许通过 HTTP 控制 Kodi』并设置『端口』、『用户名』、『密码』，之后就可以通过手机 app 遥控 kodi 了。Android 上可用 Yatse，iOS 可用 Kodi Remote。我用的是 Kodi Remote，有些功能是隐藏的，比如长按菜单按钮打开上下文菜单等。

![Kodi 遥控设置]({{ IMAGE_PATH }}/kodi-web-control.png)

『服务』里面还有一些其他设置，比如 UPnP（可取代[上一节](/tech/access-media-resource-on-raspberry-pi.html)的 MiniDLNA）、AirPlay（可取代后面的 shairport-sync）等。

如果音频是外接音箱，需要设置音频输出。设置里面选择『系统』，然后选择『音频输出』，修改『音频输出设备』。如果通过 3.5mm 插孔连接，选择 `Pi: Analogue`，如果通过 USB 声卡连接，选择 `ALSA: USB 2.0 Device`。（音频选择 ALSA 之后，将不能使用 OMX 加速。）

![Kodi 音频输出设置]({{ IMAGE_PATH }}/kodi-audio-output.png)

最后，就是安装插件了。官方插件库以英文为主，中文插件库有 [xbmc-addons-chinese](https://github.com/taxigps/xbmc-addons-chinese/raw/master/repo/repository.xbmc-addons-chinese/repository.xbmc-addons-chinese-1.2.0.zip)，和 [HDPfans](http://xbmc.hdpfans.com/repository.hdpfans.xbmc-addons-chinese.zip)，HDPfans 里面很多插件已经失效。下载 zip 文件之后，在设置里面选择『插件』，然后选择『从 zip 文件安装』。安装完成以后，可以在『获取插件』里面看到『Chinese Add-ons』和『HDPfans 中文插件库』。

插件库里的插件类型有音乐插件、视频插件、图片插件、服务、字幕、歌词等等。目前（14.2版本）基本可用的插件有：PPTV、Youku TV、百度云、奇艺视频、搜狐视频、亚洲电视、CNTV 等等，这些插件大都有一些 bug。体验最好的当属 [Youku TV](https://github.com/catro/plugin.video.youkutv)，直接模拟 Youku TV 官方 app 的界面，与盒子界面风格一致，里面提供三种分段方式：『分段』、『堆叠』、『m3u8』，不幸的是分段、堆叠这两种方式在树莓派上不能开启硬件加速，只有声音没有图像，这跟优酷 flv 的格式有关，m3u8 在每一段衔接的地方会卡一会儿，而且不能续播。百度云插件可以观看百度网盘的视频，只是缓冲速度比较慢，我这儿 4M 小水管基本不能看。

![Kodi Youku TV 插件]({{ IMAGE_PATH }}/kodi-youku-tv-addon.png)

说到本地资源管理，不得不提到刮削器。刮削器的作用是根据文件名获取电影、电视剧等相关信息，常用的有豆瓣刮削器、时光网刮削器。目前刮削器还很不智能，必须对视频重命名才能正确获取信息。具体使用方法是，在视频、音频里面添加一个目录，然后设置刮削器，每当添加新文件，需要更新资料库。

顺便提一下，目前 kodi 有个 bug，退出之后会黑屏，无法回到桌面。

### 3. 性能优化

kodi 有一些高级设置在 GUI 里面无法修改，可以编辑 `~/.kodi/userdata/advancedsettings.xml` 来修改。里面有很多设置项，可以参考[官方文档](http://kodi.wiki/view/Advancedsettings.xml)，常见的网络相关的设置有 buffermode、readbufferfactor、cachemembuffersize 等，当网络不好时可做如下设置：

```
<advancedsettings>
    <network>
        <buffermode>1</buffermode>
        <readbufferfactor>4.0</readbufferfactor>
        <cachemembuffersize>41943040</cachemembuffersize>
    </network>
</advancedsettings>
```

其中，`buffermode` 表示哪些文件需要缓存（网络、本地），设为 1 表示缓存所有文件；`readbufferfactor` 表示缓存填充速率，设为 4.0 则下载速率限制为 4.0 * 视频平均比特率；`cachemembuffersize` 表示缓冲区大小，设为 40M 则实际 RAM 占用为 3 * 40M = 120M。

## 三、Shairport Sync

shairport-sync 可作为 AirPlay 接收端，不过只支持音频。安装以后就可以直接把 iPhone 上的音乐扔给树莓派播放，比蓝牙方便很多。

kodi 里面的 AirPlay 在我这儿似乎有问题，打开以后可以搜到，但是不能播放，所以使用 shairport-sync 代替。原本我用的是 [shairport](https://github.com/abrasive/shairport)，后来发现每隔一段时间声音卡一下，所以换成了 shairport-sync，目前工作正常。

参考 [GitHub 主页](https://github.com/mikebrady/shairport-sync)安装就行了。很详细，不想写了。
