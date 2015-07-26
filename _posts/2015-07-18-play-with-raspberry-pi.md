---
layout: post
title: "树莓派折腾计划"
keywords: ["树莓派"]
description: "记录树莓派折腾的点点滴滴"
category: "tech"
tags: ["树莓派"]
---
{% include JB/setup %}

很早之前就听说过树莓派，只是一直没有机会入手，去年年底趁树莓派 2 发布之际，抢先于年前购入，从此开始了树莓派折腾之旅。

最近重燃写博客的激情，准备将这半年折腾树莓派的经历写成一系列文章，本文作为开篇。

目前暂且拟定如下计划，其中大部分已经实现，带链接的是已经记录下来的：

* 入门篇：
    * [使用 ssh 和 vnc 访问树莓派](/tech/access-raspberry-pi-via-ssh-and-vnc.html)
* 路由器篇：
    * [将树莓派打造成无线路由器](/tech/turn-raspberry-pi-into-a-router.html)（hostapd + udhcpd + iptables）
    * [使用树莓派屏蔽网页和视频广告](/tech/raspberry-pi-ad-block.html)（dnsmasq + hosts, privoxy + adblock plus, privoxy + ocplayer）
    * [使用树莓派搭建透明代理，实现科学上网](/tech/raspberry-pi-transparent-proxy.html)（ss-redir + chinadns + iptables, ss-local + privoxy）
* NAS 篇：
    * [将树莓派打造成多功能下载机](/tech/turn-raspberry-pi-into-a-downloader.html)（aria2 + transmission + xware）
    * 访问树莓派的媒体资源（minidlna + bubbleupnp + samba）
    * 将树莓派打造成电视盒子（kodi + shairport）
* 硬件篇：
    * 给树莓派加上液晶显示、温度监控和红外遥控（1602 液晶 + ds18b20 温度传感器 + 红外接收头）
    * 树莓派不用外设发射调频广播（gpio + 天线）
    * 使用树莓派 DIY 智能台灯（继电器 + 人体红外传感器）
    * 使用树莓派实现家庭监控系统（摄像头 + mjpg-streamer + motion）
* 软件篇：
    * 树莓派外网访问与远程控制（ngrok + nginx + 微信公众号）
    * 树莓派实现语音与人脸识别
* YY 篇：
    * 树莓派实现 12306 离线抢票（验证码一直在变，等快过年再整吧）
    * 树莓派实现宠物自动喂食器（一直想养一只喵星人来着）
    * 使用树莓派打造 3D 打印机（是不是感觉很高大上）
    * 使用树莓派打造四轴飞行器（想想都有点小激动）
