---
layout: post
title: "shell终端假死的处理方法"
description: "处理shell环境下突然假死的问题"
category: linux
tags: [linux, securecrt, ctrl+s, 假死]
---

### shell终端假死

在shell终端下，通常是用securecrt登陆到远程服务器时，经常会遇到“假死”的情况，即什么都操作不了，只能关掉会话，然后重新打开一个，十分郁闷。

今天实在忍受不住，去搜索了一下，原来是`Ctrl-S`这个快捷键在作怪：
> Ctrl-S 和 Ctrl-Q 分别用来暂停和继续对屏幕的输出。它们不常用，不过您可能会误按 Ctrl-S (毕竟在键盘上 S 和 D 靠的很近)。所以，如果您碰上了不管您输入什么都不能在终端屏幕上看得的怪事时，请试着按按 Ctrl-Q。注意：您在 Ctrl-S 和 Ctrl-Q 之间输入的所有字符将会被一起显示到屏幕上。

原来，`Ctrl-S`这个快捷键被按下时就会“假死”，此时，只要输入`Ctrl-Q`就可以恢复了。

再也不用重开会话了！

<!-- more -->
