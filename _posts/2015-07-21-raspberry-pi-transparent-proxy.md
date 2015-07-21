---
layout: post
title: "树莓派搭建透明代理"
keywords: ["代理", "翻墙", "科学上网", "shadowsocks"]
description: "在树莓派上搭建透明代理服务器，实现科学上网"
category: "tech"
tags: ["树莓派", "代理"]
---
{% include JB/setup %}

前面介绍了[如何将树莓派打造成无线路由器](/tech/turn-raspberry-pi-into-a-router.html)，以及[如何用树莓派屏蔽网页、视频广告](/tech/raspberry-pi-ad-block.html)，接着我们在树莓派上搭建透明代理服务器，实现科学上网。

## 一、ss-redir + chinadns + iptables

实现透明代理的方法很多，这儿介绍两种。第一种是 ss-redir + chinadns + iptables。

ss-redir 是 shadowsocks 的一部分，在 pc 上面一般用的是 ss-local。chinadns 是一个防污染 dns 服务器，和 shadowsocks 出自同一个作者之手（clowwindy）。基本原理是，先通过 chinadns 解析域名，获得真实 ip，然后通过 iptables 将指定 ip 段的请求转发给 ss-redir 监听的端口，ss-redir 将请求经过加密后转发给 shadowsocks 服务端 ss-server，由 ss-server 向目标服务器发出请求，这些过程对连接到热点的用户是透明的。

首先安装 shadowsocks，shadowsocks 有各种语言的版本，此处我们使用 libev 版本（c 版本）：

```
sudo apt-get install shadowsocks-libev
```

由于 ss-redir 需要配合 ss-server 使用，因此需要有一个墙外的 vps 运行 ss-server，对于手头不充裕的同学，推荐使用 [xvmlabs](https://xvmlabs.com/) 或 bandwagonhost，价格极低，但还算稳定，流量也够用，我已经用了一年有余。不久前，这货提供了 shadowsocks 一键安装功能，真是方(zuo)便(si)。ss-server 的安装与配置，与 ss-local、ss-redir 差别不大。

编辑配置文件 `/etc/shadowsocks-libev/config.json`：

```json
{
    "server": "x.x.x.x",
    "server_port": 1081,
    "password": "123456",
    "method": "rc4-md5",
    "local_address": "192.168.42.1",
    "local_port": 1080,
    "timeout": 60
}
```

请将 server 和 server_port 换成你的 vps 的 ip 和端口，加密方式和 ss-server 保持一致（推荐 rc4-md5，速度较快）。

shadowsocks 的启动脚本 `/etc/init.d/shadowsocks-libev` 默认启动 ss-local，请改为 ss-redir。如果你想树莓派本身也能使用代理，可以同时运行 ss-redir 和 ss-local，ss-local 的配置文件和 ss-redir 基本一致，去掉 `local_address` 设置即可（ss-server 也是一样）。

然后重启 shadowsocks 服务：

```
sudo service shadowsocks-libev restart
```

接着安装 chinadns，这个软件源里没有，需要自己编译：

```
git clone https://github.com/clowwindy/ChinaDNS
cd ChinaDNS
./configure && make
sudo make install
```

（再次建议打包成 deb 后安装，`sudo checkinstall`）

添加开机启动脚本 `/etc/init.d/chinadns`：

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          chinadns
# Required-Start:    $network $local_fs $remote_fs $syslog
# Required-Stop:     $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start ChinaDNS at boot time
### END INIT INFO

DAEMON=/usr/local/bin/chinadns
DESC=ChinaDNS
NAME=chinadns
PIDFILE=/var/run/$NAME.pid

test -x $DAEMON || exit 0

case "$1" in
  start)
    echo -n "Starting $DESC: "
    $DAEMON \
        -m \
        -c /usr/local/share/chnroute.txt \
        -p 15353 \
        1> /var/log/$NAME.log \
        2> /var/log/$NAME.err.log &
    echo $! > $PIDFILE
    echo "$NAME."
    ;;
  stop)
    echo -n "Stopping $DESC: "
    kill `cat $PIDFILE`
    rm -f $PIDFILE
    echo "$NAME."
    ;;
  restart|force-reload)
    $0 stop
    sleep 1
    $0 start
    ;;
  *)
    N=/etc/init.d/$NAME
    echo "Usage: $N {start|stop|restart|force-reload}" >&2
    exit 1
    ;;
esac

