---
layout: post
title: "Sublime Text 升级手记"
keywords: ["Sublime Text", "升级"]
description: "Sublime Text 由2到3的升级记录"
category: "tech"
tags: ["sublime text"]
---
{% include JB/setup %}

用 SublimeText 有一段时间了，一直以来用的都是2版本，但是2已经没有更新了。中间也尝试过安装3，但是用了没多久就换回来了，主要还是因为插件的兼容问题。
最近因为某个插件弄得我很不爽，于是又想升级到3，考虑到过了这么久，插件兼容性应该没有问题，于是开始动手升级。

我的操作系统是 Ubuntu 13.04 amd64，所以从[官网][official]下载了对应版本的 deb 包安装（2版本没有 deb 包，要手动安装）。

安装完第一件事就是配置各种插件。

## 插件移植

Google 了一下，找到了这个[网址][compatible_list]，上面有 `Package Control` 作者整理的插件兼容列表。
一个一个对着看了以下发现大部分插件都没问题，其中有些需要切换到另一个分支。以下是我目前使用的插件列表（按拼音排列）：

1. `ApplySyntax` 自动检测语法，检测能力一般，貌似是根据前几行的特征检测的，比如 Python 第一行一般是 `!/usr/bin/env python`。
2. `ConvertToUTF8` 支持 GBK 编码，尽管很少用到 GBK。这个插件比另外一个GBK插件好用很多。
3. `Docblokr` 代码注释，比如在函数前输入 `/**` 回车，自动添加函数说明。这功能还是相当不错的。
4. `Git` 说实话没怎么用过（其实我连 Git 都没用熟），不做评论。
5. `HexViewer` 16进制查看，有了它就不需要其他十六进制编辑器了。
6. `InputHelper` 解决 Linux 下中文输入问题，其实这办法很坑爹，后面有更好的解决办法。
7. `JsFormat` JS 格式化插件，不搞前端，很少用到。
8. `MarkdownEditing` markdown 编辑，刚装上不久，第一印象就是配色方案很丑，尚未尝试，据说很不错。
9. `Markdown Preview` markdown 转换和预览，这个还不错，只是不支持自定义转换器，我想把转换器改成 redcarpet，后面再说。
10. `Package Control` 插件管理，这个不用说，必备，需要切换到 `python3` 分支。
11. `SideBarEnhancement` 侧边栏增强，几乎也是必备，方便工程管理。
12. `SublimeClang` C/C++/ObjectiveC 代码检测、补全插件，推荐安装。作者貌似又写了一个新插件，还不完善。
13. `SublimeGDB` GDB 插件，支持源码级调试，有了它，可以抛弃 IDE 了。
14. `SublimeLinter` Lint 插件，支持多种语言，由于我基本只用 Python 和 C/C++，而这两种都有对应插件，为防止冲突，暂时禁用了。
15. `SublimePythonIDE` Python 代码检测、补全插件，原先2里面叫 `SublimeRope`，不错，很好用。
16. `SublimeREPL` ST 中嵌入解释器，写 `Python` 时用来测试很方便。
17. `Tag` 标签插件，只用过自动格式化和自动补全，不确定是不是这插件的功能。
18. `Theme - Soda` Soda 主题，这个很受欢迎而且确实很漂亮，PS：我比较喜欢 `Soda Light` 和 `soda_classic_tabs`。
19. `TrailingSpaces` 去除行尾空格，一般，好像这功能已经内置了。
20. `URLEncode` URL 编码转换，很少用。

还有一些其他插件，比如 `AdbView`，`StringUtilities` 没有列出。其实用 ST 来写 Web 是非常好的，相应的插件也非常多。由于我很少用，就没有安装。

另外，我个人还收藏了一些'小'插件，比如

1. `copy_with_line_numbers.py` 顾名思义，带行号复制。
2. `patch_exec.py` 给 `exec` 模块打补丁，以支持 Build 时输入。
3. `toggle_user_setting.py` 使用快捷键快速更改某些设置，比如 `draw_white_space`，`line_numbers`等。

这些插件可以从我的[ST 仓库][st_repo]里下载，只需要扔到 `Package/User` 下就可以使用了（有些可能需要做额外设置）。

关于插件配置，请参考相关插件的主页或Wiki。

## 字体

