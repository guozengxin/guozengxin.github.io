---
layout: post
title: "lua学习笔记(5)--函数"
description: "lua学习笔记，第五章，函数"
category: lua
tags: [programming in lua, function, lua, 函数]
comments: true
---

## 函数简介

Lua中函数调用和C语言一样，参数用小括号括越来，用逗号分隔参数，无参函数的情况下也需要写一个小括号。函数可以有返回值，也可以没有。

{% highlight lua %}
print(8*9, 9/8)
a = math.sin(3) + math.cos(10)
{% endhighlight %}

函数参数有一个特殊规则：如果函数只有一个参数，并且参数是一个字符串常量或者table构造器的话，那么可以不用小括号：

{% highlight lua %}
print "Hello"
dofile 'a.lua'
print [[ a multi line
    string ]]

f {x = 10, y = 20}
type{}
{% endhighlight %}

函数的定义也很简单，如定义一个函数求一个数组的和：

{% highlight lua %}
function add (a)
    local sum = 0
    for i = 1, #a do
        sum = sum + a[i]
    end
    return sum
end
{% endhighlight %}

函数参数的匹配和多变量赋值一样，多余的部分会被忽略，缺少的部分会用nil补齐：

{% highlight lua %}
function f(a, b) print(a, b) end
f(3)                 --> 3  nil
f(3, 4)              --> 3  4
f(3, 4, 5)           --> 3  4
{% endhighlight %}

<!-- more -->

## 多返回值

在Lua中，函数可以返回多个结果，相当一部分库函数都会返回多个结果，如：`string.find`函数，返回找到的字符串的开始和结束的下标：

