---
layout: post
title: "ANSI 控制码"
keywords: ["ANSI", "控制码"]
description: "ANSI 控制码简介"
category: "tech"
tags: ["控制码"]
---
{% include JB/setup %}

之前一直好奇如何在终端输出不同颜色的文本以及如何任意改变终端光标的位置，今天才知道原来用到了 ANSI 控制码。

参考资料：

* [维基百科](http://en.wikipedia.org/wiki/ANSI_escape_code)
* [Ubuntu手册](http://manpages.ubuntu.com/manpages/gutsy/zh_CN/man4/console_codes.4.html)

## 控制字符

控制字符，即 ASCII 控制码。ASCII 码小于 32 的字符都是控制字符。在 ASCII 中常用的控制字符包括：

名称    | 含义            | ASCII 码   | 键盘映射       | 作用
------ | --------------- | ---------- | ------------- | ---------------
BEL    | bell            | 0x07       | Ctrl+G        | 铃声
BS     | backspace       | 0x08       | Ctrl+H        | 后退一格
HT     | horizontal tab  | 0x09       | Ctrl+I        | 跳至下一个制表位
LF     | line feed       | 0x0A       | Ctrl+J        | 换行
CR     | carriage return | 0x0D       | Ctrl+M        | 回车
ESC    | escape          | 0x1B       | Ctrl+[        | 开始一个转义序列

在键盘上如何映射控制字符：按下 Ctrl 键和某个字符键，将该字符键 ASCII 编码字节的7位中，左起的两位强制定为0，从而产生出32个 ASCII 控制码之一。例如，按下 Ctrl 和字母 G（十进制编码为71，二进制为01000111），产生编码7（振铃符，十进制编码7，或二进制00000111）。
键盘上有些单个键能产生控制码。例如 Backspace 键通常产生编码8，Tab 是编码9，Enter 是编码13（有些键盘上 Enter 可能是编码10）。

## 控制序列

控制序列，即 ANSI 控制码。大多数终端模拟器（Unix，Linux，Windows）都支持 ANSI 控制码。ANSI 控制码以 ESC 字符（ASCII 27/0x1b/033）开头，对于两个字符的 ANSI 控制码，第二个字符范围是 ASCII 64-95 ('@'-'_')，然而大多数 ANSI 控制码都多于两个字符，并且以 ESC 和 [ 开头，这时将 ESC+[ （即 "\033["）称为 CSI（Control Sequence Introducer），这些控制码最后一个字符范围是 ASCII 64-126 ('@'-'~')。还有一种单字符 CSI (155/0x9B/0233)，但是不如 ESC+[ 用的多，而且可能不被某些设备支持。

### 1. 非 CSI 序列

部分非 CSI 序列：

序列      | 名称       |     作用
-------- | ---------- | --------------------------
ESC c    | RIS        |   重绘屏幕
ESC D    | IND        |   换行
ESC E    | NEL        |   新的一行
ESC H    | HTS        |   设置当前列为制表位
ESC 7    | DECSC      |   存储当前状态(光标坐标,属性,字符集)
ESC 8    | DECRC      |   恢复上一次储存的设置
ESC %    |            |   开始一个字符集选择序列
ESC (    |            |   开始一个 G0 字符集定义序列
ESC )    |            |   开始一个 G1 字符集定义序列
ESC >    | DECPNM     |   设置数字小键盘模式
ESC =    | DECPAM     |   设置程序键盘模式
ESC ]    | OSC        |   操作系统命令

有些控制序列效果可能与单个控制字符相同。

### 2. CSI 序列

CSI 序列的基本结构是：

```
CSI n1 ; n2... letter
```

最后一个字符 letter 决定 CSI 序列的动作。n1 ; n2.. 等参数是可选的，省略时将采用缺省值，一般是0或1。可以用一个问号代替参数序列。

部分 CSI 序列：

序列           | 名称       |     作用
-------------  | --------- | -------------------
CSI n A        | CUU       | 光标上移n个单位
CSI n B        | CUD       | 光标下移n个单位
CSI n C        | CUF       | 光标前移n个单位
CSI n D        | CUB       | 光标后移n个单位
CSI n E        | CNL       | 光标下移到第n行的第1列
CSI n F        | CPL       | 光标上移到第n行的第1列
CSI n G        | CHA       | 光标移动到当前行的指定列
CSI n ; m H    | CUP       | 光标移动到指定行和列(以1行1列为参照)
CSI n J        | ED        | 删除屏幕内容，0光标后(默认)，1光标前，2全屏幕
CSI n K        | EL        | 删除行内容，0光标后(默认)，1光标前，2整行
CSI n S        | SU        | 页面向上滚动n行
CSI n T        | SD        | 页面向下滚动n行
CSI n ; m f    | HVP       | 光标移动到指定行和列(以1行1列为参照)
CSI n m        | SGR       | 设置SGR参数，包括文本、背景颜色
CSI 6 n        | DSR       | 设备状态报告
CSI s          | SCP       | 保存光标位置
CSI u          | RCP       | 恢复光标位置
CSI ?25l       | DECTCEM   | 隐藏光标
CSI ?25h       | DECTCEM   | 显示光标

SGR 部分参数：

值       | 作用
-------- | ----------------------
0        | 重置所有属性
1        | 设置高亮度
4        | 打开下划线
5        | 闪烁
7        | 反转视频，交换前景色与背景色
22       | 设置正常亮度
24       | 关闭下划线
25       | 不闪烁
27       | 关闭反转视频
30-37    | 设置前景色
39       | 设置默认前景色
40-47    | 设置背景色
49       | 设置默认背景色

颜色0-7依次为：Black，Red，Green，Yellow，Blue，Magenta，Cyan，White。

## 实例

### 1. 用于 C 语言 `printf`

```
printf("\033[41;32m 字体背景是红色，字是绿色 \033[0m\n"); 
```

可设置为宏以方便使用：

```c
#define NONE         "\033[m"
#define RED          "\033[0;31m"
#define LIGHT_RED    "\033[1;31m"
#define GREEN        "\033[0;32m"
#define LIGHT_GREEN  "\033[1;32m"
//...
printf( RED "current function is %s " GREEN " file line is %d\n" NONE, __FUNCTION__, __LINE__ );
```

### 2. Shell 脚本中使用，`echo -e`

```bash
echo -ne "\033[32mtest\033[0m"                 # 显示绿色文字
echo -ne "\033[3;1H123"                        # 可以将光标移到第3行第1列处
export PS1="\[\e[34m\][\u@\h \W]\$ \[\e[0m\]"  # 修改PS1，用法稍有不同
```

显示旋转的光标，表示等待：

```bash
#!/bin/bash
charset=('|' '/' '-' '\')
i=0
echo -ne "\033[?25l"
while true; do
    echo -n "${charset[((i%4))]}"
    echo -ne "\033[1D"
    ((i++))
    sleep 0.2
done
```

除此之外，还可以做一个字符进度条，像 wget 那样，甚至可以写一个终端下的俄罗斯方块！

### 3. 设置 OSX 终端按键功能

打开终端，进入偏好设置，'设置' -> '键盘'。

1. Home 键：选择 '将字符串发送到shell'，设置字符串为 `\001`（Ctrl+A）。
2. End 键：同样设置字符串为 `\005`（Ctrl+E）。
3. Pg Up 键：同样设置字符串为 `\033[5~`（Esc[5~）。
4. Pg Dn 键：同样设置字符串为 `\033[6~`（Esc[6~）。
