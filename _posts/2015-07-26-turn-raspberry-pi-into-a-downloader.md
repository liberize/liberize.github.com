---
layout: post
title: "将树莓派打造成多功能下载机"
keywords: ["树莓派", "下载机", "aria2", "transmission", "xware"]
description: "将树莓派打造成支持 http、bt、迅雷专用链的多功能离线下载机"
category: "tech"
tags: ["树莓派"]
---
{% include JB/setup %}

住的地方办的网络是 4M 小水管，被无良的代理商坑了，之前搞得欠费了好几次，最近老是间歇性断网，这种环境下弄个下载机就很有必要了。对于一个喜欢看电影、美剧的人来说，离线下载是我用的最多的也最实用的功能。我同时用了三个下载工具，aria2 用来多线程下载 http 资源，transmission 用来下载 bt 资源，xware 用来下载迅雷专用链资源。

## 一、需要的硬件

如果只是偶尔下点东西，可以用一个大一点的 sd 卡或 u 盘，不过价格会比较贵。建议还是通过 usb 接一块机械硬盘，2.5 寸 3.5 寸都可以。

如果通过 usb 接机械硬盘，需要一个 sata 到 usb 的转接器，可以是硬盘盒、硬盘底座或者转接线。我用的是 3.5 寸的 wd10ezex 1tb 硬盘，之前买了某宝上比较火的 orico 6619us3 硬盘底座，结果在使用 dlna 和 samba 的时候，只要一传输大文件，几秒钟之内就会断开连接，`dmesg` 显示 usb disconnect，`fdisk -l` 显示硬盘由原来的 /dev/sda 变成 /dev/sdb，后来找到了[这篇文章](https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=85449)，得知是 dock 不兼容。后来在官方的[兼容列表](http://elinux.org/RPi_VerifiedPeripherals#USB_to_IDE.2FSATA)里面找到了 JM20337 这款芯片，于是在某宝上买了一根廉价的转接线（其实是 5 根线），之后再没出过问题。

另外，使用 usb 接机械硬盘的一个常见问题是供电不足。建议硬盘外接电源供电，3.5 寸硬盘因为电压是 12V 必须额外供电，同时应该保证外接电源的额定电流足够大，可以避免很多奇怪的问题。

关于硬盘文件系统的选择，如果只用于树莓派，不用考虑 win 和 mac 的兼容性，建议使用 ext2/ext3/ext4。如果使用 ntfs，需要运行 ntfs-3g 的 daemon，会占用一定的资源（空闲时也有 3-5% 的 cpu）。我用的是 ext2，相比 ext4 没有日志，减少了磁盘读写。另外，挂载的时候要加上 `noatime` 参数，同样可以减少磁盘读写。

为了让硬盘（或 u 盘）插上后自动挂载，增加如下 udev 规则 `/etc/udev/rules.d/10-usbstorage.rules`：

```
KERNEL!="sd*", GOTO="media_by_label_auto_mount_end"
SUBSYSTEM!="block",GOTO="media_by_label_auto_mount_end"
IMPORT{program}="/sbin/blkid -o udev -p %N"
ENV{ID_FS_TYPE}=="", GOTO="media_by_label_auto_mount_end"
ENV{ID_FS_LABEL}=="EFI", GOTO="media_by_label_auto_mount_end"
ENV{ID_FS_LABEL}!="", ENV{dir_name}="%E{ID_FS_LABEL}"
ENV{ID_FS_LABEL}=="", ENV{dir_name}="Untitled-%k"
ACTION=="add", ENV{mount_options}="noatime"
ACTION=="add", ENV{ID_FS_TYPE}=="vfat", ENV{mount_options}="iocharset=utf8,umask=000"
ACTION=="add", ENV{ID_FS_TYPE}=="ntfs", ENV{mount_options}="iocharset=utf8,umask=000"
ACTION=="add", RUN+="/bin/mkdir -m 777 -p /media/%E{dir_name}", RUN+="/bin/mount -o $env{mount_options} /dev/%k /media/%E{dir_name}"
ACTION=="remove", ENV{dir_name}!="", RUN+="/bin/umount -l /media/%E{dir_name}", RUN+="/bin/rmdir /media/%E{dir_name}"
LABEL="media_by_label_auto_mount_end"
```

脚本来自网络，我修改了下，增加了 noatime 参数，并且不挂载 EFI 分区。

由于树莓派 24 小时不断电，大部分时间硬盘是空闲的，为了省电，同时保护硬盘，可以设置空闲的时候自动停转（spin down），进入 standby 状态。编辑 hdparm 的配置文件 `/etc/hdparm.conf`：

```
quiet 
apm = 127
spindown_time = 60
```

`apm` 设置高级电源管理功能，越小表示越激进，0-127 允许 spin down。`spindown_time` 设置停转的超时时间，1-240 的单位是 5s，因此设为 60 表示 5 分钟没有读写硬盘将停转。

这里顺便说一下，西数硬盘声名狼藉的 C1 问题（LCC 增长很快）就跟高级电源管理特性有关，解决办法一般是用 wdidle3 将固件里的超时时间改大。

## 二、安装并配置 aria2

安装 aria2:

```
sudo apt-get install aria2
```

修改配置文件 `~/.aria2/aria2.conf`：

```
dir=/media/NAS/Downloads

daemon=true
disk-cache=32M

enable-rpc=true
rpc-listen-all=true
rpc-allow-origin-all=true

max-concurrent-downloads=1
max-connection-per-server=10
min-split-size=10M
split=10
max-overall-download-limit=0
max-download-limit=0
max-overall-upload-limit=0
max-upload-limit=0

save-session=/home/pi/.aria2/aria2.session
save-session-interval=60
force-save=false
continue=true
input-file=/home/pi/.aria2/aria2.session

on-download-complete=/usr/local/bin/auto-move-file.sh
```

其中，`dir` 为下载目录，`disk-cache` 为磁盘缓存大小，`enable-rpc` 表示打开 rpc，这样可以通过 yaaw 等 webui 远程控制，`max-concurrent-downloads` 为同时下载的任务数，`min-split-size` 和 `split` 决定了分多少块下载，即多少个线程同时下载，`on-download-complete` 是一个 hook，下载完成时自动执行脚本。这里有个 `file-allocation` 选项我没有指定，如果你使用 ext4 或 ntfs 可以指定为 `falloc`，大文件也可以瞬间分配，详见[官方文档](http://aria2.sourceforge.net/manual/en/html/aria2c.html#cmdoption--file-allocation)。

写一个简单的启动脚本 `/etc/init.d/aria2`：

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          aria2
# Required-Start:    $network $local_fs $remote_fs
# Required-Stop:     $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: High speed download utility
### END INIT INFO

case "$1" in
  start)
    echo "Starting aria2... "
    sudo -u pi /usr/bin/aria2c >/dev/null 2>&1 &
    ;;
  stop)
    echo "Stopping aria2... "
    pkill aria2c
    ;;
  *)
    echo "Usage: /etc/init.d/aria2 {start|stop}"
    exit 1
    ;;
