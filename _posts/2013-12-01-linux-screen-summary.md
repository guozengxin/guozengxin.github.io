---
layout: post
title: "linux screen命令使用小结"
description: "在linux下使用了screen命令之后，感觉这个命令确实有着强大的功能，对这个命令做一个总结。"
category: linux
tags: [linux, screen]
---
{% include JB/setup %}

### 什么是 screen 

screen是一款由 GNU 计划开发的用于命令行终端切换的自由软件。用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。screen 提供了**会话恢复**、**多窗口**、**会话共享**的功能。

### screen 相关概念

* 会话：screen 命令从 linux 终端创建出一组进程来管理多个窗口的操作，这个进程组叫一个会话。
* 窗口：一个会话中可以创建一个或多个窗口，每一个窗口都相当于一个 ssh 登录，可以执行任何 shell 程序。
* 断开会话：一个会话可以中断，在想要重新进入时可以再进入进行未完成的工作。

<!-- more -->

### screen 常用操作

1. 会话操作

- 直接用`screen`命令可以创建一个会话。
{% highlight bash %}
$ screen
{% endhighlight %}

- `screen -S`命令可以为会话指定一个别名。

{% highlight bash %}
$ screen -S test
{% endhighlight %}

- `screen -r`命令可以进入某一个会话。

{% highlight bash%}
$ screen -ls
There are screens on:
	2038.test       (Detached)
1 Sockets in /var/run/screen/S-root.
$ screen -r test
$
{% endhighlight %}

`screen -r test`或者`screen -r 2038`都可以进入 test 这个会话。事实上，输入的参数是模糊匹配的，只要输入的参数只能匹配一个会话，都可以成功进入 ，如`screen -r tes`或者`screen -r 203`。但是如果匹配到多个会话就会失败:

{% highlight bash %}
$ screen -ls    
There are screens on:
	10730.tes       (Detached)
	2038.test       (Detached)
2 Sockets in /var/run/screen/S-root.
$ screen -r tes 
There are several suitable screens on:
	10730.tes       (Detached)
	2038.test       (Detached)
Type "screen [-d] -r [pid.]tty.host" to resume one of them.
{% endhighlight %}

- `screen -d`命令可以在外部暂离一个会话。

- `screen -dr <name>`命令可以在外部使另一个终端暂离这个会话，然后在当前终端进入这个会话。

- `screen -XS <name> quit`命令可以在外部结束一个会话。

- 处于会话中，用`C-a d`命令从会话中暂离。(先按 `Ctrl+a`，再按`d`，后面的写法类似)

- `C-a :`可以在会话中输入控制台命令。如`C-a :`后输入`quit`命令可以结束掉当前会话。

- `C-a ?`显示命令帮助信息，帮助中列出了命令及其对应的快捷键，如下所示。前方的单词都可以在`C-a :`后输入进行操作，和快捷键作用相同，如`C-a :detach`等同于`C-a d`。

{% highlight bash %}
break       ^B b          license     ,             removebuf   =         
clear       C             lockscreen  ^X x          reset       Z         
colon       :             log         H             screen      ^C c      
copy        ^[ [          login       L             select      '         
detach      ^D d          meta        a             silence     _         
digraph     ^V            monitor     M             split       S         
displays    *             next        ^@ ^N sp n    suspend     ^Z z      
dumptermcap .             number      N             time        ^T t      
fit         F             only        Q             title       A         
flow        ^F f          other       ^A            vbell       ^G        
focus       ^I            pow_break   B             version     v         
hardcopy    h             pow_detach  D             width       W         
help        ?             prev        ^H ^P p ^?    windows     ^W w      
{% endhighlight %}

2. 窗口操作

在一个会话中会包含一个或者多个窗口，每个窗口相当于一个 ssh 登录。

- `C-a c` 可以在会话中创建一个窗口。

- `C-a k` 可以在会话中 kill 掉一个窗口。

- `C-a n` 在会话中切换窗口，切换到下一个窗口。(按窗口的创建顺序)

- `C-a p` 在会话中切换窗口，切换到上一个窗口。(按窗口的创建顺序)

- `C-a C-a` 在会话中切换窗口，在两个最近使用的窗口来回切换。

- `C-a 0..9` 在会话中切换到第 0..9 个窗口。

- `C-a z` 把当前会话放到后台执行，用 shell 的 fg 命令则可回去。

- `C-a w` 显示所有的窗口列表。

- `C-a "` 从列表中切换窗口。

- `C-a A` 设置窗口的别名，用别名可以方便从列表中切换窗口。

3. 窗口分割

screen 还有一个功能就是将窗口分割为多个。这样做的好处就是在同一个屏幕中观察多个 ssh 登录，或者观察多个程序的执行情况。在一些需要进行同步观察程序效果的情况下可以有着的良好的表现。

- `C-a S` 可以将当前窗口横向分割成两个区域。(多次执行将会分割成 3 个，4 个)

- `C-a X` 可以将分割的区域结束掉。

- `C-a :remove` 同`C-a X`命令。

- `C-a tab` 移动到下一个分割的区域。

- `C-a Q` 关闭除当前区域外的其他所有区域。

4. 其他技巧 

- screen 还有其他的一些技巧，在上面的介绍中也有所提及，下面列出一些自己用过的。

- `screen -wipe` 命令会清除状态为 dead 的会话（可能由于人为杀掉或者其他原因）。

- `screen -x <name>` 可以 attach 到一个处于 attached 状态的会话，而且两边的操作会同步进行。如果两边恰好处于同一个窗口，那么一边的操作会在另外一边演示，类似于屏幕共享。

- `screen -S <name> -X <cmd>` 命令可以发送一个命令到 screen 会话，如上面的`screen -XS <name> quit`命令可以在外部结束一个会话。

### 参考

1. Screen User's Manual: <http://www.gnu.org/software/screen/manual/screen.html>.
2. linux 技巧：使用 screen 管理你的远程会话: <http://www.ibm.com/developerworks/cn/linux/l-cn-screen/>.
