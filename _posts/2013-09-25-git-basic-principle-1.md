---
layout: post
title: "git基本原理（一）目录结构与对象"
description: "文章介绍git的底层原理和具体的实现方式，对这些的理解对于了解git的应用和发现它的强大是非常有用的。本文将介绍git的底层命令和高层命令的分类，以及git对象的基本知识。"
category: git
tags: [git, git对象, 版本控制, Pro Git]
---

### 系列文章

[git基本原理（一）目录结构与对象](/git/2013/09/25/git-basic-principle-1/) <br/>
[git基本原理（二）引用和打包](/git/2013/10/13/git-basic-principle-2/)

### 简介

使用了一阵 git 版本控制，了解了最基本的几个命令，对于分支、合并及分支管理等还没有接触。最近决定对 git 进行多一些的了解，以至于能够对我的代码进行更好的管理，于是想从git的底层原理和实现方式开始，进行更深层的学习。

git 的本质是基于内容寻址（content-addressable）的一套文件系统，在此基础上提供了一些高级的命令和VCS用户界面。

<!-- more -->

### 底层命令和高层命令

对于平时用到的命令`clone`, `branch`, `commit`等，这些属于git的高层命令（porcelain），但是由于git最开始的设计更像是一套工具集，而不是一个完善的版本控制系统，所以它有很多底层命令（plumbing），这些命令类似于 Linux/Unix 风格，方便从 shell 脚本调用。

### git 的目录结构

当从一个空目录创建git项目时，需要执行`git init`命令，执行完毕后会产生一个名为`.git`的隐藏目录，其结构如下：

	$ ls -F1 .git/
	branches/
	config
	description
	HEAD
	hooks/
	info/
	objects/
	refs/

几乎 git 所有的信息都保存在这个目录下，这个目录下可能还有其他文件，但是新建的 git 库，就是上面这些文件（不同版本的 git 可能会有所差别）。其中，`description`文件仅供 GitWeb 程序使用，无需关心其内容，`config`文件包含了项目的配置选项，`info`目录保存了一份不希望在`.gitignore`文件中管理的忽略模式的全局可执行文件。`hooks`目录保存了客户端或服务器的钩子脚本。

另外四个文件或者目录比较重要，是 git 的核心部分。`objects`目录存储所有数据内容，`refs`目录存储指向数据（分支）的提交对象指针，`HEAD`文件指向当前分支，`index`文件保存了暂存区域信息（由于没有进行数据操纵，所以还没有`index`文件）。

### git 对象

#### blob 对象

git 本质上是一个根据内容寻址的文件系统，意思就是，git 本身存储的都是 key-value 对。它可以根据输入的 key 进行写入和读取内容，且 key 是根据内容生成的。底层命令`hash-object`可以用来存取 key-value 对，它会将数据保存在`.git`目录并返回数据对应的键值。下面通过一些示例来演示 key-value 的存取。

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

参数`-w`的意思是 write，指示`hash-object`存储内容，不用`-w`参数则仅仅打印键值。`--stdin`表示从标准输入来读取内容，也可以从文件中读取，直接将文件名作为参数即可。命令返回一个长度为 40 的 key，使用了 SHA-1 算法根据内容计算出来的。可以看出，在`.git`目录中已经存储了该内容。

{% highlight bash %}
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
{% endhighlight %}

在目录`.git/objects`下生成了一个文件，用 key 的前两位作为子目录名，余下的 38 位作为文件名。命令`git cat-file`可以读取文件，`-p`参数为 pretty print, 打印内容。

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

现在我们可以通过 key 将文件恢复为历史版本。恢复为第一个版本：

{% highlight bash %}
$ git cat-file -p 817c8395f48d6de9de2aaa20bb2d7a3ea96b642c > test.txt
$ cat test.txt
my test content 1
{% endhighlight %}

恢复为第二个版本：

{% highlight bash %}
$ git cat-file -p 2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d > test.txt
$ cat test.txt
my test content 2
{% endhighlight %}

