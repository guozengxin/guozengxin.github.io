---
layout: post
title: "lua学习笔记(7)--迭代器和for循环"
description: "lua学习笔记，第七章，迭代器和泛型for循环"
category: lua
tags: [programming in lua, for, lua, iterator, 迭代器]
comments: true
---

这一章主要讨论如何写迭代器，先由简单的迭代器开始，然后逐渐讨论如何写复杂的迭代器。

## 迭代器和闭包(Iterators and Closures)

迭代器是一个可以遍历集合中的元素的结构. 在Lua中迭代器是一个函数，每次调用这个函数的时候，它会返回集合中的下一个元素。

任何一个迭代器需要保留上一次调用的状态，以及下个需要返回的元素。闭包的机制可以实现这个逻辑，因为闭包是一个函数及一些可访问的**non-local variable**组成，这些变量可以保存遍历的状态，包括调用的状态等。因此一个闭包包含两个函数：闭包自己和一个工厂函数，工厂函数创建闭包和它包含的变量。

如写一个示例遍历一个list，和**ipairs**不同，这个迭代器不返回索引，只返回值：

{% highlight lua %}
function values(t)
    local i = 0
    return function () 
        i = i + 1;
        return t[i]
    end
end
{% endhighlight %}

<!-- more -->

在这个示例中，`values`是**工厂函数**，每次调用这个函数时，它都会创建一个新的闭包。这个闭包保存了外部变量`t`和`i`的状态，每次调用这个闭包时，都会返回list中的下一个元素，遍历完毕时会返回**nil**.

在一个**while**循环中使用它：

{% highlight lua %}
t = {10, 20, 30}
iter = values(t)
while true do
    local element = iter()
    if element == nil then break end
    print(element)
end
{% endhighlight %}

在**泛型for**中使用迭代器更为简便：

{% highlight lua %}
t = {10, 20, 30}
for element in values(t) do
    print(element)
end
{% endhighlight %}

泛型for为迭代循环处理所有的簿记(bookkeeping)：首先调用工厂函数创建一个迭代器，内部保留迭代函数（因此不需要iter变量），然后在每次循环中调用迭代器，当迭代器返回nil时自动结束。

下面看一个高级点的示例，遍历输入文件中的所有单词。为了实现这个目的，我们保留了两个状态，当前行和当前行的当前位置，函数主体调用了`string.find`，这个函数从一行找到一个单词，可以指定开始位置，`%w+`表示查找单词，类似正则表达式，函数返回查找到内容的开始位置和结束位置。

{% highlight lua %}
function allwords()
    local line = io.read()
    local pos = 1
    return function()
        while line do
            local s, e = string.find(line, '%w+', pos)
            if s then
                pos = e + 1
                return string.sub(line, s, e)
            else
                line  = io.read()
                pos = 1
            end
        end
        return nil
    end
end
{% endhighlight %}

尽管迭代器很复杂，但是使用时依然很简单：

{% highlight lua %}
for word in allwords() do
    print(word)
end
{% endhighlight %}

## 泛型for的语义

前面看到，我们需要为每个循环创建一个闭包，在大多数情况下，这不存在问题。然后有些情况下创建闭包的代价是不能忍受的，这种情况下可以使用泛型for本身来保存迭代状态。

在前面示例中可以看到，泛型for保存了迭代器的状态。更加准确的说，它保存了三个值：迭代器函数，状态常量和一个控制变量。泛型for的语义是：

{% highlight lua %}
for <var-list> in <exp-list> do
    <body>
end
{% endhighlight %}

`var-list`是变量列表，可以有一个或者多个变量，用逗号分隔；`exp-list`是表达式列表，可以有一个或者多个表达式，用逗号分隔。通常表达式列表只有一个工厂函数。在下面的示例中，`k, v`是变量列表，`pairs(t)`是表达式列表。

{% highlight lua %}
for k, v in pairs(t) do
    print(k, v)
end
{% endhighlight %}

变量列表也可以只有一个变量：

{% highlight lua %}
for line in io.lines() do
    io.write(line, "\n")
end
{% endhighlight %}

变量列表的第一个变量称为**控制变量**,它的值如果为`nil`，表示循环结束。

for循环的运行步骤：
1. 计算in后面的表达式的值，表达式会计算for循环需要的3个值：迭代器函数，状态常量和控制变量。和多赋值一样，如果表达式返回的值不足3个，Lua会自动补齐。 
2. 在完成上面的初始化之后，for循环用两个参数:状态常量和控制变量，去调用迭代器函数（注意：对于for结构来说，状态常量没有用处，仅在初始化时获取它的值并传递给迭代函数）。
3. 将迭代函数返回的值赋给变量列表。
4. 如果返回的第一个值为nil循环结束，否则执行循环体。
5. 回到第2步再次调用迭代函数。

更精确的来说,下面的结构：

{% highlight lua %}
for var_1, ..., var_n in <explist> do
    <block>
end
{% endhighlight %}

和下面的代码逻辑相同：

{% highlight lua %}
do
    local _f, _s, _var = <explist>
    while true do
        local var_1, ..., var_n = _f(_s, _var)
        _var = var_1
        if _var == nil then break end
        <block>
    end
