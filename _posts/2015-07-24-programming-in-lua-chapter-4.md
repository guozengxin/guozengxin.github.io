---
layout: post
title: "lua学习笔记(4)--语法"
description: "lua学习笔记，第四章，语法"
category: lua
tags: [programming in lua, lua, 语言, 变量, goto]
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

<!-- more -->

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

尽可能的使用局部变量：1. 避免命名冲突，形成良好的代码风格。2. 局部变量的访问速度比全局变量快。3. 局部变量在它的作用域结束后就会被回收，而全局变量只有在程序结束时才会被回收，造成资源浪费。

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

### while

**while**循环当条件为true时，执行循环体，当条件为假时，循环结束：

{% highlight lua %}
local i = 1
while a[i] do
    print(a[i])
    i = i + 1
end
{% endhighlight %}

### repeat

**repeat-until**语句重复执行循环体直到条件为true：

{% highlight lua %}
-- 打印第一个非空的行
repeat
    line = io.read()
until line ~= ''
print(line)
{% endhighlight %}

不同于其他语言，循环体中的局部变量的作用范围包括条件语句：

{% highlight lua %}
repeat
    sqr = (sqr + x / sqr) / 2
    local error = math.abs(sqr^2 - x)
until error < x / 10000                    -- 局部变量error仍然有效
{% endhighlight %}

### for

for循环有两大类：

第一，数值for循环：

{% highlight lua %}
for var = exp1, exp2, exp3 do
    -- loop-part
end
{% endhighlight %}

for将用**exp3**作为step从初始值**exp1**到终止值**exp2**，执行loop-part。当省略**exp3**时，默认step为1.

有以下几点需要注意：
一、 三个表达式只会计算一次，在循环开始之前；
二、 循环控制变量是一个局部变量，只在for循环体内有效。一个典型的错误就是认为变量在循环结束后还有效：

{% highlight lua %}
for i = 1, 10 do print(i) end
max = i                     -- 错误，'i'不是for循环中的i
{% endhighlight %}

如果需要保留控制变量，需要进行操作将它保存：

{% highlight lua %}
local found = nil
for i = 1, a.n do
    if a[i] == value then
        found = i
        break
    end
end
print(found)
end
{% endhighlight %}

三、 循环过程中不要改变控制变量的值，那样做结果是不可预知的。

第二，范型for循环：

范型for循环遍历一个迭代器的所有元素，如：

{% highlight lua %}
for k, v in pairs(t) do print(k, v) end
{% endhighlight %}

`pairs`函数是一个方便的迭代器函数，输入一个table，每次循环，k获取key，v获取value。

范型for循环是一个简单而又强大的特性，在Lua中，有很多内置的迭代器，如按行遍历输入`io.lines`，遍历table`pairs`，遍历一个序列`ipairs`，遍历一个字符串`string.gmatch`等等。 我们也可以用函数实现自己的迭代器，这个有很多细节要讲，放到后面的章节细谈。

下面看一个具体的示例，假设有一个星期表：

{% highlight lua %}
days = {'Sunday', 'Monday', 'Tuesday', 'Wednesday',
        'Thursday', 'Friday', 'Saturday'}
{% endhighlight %}

现在想把名字转换成在一周中的次序，一个有效的方式是构造一个反向表：

{% highlight lua %}
revDays = {['Sunday'] = 1,
           ['Monday'] = 2,
           ['Tuesday'] = 3,
           ['Wednesday'] = 4,
           ['Thursday'] = 5,
           ['Friday'] = 6,
           ['Saturday'] = 7
          }

x = 'Tuesday'
print(revDays[x])         --> 3
{% endhighlight %}

用for循环可以很方便的构造反向表，而不需要手工做：

{% highlight lua %}
revDays = {}
for i, v in ipairs(days) do
    revDays[v] = i
end
{% endhighlight %}

## break, return 和 goto

### break和return
和其它语言一样，**break**语句结束一个循环，在循环外部不可以使用break。

return语句从函数返回结果，也可以不返回结果，只用于结束函数。

Lua语法规定**break**和**return**语句必须作为chunk的最后一句出现：即出现在**end**, **else**或者**until**之前。例如：

{% highlight lua %}
local i = 1
while a[i] do
    if a[i] == v then return i end
    i = i + 1
end
{% endhighlight %}

有时，为了调试方便我们需要在其它地方写return或者break，怎么办呢？

只要用`do ... end`语句包含即可：

{% highlight lua %}
function foo()
    do return end
    ...
end
{% endhighlight %}

### goto

Lua还支持**goto**语句，用`::name::`来设置一个标签，在代码的其他地方调用。另外，Lua中有一些语法规则限制goto的滥用：1. 标签遵循变量的可见性规则，不能跳进一个block中；2. 不能跳出或者跳进一个函数；3. 不能跳进局部变量的范围。

goto的一个典型的应用就是模拟**continue**语句：

{% highlight lua %}
while some_condition do
    ::redo::
    if some_other_condition then goto continue
    else if yet_another_condition then goto redo
    end
    <some code>
    ::continue::
end
{% endhighlight %}

**不能跳进局部变量的范围**的意思是：就是从某个局部变量的范围之外，跳进这个局部变量的作用范围之内，但是如果标签的位置是这个局部变量的范围的最后的非空语句之后（标签是空语句），就是允许的。这句话很难理解，看一个示例：

{% highlight lua %}
while some_condition do
    if some_other_condition then goto continue end
    local var = something
    <some code>
    ::continue::             -- ERROR 在var的作用范围之内，并且不是范围内的非空语句之后
    <some non-void code>
    ::continue::             -- 允许，在var的范围内的最后的非空语句之后 
end
{% endhighlight %}
