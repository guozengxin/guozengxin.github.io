---
layout: post
title: "git基本原理（二）引用和打包"
description: ""
category: git
tags: [git, References, 版本控制, Pro Git, Packfiles]
---
{% include JB/setup %}

### 系列文章

[git基本原理（一）目录结构与对象](/git/2013/09/25/git-basic-principle-1/) <br/>
[git基本原理（二）引用和打包](/git/2013/10/13/git-basic-principle-2/)

### 简介

在[上节](/git/2013/09/25/git-basic-principle-1/)中介绍了 git 中的目录结构和对象，本节我们但要有关 git 基本原理的另外一些内容：引用和打包。本项目的测试项目接着上节介绍。

<!-- more -->

### git 引用

在上节中，通过低级命令创建了一个完整的提交历史，最后我们可以通过`git log --stat 9ef62b`命令可以查看提交历史，但是我们必须记住`9ef62b`是最后一次提交，因此，我们必须还有一类文件记录这些 SHA-1 值，在 git 中，这类指向最后一次提交的指针称为“引用”（references 或者 refs），这些文件将被保存在`.git/refs`目录下。现在这个目录下还没有文件，其结构如下：

{% highlight bash %}
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/tags
$ find .git/refs -type f
$
{% endhighlight %}

该目录下有两个目录：`heads`和`tags`。我们可以新建一个`master`文件记住上一次提交：

{% highlight bash %}
echo "9ef62b02affa9cf8f7520f490de7871f64b7112f" > .git/refs/heads/master
$ git log master
commit 9ef62b02affa9cf8f7520f490de7871f64b7112f
Author: guozengxin <guozengxin@outlook.com>
Date:   Fri Oct 11 17:25:35 2013 +0800

    third commit

commit ea5c94680e84c557dd25588e89e02c9b3827a919
Author: guozengxin <guozengxin@outlook.com>
Date:   Fri Oct 11 17:25:06 2013 +0800

    second commit

commit 74103078726a6b12ff4e1cc6fb1906e512bdd4ea
Author: guozengxin <guozengxin@outlook.com>
Date:   Fri Oct 11 17:23:58 2013 +0800

    first commit
{% endhighlight %}

如上，在`git log`命令中就可以使用创建的引用来替代 SHA-1 值。git 中有相应的命令来更新这些引用文件 `update-ref`：

{% highlight bash %}
$ git update-ref refs/heads/master 9ef62b02affa9cf8f7520f490de7871f64b7112f
{% endhighlight %}

这条命令会创建一个引用 master。在 git 中，实质上一个分支就是指向一个工作版本的提交的指针，可以用`update-ref`命令创建一个指向第二次提交的引用，来创建一个`test`分支：

{% highlight bash %}
$ git update-ref refs/heads/test ea5c94
$ git log --pretty=oneline test
ea5c94680e84c557dd25588e89e02c9b3827a919 second commit
74103078726a6b12ff4e1cc6fb1906e512bdd4ea first commit
{% endhighlight %}

当执行`git branch [branch]`的时候，基本就是在执行`update-ref`指令，把最后一次提交的 SHA-1 值，添加到创建的引用中。

#### HEAD 标记

我们在用`update-ref`命令的时候，还是需要显式的指定最后提交的 SHA-1 值，但是当执行`git branch`命令的时候，git 是怎么知道这个值的呢？事实上，还有一个 HEAD 文件来保存指向当前分支的引用标识符。

{% highlight bash %}
$ cat .git/HEAD
ref: refs/heads/master
{% endhighlight %}

可以看到，HEAD 文件中保存的并不是指针，而是指向另一个引用。如果执行`git checkout test`，HEAD 文件就会被更新：

{% highlight bash %}
$ git checkout test
Switched to branch 'test'
$ cat .git/HEAD
ref: refs/heads/test
{% endhighlight %}

在 git 中，有对应的一个低级命令`symbolic-ref`可以设置 HEAD 文件。

{% highlight bash %}
$ git symbolic-ref HEAD
refs/heads/test
$ git symbolic-ref HEAD refs/heads/master
$ git symbolic-ref HEAD
refs/heads/master
{% endhighlight %}

不带参数，读取 HEAD 文件，带参数来设置 HEAD 文件。

在上一节[git基本原理（一）目录结构与对象](/git/2013/09/25/git-basic-principle-1/)中，我们在设置 commit 对象的时候，用命令`git commit-tree ea162a -p 741030`来设置一个 commit 对象，需要`-p`参数来指定父级对象。事实上，当我们执行`git commit`命令的时候，会创建一个 commit 对象，这个对象的父级设置为 HEAD 指向的引用的 SHA-1 值。

#### Tags

git 中的 tag 是怎么实现的呢？git 中有一个 tag 对象，它非常像 commit 对象，包含一个标签，一组数据，一个消息和一个指针，最主要的区别是 tag 对象指向一个 commit 而不是一个 tree。这是对一个分支的引用，但是永远不会变化。