可以看出，git 是根据文件内容的 SHA-1 值来作为 key 来存储内容的，而和文件名没有关系。这种对象类型称为 blob。可以通过命令`git cat-file -t`来显示对象类型：

{% highlight bash %}
$ git cat-file -t 2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d  
blob
{% endhighlight %}

#### tree 对象

git 以一种类似 UNIX 文件系统的方式存储内容，tree 对象对应着目录，可以存储文件名，blob 对象对应 inodes 或者文件内容。一个 tree 对象包含一条或多条记录，每一条记录指向一个 blob 对象或者子 tree 对象的 SHA-1 指针，并记录了对象的权限、类型和文件名信息。添加了两个文件和一个子目录后的 tree 对象信息：

{% highlight bash %}
$ git cat-file -p master^{tree} 
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 README
040000 tree 543b9bebdc6bd5c4b22136034a95dd097a57d3dd src
100644 blob 2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d test.txt
{% endhighlight %}

`master^{tree}`表示 master 分支上最新提交指向的 tree 对象。注意 src 是一个子 tree 对象。

可以通过暂存区域（index）来创建和写入 tree，必须先写文件暂存从而创建一个 index，然后将 index 中的内容写入到 tree 对象中。创建 index 的命令是 `update-index`命令来创建 index ：

{% highlight bash %}
$ git update-index --add --cacheinfo 100644 d670460b4b4aece5915caf5c68d12f560a9fe3e4 test.txt
{% endhighlight %}

由于`test.txt`不在暂存区域中，加上`--add`参数，`--cacheinfo`参数表示直接将数据插入暂存区域中，并需要指定 \<mode\> \<object\> \<path\>。本例中，模式`100644`表示普通文件（`100755`表示可执行文件，`120000`表示符号链接）。

用`write-tree`命令可以将暂存区域中的内容写入 tree 对象。如果目标 tree 不存在，`write-tree`命令会自动创建一个。

{% highlight bash %}
$ git write-tree
80865964295ae2f11d27383e5f9c0b58a8ef21da
$ git cat-file -p 80865964295ae2f11d27383e5f9c0b58a8ef21da
100644 blob d670460b4b4aece5915caf5c68d12f560a9fe3e4 test.txt
$ git cat-file -t 80865964295ae2f11d27383e5f9c0b58a8ef21da
tree
{% endhighlight %}

可以看出，确实是一个 tree 对象，存储着一个指向 blob 对象的指针。

接下来，将 test.txt 的最新版本和一个新文件 new.txt 暂存并创建一个新 tree 对象：

{% highlight bash %}
$ echo "new file" > new.txt
$ git update-index --add new.txt
$ git update-index test.txt
$ git write-tree
ea162a0d432d2352dec117377c718d10791918f5
$ git cat-file -p ea162a0d432d2352dec117377c718d10791918f5
100644 blob fa49b077972391ad58037050f2a75f74e3671e92 new.txt
100644 blob 2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d test.txt
{% endhighlight %}

这一个 tree 对象包含了两个记录，并且 test.txt 的 SHA-1 值是之前最后添加时的值`2c6f5b`。再做一个操作，将上一个 tree 对象作为子目录加入到这个 tree 对象的记录中，可以用`read-tree`命令：

{% highlight bash %}
$ git read-tree --prefix=bak 80865964295ae2f11d27383e5f9c0b58a8ef21da
$ git write-tree
531c19905cbc3be29d5ed579d3307faf5149c1f6
$ git cat-file -p 531c19905cbc3be29d5ed579d3307faf5149c1f6
040000 tree 80865964295ae2f11d27383e5f9c0b58a8ef21da bak
100644 blob fa49b077972391ad58037050f2a75f74e3671e92 new.txt
100644 blob 2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d test.txt
{% endhighlight %}

`--prefix`参数可以将一个已有的 tree 对象作为子 tree 读到暂存区域中。然后再进行`git write-tree`。此时新建的 tree 对象包含两个 blob 对象和一个子 tree 对象，该子 tree 中的文件是 test.txt 的早先版本。

