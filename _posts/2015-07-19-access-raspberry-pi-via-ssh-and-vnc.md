---
layout: post
title: "使用 ssh 和 vnc 访问树莓派"
keywords: ["树莓派", "ssh", "vnc"]
description: "在没有显示器的情况下，使用 ssh 和 vnc 访问树莓派"
category: "tech"
tags: ["树莓派"]
---
{% include JB/setup %}

拿到树莓派并安装系统以后，得想办法访问它。如果有显示器或者电视，可以通过 HDMI 线或 AV 线连接，然后插上鼠标和键盘直接操作，可是我没有任何显示设备，我只有一部笔记本，所以只能远程访问了。我一般使用 ssh 远程登录，以后的几乎所有的操作都是在 ssh 下进行的，偶尔会通过 vnc 连接图形界面。由于 vnc 没有内置，所以得先通过 ssh 登录，安装 vnc server，然后才可以通过 vnc 访问。

## 一、ssh 远程登录

将树莓派通过网线连接到路由器，使笔记本和树莓派在一个局域网里面。在笔记本上登录路由器后台，一般是 `192.168.1.1`，查看连接到路由器的设备。以我的渣斐讯路由为例，在『运行状态』『 DHCP 客户端』页面可以看到树莓派的 ip 地址是 `192.168.1.106`：

![路由器后台管理页面]({{ IMAGE_PATH }}/router-admin-page.png)

然后我们就可以通过 ssh 登录树莓派了。windows 下可以通过 PuTTY 连接，mac 和 linux 下直接在终端用 ssh 登录：

```
ssh pi@192.168.1.106
```

`pi` 用户的默认密码为 `raspberry`。

为了以后登录方便，不用输入密码，可以使用私钥登录。生成密钥对（公钥和私钥）：

```
ssh-keygen -t rsa
```

保存位置直接选默认，这样生成的私钥为 `~/.ssh/id_rsa`。

然后，将公钥 `~/.ssh/id_rsa.pub` 拷贝到树莓派的 `~/.ssh/authorized_keys`：

```
ssh-copy-id pi@192.168.1.106
```

或者用 sftp 传上去也可以，可能需要修改权限。

通过 ssh 登录以后，最好做一些基本的配置，比如修改一下软件源，运行 `raspi-config` 修改 locale 等，可参考 debian 的一些入门教程，此处略过。

## 二、安装并配置 vnc server

安装 tightvncserver：

```
sudo apt-get install tightvncserver
```

写一个简单的启动脚本 `/etc/init.d/tightvncserver`：

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          tightvncserver
# Required-Start:    $syslog
# Required-Stop:     $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: vnc server
# Description:
#
### END INIT INFO

# Check the state of the command – this'll either be start or stop
case "$1" in
  start)
    # if it's start, then start vncserver using the details below
    sudo -u pi /usr/bin/vncserver :1 -geometry 1024x720 >/dev/null 2>&1 &
    echo "vncserver started"
    ;;
  stop)
    # if it's stop, then just kill the process
    pkill Xtightvnc
    echo "vncserver stopped"
    ;;
  *)
    echo "Usage: /etc/init.d/vncserver {start|stop}"
    exit 1
    ;;
esac
exit 0
```

可以在启动参数里设置分辨率、位深、像素格式等，此处设置分辨率为 1024x720。

然后，启动 vncserver 并设置成开机启动：

```
sudo chmod +x /etc/init.d/tightvncserver
sudo service tightvncserver start
sudo update-rc.d tightvncserver defaults
```

vncserver 启动以后，就可以在笔记本上通过 vnc viewer 一类的客户端去连接树莓派的桌面了。

事实上，连上了桌面也没有什么可以做的，而且有明显的闪烁，体验很差，所以我一般只有在用 wireshark 之类的工具时才用 vnc 连接。

由于 tightvncserver 是基于 X Window 系统的，所以像 kodi (xbmc) 这种没有使用 X Window 系统的应用启动了之后是看不到窗口的。如果想通过 vnc 查看 kodi 窗口（看视频就不要想了），可以使用 [dispmanx vnc](https://github.com/patrikolausson/dispmanx_vnc)。dispmanx vnc 使用 [video core](http://elinux.org/Raspberry_Pi_VideoCore_APIs) 里面 dispmanx 相关的 api 获取屏幕快照，然后通过 libvncserver 实现 vnc 的 rfb 协议。具体编译安装过程请参照 README.md。