esac
exit 0
```

启动 aria2 并设为开机启动：

```
sudo chmod +x /etc/init.d/aria2
sudo service aria2 start
sudo update-rc.d aria2 defaults
```

也可以直接修改 `/etc/rc.local`，在末尾加上：

```
nohup sudo -u pi /usr/bin/aria2c >/dev/null 2>&1 &
```

配合 [yaaw](https://github.com/binux/yaaw) 或者 [webui-aria2](https://github.com/ziahamza/webui-aria2) 可以通过 web 页面管理下载任务。然后通过 ngrok 外网访问，就实现了远程控制。

配合 Chrome 浏览器的 [BaiduExporter 插件](https://github.com/acgotaku/BaiduExporter)，设置 rpc 地址之后，就可以直接将百度网盘的文件导出到 aria。或者如果有迅雷会员，配合 [ThunderLixianExporter 插件](https://github.com/binux/ThunderLixianExporter) 可以直接将迅雷离线导出到 aria2。最爽的是支持断点续传，如果一些资源下载失败，可以重新导出，然后继续下载。

如果你像我一样，希望下载完自动对媒体文件归类，可以将 `on-download-complete` hook 指定为如下脚本：

```bash
#!/bin/bash

if [ "$(ps -o comm= $PPID)" = 'aria2c' ]; then
    shift 2
fi

if [ "$1" = "" ]; then
    echo "usage: $(basename "$0") <file>"
    exit 0
fi

VIDEO_DIR=/home/pi/Shared/Videos
AUDIO_DIR=/home/pi/Shared/Music
IMAGE_DIR=/home/pi/Shared/Pictures

FILE_PATH="$1"
DIR_PATH="$(dirname "$FILE_PATH")"
FILE_NAME="$(basename "$FILE_PATH")"
FILE_NAME="${FILE_NAME%.*}"

auto_move()
{
    case "$1" in
        *.avi|*.mpg|*.wmv|*.mp4|*.mov|*.mkv|*.rm|*.rmvb|*.3gp|*.flv|*.swf|*.srt|*.ass)
                      echo "moving $1 to $VIDEO_DIR ..."
                      mv "$1" "$VIDEO_DIR";;
        *.mp3|*.wav|*.wma|*.mid|*.ape|*.flac)
                      echo "moving $1 to $AUDIO_DIR ..."
                      mv "$1" "$AUDIO_DIR";;
        *.jpg|*.jpeg|*.png|*.bmp|*.gif|*.tiff|*.psd|*.ico|*.svg)
                      echo "moving $1 to $IMAGE_DIR ..."
                      mv "$1" "$IMAGE_DIR";;
    esac
}

