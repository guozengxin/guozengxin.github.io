---
layout: post
title: "python科学计算相关的包安装"
description: "python科学计算相关的包很强大，如numpy和scipy等，本文介绍其安装方法。"
category: python
tags: [安装, install, numpy, scipy, lapack, blas, matplotlib, ipython, sympy]
comments: true
---

### 简介

python拥有着众多的开源包，其中用于科学计算的包也是非常丰富的，如numpy, scipy, sympy等。同时python是一个跨平台的编程语言，脚本的编写使工作更加通用和便捷，拥有上手快、实现方便等优点。所以用python做科学计算是一个不错的选择。

本文介绍一下相关包的安装。（以下介绍的是在linux系统上的安装方法，windows平台上的安装请参考[用Python做科学计算][python-scipy]）

### 相关包

- [numpy][]: numpy是一个强大的数值计算工具库，用于存储和处理数据、矩阵、矢量等类型的数据，比python自身列表等结构的处理要高效的多。
- [scipy][]: scipy是一个算法库和工具包, 包含的模块有最优化、线性代数、积分、特殊函数等一些科学与工程中常用的计算。
- [lapack][]: lapack是Linear Algebra PACKage的缩写，是一以Fortran编程语言写的，用于数值计算的函式集。scipy包依赖lapack。
- [blas][]: BLAS是Basic Linear Algebra Subprograms缩写，表示基础线性代数程序集。是一个API标准，用以规范发布基础线性代数操作的数值库。scipy包依赖blas。
- [matplotlib][]: matplotlib是一个python的绘图库，它提供了一整套和matlab相似的命令API，十分适合交互式地进行制图。
- [ipython][]: ipython是python语言的交互式工具，和matlab的交互式界面类似。
- [sympy][]: SymPy是一个符号计算的Python库。它的目标是成为一个全功能的计算机代数系统，同时保持代码简洁、易于理解和扩展。

<!-- more -->

### pip

[pip][]是python的包管理工具，集成了多种安装方法，参考[https://pypi.python.org/pypi/pip][pip]. pip可以用以下几种方法进行安装:

在线安装: 下载文件[get-pip.py](https://bootstrap.pypa.io/get-pip.py)，然后执行

{% highlight python %}
python get-pip.py
{% endhighlight %}

Ubuntu或者Debian:

{% highlight bash %}
sudo apt-get install python-pip
{% endhighlight %}

Fedora:

{% highlight bash %}
yum install python-pip
{% endhighlight %}

### 安装python包

#### numpy

numpy直接用pip安装即可：

{% highlight python %}
pip install numpy
{% endhighlight %}

#### lapack, blas

lapack, blas这两个包不是python的包，是scipy依赖的包，可以直接用yum安装：

{% highlight bash %}
yum install blas-devel lapack-devel
{% endhighlight %}

或者apt-get安装：

{% highlight bash %}
apt-get install libblas-dev liblapack-dev
{% endhighlight %}

#### matplotlib, sympy, scipy

matplotlib, sympy, scipy可以直接用pip安装, matplotlib的使用需要显卡支持，如果是在一台没有显卡的linux服务器上操作的话，这个包是无法使用的(可以正常安装)。

{% highlight python %}
pip install matplotlib
pip install sympy
pip install scipy
{% endhighlight %}

#### ipython

ipython可以直接用pip安装:

{% highlight python %}
pip install ipython
{% endhighlight %}

但是，现在最新版本的ipython2.x版本是不支持python2.6及以下的版本的。如果python是2.6或者以下版本，可以用以下的方式安装低版本的ipython:

{% highlight python %}
pip install -Iv "https://pypi.python.org/packages/source/i/ipython/ipython-1.2.1.tar.gz#md5=4ffb36697f7ca8cb4a2de0f5b30bc89c"
{% endhighlight %}

[python-scipy]: http://sebug.net/paper/books/scipydoc/ "用Python做科学计算"
[numpy]: http://www.numpy.org/ "numpy"
[scipy]: http://scipy.org/ "scipy"
[lapack]: http://www.netlib.org/lapack/ "lapack"
[blas]: http://www.netlib.org/blas/ "blas"
[matplotlib]: http://matplotlib.org/ "matplotlib"
[ipython]: http://ipython.org/ "ipython"
[sympy]: http://sympy.org/en/index.html "sympy"
[pip]: https://pypi.python.org/pypi/pip "pip"
