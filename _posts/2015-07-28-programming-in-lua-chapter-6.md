---
layout: post
title: "lua学习笔记(6)--再说函数"
description: "lua学习笔记，第六章，再说函数"
category: lua
tags: [programming in lua, function, lua, 函数, 闭包]
comments: true
---

Lua中的函数是含有**词法定界**的**第一类值**。

什么是**第一类值**？意思是在Lua中，函数是和其他类型的值有着同样功能的值，我们可以把它存储在变量中，赋值，并且作为参数和返回值。

什么是**词法定界**？意思是函数可以访问它的外部函数的变量。这个特征是强大的，因为它可以实现很多强大的函数式编程的功能，可以使你的程序更为简短。

Lua中的函数是匿名的，在程序中实际使用的函数名只是承载这个函数的一个变量，我们可以用各种方式来承载函数：

{% highlight lua %}
a = {p = print}
a.p('Hello World')       --> Hello World
print = math.sin
a.p(print(1))            --> 0.841470
sin = a.p
sin(10, 20)              --> 10  20
{% endhighlight %}

传统的创建函数的方式为：

{% highlight lua %}
function foo(x) return 2*x end
{% endhighlight %}

这实际上是一个**语法糖**，下面是原本的定义方式：

{% highlight lua %}
foo = function(x) return 2*x end
{% endhighlight %}

所以，函数定义实际上是将一个函数**值**赋给一个变量，而函数本身是匿名的。

<!-- more -->

在table库中提供了一个函数`table.sort`，可以将table中的元素排序。这个函数必须能够对不同类型的值进行排序，Lua不是尽可能多的提供参数来满足需求，而是接收一个排序函数作为参数，排序函数接收两个排序元素作为参数，并返回两者大小的关系。假设我们的表是这样的：

{% highlight lua %}
network = {
    {name = 'grauna', IP = '210.26.30.34'},
    {name = 'arraial', IP = '210.26.30.23'},
    {name = 'lua', IP = '210.26.30.36'}
}
{% endhighlight %}

如果我们想按照**name**域排序，可以这样写：

{% highlight lua %}
table.sort(network, funciton(a, b)
    return (a.name > b.name)
end)
{% endhighlight %}

一个函数接收另一个函数作为参数，如`table.sort`，我们称为**高阶函数**，高阶函数是一个强大的编程系统，有很强的灵活性。

这里我们看另一个高阶函数的用法，求一个函数的导数，用**(f(x+d)-f(x))/d**来求导（d趋于无限小）：

{% highlight lua %}
function derivative(f, delta)
    delta = delta or 1e-4
    return function(x)
        return (f(x + delta) - f(x)) / delta
    end
end

c = derivative(math.sin)
print(math.cos(5.2), c(5.2))          --> 0.46851667130038  0.46856084325086
print(math.cos(10), c(10))            --> -0.83907152907645 -0.83904432662041
{% endhighlight %}

这个函数根据参数返回了一个函数的导函数。

## 闭包

当我们用一个函数包含另一个函数时，被包含的函数有权限访问包含它的函数，这个特性叫做**词法定界**。**词法定界**加上**第一类值**，是一个强大的概念，很多其它语言是不具有这个特性的。这也是函数式编程最主要的特性之一。

让我们看一个示例。假设有一个学生名字的列表和另一个名字和分数的映射表，然后需要根据分数对名字表进行排序，我们可以这样做：

{% highlight lua %}
names = {'Peter', 'Paul', 'Mary'}
grades = [Mary = 10, Paul = 7, Peter = 8}
table.sort(names, function (n1, n2)
    return grades[n1] > grades[n2]
)
{% endhighlight %}

现在，假设我们需要创建一个函数来完成这个任务：

{% highlight lua %}
function sortbygrade(names, grades)
    table.sort(names, function (n1, n2)
        return grades[n1] > grades[n2]
    )
{% endhighlight %}

在这个函数中，可以看到匿名的比较函数有权限访问变量`grades`，这个变量是`sortbyfrade`的参数，在这里称为外部的局部变量(non-local value)。

这个特性有趣的地方在于，由于函数是第一类值，因此他们可以脱离它定义的环境。看这个示例：

{% highlight lua %}
function newCounter()
    local i = 0
    return function ()
        i = i + 1
        return i
    end
end

c1 = newCounter()
print(c1())       --> 1
print(c1())       --> 2
{% endhighlight %}

在函数`newCounter`中返回了一个匿名函数，匿名函数调用了**non-local value**`i`。然而我们真正调用这个函数的时候`print(c1())`，已经脱离了`i`的作用域，但是Lua还是利用了闭包的概念正确处理了逻辑。简单来说，闭包就是一个函数加上它所有的**non-local value**。如果我们再调用一次`newCounter`，将会创建一个新的变量`i`，所以形成了一个新闭包，接着上面的示例：

{% highlight lua %}
c2 = newCounter()
print(c2())          --> 1
print(c1())          --> 3
print(c2())          --> 2
{% endhighlight %}

所以，`c1`和`c2`是同一个函数创建的不同的闭包，两者是相互独立的。

