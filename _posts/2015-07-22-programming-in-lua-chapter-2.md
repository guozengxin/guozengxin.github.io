---
layout: post
title: "lua学习笔记(2)--类型和值"
description: "lua学习笔记，第二章, 类型和值"
category: lua
tags: [programming in lua, lua, nil]
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

### Nil

nil类型只有一个值：nil，和python中的None类似，表示什么也没有。默认情况下，变量没有经过初始化时，它的值默认为nil。

### Boolean

boolean类型有两个值：true和false，代表布尔值。但是lua中，不是只有boolean类型可以作为条件判断变量的条件，其他任何类型都可用于条件判断。条件判断认为只有boolean值`false`和`nil`判断为假，其他任何值都会判断为真。

### Number

number指实数，在lua中没有整数类型。和其它语言类似，可以用整数、小数、指数来表示实数：

{% highlight lua%}
4	0.4		4.0e-3	3e4		5E+20
{% endhighlight %}

### String


