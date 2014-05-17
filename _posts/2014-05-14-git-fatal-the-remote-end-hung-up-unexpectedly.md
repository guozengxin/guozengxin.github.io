---
layout: post
title: "git提交出现\"The remote end hung up unexpectedly\"的解决方法"
description: "The solve of \"fatal: The remote end hung up unexpectedly\"."
category: git
tags: [git, github, commit]
comments: true
---

### HTTP错误411

今天在用`http`的方式提交一个git项目到gitlab时，出现了如下错误：

	Counting objects: 56, done.
	Delta compression using up to 8 threads.
	Compressing objects: 100% (36/36), done.
	error: RPC failed; result=22, HTTP code = 411
	fatal: The remote end hung up unexpectedly
	Writing objects: 100% (38/38), 1.57 MiB, done.
	Total 38 (delta 16), reused 0 (delta 0)
	fatal: The remote end hung up unexpectedly 

不解，然后google之，得到提交出现`HTTP code = 411`错误的[原因][1].

原来这个错误是因为是由于上传的包过大 HTTP 的头出错导致的。

#### 解决办法：

设置http.postBuffer，改为一个相对较大的值。

	git config http.postBuffer 524288000

### http错误413

不幸的是，在做了上述修改后，又出现了另外一个错误：

	Counting objects: 56, done.
	Delta compression using up to 8 threads.
	Compressing objects: 100% (36/36), done.
	Writing objects: 100% (38/38), 1.57 MiB, done.
	Total 38 (delta 16), reused 0 (delta 0)
	error: RPC failed; result=22, HTTP code = 413
	fatal: The remote end hung up unexpectedly
	fatal: The remote end hung up unexpectedly

遂进行了另一番查找，找到了问题[所在][2].

在用`http/https`的方式提交本地库时，服务器会限制最大的POST大小。此时需要修改服务器的配置文件`/etc/nginx/sites-available/gitlab`，在`server`段中增加：

	client_max_body_size 128M;

大小可以自定义。然后执行`sudo service nginx restart`重新启动nginx即可。

但是由于服务器不由自己控制，所以..., 只能改为`ssh`的方式来提交本地库。

### SSH方式提交本地库

[生成ssh公钥的方式以及在github上的配置][3]

另外由于我的单机上是需要连接多个git服务器的，因为需要设置多个ssh-key，推荐用如下方式管理ssh-key.

[多个github帐号的SSH key切换][4]

### 参考

1. [如何应对 Git 操作时候的 HTTP 错误][1]
2. [GitLab出现“The remote end hung up unexpectedly”问题解决][2]
3. [Generating SSH Keys][3]
4. [多个github帐号的SSH key切换][4]

[1]: https://gitcafe.com/GitCafe/Help/wiki/%E5%A6%82%E4%BD%95%E5%BA%94%E5%AF%B9-Git-%E6%93%8D%E4%BD%9C%E6%97%B6%E5%80%99%E7%9A%84-HTTP-%E9%94%99%E8%AF%AF "如何应对 Git 操作时候的 HTTP 错误"
[2]: http://www.kaijia.me/2013/10/gitlab-fatal-the-remote-end-hung-up-unexpectedly-solved/ "GitLab出现“The remote end hung up unexpectedly”问题解决"
[3]: https://help.github.com/articles/generating-ssh-keys "Generating SSH Keys"
[4]: http://www.cnblogs.com/mackxu/p/ssh-keygen.html "多个github帐号的SSH key切换"
