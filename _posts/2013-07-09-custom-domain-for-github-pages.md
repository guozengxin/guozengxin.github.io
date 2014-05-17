---
layout: post
title: "为github pages设置个人域名"
description: "利用jekyll及github可以快速建立个人站点。如果不想用github默认的username.github.io这个域名的话，可以增加自己的域名。"
category: blog
tags: [个人域名, github]
comments: true
---

1. 利用jekyll及github可以快速建立个人站点。如果不想用github默认的username.github.io这个域名的话，可以增加自己的域名。
2. 本篇是用jekyll创建的第一篇文章，作为简单练习

<!-- more -->

### 在个人repo中设置域名

如果你想把你的个人域名example.com应用到username.github.io中，可以用下面的方法：在pages的根目录创建一个名为CNAME的文件，将域名写入这个文件：

	example.com

1. 这个文件需要在master分支创建。
2. 如果你在project pages工作，那么需要在gh-pages分支创建这个文件

### 设置DNS

接下来，就可以设置DNS地址了。这个过程可能需要一整天，要耐心等待。

根据使用的DNS地址，有两种方法进行设置：

#### 使用顶级域名

如果你使用的是顶级域名，你可以使用一个记录指向204.232.175.78

	$ dig example.com +nostats +nocomments +nocmd
	;example.com.                    IN      A
	example.com.             3259    IN      A       204.232.175.78

#### 使用二级域名

如果使用二级域名，则DNS要新建一条CNAME记录，指向username.github.com（请将username换成你的用户名）
你好
