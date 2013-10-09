---
layout: post
title: "git基本原理（一）目录结构与对象"
description: "文章介绍git的底层原理和具体的实现方式，对这些的理解对于了解git的应用和发现它的强大是非常有用的。本文将介绍git的底层命令和高层命令的分类，以及git对象的基本知识。"
category: git
tags: [git, git对象, 版本控制]
---
{% include JB/setup %}

### 简介

使用了一阵git版本控制，了解了最基本的几个命令，对于分支、合并及分支管理等还没有接触。最近决定对git进行多一些的了解，以至于能够对我的代码进行更好的管理，于是想从git的底层原理和实现方式开始，进行更深层的学习。

git的本质是基于内容寻址(content-addressable)的一套文件系统，在此基础上提供了一些高级的命令和VCS用户界面。

<!-- more -->

### 底层命令和高层命令

对于平时用到的命令`clone`, `branch`, `commit`等，这些属于git的高层命令(porcelain)，但是由于git最开始的设计更像是一套工具集，而不是一个完善的版本控制系统，所以它有很多底层命令(plumbing)，这些命令类似于Linux/Unix风格，方便从shell脚本调用。

### git 的目录结构

当从一个空目录创建git项目时，需要执行`git init`命令，执行完毕后会产生一个名为.git的隐藏目录，其结构如下：

	$ ls -F1 .git/
	branches/
	config
	description
	HEAD
	hooks/
	info/
	objects/
	refs/

几乎git所有的信息都保存在这个目录下，这个目录下可能还有其他文件，但是新建的git库，就是上面这些文件（不同版本的git可能会有所差别）。其中，`description`文件仅供GitWeb程序使用，无需关心其内容，`config`文件包含了项目的配置选项，`info`目录保存了一份不希望在 .gitignore 文件中管理的忽略模式的全局可执行文件。`hooks`目录保存了客户端或服务器的钩子脚本。

另外四个文件或者目录比较重要，是 git 的核心部分。`objects`目录存储所有数据内容，`refs`目录存储指向数据（分支）的提交对象指针，`HEAD`文件指向当前分支，`index`文件保存了暂存区域信息（由于没有进行数据操纵，所以还没有`index`文件）。

### git 对象

git 本质上是一个根据内容寻址的文件系统，意思就是，git 本身存储的都是 key-value 对。它可以根据输入的key进行写入和读取内容，且 key 是根据内容生成的。底层命令`hash-object`可以用来存取 key-value 对，它会将数据保存在`.git`目录并返回数据对应的键值。下面通过一些示例来演示 key-value 的存取。

{% highlight bash %}
$ git init
Initialized empty Git repository in /tmp/test/.git/
$ find .git/objects -type f
$ 
{% endhighlight %}

在一个空目录执行`git init`之后，生成了一个`.git`目录，可以看出，这个目录中没有任何文件。现在，我们用`git hash-object`命令来向其中添加一些内容。

{% highlight bash %}
$ echo "test content" | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
{% endhighlight %}

参数`-w`的意思是 write，指示`hash-object`存储内容，不用`-w`参数则仅仅打印键值。`--stdin`表示从标准输入来读取内容，也可以从文件中读取，直接将文件名作为参数即可。命令返回一个长度为40的 key，使用了 SHA-1 算法根据内容计算出来的。可以看出，在`.git`目录中已经存储了该内容。

{% highlight bash %}
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
{% endhighlight %}

在目录`.git/objects`下生成了一个文件，用 key 的前两位作为子目录名，余下的38位作为文件名。命令`git cat-file`可以读取文件，`-p`参数为 pretty print, 打印内容。

{% highlight bash %}
$git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
{% endhighlight %}

接下来，可以向`.git/objects`下添加更多的内容，下面用读文件的形式添加。

{% highlight bash %}
$ echo "my test content 1" > test.txt
$ git hash-object -w test.txt
817c8395f48d6de9de2aaa20bb2d7a3ea96b642c
$ echo "my test content 2" > test.txt
$ git hash-object -w test.txt
2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d
$ find .git/objects -type f
.git/objects/2c/6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d
.git/objects/81/7c8395f48d6de9de2aaa20bb2d7a3ea96b642c
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
{% endhighlight %}


