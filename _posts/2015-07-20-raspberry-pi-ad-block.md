---
layout: post
title: "树莓派屏蔽网页、视频广告"
keywords: ["树莓派", "去广告", "去视频广告"]
description: "树莓派实现屏蔽网页、视频广告"
category: "tech"
tags: ["树莓派", "去广告"]
---
{% include JB/setup %}

前面介绍了[如何将树莓派打造成无线路由器](/tech/turn-raspberry-pi-into-a-router.html)，接着我们让这个路由器更加智能，比如可以加上去网页广告、去视频广告和科学上网等功能。本来打算写成一篇文章，无奈太长，分开写吧，这篇先介绍怎样去除网页和视频广告。

## 一、去网页广告，dnsmasq + hosts

去网页广告的方法比较多，第一种是 dnsmasq + hosts。安装 dnsmasq：

```
sudo apt-get install dnsmasq
```

为了方便更新，使用别人维护的 hosts 源，将 hosts 文件转成 dnsmasq 的配置文件格式，脚本如下：

```bash
#!/bin/bash
# Address to send ads to (the RPi)
piholeIP="127.0.0.1"

# Config file to hold URL rules
eventHorizion="/etc/dnsmasq.d/adblock.conf"
whitelist=/home/pi/whitelist.txt

echo "Getting yoyo ad list..." # Approximately 2452 domains at the time of writing
curl -s -d mimetype=plaintext -d hostformat=unixhosts http://pgl.yoyo.org/adservers/serverlist.php? | sort > /tmp/matter.txt

echo "Getting winhelp2002 ad list..." # 12985 domains
curl -s http://winhelp2002.mvps.org/hosts.txt | grep -v "#" | grep -v "127.0.0.1" | sed '/^$/d' | sed 's/\ /\\ /g' | awk '{print $2}' | sort >> /tmp/matter.txt

echo "Getting adaway ad list..." # 445 domains
curl -s https://adaway.org/hosts.txt | grep -v "#" | grep -v "::1" | sed '/^$/d' | sed 's/\ /\\ /g' | awk '{print $2}' | grep -v '^\\' | grep -v '\\$' | sort >> /tmp/matter.txt

echo "Getting hosts-file ad list..." # 28050 domains
curl -s http://hosts-file.net/.%5Cad_servers.txt | grep -v "#" | grep -v "::1" | sed '/^$/d' | sed 's/\ /\\ /g' | awk '{print $2}' | grep -v '^\\' | grep -v '\\$' | sort >> /tmp/matter.txt

echo "Getting malwaredomainlist ad list..." # 1352 domains
curl -s http://www.malwaredomainlist.com/hostslist/hosts.txt | grep -v "#" | sed '/^$/d' | sed 's/\ /\\ /g' | awk '{print $3}' | grep -v '^\\' | grep -v '\\$' | sort >> /tmp/matter.txt

echo "Getting adblock.gjtech ad list..." # 696 domains
curl -s http://adblock.gjtech.net/?format=unix-hosts | grep -v "#" | sed '/^$/d' | sed 's/\ /\\ /g' | awk '{print $2}' | grep -v '^\\' | grep -v '\\$' | sort >> /tmp/matter.txt

echo "Getting someone who cares ad list..." # 10600
curl -s http://someonewhocares.org/hosts/hosts | grep -v "#" | sed '/^$/d' | sed 's/\ /\\ /g' | grep -v '^\\' | grep -v '\\$' | awk '{print $2}' | grep -v '^\\' | grep -v '\\$' | sort >> /tmp/matter.txt

echo "Getting Mother of All Ad Blocks list..." # 102168 domains!! Thanks Kacy
curl -A 'Mozilla/5.0 (X11; Linux x86_64; rv:30.0) Gecko/20100101 Firefox/30.0' -e http://forum.xda-developers.com/ http://adblock.mahakala.is/ | grep -v "#" | awk '{print $2}' | sort >> /tmp/matter.txt

echo "Getting simpleu ad list..."
curl -s https://raw.githubusercontent.com/vokins/simpleu/master/hosts | sed -e '1,/@SmartDirect/d' | grep '127\.0\.0\.1' | grep -v 'localhost' | grep -v '#' | grep -v '::' | awk '{print $2}' | sort >> /tmp/matter.txt

# Sort the aggregated results and remove any duplicates
# Remove entries from the whitelist file if it exists at the root of the current user's home folder
if [[ -f $whitelist ]];then
    echo "Removing duplicates, whitelisting, and formatting the list of domains..."
    cat /tmp/matter.txt | sed $'s/\r$//' | sort | uniq | sed '/^$/d' | grep -v -x -f $whitelist | awk -v "IP=$piholeIP" '{sub(/\r$/,""); print "address=/"$0"/"IP}' > /tmp/andLight.txt
    numberOfSitesWhitelisted=$(cat $whitelist | wc -l | sed 's/^[ \t]*//')
    echo "$numberOfSitesWhitelisted domains whitelisted."
else
    echo "Removing duplicates and formatting the list of domains..."
    cat /tmp/matter.txt | sed $'s/\r$//' | sort | uniq | sed '/^$/d' | awk -v "IP=$piholeIP" '{sub(/\r$/,""); print "address=/"$0"/"IP}' > /tmp/andLight.txt
fi

# Count how many domains/whitelists were added so it can be displayed to the user
numberOfAdsBlocked=$(cat /tmp/andLight.txt | wc -l | sed 's/^[ \t]*//')
echo "$numberOfAdsBlocked ad domains blocked."

# Turn the file into a dnsmasq config file and restart dnsmasq service
mv /tmp/andLight.txt $eventHorizion
sudo service dnsmasq restart
```

