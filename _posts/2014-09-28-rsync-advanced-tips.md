---
layout: post
title: "linux下rsync命令的一些高级用法"
description: "总结rsync命令的一些高级用法，和find命令结合使用，同步后删除源文件"
category: shell
tags: [rsync, find, remove-source-files]
comments: true
---

rsync命令是一款功能强大的同步工具，可以完成文件的完整备份和增量备份；在设置rsync用户之后可以实现在不用建立机器之间的信任关系下的进行无密码备份；同时它在多种操作机器上都可以使用。

### 1. 无需输入密码同步文件（rsync daemon的配置文件）

#### 配置文件说明

rsync命令可以通过配置文件来实现不用输入密码来同步数据，默认的配置文件在位置是`/etc/rsyncd.conf`。下面是一个示例：

	address = 10.12.0.1
	use chroot = no
	read only = no
	pid file = /var/run/rsyncd.pid
	log file = /var/log/rsync.log

	[root]
		uid = root
		gid = root
		path = /path
		hosts allow = 10.0.0.0/8 192.168.0.0/16

- `address`指定服务器IP地址
- `pid file`该选项告诉daemon把它的进程ID写入指定文件。如果文件已经存在，daemon会中止运行，而不是覆盖原文件。
- `use chroot`指定是否使用chroot，值为yes或no。一般配置为no。
- `read only`如果配成yes，表示只读，即不让客户端上传文件到服务器上。
- `log file`表示日志文件地址。

"root"下面的内存表示创建了一个rsync模块，模块名是root

- `uid`, `gid`表示linux帐户名和用户组，表示传送文件时，用哪个用户来执行。使用非root帐户时会有权限的限制。
- `path`指定允许同步的文件的目录。
- `hosts allow`限定的同步的IP地址网段。

<!-- more -->

#### 使用rsync模块名同步

在客户端机器，可以用以下的命令来同步数据：

`rsync -avP 10.12.0.1::root/path-from /path-to`

在这个命令中，无需输入密码就可以完成同步，`root`即为远程机器设置的模块名，需要用远程机器的设置的`path`进行替代，所以远程机器需要同步的目录就是`/path/path-from`

`-a`选项表示是arcive模式，表示以递归方式传输文件，并保持所有文件属性;`-v`选项启用详细消息；`-P`为断点续传(`--partial`)和显示传进度(`--progress`)两个命令的综合。

### 2. rsync配合find命令同步文件

`find`命令是linux下一个十分强大的文件查找命令，接下来将演示如何进行：

下面的find命令在当前目录下查找3天以前的所有文件：

```sh
find . -mtime +3 -type f > files.list
```

下面的rsync命令在读取需要同步的文件列表，同步到远程机器：

```sh
rsync -vP --remove-source-files --files-from=./files.list . 10.134.0.1::root/path-to
```

如果觉得写文件太麻烦，可以把两个命令合起来：

```sh
# cmd1
rsync -vP --remove-source-files --files-from=`find . -mtime +3 -type f` . 10.134.0.1::root/path-to
```

上面"cmd1"的写法在文件数特别多的时候会出错: "Argument list too long" 。所以有另外一种写法：

```sh
# cmd2
find . -mtime +3 -type f -print0 | rsync -0vP --files-from=- . 10.134.12.234::root/search/guozengxin/data/spiderPic/
```

这个命令中`-print0`和`rsync -0`的意思是find命令的输出和rsync的输入以NULL分割，预防文件名中包含奇怪的字符造成错误。`--files-from=-`表示rsync从标准输入读取文件列表。这样的命令组合不会有上一种方式的错误。

### 3. 扩展链接

1. find命令参数详解：<http://www.cnblogs.com/peida/archive/2012/11/16/2773289.html>.
2. rsync同步的艺术: <http://roclinux.cn/?p=2643>.
