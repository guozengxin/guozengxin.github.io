---
layout: post
title: "swig输入文件简介"
description: "使用swig将C或C++语言编译为其他类型语言的时候，一般需要一个声明文件来定义接口。"
category: swig
tags: [C++, swig, Python]
---
{% include JB/setup %}

### Overview

使用swig将C或C++语言编译为其他类型语言的时候，一般需要一个声明文件来定义接口。这个文件用.i或者.swg结尾，通常为具有以下形式：

{% highlight c++ %}
%module mymodule 
%{
#include "myheader.h"
%}
// Now list ANSI C/C++ declarations
int foo;
int bar(int x);
...
{% endhighlight %}

<!-- more -->

可以用以下方式运行swig，来使用这个文件：

{% highlight bash %}
swig [ options ] filename
{% endhighlight %}

例如要编译为python语言，可以：

{% highlight bash %}
swig -python xxx.i
{% endhighlight %}

### module

每个swig调用需要一个模块名，模块名用于在生成的语言中调用。根据不同的语言有不同的调用形式，如python中的import。模块名可以用以下两种方式指定。

1. **在接口文件中指定**


{% highlight c++ %}
%module(option1="value1",option2="value2",...) modulename
{% endhighlight %}

或者不指定选项：

{% highlight c++ %}
%module mymodule
{% endhighlight %}

2. **用命令行参数指定**

{% highlight bash %}
swig -python xxx.i -module modulename
{% endhighlight %} 

### 注释

接口文件中的可以加入任何C/C++方式的注释

### include


