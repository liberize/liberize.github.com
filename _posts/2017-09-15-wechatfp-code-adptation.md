---
layout: post
title: "WeChatFp 插件适配"
keywords: ["WeChatFp", "微信", "指纹支付", "Xposed"]
description: "修改 WeChatFp 代码适配不同微信版本"
category: "tech"
tags: ["Xposed"]
---
{% include JB/setup %}

以微信国内版 6.5.13 为例。

# 一、反编译

准备工具：

1. apktool
2. dex2jar + jd-gui（或 jadx）


## 1. 使用 apktool 反编译 apk

    java -jar apktool_2.2.4.jar d com.tencent.mm.apk

得到目录 com.tencent.mm，后面会用到 res 目录下的资源文件

## 2. 使用 dex2jar 将 dex 转为 jar

解压 com.tencent.mm.apk 取出 classes.dex，classes2.dex

    ./dex2jar-2.0/d2j-dex2jar.sh classes.dex
    ./dex2jar-2.0/d2j-dex2jar.sh classes2.dex

(windows 请用 d2j-dex2jar.bat)

得到文件 classes-dex2jar.jar 和 classes2-dex2jar.jar

## 3. 使用 jd-gui 查看反编译后的代码

打开 classes-dex2jar.jar，定位到 com.tencent.mm.R，打开 R.class

打开 classes2-dex2jar.jar，定位到 com.tencent.mm.plugin.wallet_core.ui.l，打开 l.class


# 二、找资源 ID 和变量名

打开 WeChatFp 源码 ObfuscationHelper.java，可以看到以下资源 ID：

    Finger_icon=new int[]{2130838280,2130838289,2130838289,2130838298}[idx];
    Finger_title=new int[]{2131236833,2131236918,2131236918,2131236964}[idx];
    passwd_title=new int[]{2131236838,2131236923,2131236923,2131236969}[idx];

和混淆后的变量名：

    PaypwdView = new String[]{"qVO","ryk","ryM","rLB"}[idx];
    PaypwdEditText = new String[]{"vyO","wjm","wjX","wDJ"}[idx];
    PayInputView = new String[]{"mOL","nnG","nnZ","nol"}[idx];
    PayTitle = new String[]{"qVK","ryg","ryI","rLw"}[idx];
    Passwd_Text = new String[]{"qVK","ryz","rzb","rLQ"}[idx];

只需要找到对应的 ID 和变量名，加进去就行了，以下 1-3 为资源，4-8 为控件对应的变量

## 1. Finger_icon - 指纹图标

res/drawable-xxhdpi-v4 找到指纹图标 agl.png，

res/values/public.xml，搜索『agl』，找到

    <public type="drawable" name="agl" id="0x7f02031a" />

(可能有多个，选 type="drawable" 的那个)

0x7f02031a 转成十进制 2130838298

## 2. Finger_title - 指纹标题文本

res/values/strings.xml，搜索『请验证指纹』，找到

    <string name="dtx">请验证指纹</string>

（可能有多个，随便选一个）

res/values/public.xml，搜索『dtx』，找到

    <public type="string" name="dtx" id="0x7f081864" />

0x7f081864 转成十进制 2131236964

## 3. passwd_title - 密码标题文本

res/values/strings.xml，搜索『请输入支付密码』，找到

    <string name="du2">请输入支付密码</string>

res/values/public.xml，搜索『du2』，找到

    <public type="string" name="du2" id="0x7f081869" />

0x7f081869 转成十进制 2131236969

## 4. PaypwdView - 密码面板（EditHintPasswdView）

com.tencent.mm.plugin.wallet_core.ui.l 搜索『EditHintPasswdView』，找到

    public EditHintPasswdView rLB;

## 5. PaypwdEditText - 密码文本框（TenpaySecureEditText）

点击 EditHintPasswdView 跳转到定义，搜索『TenpaySecureEditText』找到

private TenpaySecureEditText wDJ;

## 6. PayInputView - 输入键盘（View）

搜索『isShown』找到后面紧跟『setVisibility』的地方，

    if (!l.this.nol.isShown()) {
        l.this.nol.setVisibility(0);
    }

点击 nol 跳转到定义，

    protected View nol;

## 7. PayTitle - 对话框标题（TextView）

因为标题文本为 Finger_title 或 passwd_title，按 passwd_title （步骤 3 的结果）去找，

com.tencent.mm.R 搜索『2131236969』，找到

    public static final int fja = 2131236969;

com.tencent.mm.plugin.wallet_core.ui.l 搜索『fja』，找到

    this.rLw.setText(R.l.fja);

点击 rLw 跳转到定义，

    public TextView rLw;

## 8. Passwd_Text - 密码/指纹切换（TextView）

res/values/strings.xml，搜索『使用指纹』，找到

    <string name="dn8">使用指纹</string>

res/values/public.xml，搜索『dn8』，找到

    <public type="string" name="dn8" id="0x7f08176c" />

0x7f08176c 转成十进制 2131236716，com.tencent.mm.R 搜索『2131236716』，找到

    public static final int ffD = 2131236716;

com.tencent.mm.plugin.wallet_core.ui.l 搜索『ffD』，找到

    this.rLQ.setText(getContext().getString(R.l.ffD));

点击 rLQ 跳转到定义，

    public TextView rLQ;


最后感谢作者 @dss16694 开源此插件的代码。
