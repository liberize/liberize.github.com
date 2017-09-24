---
layout: post
title: "修改微信集成 WechatFp 插件"
keywords: ["WechatFp", "微信", "指纹支付", "Xposed"]
description: "将 WechatFp 代码注入到微信"
category: "tech"
tags: ["Xposed"]
---
{% include JB/setup %}

以微信国内版 6.5.13 为例。

## 一、原理

反编译微信 apk，得到 smali 代码，然后修改 smali 代码，注入 WechaFp 的逻辑，最后再编译回去。

由于 smali 接近汇编，裸写有一定难度，而且容易出错，因此采用的方法是：

建一个工程，加入涉及到的几个微信类（不需要实现），将 WechatFp 的主要逻辑抽出来，然后在微信类里原来 WechatFp hook 的几个地方注入调用代码，编译该工程，得到 apk，然后反编译该 apk，攫取部分 smali 代码，注入到微信的 smali 代码里。

两个基本原则是：

1. 尽量将逻辑包装成一个方法，微信类里只需调用该方法，最小化修改。
2. 需要引用的微信变量尽量不要在 WechatFp 的逻辑里直接使用，因为微信变量名是混淆过的，不同版本不一样。

以上方法应该也能用于 WechatLuckyMoney。

## 二、反编译微信 apk

准备工具：

1. apktool

### 1. 使用 apktool 反编译 apk

    java -jar apktool_2.2.4.jar -r d com.tencent.mm.apk

`-r` 参数表示不反编译资源，因为微信的资源做过特殊处理，放在了外部 r 目录下，apktool 处理会有问题（回编出错）。

### 2. 去掉微信的 dex 校验

打开 smali/com/tencent/mm/d/a.smali，去掉 793 行的

    if-eqz v0, :cond_2

此处参考『(微信的二次打包)[http://www.jianshu.com/p/a0e6b3f15d78]』。

## 三、注入 WechatFp 的代码

### 1. 下载 WechatFpInject 代码

首先，下载我修改的工程代码，

    git clone https://github.com/liberize/WechatFpInject

其中微信混淆过的变量名主要在 com/tencent/mm/plugin/wallet_core/ui/l.java 里：

    public class l {
        public EditHintPasswdView rLB;  // passwdView
        protected View nol;             // payInputView
        public TextView rLw;            // payTitle
        public TextView rLQ;            // payMethod
        ...
    }

还有一个在 com/tencent/mm/wallet_core/ui/formview/EditHintPasswdView.java 里：

    public class EditHintPasswdView extends RelativeLayout {
        private TenpaySecureEditText wDJ;
        ...
    }

这些混淆过的变量名可以参考上篇文章『(WechatFp 插件适配)[(/tech/wechatfp-code-adaptation.html)]』获得。

### 2. 反编译得到 smali 代码

Android Studio 菜单 Build -> Build APK，编译 WechatFpInject 工程，得到 app-debug.apk。注意这个 apk 是不能运行的。

使用 apktool 反编译此 apk，

    java -jar apktool_2.2.4.jar d app-debug.apk

### 3. 注入 smali 代码到微信

#### 复制独立的 smali 文件

复制 smali 目录下的以下文件到微信的 smali 目录下相同路径：

1. android/support/v4/os/CancellationSignal\*.smali 3 个文件（指纹库依赖）
2. com/wei 整个目录（指纹库）
3. me/liberize 整个目录（WechatFp 的逻辑）

2、3 可以把里面的 BuildConfig.smali 和 R\*.smali 删除。

#### 注入 WechatFingerprint.enable 方法调用

复制 smali/com/tencent/mm/plugin/wallet/pay/ui/WalletPayUI.smali 里的

    invoke-static {p0}, Lme/liberize/wechatfpinject/WechatFingerprint;->enable(Landroid/app/Activity;)V

到微信 smali_classes3/com/tencent/mm/plugin/wallet/pay/ui/WalletPayUI.smali 和 smali_classes2/com/tencent/mm/plugin/wallet/balance/ui/WalletBalanceFetchPwdInputUI.smali 文件里 onCreate 方法的 .prologue 后。

    .method public onCreate(Landroid/os/Bundle;)V
        .locals 8

        .prologue

        ...

#### EditHintPasswdView 加入 getEditText 方法

复制 smali/com/tencent/mm/wallet_core/ui/formview/EditHintPasswdView.smali 里的

    .method public getEditText()Lcom/tenpay/android/wechat/TenpaySecureEditText;
        .locals 1

        .prologue
        .line 19
        iget-object v0, p0, Lcom/tencent/mm/wallet_core/ui/formview/EditHintPasswdView;->wDJ:Lcom/tenpay/android/wechat/TenpaySecureEditText;

        return-object v0
    .end method

到微信 smali_classes2/com/tencent/mm/wallet_core/ui/formview/EditHintPasswdView.smali 文件里。

#### 注入 WechatFingerprint.init 方法调用

复制 com/tencent/mm/plugin/wallet_core/ui/l.smali 里的

    iget-object v0, p0, Lcom/tencent/mm/plugin/wallet_core/ui/l;->rLB:Lcom/tencent/mm/wallet_core/ui/formview/EditHintPasswdView;

    iget-object v1, p0, Lcom/tencent/mm/plugin/wallet_core/ui/l;->rLw:Landroid/widget/TextView;

    iget-object v2, p0, Lcom/tencent/mm/plugin/wallet_core/ui/l;->nol:Landroid/view/View;

    iget-object v3, p0, Lcom/tencent/mm/plugin/wallet_core/ui/l;->rLQ:Landroid/widget/TextView;

    const v4, 0x7f02031a

    const v5, 0x7f081864

    const v6, 0x7f081869

    invoke-static/range {v0 .. v6}, Lme/liberize/wechatfpinject/WechatFingerprint;->init(Lcom/tencent/mm/wallet_core/ui/formview/EditHintPasswdView;Landroid/widget/TextView;Landroid/view/View;Landroid/widget/TextView;III)V

到微信 smali_classes2/com/tencent/mm/plugin/wallet_core/ui/l.smali 文件构造函数的 return-void 之前

    .method public constructor <init>(Landroid/content/Context;)V
        ...

        return-void
    .end method

#### 注入 WechatFingerprint.savePasswd 和 WechatFingerprint.disable 方法调用

复制 com/tencent/mm/plugin/wallet_core/ui/l.smali 里的

    iget-object v0, p0, Lcom/tencent/mm/plugin/wallet_core/ui/l;->rLB:Lcom/tencent/mm/wallet_core/ui/formview/EditHintPasswdView;

    invoke-static {v0}, Lme/liberize/wechatfpinject/WechatFingerprint;->savePasswd(Lcom/tencent/mm/wallet_core/ui/formview/EditHintPasswdView;)V

    invoke-static {}, Lme/liberize/wechatfpinject/WechatFingerprint;->disable()V

到微信 smali_classes2/com/tencent/mm/plugin/wallet_core/ui/l.smali 文件的 dismiss 方法的 return-void 之前


    .method public dismiss()V
        ...

        return-void
        ...

    .end method

## 4. 编译修改后的微信 apk

    java -jar ../apktool_2.2.4.jar b com.tencent.mm

com.tencent.mm/dist 目录下可以看到生成的 com.tencent.mm.apk

然后，提取原 apk 的 r 目录，并添加到新生成的 apk 里。

最后重新签名，就完成了。