#### commit 对象

在前面的测试中建立了几个 tree 对象，它记录了存储的文件信息。但是我们必须记住 SHA-1 值才能取出值。因此还必须有一个对象存储这些 key，并且最好能够存储操作人的相关信息。在 git 中，这一对象被称为 commit 对象，我们可以用`commit-tree`命令来创建 commit 对象。

{% highlight bash %}
$ echo "first commit" | git commit-tree 808659
74103078726a6b12ff4e1cc6fb1906e512bdd4ea
$ git cat-file -p 74103078726a6b12ff4e1cc6fb1906e512bdd4ea
tree 80865964295ae2f11d27383e5f9c0b58a8ef21da
author guozengxin <guozengxin@outlook.com> 1381483438 +0800
committer guozengxin <guozengxin@outlook.com> 1381483438 +0800

first commit
{% endhighlight %}

在这里，我们将创建的第一个 tree 对象作为参数（只需要指定前6位即可），创建了一个 commit 对象。用`cat-file`查看 commit 对象的内容，可以看到保存了第一个 tree 对象，以及作者、提交者、时间和注释信息。

然后我们用同样的方法，创建另外两个 commit 对象，每一个都指向之前的 commit 对象。

{% highlight bash %}
$ echo "second commit" | git commit-tree ea162a -p 741030
ea5c94680e84c557dd25588e89e02c9b3827a919
$ echo "third commit" | git commit-tree 531c19 -p ea5c94
9ef62b02affa9cf8f7520f490de7871f64b7112f
{% endhighlight %}

`commit-tree`命令的`-p`参数指定 commit 对象的上一个 commit 对象。现在在项目中就有了 3 个提交对象，每一个对象对应着一次`git commit`，现在运行`git log`就会看到提交历史：

{% highlight bash %}
$ git log --stat 9ef62b
commit 9ef62b02affa9cf8f7520f490de7871f64b7112f
Author: guozengxin <guozengxin@outlook.com>
Date:   Fri Oct 11 17:25:35 2013 +0800

    third commit

 bak/test.txt |    1 +
 1 files changed, 1 insertions(+), 0 deletions(-)

commit ea5c94680e84c557dd25588e89e02c9b3827a919
Author: guozengxin <guozengxin@outlook.com>
Date:   Fri Oct 11 17:25:06 2013 +0800

    second commit

 new.txt  |    1 +
 test.txt |    2 +-
 2 files changed, 2 insertions(+), 1 deletions(-)

commit 74103078726a6b12ff4e1cc6fb1906e512bdd4ea
Author: guozengxin <guozengxin@outlook.com>
Date:   Fri Oct 11 17:23:58 2013 +0800

    first commit

 test.txt |    1 +
 1 files changed, 1 insertions(+), 0 deletions(-)
{% endhighlight %}

现在，我们就通过低级操作创建了一个 git 项目历史，这和`git add`、`git commit`等命令所进行的操作基本相同。创建的所有的 blob 对象、tree 对象和 commit 对象都保存在`.git/objects`目录下：

{% highlight bash %}
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
.git/objects/81/7c8395f48d6de9de2aaa20bb2d7a3ea96b642c # test.txt v1
.git/objects/2c/6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d # test.txt v2
.git/objects/80/865964295ae2f11d27383e5f9c0b58a8ef21da # tree 1 
.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
.git/objects/ea/162a0d432d2352dec117377c718d10791918f5 # tree 2
.git/objects/ea/5c94680e84c557dd25588e89e02c9b3827a919 # second commit
.git/objects/53/1c19905cbc3be29d5ed579d3307faf5149c1f6 # tree 3
.git/objects/74/103078726a6b12ff4e1cc6fb1906e512bdd4ea # first commit
.git/objects/9e/f62b02affa9cf8f7520f490de7871f64b7112f # third commit
{% endhighlight %}

其中，blob 对象基本可以存储任何内容，但 tree 对象和 commit 对象有固定的格式。

### 参考

1. Pro Git（中文版）: <http://git.oschina.net/progit/>