闭包在很多情况下是一个很有用的功能，如前面的示例，可以作为`table.sort`函数的参数。闭包也可以让我们在函数中创建函数，如`newCounter`，这一特性可以让我们在**函数式编程**中创造无限的可能。闭包在创建**回调函数**中也非常有用，一个典型的示例就是创建按键的回调函数，如，计算器程序需要对每个数字写一个回调函数：

{% highlight lua %}
function digitButton(digit)
    return Button{label = tostring(digit),
                  action = function()
                      add_to_display(digit)
                  end
                 }
end
{% endhighlight %}

在这个示例中，参数数`digit`是**non-local value**，在外部调用中Lua会正确处理。

闭包也可以应用在重定义库函数，如重定义`math.sin`函数，使其接收一个度数而不是弧度：

{% highlight lua %}
oldSin = math.sin
math.sin = function(x)
    return oldSin(x * math.pi / 180)
end
{% endhighlight %}

更清楚的方式：

{% highlight lua %}
do
    local oldSin = math.sin
    local k = math.pi / 180
    math.sin = function(x)
        return oldSin(x * k)
    end
end
{% endhighlight %}

这样，旧的函数是一个局部变量，我们只能通过重定义的函数来访问。

当我们运行一段不信任的代码时，利用闭包我们可以创建一个安全环境（**沙箱**）。比如可以使用闭包重定义io库的`open`函数：

{% highlight lua %}
do
    local oldOpen = io.open
    io.open = function(filename, mode)
        if access_OK(filename, mode) then
            return oldOpen(filename, mode)
        else
            return nil, 'access denied'
        end
    end
end
{% endhighlight %}

## 非全局函数

函数不仅可以保存在全局变量中，也可以作为局部变量和表的元素来保存。如果`io.read`，`math.sin`都是作为表中的一个域来保存的。用下面语法来创建：

{% highlight lua %}
Lib = {}
Lib.foo = function(x, y) return x + y end
Lib.goo = function(x, y) return x - y end

print(Lib.foo(2, 3), Lib.goo(2, 3))            --> 5     -1
{% endhighlight %}

也可以直接初始化：

{% highlight lua %}
Lib = {
    foo = function(x, y) return x + y end
    goo = function(x, y) return x - y end
}
{% endhighlight %}

Lua也提供了另一种语法创建这样的函数：

{% highlight lua %}
Lib = {}
function Lib.foo(x, y) return x + y end
function Lib.goo(x, y) return x - y end
{% endhighlight %}

当把函数保存在一个局部变量时，得到一个局部函数，局部函数只有在当前的chunk中有效，在chunk中的其它代码可以调用这个函数：

{% highlight lua %}
local f = function(<params>)
    <body>
end
{% endhighlight %}

另一种方式：
{% highlight lua %}
local function f(<params>)
    <body>
end
{% endhighlight %}

有一个问题在于定义递归局部函数的时候，常规的方法可能不生效：

{% highlight lua %}
local fact = function(n)
    if n == 0 then return 1
    else return n * fact(n - 1)        -- ERROR, fact找不到
    end
end
{% endhighlight %}

当Lua调用`fact(n - 1)`的时候，局部变量`fact`还没有被定义，因此就会去找全局变量`fact`，然而并没有找到。所以要用下面的方式创建：

{% highlight lua %}
local fact
fact = function(n)
    if n == 0 then return 1
    else return n * fact(n - 1)
    end
end
{% endhighlight %}

然而在上面第二种定义方式中，不存在这样的问题，因为：

{% highlight lua %}
local function foo(<params>)
    <body>
end

-- 等同于

local foo;
foo = function(<params>)
    <body>
end
{% endhighlight %}

## 正确的尾调用

Lua中函数的另一个重要特征是可以正确处理**尾调用**。**尾调用**是一种类似在结尾的goto调用，当函数最后一个语句是调用另一个函数时，我们称为尾调用：

{% highlight lua %}
function f(x)
    return g(x)
end
{% endhighlight %}

函数f在调用g后不用再做别的事情，这种情况下，g结束后不需要再返回f，而可以直接返回到f的调用者。所以尾调用后不需要保留调用栈信息，拥有更快的调用速度。

由于尾调用不保留栈信息，所以尾递归可以是无限层的，下面的程序不论n为何值不会导致栈溢出：

{% highlight lua %}
function foo(n)
    if n > 0 then return foo(n - 1) end
end
{% endhighlight %}

但是有一些情况类似于尾调用，但不是尾调用。如：

{% highlight lua %}
function f(x) 
    g(x)
end
{% endhighlight %}

在这个函数中，g执行结束后，f必须释放掉g返回的临时变量才能结束，因此不是尾调用。类似的，下面的返回式都不是尾调用：

{% highlight lua %}
return g(x) + 1
return x or g(x)
return (g(x))
{% endhighlight %}

在Lua中，只有式子`return func(args)`才是一个正确的尾调用，但`func`和它的参数都可以很复杂，因为Lua会在调用之前完成所有的计算。如，下面的调用是一个正确的尾调用：

{% highlight lua %}
return x[i].foo(x[j] + a * b, i + j)
{% endhighlight %}