在 git 中，有两种类型的 tag，含附注的（annotated）和轻量级的（lightweight）。lightweight 标签就像是一个不会变化的引用，事实上就是一个指向特定提交对象的引用。而 annotated 标签实际是存储在仓库中的一个独立对象，包含较验信息，标签名字，说明等等附加信息。我们可以用`update-ref`来简单创建一个 lightweight tag：

{% highlight bash %}
$ git update-ref refs/tags/v1.0 9ef62b02affa9cf8f7520f490de7871f64b7112f
$ cat .git/refs/tags/v1.0 
9ef62b02affa9cf8f7520f490de7871f64b7112f
{% endhighlight %}

这就是 lightweight tag 的全部内容。

annotated tag 要更复杂，git 需要创建一个 tag 对象，然后写入一个指向它而不是直接指向 commit 对象的引用。用如下方法创建 annotated tag：

{% highlight bash %}
$ git tag -a v1.1 ea5c94680e84c557dd25588e89e02c9b3827a919 -m "test tag"
$ cat .git/refs/tags/v1.1 
805b4b0970dc79a18c1fcf9bb9f31c2dbd000aef
{% endhighlight %}

`805b4b`是所创建的 tag 对象的 SHA-1 值。可以用`git cat-file`来查看这个对象：

{% highlight bash %}
$ git cat-file -p 805b4b
object ea5c94680e84c557dd25588e89e02c9b3827a919
type commit
tag v1.1
tagger guozengxin <guozengxin@outlook.com> Wed Oct 16 23:29:56 2013 +0800

test tag
{% endhighlight %}

对象的内容展示了指向的对象的 SHA-1 值，指向对象的类型，tag 名称，标签作者信息和时间，以及注释。值得注意的是指向对象的类型不一定必须是 commit 对象，也可以指向 blob 对象和 tree 对象，但是这两种平时很难用到。

#### Remotes

现在介绍 git 的第四种引用：远程引用（remote reference）。如果在仓库中添加了一个 remote 并把代码推送过去。git 会把最后一次推送的分支保存到 `refs/remotes`目录下。

{% highlight bash %}
$ git remote add origin git@github:guozengxin/git-test.git
$ git push origin master
Enter passphrase for key '/home/guozengxin/.ssh/github_id_rsa':
Counting objects: 9, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (9/9), 749 bytes, done.
Total 9 (delta 0), reused 0 (delta 0)
To git@github:guozengxin/git-test.git
 * [new branch]      master -> master
{% endhighlight %}

然后查看`refs/remotes/origin/master`这个文件：

{% highlight bash %}
$ cat .git/refs/remotes/origin/master 
9ef62b02affa9cf8f7520f490de7871f64b7112f
{% endhighlight %}

发现它保存了最后一次提交对象的指针。

remote 引用和分支的主要区别是，remote 引用是不能被 checkout 的。git 只是为最后推送做一个标记。

### git 打包

到目前为止，git 仓库中一共有11个对象：4 个 blog, 3 个 tree，3 个 commit 以及 1 个tag：

{% highlight bash %}
$ find .git/objects/ -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
.git/objects/81/7c8395f48d6de9de2aaa20bb2d7a3ea96b642c # test.txt v1
.git/objects/2c/6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d # test.txt v2
.git/objects/80/865964295ae2f11d27383e5f9c0b58a8ef21da # tree 1 
.git/objects/80/5b4b0970dc79a18c1fcf9bb9f31c2dbd000aef # tag v1.1
.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
.git/objects/ea/162a0d432d2352dec117377c718d10791918f5 # tree 2
.git/objects/ea/5c94680e84c557dd25588e89e02c9b3827a919 # second commit
.git/objects/53/1c19905cbc3be29d5ed579d3307faf5149c1f6 # tree 3
.git/objects/74/103078726a6b12ff4e1cc6fb1906e512bdd4ea # first commit
.git/objects/9e/f62b02affa9cf8f7520f490de7871f64b7112f # third commit
{% endhighlight %}

接下来，我们增加一个文件来增加文件的总大小。

{% highlight bash %}
$ wget http://www.sina.com.cn -O sina.html
--2013-11-24 01:21:39--  http://www.sina.com.cn/
Resolving www.sina.com.cn... 202.108.33.60
Connecting to www.sina.com.cn|202.108.33.60|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 706355 (690K) [text/html]
Saving to: "sina.html"

100%[===========================================================================================>] 706,355     --.-K/s   in 0.01s   

2013-11-24 01:21:40 (59.3 MB/s) - "sina.html" saved [706355/706355]
$ git add sina.html
$ git commit -m "add sina.html"
[master df27665] add sina.html
 2 files changed, 9058 insertions(+), 1 deletions(-)
 delete mode 100644 bak/test.txt
 create mode 100644 sina.html
{% endhighlight %}

查看 sina.html 文件的 blob 对象的 SHA-1 值：

{% highlight bash %}
$ git cat-file -p master^{tree}
100644 blob fa49b077972391ad58037050f2a75f74e3671e92    new.txt
100644 blob d103b2f0b1d3170693d04e672b069032fb4108f7    sina.html
100644 blob 2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d    test.txt
{% endhighlight %}

