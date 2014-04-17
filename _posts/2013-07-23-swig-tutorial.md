---
layout: post
title: "swig入门介绍"
description: "通过swig，可以在tcl, perl, python, java, c#中执行c函数，加快程序运行速度"
category: swig
tags: [C++, Python, swig]
---

### OverView

如果觉得脚本执行语言执行太慢，可以使用[SWIG](http://www.swig.org/)将C函数嵌入到[Tcl](http://www.tcl.tk/), [Perl](http://www.perl.org/), [Python](http://www.python.org/), [Java](http://java.sun.com/)和[C#](http://msdn.microsoft.com/netframework)中执行。

<!-- more -->

本文中的C代码：
{% highlight c++ %}
/* File : example.c */

#include <time.h>
double My_variable = 3.0;

int fact(int n) {
	if (n <= 1) return 1;
	else return n*fact(n-1);
}

int my_mod(int x, int y) {
	return (x%y);
}

char *get_time()
{
	time_t ltime;
	time(&ltime);
	return ctime(&ltime);
}
{% endhighlight %}

### 接口文件

要将上述c代码放到你喜欢的脚本语言中执行，你需要为它写一个接口文件。上述c函数的接口文件可以这样写：要将上述c代码放到你喜欢的脚本语言中执行，你需要为它写一个接口文件。上述c函数的接口文件可以这样写：要将上述c代码放到你喜欢的脚本语言中执行，你需要为它写一个接口文件。上述c函数的接口文件可以这样写：

{% highlight python %}
/* example.i */
%module example
%{
/* Put header files here or function declarations like below */
extern double My_variable;
extern int fact(int n);
extern int my_mod(int x, int y);
extern char *get_time();
%}

extern double My_variable;
extern int fact(int n);
extern int my_mod(int x, int y);
extern char *get_time();
{% endhighlight %}

### 编译为python语言

用SWIG把C语言编译为python模块很简单，只需要执行以下步骤：

{% highlight bash %}
$ swig -python example.i
$ gcc -fPIC -I/usr/include/python2.6 -c example_wrap.c -o example_wrap.o
$ gcc -fPIC -c example.c -o example.o
$ gcc -shared example.o example_wrap.o -o _example.so
{% endhighlight %}

然后可以用以下方法调用上述模块：
{% highlight python %}
>>> import example
>>> example.fact(5)
120
>>> example.my_mod(7,3)
1
>>> example.get_time()
'Tue Jul 23 23:10:17 2013\n'
{% endhighlight %}

### 参考

1. swig tutorial: [http://www.swig.org/tutorial.html](http://www.swig.org/tutorial.html)
