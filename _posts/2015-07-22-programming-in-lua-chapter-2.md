---
layout: post
title: "lua学习笔记(2)--类型和值"
description: "lua学习笔记，第二章, 类型和值"
category: lua
tags: [programming in lua, lua, nil, table]
comments: true
---

## 8种数据类型

Lua是动态类型语言，不需要类型定义，每个值携带了自己的类型。

在lua中，有8种基本类型：`nil`, `boolean`, `number`, `string`, `userdata`, `function`, `thread`和`table`。可以用`type`函数来输入一个类型的值：

{% highlight lua %}
print(type("hello"))  	--> string
print(type(3.0))		--> number
print(type(print))		--> function
print(type(true))		--> boolean
print(type(nil))		--> nil
print(type({}))			--> table
{% endhighlight %}

<!-- more -->

## Nil

nil类型只有一个值：nil，和python中的None类似，表示什么也没有。默认情况下，变量没有经过初始化时，它的值默认为nil。

## Booleans

boolean类型有两个值：true和false，代表布尔值。但是lua中，不是只有boolean类型可以作为条件判断变量的条件，其他任何类型都可用于条件判断。条件判断认为只有boolean值`false`和`nil`判断为假，其他任何值都会判断为真。

## Numbers

number指实数，在lua中没有整数类型。和其它语言类似，可以用整数、小数、指数来表示实数：

{% highlight lua%}
4	0.4		4.0e-3	3e4		5E+20
{% endhighlight %}

## Strings

### 普通字符串
lua是8位字节的，所以lua中的字符串可以存储任何数值字符。也可以存储Unicode字符（UTF-8, UTF-16，等）。

lua的string是不可更改的，只能通过创建一个新的字符串来实现更改：

{% highlight lua %}
a = "one string"
b = string.gsub(a, "one", "another")
print(a)
print(b)
{% endhighlight %}

lua的string是自动分配内存的，你不必担心内存的分配和回收。

可以用`#`操作符来获取string的长度:

{% highlight lua %}
a = 'hello'
print(#a)       --> 5
print(#'abc')   --> 3
{% endhighlight %}

lua中字符串可以有两种限定方式，单引号和双引号，这两种表示方式没有区别：

{% highlight lua %}
a = 'string'
b = "string"
{% endhighlight %}

### Long Strings(多行字符串)

lua中可以用`[[..]]`表示多行字符串：

{% highlight lua %}
page = [[
<html>
<head>
  <title>An HTML Page</title>
</head>
<body>
</body>
</html>
]]
{% endhighlight %}

有时候，我们需要在字符中包含`[[`或者`]]`，那么可以在多行界定符之间加入任何字符：`[==[`和`]==]`。相似的逻辑也可以应用到多行注释中：用`--[=[`和`]=]`作为注释的开始和结束。

## 强制转换

lua会在number和string类型之间自动转换，当一个字符串进行算术操作时，字符串会自动转换成数字：

{% highlight lua %}
print('10' + 1)        --> 11
print('10 + 1')        --> 10 + 1
print('2e-10' * '2')   --> 4e-10
print('a' + 1)         --> ERROR
{% endhighlight %}

在其他需要数字的场合，例如一个函数需要数字作为参数时，字符串也有可能转换成数字。

数字也可以自动转换为字符串，如在连接两个数字时(`..`操作是字符串连接操作)：

{% highlight lua %}
-- 数字和连接符之间必须要有空格，否则会被当作小数点
print(10 .. 20)      --> 1020
{% endhighlight %}

在平时编程过程中，不要依赖于类型的自动转换，毕竟字符串和数字是不同的值，`10 == '10'`是false。

我们可以用`tonumber`和`tostring`这两个函数进行转换：

{% highlight lua %}
a = tonumber('10')
print(type(a))           --> number
b = tostring(10)
print(type(b))           --> string
{% endhighlight %}

## Tables

### 基本概念和构建

在lua中，表是一个很强大的数据结构，它用关联数组实现，关联数组不仅可以用数字作为索引，也可以用其它任何值作为索引。

lua中的表是lua动态分配的对象，程序只是负责控制这个对象的引用，在lua中，我们无需声明table，只能用`{}`来创建：

{% highlight lua %}
a = {}
k = 'x'
a[k] = 10
a[5] = 'this is 5'
print(a['x'])             --> 10
k = 5
print(a[k])               --> this is 5
{% endhighlight %}

每个表都可以拥有不同的类型的索引：

{% highlight lua %}
a = {}
for i = 1, 100 do
	a[i] = i * i
end
print(a[3])           --> 9
a['x'] = 10
print(a['x'])         --> 10
print(a['y'])         --> nil
{% endhighlight %}

注意上面最后一行，和全局变量一样，没有初始化的域会默认为nil。并且也可以将一个域置为nil来删除它。

lua也可以用点号来访问一个成员：

{% highlight lua %}
a = {}
a.x = 10
print(a.x)             --> 10
{% endhighlight %}

这里经常犯错的是把**a.x**和**a[x]**搞混，**a.x**指的是以字符串"x"为索引域a['x']，而**a[x]**指的是以变量x的值为索引的域。

### 数组

为了方便表示数组，只需要将表的下标都设置为数字即可：

{% highlight lua %}
a = {}
for i = 1, 10 do
	a[i] = io.read()
end
{% endhighlight %}

下标可以设为任何数字，但是lua的惯例是以数字1开始。

lua中可以用`#a`来计算数组的长度，前提是数组是从下标1开始，并且数组中所有的元素都不为nil，因为遇到第一个nil会被当作数组的结束：

{% highlight lua %}
for i = 1, #a do
	print(a[i])
end
{% endhighlight %}

## Functions

函数是第一类值：和其他变量相同，它可以存储在变量中，也可以作为参数和返回值。这个特性有很大的灵活性，Lua对函数式编程也有很多的支持，包含很多特性。

Lua可以调用lua或者C实现的函数，Lua所有的标准库都是用C实现的。

## Userdata and Threads

userdata可以将C数据存放在Lua变量中，userdata在Lua中除了赋值和相等比较外没有预定义操作。线程是lua协同操作时的类型。