查看这个 blog 对象的大小：

{% highlight bash %}
$ git cat-file -s d103b2f0b1d3170693d04e672b069032fb4108f7
706355
{% endhighlight %}

这个对象的内容有706K。接下来，我们修改一下 sina.html 文件的内容并提交，看看会发生什么：

{% highlight bash %}
$ echo test content >> sina.html 
$ git commit -am "add content to sina.html"
[master b30c784] add content to sina.html
 1 files changed, 1 insertions(+), 1 deletions(-)
$ git cat-file -p master^{tree}
100644 blob fa49b077972391ad58037050f2a75f74e3671e92    new.txt
100644 blob 76c5d4dbcf2930d9e9a1cc9f323544fcf88dea8a    sina.html
100644 blob 2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d    test.txt
{% endhighlight %}

可以看到，对文件内容做修改后，git 用一个完全不同的对象来保存 sina.html。到此为止 git 保存了两个 706K 大小的对象，但是之间的差异非常小。那可不可以保存文件之间的差异来节省空间呢？

事实上，git 默认保存对象的模式叫松散对象(loose object)格式，当仓库中有太多松散对象，或者手工调用`git gc`命令，或推送至远程服务器时，git 都会将这些对象打包至一个叫 packfile 的二进制文件以节省空间。手动执行 `git gc`：

{% highlight bash %}
$ git gc
Counting objects: 16, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (12/12), done.
Writing objects: 100% (16/16), done.
Total 16 (delta 1), reused 0 (delta 0)
$ find .git/objects/ -type f         
.git/objects/pack/pack-dfad34d64e5d04e7a69e91131c48f7df784d514a.pack
.git/objects/pack/pack-dfad34d64e5d04e7a69e91131c48f7df784d514a.idx
.git/objects/info/packs
{% endhighlight %}

可以看到，之前 git 创建的对象都不存在了，但是多了另外三个文件。`packs`文件保存了 packfile 的文件名，`.idx` 文件保存了对象的 offset 信息，方便快速定位到任意一个对象。`.pack`对象保存了所有被压缩的对象。

{% highlight bash %}
$ ls -sh .git/objects/pack/pack-dfad34d64e5d04e7a69e91131c48f7df784d514a.pack
140K .git/objects/pack/pack-dfad34d64e5d04e7a69e91131c48f7df784d514a.pack
{% endhighlight %}

新生成的 packfile 文件只有 140K 的大小，而刚才的 sina.html 文件就有 706K，通过打包操作大大减少了磁盘占用。可以`git verify-pack -v`操作来观察打包文件的结构：

{% highlight bash %}
$ git verify-pack -v .git/objects/pack/pack-dfad34d64e5d04e7a69e91131c48f7df784d514a.idx 
2c6f5b15d2c42d7e2068aa2aeddef1c6ea025b4d blob   18 28 142118
531c19905cbc3be29d5ed579d3307faf5149c1f6 tree   101 104 142297
6e26ba5798cf7217be76518caddd17dd82f18241 tree   108 109 142146
74103078726a6b12ff4e1cc6fb1906e512bdd4ea commit 183 121 628
76c5d4dbcf2930d9e9a1cc9f323544fcf88dea8a blob   706368 141116 1002
805b4b0970dc79a18c1fcf9bb9f31c2dbd000aef tag    139 126 749
80865964295ae2f11d27383e5f9c0b58a8ef21da tree   36 47 142401
9cb0777e90ea2b17462298c243421d26b6d2da82 tree   108 109 875
9ef62b02affa9cf8f7520f490de7871f64b7112f commit 231 150 327
b30c7845975a9e0067924aced745c49afe9ec6f2 commit 243 161 12
d103b2f0b1d3170693d04e672b069032fb4108f7 blob   29 42 142255 1 76c5d4dbcf2930d9e9a1cc9f323544fcf88dea8a
d670460b4b4aece5915caf5c68d12f560a9fe3e4 blob   13 22 142448
df276650e5396cb15aa4768bc0edf25f8c592970 commit 232 154 173
ea162a0d432d2352dec117377c718d10791918f5 tree   71 76 142470
ea5c94680e84c557dd25588e89e02c9b3827a919 commit 232 151 477
fa49b077972391ad58037050f2a75f74e3671e92 blob   9 18 984
non delta: 15 objects
chain length = 1: 1 object
.git/objects/pack/pack-dfad34d64e5d04e7a69e91131c48f7df784d514a.pack: ok
{% endhighlight %}

事实上，git 打包对象时，会查找命名及尺寸都比较相近的文件，并只保存文件不同版本之间的差异内容。`d103b2`这个 blob 是 sina.html 文件的第一个版本，这个对象引用了`76c5d4`这个 blob 对象，即文件的第二个版本。`76c5d4`占用了 706368 字节，但是`d103b2`只占用了 29 字节，只保存了文件的差异。将较新的文件保存完整版本是因为大部分情况下我们都是需要快速访问最新的版本。

### 参考

1. Pro Git（中文版）: <http://git.oschina.net/progit/>