为了字体问题郁闷了好几天，现在才勉强搞定。折腾了很久，得出一个很深的感触：ST 对中文的支持太差了！
且不说不支持GBK编码，输入法不能用，还有中文自动换行问题，单就中文字体支持这一点就可以看的出来：

* 不支持比例字体的 `Bold` 和 `Italic`
* 中英文两种字体一起显示时可能不在一条水平线上
* 中文 `Bold` 和 `Regular` 宽度不同时会发生文字重叠

其中第三条最坑爹，因为我试了几十种字体发现只有微软雅黑 `Bold` 和 `Regular` 宽度才是相同的。基于以上原因，字体的选择要符合以下条件：

1. 字体要有Bold和Italic版本
2. 字体为等宽字体，因为：
    1. 比例字体用作编程字体效果不佳（个人喜好）
    2. 比例字体在ST中Bold和Italic不能显示（即使有对应版本）
3. 若字体不包含中文，需要找到一款中文字体满足以下条件：
    1. 最好是TrueType字体，而且笔画不能太粗，渲染后应当清晰可见
    2. 中文字体和英文字体一起显示时应当显示在同一水平线上
    3. 中文字体要有Bold和Italic版本，并且Bold版本字符宽度要与Regular版本相同

其实包含中文的等宽字体看起来效果很差，比如 `WenQuanYi Micro Hei Mono`，英文字母和中文字符宽度相同，效果可想而知。这里要提到 `YaHei Consolas Hybrid` 这款混合字体，我个人非常喜欢，但是混合之后就不是等宽字体了，所以杯具了！因此还是中英文混搭比较好，这样也更加灵活。中英文混搭需要借助 Ubuntu 的 `fonts.conf` 文件来实现，ST本身不支持设置 Fallback 字体。

常见的适合编程的英文字体有：`Monaco`、`Consolas`、`Inconsolata`、`DejaVu Sans Mono`、`Courier New`等。但是 `Monaco` 和 `Inconsolata`没有粗体和斜体，所以很郁闷。Google之，可以找到对应的替代字体 `LucidaMonoEF` 和 `Inconsolata LGC`。
中文字体以黑体和圆体为佳。以下是我测试过的几组显示基本在一条水平线上的搭配：

* DejaVu Sans Mono -- WenQuanYi Micro Hei
* Inconsolata      -- STXihei
* Monaco           -- FZLanTingHei-M-GBK
* LucidaMonoEF     -- STHeiti
* LucidaMonoEF     -- LEXUS-HeiS-Medium-U

纠结了很久，最后我选择了 `Inconsolata LGC` 和 `微软雅黑`。选择雅黑主要基于上述 3.3，选择 `Inconsolata LGC` 是因为它跟 `微软雅黑` 勉强能够显示在一条水平线上，而且它也确实比较好看。

**更新：** 现在用的是 `LucidaMonoEF` 和 `STHeiti`。见后面截图。

配置起来很简单，在 `Settings` - `User` 中加入：

```json
{
    "font_face": "Inconsolata LGC",
    "font_size": 11,
    "line_padding_top": 1,
    "line_padding_bottom": 1
}
```

新建 `~/.config/fontconfig/fonts.conf`，内容如下：

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <match target="pattern">
    <test name="family" qual="any">
      <string>Inconsolata LGC</string>
    </test>
    <edit name="family" binding="strong" mode="append">
      <string>Microsoft YaHei</string>
    </edit>
  </match>
</fontconfig>
```

另外，我顺便改了一下 Ubuntu 系统的默认字体为华文黑体（`STHeiti`），显示效果比我之前用的微米黑好多了！
系统默认的是文泉驿正黑，可以在上述 `fonts.conf` 中修改，但只对当前用户有效。
我的办法是，先删除 `/etc/fonts/conf.d/64-wqy-zenhei.conf`（貌似是这个路径），然后修改 `/etc/fonts/conf.d/69-language-selector-zh-cn.conf` 将你喜欢的字体添加到 `serif`，`sans-serif` 和 `monospace` 字体族列表最前面就行了。
这里需要注意在每个 `edit` 标签中加入 `binding="strong"`，这一项在 Ubuntu 旧版本里面是有的，我之前没加，结果怎么改都没效果，改过之后应该是

```xml
<edit name="family" mode="prepend" binding="strong">
    <string>...</string>
