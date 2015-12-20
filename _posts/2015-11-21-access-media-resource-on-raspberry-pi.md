---
layout: post
title: "访问树莓派的媒体资源"
keywords: ["树莓派", "ftp", "samba", "minidlna", "bubbleupnp"]
description: "通过 samba、dlna 等方式远程访问树莓派上媒体资源"
category: "tech"
tags: ["树莓派"]
---
{% include JB/setup %}

下载机搭建好了以后，接下来就该考虑远程访问树莓派上的资源了。对于普通文件，可以用 FTP、Samba。对于媒体文件，可以用 MiniDLNA，再配合 BubbleUPnP 可以在 Android 设备上使用 MX Player 播放音视频。

## 一、FTP

FTP 是一种比较古老的方式，用起来并不是很方便，所以仅作为备用方案。

由于之前已经配置好了 SSH 访问，所以可以直接使用 SFTP，无需安装 FTP 服务端。一般的 FTP 客户端都应该支持 SFTP 了。缺点是比 FTP 速度慢一些。

## 二、Samba

Samba 是一种比较方便的、跨平台文件共享方式，Windows、Mac OS、Linux 都可以直接访问 Samba 共享的资源，无需安装额外客户端。

安装 Samba：

```
sudo apt-get install samba
```

修改配置文件 `/etc/samba/smb.conf`，加上：

```ini
[share]
    path = /media/NAS
    valid users = root pi
    browseable = yes
    public = yes
    writable = yes
```

然后重启 Samba 服务：

```
sudo service samba restart
```

然后在另一台电脑上测试一下能否访问共享目录。以 Mac 为例，确保 pc 和 树莓派在同一局域网内，在 Finder 左侧 Shared 里面应该已经可以看到 `raspberrypi`，以 pi 的身份登陆，可以看到两个共享目录，分别是 `pi` 和 `share`，这里的 `share` 就是 pi 上面共享的 `/media/NAS`。里面如果有视频，可以直接在 Mac 上播放了。

经过测试 samba 读写速度分别为 7.9 MB/s 和 8.4 MB/s，考虑到树莓派是百兆网口，这个速度还算正常。进一步的性能优化可以参考[这里](https://www.samba.org/samba/docs/man/Samba-HOWTO-Collection/speed.html)，比如加上：

```ini
[global]
    socket options = TCP_NODELAY IPTOS_LOWDELAY
```

读的速度可以提升到 9.4 MB/s，写的速度基本没变。

## 三、MiniDLNA + BubbleUPnP

DLNA 是一种比较常见、广泛支持的数字媒体设备之间互联互通的解决方案。MiniDLNA 是一个支持 DLNA/UPnP 协议的轻量级的服务端软件。

DLNA 目前还有一些缺点，比如不支持外挂字幕，不支持 rmvb 格式等，这些基本都是可以解决的。字幕问题，如果使用 BubbleUPnP + MX Player，可以在 MX Player 中加载本地字幕。rmvb 格式问题，可以下载源码，自己打补丁，参考[这篇文章](http://blog.csdn.net/Haven200/article/details/43039261)。

安装 MiniDLNA：

```
sudo apt-get install minidlna
```

修改配置文件 `/etc/minidlna.conf`：

```
media_dir=A,/media/NAS/Music
media_dir=P,/media/NAS/Pictures
media_dir=V,/media/NAS/Videos
inotify=yes
friendly_name=Raspberry Pi Media Server
```

其中，`A,` `P,` `V,` 后面分别是音频、图片、视频目录，`inotify` 表示是否自动发现新文件，`friendly_name` 是显示的名称。

然后重启 MiniDLNA 服务：

```
sudo service minidlna restart
```

浏览器访问树莓派 8200 端口可以看到当前媒体文件数目。

如果加入新文件后没有自动识别，执行 `sudo minidlnad -R` 刷新数据库。

目前很多播放器 app 都支持 DLNA，比如 iOS 上的 AVPlayer 等。Android 上目前还没有发现支持 DLNA 又好用的播放器，建议使用 BubbleUPnP + MX Player。

BubbleUPnP 是一个 DLNA 文件管理器，本身不提供播放功能，但可以调用外部播放器播放 DLNA 服务器上的视频，需要同时安装服务端和客户端。

到[官网](http://www.bubblesoftapps.com/bubbleupnpserver/)下载最新的服务端，选通用的 `Zip archive` 就行了。

放到树莓派上，解压，加上可执行权限：

```
unzip -d BubbleUPnPServer BubbleUPnPServer-distrib.zip
cd BubbleUPnPServer
chmod +x launch.sh
```

写一个简单的启动脚本 `/etc/init.d/bubbleupnp`：

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          BubbleUPnP
# Required-Start:    $network $local_fs $remote_fs
# Required-Stop:     $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: UPnP server by bubble soft
# Description:
#
### END INIT INFO

cd /path/to/BubbleUPnPServer

case "$1" in
  start)
    ./launch.sh >/dev/null 2>&1 &
    echo "BubbleUPnP started."
    ;;
  stop)
    ps aux | grep 'BubbleUPnP' | grep -v grep | awk '{print $2}' | xargs -r kill -HUP
    echo "BubbleUPnP stopped."
    ;;
  *)
    echo "Usage: /etc/init.d/bubbleupnp {start|stop}"
    exit 1
    ;;
esac
exit 0
```

启动 BubbleUPnPServer 并设为开机启动：

```
sudo chmod +x /etc/init.d/bubbleupnp
sudo service bubbleupnp start
sudo update-rc.d bubbleupnp defaults
```

在 Android 设备上安装 BubbleUPnP 客户端，找 DLNA 服务器 `Raspberry Pi Media Server`，连上以后就可以看到树莓派上的媒体文件，点击视频可以选择使用 MX Player 或其他播放器播放。

（由于间隔太久，一些细节已经记不清了，如果有疏漏，请指出）
