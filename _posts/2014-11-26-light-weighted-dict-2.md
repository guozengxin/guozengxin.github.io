---
layout: post
title: "Linux终端的英汉词典(2)"
description: "编写一个轻量级的词典程序，数据来源于iciba.com，python语言编写，在linux平台上运行."
category: python
tags: [python, ldict, spider, lxml, linux, 终端]
comments: true
---

## 1. ldict简介

本文介绍[ldict][]的编写中的其它技术。包括**终端输出彩色文字**，**简单的交互式输入**实现等。

本项目的github地址: <https://github.com/guozengxin/ldict>

**安装方法**：

```python
python setup.py install
```

**依赖的库**: [lxml][]

**适用系统**：Linux, 装有Python运行环境的系统

<!-- more -->

## 2. 输出彩色文字

为了使输出的翻译结果更加清晰明了，加入了彩色文字输出的代码。在终端上输出彩色文字的格式为：

```bash
echo -e "\033[0;32m Hello World. \033[0m \n"
```

这段代码可以在终端上输出绿色的Hello World，其中`\033`是开始标记，`[0;`是ANSI控制码，`32`表示的是字的颜色，`\033[0m`表示清除颜色设置，如果不清除会影响后续输出的颜色。

有关字体色，背景色的说明：

字背景颜色(40-49)| 字背景颜色(30-39)
-----------------|:-----------------
40:黑            | 30:黑
41:深红          | 31:红
42:绿            | 32:绿
43:黄            | 33:黄
44:蓝            | 34:蓝
45:紫            | 35:紫
46:深绿          | 36:深绿
47:白            | 37:白

控制码的说明：

控制码           | 说明
-----------------|------------------
\33[0m           | 关闭所有属性
\33[1m           | 设置高亮度
\33[4m           | 下划线
\33[5m           | 闪烁
\33[7m           | 反显
\33[8m           | 消隐
\33[30m - \33[37m| 设置前景色
\33[40m - \33[47m| 设置背景色
\33[nA           | 光标上移n行
\33[nB           | 光标下移n行
\33[nC           | 光标右移n行
\33[nD           | 光标左移n行
\33[y;xH         | 设置光标位置
\33[2J           | 清屏
\33[K            | 清除从光标到行尾的内容
\33[s            | 保存光标位置
\33[u            | 恢复光标位置
\33[?25l         | 隐藏光标
\33[?25h         | 显示光标

在ldict中，写了一个简易的模块用来打印彩色文字：[源文件github地址](https://github.com/guozengxin/ldict/blob/master/ldutil/colorprint.py)

## 3. 简单交互式输入

为了实现交互式输入，就必须一个字符一个字符的读。程序中调用了两个模块实现了这一功能：[tty][]和[termios][]。

以下是我实现的获取字符的函数：

```python
import termios, tty
class _Getch:
    def __call__(self, length=1):
        fd = sys.stdin.fileno()
        old_settings = termios.tcgetattr(fd)
        try:
            tty.setraw(sys.stdin.fileno())
            ch = sys.stdin.read(length)
        finally:
            termios.tcsetattr(fd, termios.TCSADRAIN, old_settings)
        return ch
```

下面实现了一个从命令行获取一个单词的函数：
在获取一个字符的时候，我在函数中对下面几个字符做了处理：

1. 如果是回车符(`\r\n`)，则返回整个串，退出函数。
2. 如果是制表符(`\t`)，则什么也不做，继续读下一个字符。
3. 如果是删除符(`\b`)，则输出`\b \b`，意思是往回退一个字符，然后写一个空白，再往回退一个字符。这里如果只退的话，不会删除字符。
4. 如果是退出符(`\03\04`)，则结束程序，这两个字符分别是`Ctrl+C`和`Ctrl+D`。
5. 如果新字符的长度为3，则有可能是向上键(`\x1b[A`)或者(`向下键`)，对这两种字符来回顾历史记录。

```python
getch = _Getch()
def myinput(flag='> '):
    chars = []
    sys.stdout.write(flag)
    nowIndex = len(historyList)
    hislen = nowIndex
    while True:
        newChar = getInput()
        newLen = len(newChar)
        if newLen == 1:
            if str(newChar) in '\r\n':
                print ''
                break
            if str(newChar) in '\t':
                continue
            elif newChar == '\b':
                if chars:
                    del chars[-1]
                    sys.stdout.write('\b \b')
            elif str(newChar) in '\03\04':
                sys.exit(-1)
            else:
                sys.stdout.write(newChar)
                chars.append(newChar)
        elif newLen == 3:
            if newChar == '\x1b[A':
                if nowIndex > 0:
                    if nowIndex == hislen:
                        removeShow(chars)
                    elif nowIndex > 0:
                        removeShow(historyList[nowIndex])
                    sys.stdout.write(historyList[nowIndex - 1])
                    nowIndex -= 1
            elif newChar == '\x1b[B':
                if nowIndex < hislen:
                    removeShow(historyList[nowIndex])
                    if nowIndex < hislen:
                        sys.stdout.write(historyList[nowIndex + 1])
                    elif nowIndex == hislen - 1:
                        sys.stdout.write(''.join(chars))
                    nowIndex += 1

    if nowIndex < hislen:
        return historyList[nowIndex]
    else:
        return ''.join(chars)
```

## 总结

这两篇文章就是我写这样一个小程序的总结，目前在交互式输入和字符编码的输入方面还有一些bug，以后有时间再做做优化。

[termios]: https://docs.python.org/2/library/termios.html
[tty]: https://docs.python.org/2/library/tty.html
[lxml]: http://lxml.de/
[ldict]: https://github.com/guozengxin/ldict
[colorprint]: http://www.cnblogs.com/ruihong/archive/2012/10/22/linux_terminal_output_color_text.html
