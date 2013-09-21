---
layout: post
title: "python函数之 filter map reduce"
description: "python中有很多设计巧妙且实用的函数，能极大的减少开发的工作量，使代码看起来相当的简洁。本文介绍其中的几个：filter, map 和 reduce"
category: python
tags: [python, filter, map, reduce]
---
{% include JB/setup %}

### 简介

[**filter()**](http://docs.python.org/2/library/functions.html#filter), [**map()**](http://docs.python.org/2/library/functions.html#map) 和 [**reduce()**](http://docs.python.org/2/library/functions.html#reduce)函数是python语言中几个操作列表的内置函数，用简洁的表达实现复杂功能。

<!-- more -->

### filter

#### 函数说明

	filter(function, iterable)

[**filter()**](http://docs.python.org/2/library/functions.html#filter)对*iterable*的元素逐一调用*function*，留下function返回true的元素。*iterable*可以是一个sequence，或者是任何一个支持迭代的容器。如果*iterable*的类型是string或者tuple, 结果仍然不变，如果是其他类型的，结果将是一个list。

如果*function*是None, 即会使用identity函数, 即会过滤掉*iterable*中为假的元素。

[**filter()**](http://docs.python.org/2/library/functions.html#filter)和python中的列表推导有着相似的功能：如果*function*不为None，则*filter(function, iterable)*和*[item for item in iterable if function(item)]*相同；如果*function*为None，则和*[item for item in iterable if item]*相同。

#### 示例1

{% highlight python %}
>>> a = [1, 2, 3, 4, 5, 6, 7]
>>> b = filter(lambda x:x>5, a)
>>> print b
[6, 7]
{% endhighlight %}

上面示例用到**lambda**表达式，作用是返回*x>5*的值，filter将列表*a*中大于5的值保留下来了。

#### 示例2

{% highlight python %}
>>> a = [0, False, 1, 2, 3, 4, 5, 6, 7] 
>>> b = filter(None, a)
>>> print b
[1, 2, 3, 4, 5, 6, 7]
{% endhighlight %}

这个示例演示了*function*为None的情况，列表a中所有为假的元素都被过滤掉了。

### map

#### 函数说明

	map(function, iterable, ...)

[**map()**](http://docs.python.org/2/library/functions.html#map)对*iterable*中的元素逐一应用*function*，返回一个列表类型的结果。如果传入了多个*iterable*，则*function*必须平等的传入多个参数（*iterable*的数目对应参数的数目），如果某一个*iterable*的数目比其他的少，会用*None*补齐。

如果*function*为None，则会使用*identity*函数代替。如果参数中包含多个*iterable*，则会返回一个类型为tuple的list（和转置运算比较类似）。

#### 示例1

{% highlight python %}
>>> a = [0, 1, 2, 3, 4, 5]    
>>> b = map(lambda x:x+3, a) 
>>> print b
[3, 4, 5, 6, 7, 8]
{% endhighlight %}

将列表*a*中所有的元素都加3.

#### 示例2

{% highlight python %}
>>> a = [1, 2, 3]
>>> b = [4, 5, 6] 
>>> c = map(lambda x,y:x+y, a, b)
>>> print c
[5, 7, 9]
{% endhighlight %}

这个例子中传入了2个列表，将对应列相加。

#### 示例3

{% highlight python %}
>>> a = [1, 2, 3]                
>>> b = [4, 5, 6]                
>>> c = map(None, a, b)                
>>> print c
[(1, 4), (2, 5), (3, 6)]
{% endhighlight %}

*function*参数为*None*的情况。

### reduce

#### 函数说明

	reduce(function, iterable[, initializer])

*function*是一个接受两个参数的函数，[**reduce()**](http://docs.python.org/2/library/functions.html#reduce)函数将*iterable*中的值逐个应用*function*：先把前两个应用*function*，再把返回值和第三个值应用*function*, 依此类推，最后返回的值作为结果。例如*reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])*产生的效果为*((((1+2)+3)+4)+5)*

如果可选参数*initializer*被使用，那个这个值作为初始值，[**reduce()**](http://docs.python.org/2/library/functions.html#reduce)会先将*initializer*和*iterable*中的第一个元素作为参数应用*function*。

如果指明*initializer*且*iterable*为空，则[**reduce()**](http://docs.python.org/2/library/functions.html#reduce)返回*initializer*；如果*initializer*未指明，且*iterable*只有一个元素，则[**reduce()**](http://docs.python.org/2/library/functions.html#reduce)返回第一个元素。如果*initializer*未指明，且*iterable*为空，将会发生异常。

#### 示例1

{% highlight python %}
>>> a = [1, 2, 3, 4, 5]
>>> b = reduce(lambda x,y:x*y, a) 
>>> print b
120
{% endhighlight %}

求列表中所有元素的乘积。

#### 示例2

{% highlight python %}
>>> a = [1, 2, 3, 4, 5]          
>>> b = reduce(lambda x,y:x*y, a, 5)
>>> print b
600
{% endhighlight %}

当指定初始值时，变为(5 * 1 * 2 * 3 * 4 * 5).

### 参考

1. python内置函数：<http://docs.python.org/2.7/library/functions.html>.
2. python几个内置函数之-filter，map,reduce：<http://blog.csdn.net/shark0001/article/details/1363564>.
