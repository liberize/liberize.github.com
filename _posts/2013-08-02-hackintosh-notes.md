---
layout: post
title: "黑苹果安装手记"
keywords: ["黑苹果", "Acer 4738G"]
description: "Acer 4738G 上安装黑苹果的记录"
category: "tech"
tags: ["黑苹果"]
---
{% include JB/setup %}

去年暑假第一次接触黑苹果，由于当时显卡驱动始终搞不定，显示效果很差，一怒之下就把 OS X 分区给删除了。今年用了半个学期的 Ubuntu，上个月初，一时心血来潮，翻了翻远景，发现显卡驱动已经有人整成功了，于是又开始动手折腾了。玩过黑苹果的人都知道，驱动是需要解决的主要问题。由于我的笔记本 acer 4738g 算是老本了，大部分驱动问题前人已经解决了，所以整个过程比较轻松。

笔记本配置：

* 型号：Acer Aspire 4738g
* 系统：Mac OS X ML 10.8.3            (Chameleon)
* 显卡：AMD Radeon HD 6370M 1024M     (ATY_Init)
* 声卡：Realtek ALC 272               (VoodooHDA)
* 网卡：Broadcom BCM57780             (BCM5722D)
* 无线：Atheros AR5B97                (IO80211Family)

## 引导

爬了几天的帖子，发现很多大神都使用 Clover 引导，不过我比较懒，所以还是使用变色龙来引导。开始是用 Grub4Dos 加载 wowpc.iso 来引导 OS X 的，在 menu.lst 中加入：

```
title Goto Chameleon (win)
root (hd0,0)
map /myboot/images/wowpc.iso (0xff) || map --mem /myboot/images/wowpc.iso (0xff)
map --hook
chainloader (0xff)
```

使用 wowpc.iso 的好处是简单，但是如果要修改变色龙的设定就比较麻烦了。在 OS X 下双击挂载 wowpc.iso，将里面的内容拷贝到 ~/bootmedia 文件夹下，修改 Extra/com.chameleon.boot.plist，改完以后，打开终端，执行：

```bash
cd ~
hdiutil makehybrid -o wowpc.iso bootmedia/ -iso -hfs -joliet -eltorito-boot bootmedia/usr/standalone/i386/cdboot -no-emul-boot -hfs-volume-name "Chameleon" -joliet-volume-name "Chameleon" -iso-volume-name "Chameleon"
```

即可在 ~ 目录下生成新的 wowpc.iso。

最后我还是选择安装 Mac 版变色龙。用 pkg 安装后，重启出现

```
boot0: test
boot0: error
```

意识到我可能是 4KB 磁盘，所以重新手动安装了一次：

```bash
# 安装到 MBR
sudo ./fdisk440 -f boot0hfsNV -u -y /dev/rdisk0
# 安装到 Mac 分区的 PBR
sudo dd if=boot1hNV of=/dev/rdisk0s3 bs=4096
# 复制 boot 到根目录
sudo cp boot /
```

其实，关键是 `bs=4096`。以上命令可以在 Mac 或 Linux 下执行。

引导成功后修改变色龙参数如下（/Extra/com.chameleon.boot.plist 内容）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Default Partition</key>
    <string>hd(0,3)</string>
    <key>Graphics Mode</key>
    <string>1366x768x32</string>
    <key>Hide Partition</key>
    <string>hd(0,5) hd(0,6)</string>
    <key>Instant Menu</key>
    <string>Yes</string>
    <key>UseKernelCache</key>
    <string>Yes</string>
</dict>
</plist>
```

## 显卡驱动

显卡驱动修改主要参考以下帖子：

* [ATI 5系和6系显卡驱动&修改FB探讨](http://bbs.pcbeta.com/viewthread-1060313-1-1.html)
* [ATI Mobility HD 5650 取 EDID 开启 QE/CI 过程分享(Toshiba Sony Dell 成功)](http://bbs.pcbeta.com/viewthread-846572-1-1.html)
* [EDID之进阶篇](http://bbs.pcbeta.com/viewthread-1125390-1-1.html)
* [Acer 4738G驱动及配置分享，免修改kext驱动6370M](http://bbs.pcbeta.com/viewthread-1354667-1-1.html)

### 1. 添加显卡 ID

显卡ID：0x68E41002。

1. 打开 `ATI5000Controller.kext/Contents/Info.plist`，在 `ATY,Eulemur` 下 `IOPCIMatch`中添加显卡 ID。
2. 打开 `AMDRadeonAccelerator.kext/Contents/Info.plist`，在 `AMDCedarGraphicsAccelerator` 下 `IOPCIMatch`中添加显卡 ID。

### 2. 修改 FrameBuffer

将 `radeon_bios_decode` 和 `redsock_bios_decoder` 的结果整理如下：

接口       | senseid     | txmit      | enc
---------- | ----------- | ---------- | -----------
LVDS       | 0x04        | 0x10       | 0x00
HDMI       | 0x01        | 0x11       | 0x02
VGA        | 0x08        | 0x00       | 0x10

FrameBuffer 使用 Eulemur，原始参数：

```
04000000140000000001000001020104
00080000000200000071000012040402
10000000100000000000000000100001
```

修改参数为：

```
02000000400000000901000010000004
00080000040200000071000011020101
10000000100000000001000000000201
(屏蔽 VGA，将其 senseid 改为 0x01)
```

使用 0xED 打开 `ATI5000Contrller.kext/Contents/MacOS/ATI5000Contrller`，进行替换。

### 3. 加载 FrameBuffer

可以使用 `ATY_Init.kext` 或修改 DSDT 来实现，简单起见，使用 `ATY_Init.kext`。

### 4. 注入 EDID

这一步不是必须，但是注入 EDID 之后可以识别显示器，添加分辨率等。
通过 IORegistryExplorer 查看，VendorID 为 4ca3，ProductID 为 3050。
提取的 EDID 为：

```
00FFFFFFFFFFFF004CA3503000000000
00140103802013780A09E59757548A27
22505400000001010101010101010101
010101010101561E5610510016303020
250035AE100000190000000F00000000
00000000001EB4027400000000FE0053
414D53554E470A2020202020000000FE
00313430415430312D4730340A2000F9
```

在 `/System/Library/Displays/Override/DisplayVendorID-4ca3/DisplayProductID-3050` 中加入以下内容定制显示名称和可选分辨率：

```xml
    <key>DisplayProductName</key>
    <string>Samsung SEC3050</string>
    <key>dmdg</key>
    <data>
    AAAAAg==
    </data>
    <key>scale-resolutions</key>
    <array>
        <data>
        AAAFAAAAAtAAAAAB
        </data>
        <data>
        AAAFAAAAAwAAAAAB
        </data>
        <data>
        AAAFUAAAAwAAAAAB
        </data>
    </array>