exit 0
```

其中，`-m` 参数表示启用压缩指针（DNS pointer mutation），`-c` 指定 chnroute 文件，`-p` 指定监听的端口，没有指定将使用 dns 默认的 53 端口，`-s` 指定下游 dns 服务器。

建议配合 dnsmasq 使用，支持缓存、tcp 查询，只需要修改 dnsmasq 的配置文件：

```
no-resolv
server=127.0.0.1#15353
```

使用 chnroute 文件可以区分国内外 ip，对于国内 ip 可以使用国内的 dns 服务器，如 114.114.114.114。chnroute 文件通过如下命令更新：

```
curl 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | grep ipv4 | grep CN | awk -F\| '{ printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > chnroute.txt
```

可以用 crontab 添加定时任务，每周更新一次。

然后启动 chinadns 服务，并设为开机启动：

```
sudo chmod +x /etc/init.d/chinadns
sudo service chinadns start
sudo update-rc.d chinadns defaults
```

为了让连到热点的设备使用我们搭建的 dns 服务器，修改 udhcpd 或 dnsmasq 的配置文件，设置 dns 服务器地址为 `192.168.42.1`，参考[这里](/tech/turn-raspberry-pi-into-a-router.html#二、安装并配置-udhcpd)。

最后，添加 iptables 规则。参考[这里](/tech/turn-raspberry-pi-into-a-router.html#三、配置-iptables、ifconfig)，通过启动脚本的方式自动配置 iptables 规则。

修改 `/etc/init.d/iptables`，在 `do_start` 函数里面添加：

```bash
# shadowsocks rules
iptables -t nat -N SHADOWSOCKS

# - google, chrome, youtube, appspot, blogspot, blogger, feedburner (google)
iptables -t nat -A SHADOWSOCKS -p tcp -d 173.194.0.0/16 -j REDIRECT --to-ports 1080
iptables -t nat -A SHADOWSOCKS -p tcp -d 203.208.32.0/19 -j REDIRECT --to-ports 1080
iptables -t nat -A SHADOWSOCKS -p tcp -d 216.239.32.0/19 -j REDIRECT --to-ports 1080
iptables -t nat -A SHADOWSOCKS -p tcp -d 74.125.0.0/16 -j REDIRECT --to-ports 1080
iptables -t nat -A SHADOWSOCKS -p tcp -d 64.233.160.0/19 -j REDIRECT --to-ports 1080
iptables -t nat -A SHADOWSOCKS -p tcp -d 216.58.192.0/19 -j REDIRECT --to-ports 1080

iptables -t nat -A SHADOWSOCKS -p tcp -j RETURN
iptables -t nat -A PREROUTING -p tcp -j SHADOWSOCKS
```

在 `do_stop` 函数里添加：

```bash
# shadowsocks rules
iptables -t nat -D PREROUTING -p tcp -j SHADOWSOCKS
iptables -t nat -F SHADOWSOCKS
iptables -t nat -X SHADOWSOCKS
```

这里面我只加了 google 的 ip 段，其他 ip 段请按以下步骤添加（取自[这里](https://hong.im/configure-an-openwrt-based-router-to-use-shadowsocks-and-redirect-foreign-traffic.html)）：

1. 使用 `dig` 或 `nslookup` 查询 chinadns 获得正确 ip。
2. 借助 [APNIC 的 whois 工具](http://wq.apnic.net/apnic-bin/whois.pl)，查询 ip 所属的 ip 段，注意单个 ip 和 ip 段的取舍，比如 google 可以加入整个 ip 段，其他的一些小站可能只有一个 ip，就只用加一个了。
3. 按以上 google 的格式添加 ip 段。

或者暴力一点，所有的国外请求均走代理，参考[这里](https://gist.github.com/wen-long/8644243)。

做完以后重启 iptables 服务，基本就大功告成了。

以上方法的缺点是配置比较麻烦，而且更新代理规则比较麻烦（得先将域名转成 ip）。

## 二、ss-local + privoxy

[上篇文章](/tech/raspberry-pi-ad-block.html)介绍了使用 privoxy 去广告，其实 privoxy 本身就是一个代理程序，由于支持 intercepting proxy，可用作透明代理。只需要写一些规则，对于匹配的 url 转发给下一级代理 shadowsocks，配置非常简单。

ss-local 的安装与配置跟 ss-redir 基本一致，不再详述。

privoxy 的安装与配置参照上篇文章，亦不再详述。只需要增加一个 actionsfile，并打开 `accept-intercepted-requests`：

```
actionsfile shadowsocks.action
accept-intercepted-requests 1
```

actionsfile `/etc/privoxy/shadowsocks.action` 内容仿照以下格式：

```
{+forward-override{forward-socks5 127.0.0.1:1080 .}}
.google.com
.google.com.hk
.googleapis.com
.googlecode.com
.googleusercontent.com
.googlevideo.com
.google-analytics.com
.chrome.com
.android.com
.gstatic.com
.appspot.com
.youtube.com
.ytimg.com
.goo.gl
.blogspot.com
.blogger.com
.feedburner.com
```

同样地，这里只加了 google 的一些网站，其他的自己添加吧，一目了然。

最后，别忘了添加 iptables 规则，同样参考[上篇文章](/tech/raspberry-pi-ad-block.html#二、去网页广告，privoxy-+-adblock-plus-规则)。

这种方法的优点自然是简单，缺点是受 privoxy 限制，只支持 http 和 https。

实际上，我是两种方法一起用的，将 privoxy 的 iptables 规则加在 shadowsocks 规则前面，这样 http、https 走第二种方法，其他协议走第一种方法，这样所有的协议都支持，而且对去广告没有影响。
