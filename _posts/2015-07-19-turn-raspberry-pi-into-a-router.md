---
layout: post
title: "将树莓派打造成无线路由器"
keywords: ["树莓派", "路由器", "hostapd", "udhcpd", "iptables"]
description: "将树莓派打造成无线路由器"
category: "tech"
tags: ["树莓派", "路由器"]
---
{% include JB/setup %}

家里有个很挫的无线路由器，不能刷 OpenWRT，没什么可玩性，只好拿树莓派开刀了，顺便学点网络相关的知识。

既然是无线路由器，除了树莓派以外还需要一张无线网卡，本人使用的是 EP-N8508GS，这是一款廉价的无线网卡，支持软 AP，但是无法使用 aircrack，如果想用来破解无线网络，还是不要买这款了。如果你用的是其他型号的无线网卡，本文的大部分步骤应该仍然有效。如果是用的 360、百度、小米之类的随身 WiFi，可能有驱动问题，请参考[这篇文章](http://www.freemindworld.com/blog/2013/131010_360_wifi_in_linux.shtml)解决。

## 一、安装并配置 hostapd

为了使用无线网卡 AP 功能，需要安装 hostpad。如果你的芯片型号在支持之列，请直接从软件源安装：

```
sudo apt-get install hostapd
```

EP-N8508GS 使用的芯片型号是 Realtek RTL8188CUS，不能使用软件源里面的 hostapd，放狗搜发现国外大神的[这篇文章](http://www.jenssegers.be/blog/43/realtek-rtl8188-based-access-point-on-raspberry-pi)，使用 Realtek 官方提供的 hostapd 源码编译，经测试，工作正常。大致步骤如下：

从 GitHub 下载最新源码并解压：

```
wget https://github.com/jenssegers/RTL8188-hostapd/archive/v2.0-beta.tar.gz
tar -zxvf v2.0-beta.tar.gz
```

进入源码目录，编译安装：

```
cd RTL8188-hostapd-2.0-beta/hostapd
sudo make
sudo make install
```

（建议养成良好的习惯，不要直接 `make install`，先使用 `checkinstall` 打包成 deb，然后再安装，方便管理，此处略过。）

然后打开 hostapd 的配置文件 `/etc/hostapd/hostapd.conf`，修改 `ssid`、`wpa_passphrase` 为 WiFi 热点的名称和密码，比如我的：

```ini
# Basic configuration

interface=wlan0
ssid=Raspberry_Pi
channel=6
#bridge=br0

# WPA and WPA2 configuration

macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=123456
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
wmm_enabled=0

# Hardware configuration

driver=rtl871xdrv
ieee80211n=1
hw_mode=g
device_name=RTL8188CUS
manufacturer=Realtek
```

如果使用软件源的 hostapd，请修改 `/etc/default/hostapd`，设置 conf 文件路径：

```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

最后，重启 hostapd 服务，并设置开机启动：

```
sudo service hostapd restart
sudo update-rc.d hostapd defaults
```

## 二、安装并配置 udhcpd

为了让连到热点的设备能通过 DHCP 获取 IP，需要安装 DHCP 服务程序，可以使用 udhcpd 或 dnsmasq。安装 udhcpd：

```
sudo apt-get install udhcpd
```

然后打开 udhcpd 配置文件 `/etc/udhcpd.conf`，修改 `start`、`end` 为 DHCP 分配的 IP 段的起始和结束 IP，并修改 `opt dns`、`opt subnet`、`opt router` 等选项设置 DNS、子网掩码、路由器 IP，例如：

```
start           192.168.42.2        #default: 192.168.0.20
end             192.168.42.20       #default: 192.168.0.254

interface       wlan0               #default: eth0

remaining       yes                 #default: yes

opt     dns     192.168.42.1
option  subnet  255.255.255.0
opt     router  192.168.42.1
option  dns     8.8.8.8             # appened to above DNS servers
option  lease   864000              # 10 days of seconds
```

这个地方我设置路由器 IP 为 `192.168.42.1`，并设置分配的 IP 范围为 `192.168.42.2` - `192.168.42.20`，所有设备都在 192.168.42.0/24 这个子网内。

注意：上面设置 dns 服务器地址为 `192.168.42.1` 是因为我在树莓派上搭建了一个 DNS 服务器，如果你没有，请设置成其他公用 DNS，比如 `114.114.114.114`。

接着修改 `/etc/default/udhcpd`，注释掉 `DHCPD_ENABLED="no"` 这一行。

重启 udhcpd 服务，并设置开机启动：

```
sudo service udhcpd restart
sudo update-rc.d udhcpd defaults
```

## 三、配置 iptables、ifconfig

打开网络接口配置文件 `/etc/network/interfaces`，设置 wlan0 为静态 ip，如下：

```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0

iface wlan0 inet static
    address 192.168.42.1
    netmask 255.255.255.0
#    gateway 192.168.1.1
#    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

iface default inet dhcp
```

然后使配置生效：

```
sudo ifdown wlan0
sudo ifup wlan0
```

接着开始配置 NAT。修改 `/etc/sysctl.conf`，打开内核 IP 转发，在末尾添加：

```
net.ipv4.ip_forward=1
```

然后，添加 iptables 规则，将 wlan0 的包通过 eth0 转发：

```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

由于 iptables 设置重启后将消失，常用的方法是先保存下来：

```
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

然后启动时加载，编辑 `/etc/network/interfaces`，添加下面这一行:

```
up iptables-restore < /etc/iptables.ipv4.nat
```

也可以写一个启动脚本 `/etc/init.d/iptables`，执行以上 iptables 命令：

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          iptables
# Required-Start:    mountkernfs $local_fs
# Required-Stop:     mountkernfs $local_fs
# X-Start-Before:    networking
# X-Stop-After:      networking
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Iptables
# Description:       Debian init script for iptables
### END INIT INFO

. /lib/lsb/init-functions

function do_start {
    log_daemon_msg "Starting iptables service" "iptables"

    # hostapd rules
    if ls /etc/rc*.d/*hostapd; then
        iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
        iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
        iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
    fi

    log_end_msg $?
}

function do_stop {
    log_daemon_msg "Stopping iptables service" "iptables"

    # hostapd rules
    if ls /etc/rc*.d/*hostapd; then
        iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
        iptables -D FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCE$
        iptables -D FORWARD -i wlan0 -o eth0 -j ACCEPT
    fi

    log_end_msg $?
}

case "$1" in
    start)
        do_start
    ;;
    stop)
        do_stop
    ;;
    restart)
        do_stop
        do_start
    ;;
    *)
        echo "Usage: /etc/init.d/iptables {start|stop|restart}"
        exit 1
    ;;
esac

exit 0
```

然后，增加可执行权限，并设置开机自动执行：

```
sudo chmod +x /etc/init.d/iptables
sudo update-rc.d iptables defaults
```

以上，AP 的配置就基本完成了，重启一下，看看手机或笔记本是不是能连上你的 WiFi 热点。

## 四、AP 模式和普通模式切换

如果你想切换 AP 模式和普通模式，请保留 `/etc/network/interfaces` 里面注释掉的两行，并使用上面的启动脚本的方式配置 iptables。

打开 `/etc/wpa_supplicant/wpa_supplicant.conf`，设置 WiFi 的名称、密码以自动连接：

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="PHICOMM_68E238"
    proto=RSN
    key_mgmt=WPA-PSK
    pairwise=CCMP TKIP
    group=CCMP TKIP
    psk="123456"
}
```

最后，可使用下面的脚本切换（需要 root 权限）：

```bash
#!/bin/bash

if [ "$1" = 'on' ]; then
    service hostapd start
    update-rc.d hostapd defaults
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
    ifdown wlan0
    sed -i -e 's/^\( *address\) 192\.168\.1\.120$/\1 192\.168\.42\.1/' \
           -e 's/^\( *gateway .*\)$/#\1/' \
           -e 's/^\( *wpa-conf .*\)$/#\1/' \
           /etc/network/interfaces
    ifup wlan0
elif [ "$1" = 'off' ]; then
    service hostapd stop
    update-rc.d hostapd remove
    iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
    iptables -D FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -D FORWARD -i wlan0 -o eth0 -j ACCEPT
    ifdown wlan0
    sed -i -e 's/^\( *address\) 192\.168\.42\.1$/\1 192\.168\.1\.120/' \
           -e 's/^#\( *gateway .*\)$/\1/' \
           -e 's/^#\( *wpa-conf .*\)$/\1/' \
           /etc/network/interfaces
    ifup wlan0
else
    echo "usage: $(basename "$0") <on|off>"
    exit 1
fi
```

此处，普通模式同样使用静态 IP：`192.168.1.120`。

如果 `ipdown`、`ifup` 有问题，过一会儿再依次执行 `ipdown`、`ifup`，或者直接重启。

如果想继续给树莓派加上过滤网页和视频广告、科学上网等功能请阅读[下篇](/tech/raspberry-pi-ad-block.html)。

参考：

* <http://elinux.org/RPI-Wireless-Hotspot>