{% highlight lua %}
s, e = string.find('hello Lua users", "Lua")
print(s, e)              --> 7    9
{% endhighlight %}

在定义函数时，只要在**return**关键字后面列出所有需要返回的值，即可以返回多个值。如定义一个函数返回列表中最大的元素及其索引：

{% highlight lua %}
function maxinum(a)
    local mi = 1
    local m = a[mi]
    for i = 1, #a do
        if a[i] > m then
            mi = i;
            m = a[i]
        end
    end
    return m, mi
end

print(maximum({8, 10, 23, 12, 5}))        --> 23 3
{% endhighlight %}

Lua总是调整函数返回值的个数去适应调用环境，当作为语句调用时，会忽略所有的返回值；当作为表达式调用时，会默认保留第一个返回值；当调用作为表达式最后一个参数或者仅有的一个参数时，会根据变量的个数尽可能的匹配多个值，不足补nil。下面列举一些示例来说明上面的几种情况。

假设有以下三个函数：

{% highlight lua %}
function foo0() end
function foo1() return 'a' end
function foo2() return 'a', 'b' end
{% endhighlight %}

在一个多变量赋值中，函数调用作为最后一个值，会尽可能的匹配多个返回结果：

{% highlight lua %}
x, y = foo2()               -- x = 'a', y = 'b'
x = foo2()                  -- x = 'a', ignore 'b'
x, y, z = 10, foo2()        -- x = 10, y = 'a', z = 'b'
{% endhighlight %}

如果函数没有返回值，但是式子中又需要值，这时会用nil补齐：

{% highlight lua %}
x, y = foo()            -- x = nil, y = nil
x, y = foo1()           -- x = 'a', y = nil
x, y, z = foo2()        -- x = 'a', y = 'b', z = nil
{% endhighlight %}

如果函数调用不作为表达式中的最后一个元素时，将只保留第一个返回值：

{% highlight lua %}
x, y = foo2(), 20      -- x = 'a', y = 20
x, y = foo0(), 20, 30  -- x = nil, y = 20
{% endhighlight %}

当一个函数调用作为另一个函数调用的最后一个参数位置时，所有的返回值将作为参数，如果不是最后一个参数位置，将只使用第一个返回值：

{% highlight lua %}
print(foo0())        --> 
print(foo1())        --> a
print(foo2())        --> a    b
print(foo2(), 1)     --> a    1
print(foo2() .. 'x')  --> ax
{% endhighlight %}

当将函数调用用于构造表时，将使用全部的返回值：

{% highlight lua %}
t = {foo0()}             -- t = {}
t = {foo1()}             -- t = {'a'}
t = {foo2()}             -- t = {'a', 'b'}
{% endhighlight %}

如果是`return f()`语句，将返回函数f返回的全部值：

{% highlight lua %}
function foo(i)
    if i == 0 then return foo0()
    elseif i == 1 then return foo1()
    elseif i == 2 then return foo2()
    end
end

print(foo(1))           --> a
print(foo(2))           --> a  b
print(foo(0))           --> (no results)
print(foo(3))           --> (no results)
{% endhighlight %}

也可以将函数调用用小括号括越来，强制只使用第一个返回值：

{% highlight lua %}
print((foo0()))             --> nil
print((foo1()))             --> a
print((foo2()))             --> a
{% endhighlight %}

Lua中有一个预定义函数：**unpack**，接受一个数组作为输入参数，返回数组的所有元素。

{% highlight lua %}
print(unpack{10,20,30})           --> 10   20   30
a, b = unpack{10, 20, 30}         -- a = 10, b = 20
{% endhighlight %}

**unpack**函数可以使用下标指明数组的开始和结束：

{% highlight lua %}
print(unpack({1, 2, 3, 4, 5, 6}, 3, 5))           --> 3 4 5
{% endhighlight %}

## 可变参数

Lua函数可以接收可变参数，和C语言中的**printf**函数类似。下面看一个简单的示例，计算所有参数的和：

{% highlight lua %}
function add(...)
    local s = 0
    for i, v in ipairs{...} do
        s = s + v
    end
    return s
end

print(add(3, 4, 10))        --> 17
{% endhighlight %}

在Lua中，用**(...)**表示函数可以接收可变参数，**...**表示接收到了参数列表，如用表达式**{...}**来通过数组访问接收到的参数。如：

{% highlight lua %}
function foo(a, b, c)
-- 等同于
function foo(...)
    local a, b, c = ...
{% endhighlight %}

也可以在固定参数后面加上可变参数，Lua会将前面的参数传给固定参数，将后面的参数放到可变参数列表arg中：

{% highlight lua %}
function g(a, b, ...) end

g(3)                 -- a = 3, b = nil, arg = {n = 0}
g(3, 4)              -- a = 3, b = 4, arg = {n = 0}
g(3, 4, 5, 8)        -- a = 3, b = 4, arg = {5, 8; n = 2}
{% endhighlight %}

如果在获取返回值时，只想要string.find返回的第二个值，可以有两种方法：

第一种方法是使用虚变量：

{% highlight lua %}
local _, x = string.find(s, p)
{% endhighlight %}

另一种方法是使用可变参数声明一个函数：

{% highlight lua %}
function select(n, ...)
    return arg[n]
end

print(string.find("hello hello", " hel"))        --> 6, 9
print(select(1, string.find("hello hello", " hel"))    --> 6
print(select(2, string.find("hello hello", " hel"))    --> 9
{% endhighlight %}

## 命名参数

Lua中的参数是位置相关的：当调用函数时，Lua是用参数的位置来匹配参数的。然而，有时候需要用名称来匹配参数。例如库函数`os.rename`就是靠名称来匹配的：

{% highlight lua %}
-- invalid code
os.rename(old="temp.lua", new = "new.lua")
{% endhighlight %}

上面的代码是不合法的，Lua不支持这种语法。正确的做法是用一个构造器，并作为函数的唯一参数来实现这个功能：

{% highlight lua %}
os.rename(old='temp.lua', new = 'new.lua')
{% endhighlight %}

根据这种方法我们可以重新实现这个函数：

{% highlight lua %}
funciton rename(arg)
    return os.rename(arg.old, arg.new)
end
{% endhighlight %}

当需要的参数很多时，这种传参的方法非常有用，而且不用一一匹配参数的位置，例如在GUI库创建window的函数中：

{% highlight lua %}
function Window(options)
    if type(options.title) ~= 'string' then
        error('no title')
    elseif type(options.width) ~= 'number' then
        error('no width')
    elseif type(options.height) ~= 'number' then
        error('no height')
    end

    _Window(options.title,
            options.x or 0,
            options.y or 0,
            options.width, options.height,
            options.background or 'white',
            options.border
           )
end

w = Window{x = 0, y = 0, width = 300, height = 200,
           title = 'Lua', background = 'blue',
           border = true
          }
{% endhighlight %}

**Window**函数就可以自由的去构造自己的实参列表，而不用去考虑参数的顺序。
