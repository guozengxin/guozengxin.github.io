---
layout: post
title: "linux下dd命令介绍"
description: "dd命令是Linux下操作磁盘的命令，可以进行数据拷贝，并且拷贝过程中能够进行指定的转换。"
category: linux
tags: [linux, unix, dd]
comments: true
---

### 简介

**dd**命令是Linux/UNIX下一个操作磁盘的命令，可以指定大小拷贝数据，并在拷贝中可以进行指定的转换。大家都知道，Linux/UNIX下常用的拷贝命令是**cp**, **dd**命令与**cp**命令的主要区别是：1. **dd**命令是对块进行操作的，**cp**是对文件进行操作；2. 和**cp**比较，**dd**命令更低级，通过合适的运用能完成一些特殊的操作。

<!-- more -->

### dd命令参数说明

* `if=FILE` 从*FILE*文件中读数据而不是标准输出（**dd**命令默认从标准输入读数据）

* `of=FILE` 输出到*FILE*文件而不是标准输出（**dd**命令默认输出到标准输出）

* `ibs=BYTES` 设置输入的块大小为*BYTES*字节，即**dd**命令一次读多少个字节。（默认大小是512 bytes）

* `obs=BYTES` 设置输出的块大小为*BYTES*字节，即**dd**命令一次写多少个字节。（默认大小是512 bytes）

* `bs=BYTES` 同时设置输入和输出的块大小为*BYTES*字节，这个设置会覆盖`ibs`和`obs`设置。另外，如果没有设置`conv`选项，每一个输入块会拷贝为每一个输出块。

* `cbs=BYTES` 设置转换的块大小为*BYTES*字节，即**dd**命令一次转换多少个字节。

* `skip=BLOCKS` 拷贝跳过输入文件开头的*BLOCKS*个块，块的大小由`ibs`指定。

* `seek=BLOCKS` 拷贝跳过输出文件开头的*BLOCKS*个块，块的大小由`obs`指定。

* `count=BLOCKS` 从输入文件拷贝*BLOCKS*个块，块的大小由`ibs`指定。（默认拷贝整个输入文件）

* `conv=CONVERSION[,CONVERSION]...` 按照*CONVERSION*指定的参数转换文件。*CONVERSION*的参数下一节说明。

* `iflag=FLAG[,FLAG]...` 以*FLAG*参数指定的方式试问输入文件。*FLAG*参数后面介绍。

* `oflag=FLAG[,FLAG]...` 以*FLAG*参数指定的方式试问输出文件。*FLAG*参数后面介绍。

### 转换参数说明

* `ascii` 从[**EBCDIC**](http://zh.wikipedia.org/wiki/EBCDIC)转换为[**ASCII**](http://zh.wikipedia.org/zh-cn/ASCII), 这是1:1的转换。

* `ebcdic` 从[**ASCII**](http://zh.wikipedia.org/zh-cn/ASCII)转换为[**EBCDIC**](http://zh.wikipedia.org/wiki/EBCDIC), 这是1:1的转换。

* `ibm` 从[**ASCII**](http://zh.wikipedia.org/zh-cn/ASCII)转换为交替的**EBCDIC**编码，这不是1:1的转换。`acsii`, `ebcdic`和`ibm`这3个参数是互斥的。

* `block` 对输入的每一行，输出为`cbs`参数指定的字节数。不够的部分以空格填充。该选项与`ascii`, `ebcdic`, `ibm`, 和`unblock`选项冲突。

* `unblock` 在的每一个`cbs`大小的输入块上，去除多余的尾随空格。该选项与`ascii`, `ebcdic`, `ibm`, 和`block`选项冲突。

* `lcase` 将所有字母转换为小写。

* `ucase` 将所有字母转换为大写。

* `swab` 交换输入的每对字节。

* `noerror` 当读取错误时继续转换。

* `nocreat` 不创建输出文件，即命令执行前输出文件必须存在。

* `excl` 创建输出文件，如果输出文件已经存在，则发生错误。该选项与`nocreate`冲突。

* `notrunc` 不截断输出文件。

* `sync` 用**zero**字节将每个输入块填充到由`ibs`值指定的长度，如果指定了`block`或者`unblock`选项，则用改用空格填充。

* `fdatasync` 在命令结束之前将数据写入磁盘。

* `fsync` 在命令结果之前将数据和元信息写入磁盘，与`fdatasync`的区别是`fdatasync`只影响文件的数据部分，而`fsync`还影响文件的属性部分。

### *FLAG*参数说明

* `append` 以*append*方式写文件。

* `cio` 以并发的I/O方式操作数据。

* `direct` 以直接的I/O机制操作数据，避免进行数据缓存。

* `directory` 如果文件不是一个目录，则失败。大多数的操作系统不允许I/O到一个目录，所以这个参数被限制。

* `sync` 使用同步的I/O方式对数据和文件属性。

* `nonblock` 使用非阻塞的I/O方式。

* `noatime` 不更新文件的访问时间。

* `nofollow` 不进入符号链接。

* `nolinks` 如果文件是multiple硬连接，则失败。

* `binary` 二进制I/O。

* `text` 文本I/O。

### dd命令应用示例

(1). 将本地的/dev/hdb整盘备份到/dev/hdd

```bash
dd if=/dev/hdb of=/dev/hdd
```

(2). 备份磁盘开始的512个字节大小的MBR信息到指定文件

```bash
dd if=/dev/hda of=/root/image count=1 bs=512
```

(3). 将备份的MBR信息写到磁盘开始部分

```bash
dd if=/root/image of=/dev/had
```

(4). 利用/dev/zero文件，创建一个大小为256M的文件

```bash
dd if=/dev/zero of=/swapfile bs=1024 count=262144
```

(5). 利用随机的数据填充硬盘，用来销毁数据

```bash
dd if=/dev/urandom of=/dev/hda1
```

(6). 测试硬盘的读写速度

```bash
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
dd if=/root/1Gb.file bs=64k | dd of=/dev/null
```

(7). 确定硬盘的最佳块大小（给`bs`设置不同大小的数值来测试读写速度）
	
```bash
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
dd if=/dev/zero bs=2048 count=500000 of=/root/1Gb.file
dd if=/dev/zero bs=4096 count=250000 of=/root/1Gb.file
dd if=/dev/zero bs=8192 count=125000 of=/root/1Gb.file
```

(8). 将 ASCII 文本文件转化为 EBCDIC

```bash
dd if=text.ascii of=text.ebcdic conv=ebcdic
```

(9). 将变长记录的 ASCII 文件 /etc/passwd 转换为一个固定长度为 132 字节的 EBCDIC 纪录

```bash
dd if=/etc/passwd cbs=132 conv=ebcdic of=/tmp/passwd.ebcdic
```

(10). 将 dd 命令作为一个过滤器使用（将目录列表显示为大写字母）

```bash
ls -l | dd  conv=ucase
```

### 参考

1. dd命令参考大全: <http://study.chyangwa.com/IT/AIX/aixcmds2/dd.htm>.

2. dd(1) - Linux man page: <http://linux.die.net/man/1/dd>.

3. Linux 下的dd命令使用详解: <http://www.cnblogs.com/Centaurus/archive/2013/05/25/3099487.html>.
