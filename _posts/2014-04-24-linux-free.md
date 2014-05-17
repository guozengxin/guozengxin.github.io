---
layout: post
title: "Linux系统free命令"
description: "Linux系统下的free命令可以查看系统内存的使用情况"
category: linux
tags: [Linux, 内存管理, free命令]
comments: true
---

Linux系统中的[free](http://linux.die.net/man/1/free)命令是一个查看系统内存、交换区使用情况的命令。

### free命令输出

在shell下输入free命令：

	$ free
	             total       used       free     shared    buffers     cached
	Mem:      16332276   15024604    1307672          0     254168   11476680
	-/+ buffers/cache:    3293756   13038520
	Swap:      4194296       9588    4184708

<!-- more -->

#### 第二行

free的输出共有4行，第二行的输出是从操作系统角度来看的，在这一行中：

* 第一列（total）表示总量
* 第二列（used）是已经使用的量
* 第三列（free）是空闲的量
* 第四列（shared）是系统共享内存的量（这一列已经被废弃掉了，一般情况下都是0）
* 第五列（buffer）是系统buffer的缓冲区数据，这些数据是操作系统还未写到磁盘中的数据
* 第六列（cached）是系统缓存的数据，这些数据是操作系统从磁盘中读出来放到内存中用以提高操作系统运行速度的数据

在这一行中，第一列等于第二列和第三列的和。

#### 第三行

输出的第三行是从应用程序的角度看，内存的使用情况：

* 第一列（used）表示从应用程序的角度，内存使用的量
* 第二列（free）表示从应用程序的角度，内存使用的量

第三行的输出和第二行有以下关系：

	第三行的（used） = 第二行的（used）- 第二行的（buffers） - 第二行的（cached）

这个等式是说明了操作系统认为内存的使用量是应用程序使用量和操作系统缓存的量之和。

同理有：

	第三行的（free）= 第二行的（free）+ 第二行的（buffers） + 第二行的（cached）

#### 第四行

输出的第四行是交换区的信息，分别是交换的总量（total），使用量（used）和有多少空闲的交换区（free）。

### 释放内存

当系统的内存使用比较紧张时，可以用以下的命令释放一些被操作系统 cached 的内存：

	echo 3>/proc/sys/vm/drop_caches

### 参数

`-b, -k, -m, -g` 参数表示输出结果的单位，分别是字节，K字节，M字节，G字节。如：

	# free -m
				 total       used       free     shared    buffers     cached
	Mem:         15949      15727        222          0        221      13094
	-/+ buffers/cache:       2411      13538
	Swap:         4095         10       4085

`-o` 使用旧的格式，不输出第三行（-/+ buffers/cache）
`-s [delay]` 动态显示输出结果，第隔`delay`秒输出刷新一次结果
`-c [times]` 和`-s`参数配合使用，刷新`times`次之后结束
