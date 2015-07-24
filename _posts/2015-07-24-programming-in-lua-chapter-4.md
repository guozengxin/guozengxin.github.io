---
layout: post
title: "lua学习笔记(4)--语法"
description: "lua学习笔记，第四章，语法"
category: lua
tags: [programming in lua, lua, 语言]
comments: true
---

Lua支持方便的语句，包括赋值、控制、函数调用等，以及多变量赋值和局部变量声明。

## 赋值

Lua支持传统赋值以及非传统的多变量赋值：

{% highlight lua %}
a = "hello" .. "world"
t.n = t.n + 1
a, b = 10, 2 * x
{% endhighlight %}

上面最后一行语句将10赋给变量a，将2*x赋给变量b。

在多变量赋值中，Lua首先计算所有的值，然后才执行赋值，所以可以用来进行交换变量：

{% highlight lua %}
x, y = y, x
a[i], a[j] = a[j], a[i]
{% endhighlight %}

当值的个数和变量的个数不一致时，Lua会自动适应，如果值的个数大于变量的个数，Lua会丢弃掉多余的值，当变量的个数大于值的个数时，Lua会将多余的变量赋为nil。

{% highlight lua %}
a, b, c = 0, 1
print(a, b, c)             --> 0 1 nil
a, b = a+1, b+1, b+2
print(a, b)                --> 1 2
a, b, c = 0
print(a, b, c)             --> 0 nil nil
{% endhighlight %}

多变量赋值通常情况下用于变量交换，以及函数赋值：`a, b = f()`，一般不建议将几个无关的变量的赋值写在同一行。

## 局部变量

Lua支持局部变量，用`local`关键字创建局部变量：

{% highlight lua %}
j = 10           --> global variable
local i = 1      --> local variable
{% endhighlight %}

局部变量的作用域是代码块，代码块是指一个控制结构、函数体或者一个trunk内。

{% highlight lua %}
x = 10
local i = 1              --> 作用域trunk

while i <= x do
	local x = i * 2     --> 作用域while语句
	print(x)
	i = i + 1
end

if i > 20 then
	local x             --> 作用域`then`
	x = 20
	print(x + 2)        --> 22
else
	print(x)            --> 10 (这里x是全局变量)
end

print(x)                --> 10 (这里x是全局变量)
{% endhighlight %}

注意在交互模式下每一行是一个trunk，所以如果用local定义变量的话，这个变量只有在当前行有效。为了解决这个问题，我们可以用`do-end`来限定代码块：

{% highlight lua %}
do
	local a2 = 2*a
	local d = 1
end
{% endhighlight %}

尽可能的使用局部变量：1. 避免命名冲突，形成良好的代码风格。2. 局部变量的访问速度比全局变量快。3. 局部变量在它的作用域结束后就会被回收，而全局变量只有在程序结束时才会被回收，千万资源浪费。

内层作用域的变量会覆盖外层作用域的同名变量，并且如果变量没有进行初始化时，它的值为nil:

{% highlight lua %}
local a, b = 1, 20
if a < b then
	print(a)     --> 1
	local a      --> 赋值nil
	print(a)     --> nil
end
print(a, b)      --> 1 10
{% endhighlight %}

## 控制结构

Lua提供了4种控制结构：if, while, repeat, for，控制条件可以是任意值，Lua认为只有false和nil会判断为假，其它作为值都为真（和其它语言不同，Lua认为数字0以及空字符串都为真）。

### if then else

if语句有以下几种形式：

{% highlight lua %}
if a < 0 then
	a = 0
end

if a < b then 
	return a
else
	return b
end

if op == '+' then
	r = a + b
elseif op == '-' then
	r = a - b
elseif op == '*' then
	r = a * b
elseif op == '/' then
	r = a/b
end
{% endhighlight %}

### while语句