```

## 声卡驱动

声卡驱动主要有两种思路：仿冒驱动或万能驱动。仿冒驱动修改了无数次，始终不能识别设备，心灰意冷，弃之。万能驱动即 VoodooHDA，虽然可用，但不完美。

旧版本 VoodooHDA 只有在睡眠之后才能发声，新版本无需睡眠即可发声，但一段时间之后声音模糊消失，睡眠之后正常，至今无解。

提取的 Pin Default 值：

```
Address: 0
Vendor Id: 0x10ec0272

11      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black
12      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black
13      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black
14      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black
15      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black
16      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black
17      10 01 13 99     [Fixed]     Speaker at Int ATAPI    Unknown
18      40 98 a1 03     [Jack]      Mic at Ext Left         Pink        VREF_80
19      30 09 a3 99     [Fixed]     Mic at Int ATAPI        Unknown     VREF_80
1a      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black       VREF_HIZ
1b      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black       VREF_HIZ
1d      2d 99 17 40     [N/A]       Speaker at Ext N/A      Pink
1e      f0 11 11 41     [N/A]       Speaker at Ext Rear     Black
21      20 40 21 03     [Jack]      HP Out at Ext Left      Green
```

修正的 Pin Default 值：

```
17      40 01 13 90     [Fixed]     Speaker at Int ATAPI    Unknown
18      20 90 81 01     [Jack]      Mic at Ext Left         Pink        VREF_80
19      10 01 a3 90     [Fixed]     Mic at Int ATAPI        Unknown     VREF_80
21      50 40 21 01     [Jack]      HP Out at Ext Left      Green
```

PathMap：

```
[Fixed] Speaker at Int ATAPI    17 > 0f > 02 = 23 > 15 > 2
[Jack]  Mic at Ext Left         08 > 23 > 18 = 8  > 35 > 24
[Fixed] Mic at Int ATAPI        09 > 22 > 19 = 9  > 34 > 25
[Jack]  HP Out at Ext Left      21 > 0d > 03 = 33 > 13 > 3
```

在 NodesToPatch 中修正 Speaker 节点和 Headphone 节点的 Config 值，将 Config 最后两个数字改为 50 和 5f，即可实现耳机和扬声器的自动切换（睡眠以后）：

```xml
    <key>NodesToPatch</key>
    <array>
        <dict>
            <key>Codec</key>
            <integer>0</integer>
            <key>Config</key>
            <string>0x99130150</string>
            <key>Node</key>
            <integer>23</integer>
        </dict>
        <dict>
            <key>Codec</key>
            <integer>0</integer>
            <key>Config</key>
            <string>0x0321405f</string>
            <key>Node</key>
            <integer>33</integer>
        </dict>
    </array>
```

由于很少用到麦克风，所以 Mic 未测试。

## 网卡和无线

网卡和无线很容易解决，直接安装对应的 kext 即可。无线还可以使用 DSDT 解决，参考 [SomeRy 的帖子](http://bbs.pcbeta.com/viewthread-1354667-1-1.html)。

## 其他硬件

PS2 驱动：VoodooPS2Controller.kext。可解决键盘、触摸板的识别问题。

睡眠：在 DSDT 中删除 EHC1 部分即可正常睡眠、唤醒，而且可以自动睡眠、手动睡眠（按电源键或 Fn+F4），唤醒只能按电源键。修改 DSDT 可以合盖睡眠：搜索 `Device (LID0)`，找到：

```
    Method (_LID, 0, NotSerialized)
    {
        If (ECON)
        {
            If (LEqual (^^LPCB.EC0.LID2, Zero))
            {
                Return (One)
            }
            Else
            {
                Return (Zero)
            }
        }
```

在 Else 里面，return 之前添加 `Notify (SLPB, 0x80)` 即可。
