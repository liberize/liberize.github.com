---
layout: post
title: "OS X 使用技巧"
keywords: ["Mac OS X", "技巧"]
description: "OS X 使用技巧"
category: "tech"
tags: ["mac os x"]
---
{% include JB/setup %}

用 OS X 有些天了，总结一些小技巧，以备忘。

## 快捷键设置

### 1. 更改修饰键

由于我主要使用笔记本上自带的 PS2 键盘，加之比较习惯 Windows 和 Linux 的操作习惯。故在“系统偏好设置” -> “键盘” -> “修饰键”中做如下修改：

* control => command （对应 PS2 和 USB 键盘上 Ctrl 键）
* option  => control （对应 PS2 键盘上 Win 徽标键，USB 键盘上 Alt 键）
* command => option  （对应 PS2 键盘上 Alt 键，USB 键盘上 Win 徽标键）

### 2. 更改 Home 键和 End 键

习惯了按 Home 和 End 键跳到行首和行尾，转到 OS X 很不习惯，果断改之。首先，运行 KeyFixer，将会在 `~/Library/KeyBindings` 下生成 `DefaultKeyBinding.dict` 文件，这样在多数应用程序中就可以按 Home 和 End 键跳到行首和行尾啦，但是 Firefox 不行（吐槽一下，Firefox 很多地方与 OS X 的融合不是很好，比如网页中右键没有“在字典中查找”，还有打开快播时顶栏和 Dock 消失等等，而 Chrome 没有这些问题）。然后，参照 [ANSI 控制码](/post/ansi-escape-code.html#toc_7) 一文修改终端 Home 和 End 键的行为。这样一来用起来就顺手多了。

### 3. 设置应用程序快捷键

OS X 下设置应用程序快捷键只需在“系统偏好设置” -> “键盘” -> “键盘快捷键”中修改就行了，基本上所有应用程序快捷键都可以设置，实在是太方便了！
比如让 Chrome 按 F5 刷新，只需选择应用程序为 Google Chrome，设置菜单标题为 “重新加载此页”，键盘快捷键为 F5 就可以了。

## Sublime Text 设置

提到 OS X 下 Markdown 编辑器，一般都会想到 Mou，可是 Mou 对 Fenced Code Block 支持不是很好，而且不能识别 YAML Front Matter。后来在搜索过程中发现一个牛X的在线 Markdown 编辑器 —— StackEdit，支持众多的 Markdown 扩展语法，支持实时预览、保存到本地、同步到 Google Drive 和 Dropbox、发布到 Github 等，而且以 Chrome 应用形式安装以后可以离线编辑，缺点是同样不能识别 YAML Front Matter。最后我还是决定使用 Sublime Text，配合自定义的语法高亮。

对于 Sublime Text，不得不提到的是 OS X 下很多默认设置都不合理，所以需要在 Settings - User 中按如下设置：

```yaml
"open_files_in_new_window": false,
"close_windows_when_empty": false,
"find_selected_text": true
```

至于各项什么意思，看名字便知。

## 一些好用的 App

### 1. XtraFinder

对我来说，基本必不可少，它有如下功能：

* 标签页浏览
* Cmd + X 剪切，Del 删除
* 打开终端，显示隐藏文件，以 Root 身份启动
* 右键新建文件
* 还有很多，不记得了

还有一款 TotalFinder 貌似功能差不多，没用过。

### 2. BetterZip

这个用的人应该不少，支持浏览压缩包文件，用起来和 Windows 下的 7Zip、WinRAR 差不多。不得不说，OS X 默认的归档管理器实在太坑爹了，双击自动解压，很多时候其实只需要查看压缩包中的部分文件，却不得不将其全部解压。

### 3. QuickSilver / Alfred

通过键盘快捷操作，很强大。觉得用 Spotlight 就够了，所以没有去折腾。

## 其他使用技巧

暂时就写这么多了，还有一些其他的使用技巧，比如

* 使用 DictUnifier 转换星际译王词典并自动添加到 Mac 自带字典程序。
* 利用 Automator 创建服务使得像图像缩放旋转，转换图像格式，给视频下载字幕（参考[这里](http://fduo.org/use-applescript-to-fetch-subtitles-from-shooter-cn/)）等常用操作都可以在 Finder 中右键完成。
* 创建并使用内存盘，加快系统运行速度（参考[这里](/post/create-ramdisk-in-osx.html)）。