</edit>
```

这是目前的效果：

![sublime text screenshot]({{ IMAGE_PATH }}/sublime-text-font-preview.png)

[official]: http://www.sublimetext.com/3
[compatible_list]: https://github.com/wbond/sublime_package_control/wiki/Sublime-Text-3-Compatible-Packages
[st_repo]: https://github.com/liberize/sublime-text-plugins

## 中文支持

### 1. 输入法

Sublime Text 在 linux 下无法使用中文输入法，解决方法很多，比如使用 `InputHelper` 插件等。目前最完美的解决办法是 `cjacker` 提出使用的 `LD_PRELOAD` 加载共享库的[方法][cjacker_solution]。

其实我很久之前就在 ST 论坛看到了那篇帖子，并且也试过，效果很不错。但是随后我就发现了一个瑕疵：
由于使用了设置 `LD_PRELOAD` 环境变量的方法，ST 的所有子进程将继承这一环境变量，从而这一改变将影响所有子进程，这会导致某些子进程不能正常运行。
比如，`Preferences - Browse Packages` 原本应该打开 nautilus，但是现在打不开了，因为 nautilus 使用的是 Gtk3， 而 ST 和 libsublime-imfix.so 使用的是 Gtk2，两种库不能混用。

解决思路很简单，只要清除 `LD_PRELOAD` 环境变量就行了。

一开始我想写个小插件，在插件加载时将 `LD_PRELOAD` 设为空，比如

```python
import os
os.environ['LD_PRELOAD'] = ''
```

结果发现没起作用，原因是 ST3 将插件加载功能放到了一个独立的进程 `plugin_host` 里，使用插件将只能改变 `plugin_host` 的环境变量。

然后，我将问题归结为 linux 系统中如何通过某种方式改变其他进程环境变量。首先就想到了 `GDB`， 利用 `GDB` 的 `attach` 功能来实现，写成脚本如下：

```bash
#!/bin/sh
st_pid=`pgrep sublime_text`
[ -z "$st_pid" ] && exit 1
gdb <<EOF >/dev/null 2>&1
attach $st_pid
call putenv ("LD_PRELOAD=")
detach
quit
EOF
exit 0
```

以 root 身份运行这个脚本就可以了。但是这也不是一个好的解决办法，因为开销大、速度慢。

接着，我又想到可以在共享库中设置（早该想到的），于是在 `sublime_imfix.c` 中加入了以下函数：

```c
void __attribute__ ((constructor)) on_load(void)
{
    // Clear `LD_PRELOAD` environment variable
    putenv("LD_PRELOAD=");
}
```

这个函数将在共享库加载时自动执行。经过测试，效果很好。

最后的代码和共享库可以在我的 [Github 仓库][st_repo]里找到。

### 2. 自动换行

ST 的中文换行一直都有问题，原因是对单词（word）的识别有问题，除非遇到英文或空格，否则它把整个中文段落都识别为一个单词。其实类似的问题在 Jekyll 里也存在，`truncatewords` 同样也不对。解决方法是让它把每个中文字符识别为一个单词。通过在语法里增加以下内容可以让 ST 正确换行：

```xml
<dict>
    <key>match</key>
    <string>[$£¥‘“〈《「『【〔＄（［｛｢￡￥]*[&#x3000;-&#x9fff;][!%,.:;?¢°’”‰′″℃、。々〉》」』】〕ぁぃぅぇぉっゃゅょゎ゛゜ゝゞァィゥェォッャュョヮヵヶゕゖㇰㇱㇲㇳㇴㇵㇶㇷㇸㇹㇺㇻㇼㇽㇾㇿ・ーヽヾ！％），．：；？］｝｡｣､･ｧｨｩｪｫｬｭｮｯｰﾞﾟ￠]*</string>
    <key>name</key>
    <string>meta.cjkword</string>
</dict>
```

这段代码来自 <http://typeof.net/m/sublime-text-2-chinese-wrap-fix.html>。对 CJK 字符都有效。
为了解决避头尾，这里增加了些表示标点的码位，核心就是个 `[&#x3000;-&#x9fff;]` 而已。

其实这方法我尝试过，但没有成功。

[cjacker_solution]: https://www.sublimetext.com/forum/viewtopic.php?f=3&t=7006&sid=f1493bca62b08b3961988d5b4647be84&start=10#p41343
[st_repo]: https://github.com/liberize/sublime-text-plugins/tree/master/sublime_imfix

## Markdown

之前在我的 Ubuntu 上写 Markdown 一直用的是 [`ReText`][retext]，但是这个 ReText 除了能实时预览外貌似没有其他优势，连替换功能都没有，另外我也不大喜欢专门安装一个编辑器来写 Markdown，所以自然想到用 Sublime Text 来代替。由于 ST 不支持嵌入网页，所以实时预览功能自然不可能实现了，但是可以将转换的 html 在浏览器中打开，配合 LiveReload 之类的插件，差不多也能实现实时预览的功能。另外，使用 ST 最大的优点就是通过插件支持丰富的自定义功能。

### 1. 语法

如果你喜欢使用 [`Github Flavored Markdown (GFM)`][gfm]，那么 [`Knockdown`][knockdown] 插件几乎就是必不可少了。`Knockdown` 支持 `GFM` 的语法高亮，其中最有用的当属 `Fenced Code Block` 支持。经过测试发现 `Knockdown` 本身的配色方案在我的 Ubuntu 上显示效果并不好。于是我把其中与 Markdown 相关的部分 copy 出来，写到了我的默认配色方案 [`Juicy`][juicy] 中，并精简了很多内容。晒晒效果：

![Markdown Syntax][markdown_syntax]

### 2. 预览

如果你希望能随时预览 Markdown 的转换效果，可以使用 [Markdown Preview][markdown_preview] 插件。目前这个插件支持 [`python-markdown2`][markdown2] 和 [`Github API`][github_api] 两种转换方式。前者使用 `python-markdown2` 作为转换引擎，支持包括 `Fenced Code Block` 在内的一些扩展特性；后者使用 Github 提供的 API 进行在线转换。由于我平时主要用 Markdown 写博客文章，而我的 Github Page 使用 [`redcarpet`][redcarpet] 作为 Markdown 转换器，所以自然也希望 Preview 功能使用 redcarpet 做转换器，遗憾的是 `Markdown Preview` 插件并不支持自定义转换器，所以我随手写了个 `markdown_preview.py` 用来代替 `Markdown Preview` 插件。

其实原理很简单，就是将当前编辑器的文本或选中文本保存到临时目录，然后调用 `redcarpet.rb` 进行转换。`redcarpet.rb` 中使用了 `pygments` 提供代码的语法高亮，同时增加了去除 YAML 头部的功能。

代码见[这里][markdown_preview_2]（没学过 ruby，照葫芦画瓢，见笑）。

目前存在的问题是，由于每次都将文本保存到磁盘，再从磁盘读取，而且还要启动 ruby 解释器，所以效率偏低。可以在 `markdown_preview.py` 使用 `subprocess` 模块启动 `redcarpet.py`，并通过管道传递数据（试过，不知为何没有成功）。

如果你使用其他转换器，如 `kramdown`，`rdiscount`，`maruku` 等略加修改应该就可以使用。

### 3. 编辑

Markdown 编辑插件目前只发现 [`MarkdownEditing`][markdown_editing]。这个插件除了提供大量的快捷键、snippet以外，还自带了语法定义和配色方案，不过貌似不支持 `Fenced Code Block` 等扩展语法，还有默认的配色方案我也不喜欢，所以直接去掉了。还有一个比较郁闷的地方是，作者似乎只考虑到了 OSX 用户，所以只设了 OSX 的 keymap，可以自己加到 `Default.sublime-keymap` 中，不过不确定是否都能使用。

[retext]: http://sourceforge.net/p/retext/home/ReText/
[gfm]: http://github.github.com/github-flavored-markdown/
[knockdown]: https://github.com/aziz/knockdown
[juicy]: https://github.com/liberize/sublime-text-plugins/tree/master/color_scheme
[markdown_syntax]: {{ IMAGE_PATH }}/markdown-syntax-highlight.png
[markdown_preview]: https://github.com/revolunet/sublimetext-markdown-preview
[markdown_preview_2]: https://github.com/liberize/sublime-text-plugins/tree/master/markdown_preview
[markdown2]: https://github.com/trentm/python-markdown2
[github_api]: http://developer.github.com/v3/
[redcarpet]: https://github.com/vmg/redcarpet
[markdown_editing]: https://github.com/ttscoff/MarkdownEditing