脚本来自 [pi-hole](https://github.com/jacobsalmela/pi-hole)，我做了适当修改，加了 simpleu 的 hosts 源，以屏蔽国内广告。如果不需要其中的某些源，把对应的 `echo` 和 `curl` 行注释掉就可以了。另外，可以指定白名单，默认路径是 `/home/pi/whitelist.txt`，格式为正则表达式，比如 `www\.baidu\.com`。

为了定期更新 dnsmasq 的配置文件，编辑 crontab：

```
sudo crontab -e
```

加入一项定时任务：

```
@weekly /usr/local/bin/update-dnsmasq-conf.sh
```

后面是脚本路径，此处设定为每周执行一次。

接下来还需要让连接到热点的设备使用我们自己搭建的 DNS，编辑 udhcpd 的配置文件 `/etc/udhcpd.conf`，设置 `opt dns` 选项：

```
opt     dns     192.168.42.1
```

其中，`192.168.42.1` 是无线网卡的 IP。

dnsmasq 带有 DHCP 功能，可以取代 udhcpd，如果使用 dnsmasq 作为 DHCP 服务器，请参照文档修改。

改完以后，重启 udhcpd 服务，将设备连接到热点，检查 DNS 是否已设置成功。

以上方法的缺点是，通过 hosts 屏蔽广告，误杀比较严重，因为只能匹配域名，无法匹配详细的 url。

## 二、去网页广告，privoxy + adblock plus 规则

我最后用的是第二种方法：privoxy + adblock plus 规则。

privoxy 支持 intercepting proxy，支持基于规则的请求拦截、内容过滤，配合完善、精准的 adblock plus 规则，去广告效果可以与 adblock plus 媲美。只是从 abp 规则向 privoxy 规则的转换比较复杂，找了几个脚本（比如[这个](https://github.com/Andrwe/privoxy-blocklist)），处理得都不是很好，最后用了一个 [haskell 程序](https://projects.zubr.me/wiki/adblock2privoxy)。只是作者没有提供 arm 版本的二进制文件，只能自己编译，而 haskell 的编译巨坑无比，软件源里面的 ghc 版本太旧，没有带 ghci，导致有些包编译不过，所以只能先用旧的 ghc 编译新的 ghc，这其中无数的坑自然不必说，因此我只好在另一台 vps 上面跑 adblock2privoxy 二进制文件，然后让树莓派定期去拉取生成的 privoxy 规则文件。

直到不久之前，从 stackoverflow 上面[这个回答](http://stackoverflow.com/a/29380559) 的评论中得知 debian 的 unstable 源里面提供了带 ghci 的 ghc-7.8.4，于是我赶紧修改软件源，装上了新版 ghc，然后按官网说明编译二进制文件，中间运行 `cabal update` 和 `cabal install ...` 的时候无数次卡死，不过最后还是成功编译出来了，并且运行正常。附上编译好的二进制文件：<a href="http://pan.baidu.com/s/1ntxfIB3" title="前往网盘下载"><button class="blue"><i class="icon-download-alt"></i> 百度网盘</button></a>。

废话少说，先安装 privoxy：

```
sudo apt-get install privoxy
```

（我是自己编译的最新版 3.0.23，不过忘了为什么要自己编译了。）

然后编辑配置文件 `/etc/privoxy/config`，设置 `actionsfile` 和 `filterfile`，并打开 `accept-intercepted-requests`：

```
actionsfile ab2p.action
actionsfile ab2p.system.action
filterfile ab2p.filter
filterfile ab2p.system.filter

accept-intercepted-requests 1
```

接着按照官方的说明，配置 adblock2privoxy。

元素隐藏功能需要 web server 支持，我用的 nginx，在配置文件加上：

```
location ~ ^/[^/.]+\..+/ab2p.css$ {
    # first reverse domain names order
    rewrite ^/([^/]*?)\.([^/.]+)(?:\.([^/.]+))?(?:\.([^/.]+))?(?:\.([^/.]+))?(?:\.([^/.]+))?(?:\.([^/.]+))?(?:\.([^/.]+))?(?:\.([^/.]+))?/ab2p.css$ /$9/$8/$7/$6/$5/$4/$3/$2/$1/ab2p.css last;
}

location ~ (^.*/+)[^/]+/+ab2p.css {
    # then try to get CSS for current domain
    # if it is unavailable - get CSS for parent domain
    try_files $uri $1ab2p.css;
}
```

使用如下脚本运行 adblock2privoxy，更新 privoxy 配置：

```bash
#!/bin/bash

privoxy_dir=/etc/privoxy
web_dir=/var/www/privoxy
task_file=/home/pi/adblock2privoxy/ab2p.task
adblock_lists=(https://easylist-downloads.adblockplus.org/easylistchina.txt)

echo "=> running adblock2privoxy ..."
if [ -f $task_file ]; then
    adblock2privoxy -t $task_file
else
    adblock2privoxy -p $privoxy_dir \
                    -w $web_dir \
                    -d 192.168.42.1 \
                    -t $task_file \
                    ${adblock_lists[@]}
fi

echo "=> restarting privoxy service ..."
sudo service privoxy restart
```

这里我只用了 EasyListChina，因为浏览国外网站比较少，而 EasyList 体积很大，担心影响速度。可以在括号内加入其他订阅地址，空格分隔。

可以在 crontab 加上定时任务，每周更新一次。

最后，添加 iptables 规则，将 80 端口的请求转发给 privoxy 监听的 8118 端口。参考[上篇文章](/tech/turn-raspberry-pi-into-a-router.html) ，通过启动脚本的方式自动配置 iptables 规则。

在 `do_start` 函数中加上：

```
# privoxy rules
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8118
```

在 `do_stop` 函数中加上：

```
# privoxy rules
iptables -t nat -D PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8118
```

## 三、去视频广告

众所周知，去除视频广告不能直接屏蔽广告 url，不然会黑屏，一般采取替换 flash 播放器的方法。之前有个 OpenGG Clean Player，可惜后来被一些无耻的人滥用，作者无奈关掉了，好在后继有人，Gesion 继续维护了这个项目，见其[博客](http://blog.onens.com/onens-clean-player.html)。此处我们将通过 privoxy + Onens.Clean.Player 实现去视频广告的功能。

原理很简单，利用 privoxy 的强大功能，在视频页面插入 Onens.Clean.Player 的 js 脚本即可，另外需要解决一个跨域访问的问题。

编辑 privoxy 配置文件，分别增加一个 `actionsfile` 和 `filterfile`：

```
actionsfile ocplayer.action
filterfile ocplayer.filter
```

`ocplayer.action` 内容：

```
{+forward-override{forward api.youku.com:80}}
.youku.com/crossdomain\.xml$

{+filter{ocplayer}}
.youku.com/.*\.html
.tudou.com/.*\.html
.iqiyi.com/.*\.html
.sohu.com/.*\.shtml
```

我只加了 youku、tudou、iqiyi、sohu 的链接，只有匹配这些 url 时才有效。

`ocplayer.filter` 内容：

```
FILTER: ocplayer Insert onens.clean.player.user.js
s@</head>@\n<script type="text/javascript" src="http://gesion.duapp.com/script/onens.clean.player.user.js"></script>\n$0\n@
```

两个文件放到 privoxy 的配置文件目录 `/etc/privoxy/`，然后重启 privoxy 服务即可。