cd "$DIR_PATH"
case "$FILE_PATH" in
    *.tar.bz2|*.tbz2) tar jxvf "$FILE_PATH" -C "$FILE_NAME";;
    *.tar.gz|*.tgz)   tar zxvf "$FILE_PATH" -C "$FILE_NAME";;
    *.bz2)            bunzip2 -c "$FILE_PATH" > "$FILE_NAME";;
    *.rar)            mkdir -p "$FILE_NAME" && cd "$FILE_NAME" && unrar x "$FILE_PATH" && cd ..;;
    *.gz)             gunzip -c "$FILE_PATH" > "$FILE_NAME";;
    *.tar)            tar xvf "$FILE_PATH" -C "$FILE_NAME";;
    *.zip)            unzip -d "$FILE_NAME" "$FILE_PATH";;
    *.7z)             7z x -o"$FILE_NAME" "$FILE_PATH";;
    *)                auto_move "$FILE_PATH"; exit 0;;
esac

[ "$?" != 0 ] && exit 1

rm -f "$FILE_PATH"
for file in $(find "$FILE_NAME" -type f); do
    auto_move $file
done
```

其中，`VIDEO_DIR``AUDIO_DIR``IMAGE_DIR` 分别是视频、音频、图片的目录，如果下载的文件为压缩文件，会自动解压，然后归类。需要安装 `p7zip` 和 `unrar`（non-free version），另外源里面的 `zip`、`p7zip` 可能会出现中文乱码，需要打 patch。

## 三、安装并配置 transmission

安装 transmission：

```
sudo install transmission-cli transmission-common transmission-daemon
```

修改配置文件 `/etc/transmission-daemon/settings.json`：

```json
{
    "download-dir": "/media/NAS/Downloads", 
    "rpc-enabled": true, 
    "rpc-password": "123456", 
    "rpc-port": 9091, 
    "rpc-url": "/", 
    "rpc-username": "pi", 
}
```

我对 bt 没有很深的研究，只改了少数几个选项。其中 `rpc-password` 会自动改成加密的形式，不要奇怪。另外注意：必须先停了 transmission 的进程再修改配置文件，不然 transmission 退出的时候会把配置覆盖掉。

transmission 自带 webui，如果没有修改 `rpc-url`，可通过 http://<host>:9091/transmission/web/ 访问。如果不喜欢默认的 webui，可以使用第三方的 webui，比如 [transmission-web-control](https://github.com/ronggang/transmission-web-control) 和 [kettu](https://github.com/endor/kettu)（比较简洁）。在启动脚本 `/etc/init.d/transmission-daemon` 里通过 `TRANSMISSION_WEB_HOME` 环境变量设置 web home 目录，比如：

```
export TRANSMISSION_WEB_HOME="/var/www/kettu"
```

## 四、安装并配置 Xware

迅雷发布的路由器版远程下载工具 Xware，因为有 arm 版可以安装在树莓派上，也算是原生的解决方案了。

请到[迅雷官网](http://g.xunlei.com/forum-51-1.html)下载最新版 Xware，目前版本是 1.0.31，下载 Xware1.0.31_armel_v5te_glibc.zip。

解压之后，直接运行 `./portal`，连接成功后将打印 `THE ACTIVE CODE`，记住这个激活码，然后登录 <http://yuancheng.xunlei.com/>，使用激活码激活。打开任务管理页面，如果下载机上有一个小绿点表示下载机在线可用。

为了方便，写一个简单的启动脚本 `/etc/init.d/xware`：

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          xware
# Required-Start:    $network $local_fs $remote_fs
# Required-Stop:     $remote_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: thunder xware
# Description:
#
### END INIT INFO

cd /home/pi/Workspace/Xware

# Check the state of the command – this'll either be start or stop
case "$1" in
  start)
    # if it's start, then start xware using the details below
    sudo -u pi ./portal >/dev/null 2>&1 &
    sudo -u pi inotifywait -m -r -q --format '%w%f' -e moved_to /media/NAS /media/NAS/TDDOWNLOAD | while read file; do
        sudo -u pi /usr/local/bin/auto-move-file.sh "$file" >/dev/null 2>&1
    done &
    echo "Xware started."
    ;;
  stop)
    # if it's stop, then just kill the process
    sudo -u pi ./portal -s >/dev/null 2>&1 &
    killall inotifywait
    echo "Xware stopped."
    ;;
  *)
    echo "Usage: /etc/init.d/xware {start|stop}"
    exit 1
    ;;
esac
exit 0
```

运行目录请修改成你的解压目录，建议使用 pi 用户运行（迅雷这种<del>流氓</del>公司的软件，你不会想用 root 运行它的...）。这里用 inotifywait 监视文件系统变化，当下载完成以后自动运行前面的 `auto-move-file.sh` 归类脚本。

启动 Xware 并设为开机启动：

```
sudo chmod +x /etc/init.d/xware
sudo service xware start
sudo update-rc.d xware defaults
```

同样地，也可以直接修改 `/etc/rc.local`，在末尾加上：

```
nohup sudo -u pi ./portal >/dev/null 2>&1 &
```

迅雷远程也可以通过 webui 管理下载任务，之前激活设备的时候我们看到的就是它的 webui 了。

下载机打造完成以后，可以通过 samba、ftp 或 dlna 访问里面的资源，这个将在下一篇文章中介绍。
