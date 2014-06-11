---
layout: post
title: "在 OS X 中创建并使用内存盘"
keywords: ["Mac OS X", "内存盘"]
description: "在 OS X 中创建并使用内存盘"
category: "tech"
tags: ["mac os x", "内存盘"]
---
{% include JB/setup %}

由于我机子上有 8G 内存，平时基本用不到这么多，于是想到拿出一部分作为内存盘使用，然后把系统临时文件目录设置到内存盘里，这样可以加快运行速度，同时保护硬盘。所以，每次重装系统，设置内存盘都是必做的事情之一。

## 内存盘概述

Windows 对内存盘没有原生支持，但是有很多软件可以做到，只是 Windows 下的内存盘貌似都是静态分配的，也就是说如果分配了2个G的内存盘，那么这2个G就不能被其他程序使用了，即使实际占用小于2个G。

Linux 对内存盘支持最好，设置内存盘最方便，只要在 `/etc/fstab` 中加入一行：

```
tmpfs     /tmp    tmpfs  defaults,size=3G       0 0
```

就可以设置一个 3G 的内存盘，并且开机自动挂载到 `/tmp`。注意：Linux 下内存盘是动态分配的，3G 是最大大小。

Mac OS X 下设置内存盘同 Linux 一样不需要第三方软件来完成，利用系统自带的 `hdiutil` 和 `diskutil` 这两个命令行工具就行了：

```bash
DISK_NAME=RamDisk
DISK_SPACE=3072
diskutil erasevolume HFS+ $DISK_NAME `hdiutil attach -nomount ram://$(($DISK_SPACE*1024*2))`
```

OS X 下的内存盘也是动态分配的。只是默认情况下每次注销内存盘都会消失，登录时需要重新创建，而重新创建速度很慢，而且原来文件会消失。

## OS X 下创建内存盘的理想方法

为了解决前面提到的问题，达到 Linux 下同样效果，我搜索了很多关于 Mac 下使用内存盘的文章，也试过了很多 App，但是都存在这个问题。没办法，只好自己动手了。经过一番摸索，终于实现了只开机创建一次，注销时不会消失。

首先，创建 `~/Scripts/create_ram_disk.sh` 内容如下：

```bash
#!/bin/bash

DISK_NAME=RamDisk
MOUNT_PATH=/Volumes/$DISK_NAME
DISK_SPACE=3072

if [ ! -e $MOUNT_PATH ]; then
    device=`hdiutil attach -notremovable -nomount ram://$(($DISK_SPACE*1024*2))`
    [ -z "$device" ] && exit 1
    diskutil erasevolume HFS+ $DISK_NAME $device
    diskutil disableJournal $device
    diskutil disableOwnership $device
    mkdir $MOUNT_PATH/Caches
fi
exit 0
```

这是创建内存盘的主要代码，其实和前面方法是一样的，只是多加了点内容。

然后，在 `/Library/LaunchDaemons` 下新建 `com.me.ramdisk.plist`，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.me.ramdisk</string>
    <key>ProgramArguments</key>
    <array>
        <string>bash</string>
        <string>/Users/liberize/Scripts/create_ram_disk.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

并修改文件所有者和权限，终端下运行：

```bash
cd /Library/LaunchDaemons
sudo chown root:wheel com.me.ramdisk.plist
sudo chmod 644 com.me.ramdisk.plist
```

这样做将在开机时（非登陆时）自动运行创建内存盘的脚本，其中 `/Users/liberize/Scripts/create_ram_disk.sh` 是脚本路径。

最后，为了避免每次注销时卸载 RamDisk，终端下运行：

```bash
sudo defaults write /Library/Preferences/SystemConfiguration/autodiskmount AutomountDisksWithoutUserLogin -bool true
```

重启即可。

由于 OS X 临时文件主要在 `~/Library/Caches` 下，所以可以把 `~/Library/Caches` 链接到 Ramdisk：

```bash
cd ~/Library
rm -rf Caches
ln -s /Volumes/RamDisk/Caches Caches
```
