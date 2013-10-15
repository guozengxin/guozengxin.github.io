---
layout: post
title: "git基本原理（二）引用和打包"
description: ""
category: git
tags: [git, References, 版本控制, Pro Git, Packfiles]
---
{% include JB/setup %}

### 系列文章

[git基本原理（一）目录结构与对象](/git/2013/09/25/git-basic-principle-1/)

### 简介

在[上节]</git/2013/09/25/git-basic-principle-1/>中介绍了 git 中的目录结构和对象，本节我们但要有关 git 基本原理的另外一些内容：引用和打包。本项目的测试项目接着上节介绍。

### git 引用

在上节中，通过低级命令创建了一个完整的提交历史，最后我们可以通过`git log --stat 9ef62b`命令可以查看提交历史，但是我们必须记住`9ef62b`是最后一次提交，因此，我们必须还有一类文件记录这些 SHA-1 值，在 git 中，这类指向最后一次提交的指针称为“引用”（references 或者 refs），这些文件将被保存在`.git/refs`目录下。现在这个目录下还没有文件，其结构如下：

{% highlight bash %}

{% endhighlight %}