end
{% endhighlight %}

如果迭代函数是`f`, 状态常量是`s`, 并且控制变量的初始值是`a0`, 控制变量将做如下循环：`a1 = f(s, a0)`，`a2 = f(s, a1)` ... 直到`ai`为nil，如果**for**有其它值，它们将通过调用函数`f`返回。

## 无状态迭代器

**无状态迭代器**指不保存任何状态的迭代器。因此，我们可以在多个循环中使用同一个无状态的迭代器，避免创建新的闭包的开销。

刚刚看到，**for循环**调用迭代器函数要使用两个参数：状态常量和控制变量。一个无状态迭代器产生下一个元素只使用这两个值。典型的示例是`ipairs`：

{% highlight lua %}
a = {'one', 'two', 'three'}
for i, v in ipairs(a) do
    print(i, v)
end
{% endhighlight %}

迭代的状态包括被遍历的表（状态常量）、当前索引（控制变量）。我们可以自己实现这个迭代器：

{% highlight lua %}
local function iter(a, i)
    i = i + 1
    local v = a[i]
    if v then
        return i, v
    end
end

function ipairs(a)
    return iter, a, 0
end
{% endhighlight %}

当Lua在for循环中，调用`ipairs(a)`时，它首先得到3个值：迭代函数`iter`，状态常量`a`，控制变量的初始状态`0`。然后Lua调用`iter(a,0)`，将得到返回值`1, a[1]`，下一次调用`iter(a, 1)`，返回`2, a[2]`,直到循环结束。

`pairs`函数，是一个迭代器遍历table中的所有元素。除了迭代器函数是`next`函数：

{% highlight lua %}
function pairs(t)
    return next, t, nil
end
{% endhighlight %}

函数`next(t, k)`，`k`是表`t`中的一个key，返回下一个key以及对应的value。`next(t, nil)`将返回第一个key-value对。在循环中也可以直接调用`next`函数：

{% highlight lua %}
for k, v in next, t do
    <loop body>
end
{% endhighlight %}

for循环会自动适应参数，所以这个for循环会获取到`next, t, nil`，可以看到和`pairs(t)`的是完全一致的。

用无状态的迭代器来遍历一个链表也是比较有趣的：

{% highlight lua %}
local function getnext(list, node)
    if not node then
        return list
    else
        return node.next
    end
end

function traverse(list)
    return getnext, list, nil
end
{% endhighlight %}

技巧是使用链表的主节点`list`作为状态常量，然后用当前值作为控制变量。当第一次调用`getnext`的时候，`node`是nil，因此返回`list`作为第一个元素。在接下来的调用中，`node`参数不是nil，因此返回`node.next`。

## 复杂状态的迭代器

有时候，只有一个状态常量和一个控制变量的迭代器并不能满足需求，我们需要保存更多的状态。简单的解决办法是使用迭代器，还有一种方法是将所有的信息打包到一个table中，用这个table作为状态常量。使用table可以让迭代器拥有更多的data，并且可以遍历过程中改变data。虽然状态常量是一个不变的table，但是过程中可以改变table中的值。

作为这个技术的一个示例，我们重写函数`allwords`，来遍历输入文件中的所有单词。这次我们用一个table来保存`line`和`pos`变量。

{% highlight lua %}
local iterator

function allwords()
    local state = {line = io.read(), pos = 1}
    return iterator, state
end

function iterator(state)
    while state.line do
        local s, e = string.find(state.line, '%w+', state.pos)
        if s then
            state.pos = e + 1
            return string.sub(state.line, s, e)
        else
            state.line = io.read()
            state.pos = 1
        end
    end
    return nil
end
{% endhighlight %}

无论何时，都尽量尝试去写无状态的迭代器，而是由for来保存状态，因为创建对象花费的代价要高。如果不能用无状态迭代器来实现，也要用闭包去实现，因为通常情况下闭包要比创建table的代价小；另外处理**non-local variables**的速度要比table快。后续章节中将会介绍用协程创建迭代器，这种方式将更强，但更复杂。

## 真正的迭代器

上面介绍的迭代器并不是真正意义上的迭代器，因为它并没有迭代，它和python中的**生成器**很类似。

现在介绍一种方式创建一个在内部完成迭代的迭代器。当使用这样的迭代器时，不需要写一个循环，我们仅仅写出每一次迭代需要处理的任务作为参数来调用迭代器就可以了。具体的说，迭代器接受一个函数作为参数，函数在迭代器内部被调用。

下面看一个具体的示例：

{% highlight lua %}
function allwords(f)
    for l in io.lines() do
        for w in string.gfind(l, '%w+') do
            f(w)
        end
    end
end
{% endhighlight %}

如果我们要把单词打印出来，只需要：

{% highlight lua %}
allwords(print)
{% endhighlight %}

更一般的做法是用匿名函数作为参数，下面示例打印出单词**helllo**出现的次数：

{% highlight lua %}
local count = 0
allwords(function (w)
    if w == 'hello' then count = count + 1 end
end)
print(count)
{% endhighlight %}

用for结构也可以完成同样的任务：

{% highlight lua %}
local count = 0
for w in allwords() do
    if w == 'hello' then count = count + 1 end
end
{% endhighlight %}
