---
layout: post
title: "lua学习笔记(11)--数据结构"
description: "lua学习笔记，第十一章，数据结构"
category: lua
tags: [Programming in Lua, 数据结构, table, 数组, 集合, 包]
comments: true
---

在Lua中， table是Lua中唯一的数据结构。其它语言提供的数组、记录、列表、栈、集合等数据结构，在Lua都可以用table来实现。并且在Lua中table都很好的实现了这些数据结构。 

在传统的C语言或者Pascal中，我们经常使用数组和列表来实现大部分的数据结构，在Lua不仅可以用table完成同样的功能，而且table的功能更强大。通过table很多算法的实现都简化了，比如在lua中很少去写一个搜索算法，因为Lua本身就提供了这样的功能。

需要花一段时间来学习怎么有效的使用table。在这一章中，将会介绍用table来实现一些典型的数据结构。

<!-- more -->

## 数组

在Lua的实现数组只需要用整数索引table就可以。然而，数组没有固定大小，是按需增长的。不过通常我们都会给数组定义一个大小，比如下面的代码：

{% highlight lua %}
a = {}
for i = 1, 1000 do
    a[i] = 0
end
{% endhighlight %}

操作符(**#**)用来得到数组的大小：

{% highlight lua %}
print(#a)
{% endhighlight %}

可以从索引0, 1或者任何值开始一个数组，比如索引可以从*-5*到*5*：

{% highlight lua %}
a = {}
for i = -5, 5 do
    a[i] = 0
end
{% endhighlight %}

但是在Lua中的惯例是数组从索引1开始，Lua的标准库都遵循这个标准，以及操作**#**。如果数组不是从下标1开始，将无法使用这些库。

数组可以直接使用构造器来初始化：

{% highlight lua %}
squares = {1, 4, 7, 10}
{% endhighlight %}

## 矩阵和多维数组

在Lua中有两种方式来表示矩阵。第一个是嵌套数组。比如可以用下面的代码构造一个*N\*M*的数组：

{% highlight lua %}
mt = {}
for i = 1, N do
    mt[i] = {}
    for j = 1, M do
        mt[i][j] = 0
    end
end
{% endhighlight %}

在Lua中table是对象，所以你必须显式的创建每一行。这多少有点麻烦，但是提供了更多的灵活性。比如**三角矩阵**可以给每一行创建不同的大小，来节省空间。

创建矩阵的第二个方法将行列组合起来。如果下标都是整数，可以把行标乘以一个常量加上列标。用这个方法构造一个*N\*M*的数组：

{% highlight lua %}
mt = {}
for i = 1, N do
    for j = 1, M do
        mt[(i=1) * M + j] = 0
    end
end
{% endhighlight %}

如果索引都是字符串的话，可以用一个字符将两个索引串连接起来构造一个单一的下标。如两个下标分别为*s*和*t*，假定*s*和*t*都不包含冒号，那么就可以用*[s..':'..t]*来表示矩阵的下标。如果*s*和*t*中可能含有冒号的话，可以用控制字符来连接，比如`\n`。

第二种表示方法的优势是节省空间，特别对于**稀疏矩阵**来说。由于稀疏矩阵大部分的元素都为空，在Lua中可以用*nil*来表示。由于table天生就具有稀疏的特性，所以用第二种方式只需要存储那些不为*nil*的元素就可以了。

## 链表

由于table是动态实体，可以很简单的实现链表。每个节点是一个table，指针是这个表的一个域，并且是另外一个节点的引用。比如，我们实现一个简单的列表，每个节点含有两个域：`next`和`value`。根节点：

{% highlight lua %}
list = nil
{% endhighlight %}

用值`v`创建一个节点，并插入到链表头部：

{% highlight lua %}
list = {next = list, value = v}
{% endhighlight %}

遍历列表：

{% highlight lua %}
local l = list
while l do
    print(l.value)
    l = l.next
end
{% endhighlight %}

其它列表，像**双向链表**或者**循环链表**，都可以简单的用table来实现。但是在Lua很少需要那样的数据结构，因为通常都会有比链表更简单的方式来实现。

## 队列和双向队列

一个简单的实现队列的方式是用table库中的`insert`和`remove`函数。这两个函数可以从数组任何位置插入和移除元素。然而，这种方式在应对大数据结构时效率很低。更加有效的实现方式是使用两个索引，一个表示第一个元素另一个表示最后一个元素：

{% highlight lua %}
function ListNew()
    return {first = 0, last = -1}
end
{% endhighlight %}

为了避免全局变量的干扰，我们将所有的列表操作定义在一个table内部，名为`List`。因此，我们重写上面的示例：

{% highlight lua %}
List = {}
function List.new()
    return {first = 0, last = -1}
end
{% endhighlight %}

现在我们可以在常数时间在队列的两端进行插入和删除操作：

{% highlight lua %}
function List.pushfirst(list, value)
    local first = list.first - 1
    list.first = first
    list[first] = value
end

function List.pushlast(list, value)
    local last = list.last + 1
    list.last = last
    list[last] = value
end

function List.popfirst(list)
    local first = list.first
    if first > list.last then error('list is empty') end
    local value = list[first]
    list[first] = nil
    list.first = first + 1
    return value
end

function List.poplast(list)
    local last = list.last
    if list.first > last then error('list is empty') end
    local value = list[last]
    list[last] = nil
    list.last = last - 1
    return value
end
{% endhighlight %}

从队列的定义上来讲，只能调用`pushlast`和`popfirst`，这样*first*和*last*都会持续增加。然而在Lua中，用table来实现数组，索引值可以是从1到20，也可以是从16777216到16777236. 另外，Lua使用双精度表示数字，假定每秒执行100万次插入操作，在数值溢出前程序可以执行200年。

## 集合和包

假定你想列出一段源码中所有的标识符，并且滤掉预留的单词。传统的C程序员会将预留的单词放到一个数组中，然后遍历源代码，查找每个标识符是否在这个数组中。

在Lua中，用table可以更加简单的实现这样的集合，只要集合的元素作为索引值存储就可以了。在查找元素的时候不需要遍历，只需要对给定元素，看看以它为下标的值是否为*nil*。

{% highlight lua %}
reserved = {
    ['while'] = true,
    ['end'] = true,
    ['function'] = true,
    ['local'] = true,
}

fow w in allwords() do
    if not reserved[w] then
        <do something>
    end
end
{% endhighlight %}

还可以用辅助函数构造出更清晰的集合：

{% highlight lua %}
function Set(list)
    local set = {}
    for _, l in ipairs(list) do set[l] = true end
    return set
end

reserved = Set{'while', 'end', 'function', 'local', }
{% endhighlight %}

**包**，也可以称为**多集合**，与通常的集合不同的是，同一个元素可以出现多次。Lua中表示包的方法和表示集合的方法类似，除了每个元素需要加一个计数器。插入一个元素并计数：

{% highlight lua %}
function insert(bag, element)
    bag[element] = (bag[element] or 0) + 1
end
{% endhighlight %}

删除一个元素并减计数：

{% highlight lua %}
function remove(bag, element)
    local count = bag[element]
    bag[element] = (count and count > 1) and count - 1 or nil
end
{% endhighlight %}

## 字符串缓冲

假设你现在想把很多小字符串拼成一个大字符串，例如按行读取一个文件。你的代码可能是这样的：

{% highlight lua %}
local buff = ''
for line in io.lines() do
    buff = buff .. line .. '\n'
end
{% endhighlight %}

尽管这段代码看上去很正常，但是当文件特别大时，效率会很低。例如对于1M的文件，读取需要1分钟。

为什么会很大呢？假设我们正在读一个文件，每行20字节，我们已经读了2500行，所以现在*buff*的大小是50KB。现在当读下一行的时候，Lua将会创建一个新的50020字节的string，并且把原来的字符串拷贝到新的字符串中。这意味着，后面每读一行，都会进行至少50KB大小的分配和拷贝工作。下面这行语句是导致这个问题的罪魁祸首：

{% highlight lua %}
buff = buff .. line .. '\n'
{% endhighlight %}

当后面又读100行的时候，Lua进行了5MB的移动；当读了350KB的文件的时候，Lua将进行50GB的移动！

事实上这个问题不只是存在于Lua，其它的采用字符串不可变特性的语言都存在这个问题。如Java。

对于小的字符串，上面的这个情形并不是一个问题。为了读整个文件，Lua提供了`io.read('*a')`选项，一次性读取整个文件。然而，有时候我们不得不面对这个效率问题，就像Java提供了*StringBuffer*来改善这个问题。Lua中提供了`table.concat`函数，返回了给出的列表中的所有字符串的连接。我们可以重写上面的代码：

{% highlight lua %}
local t = {}
for line in io.lines() do
    t[#t + 1] = line .. '\n'
end
local s = table.concat(t)
{% endhighlight %}

这个循环读取350KB的文件只需要0.5s，但上一段代码需要1分钟。（`io.read(*a)`仍然是最快的）

还可以进行优化！`concat`函数接收第二个可选的参数，代码字符串之间的分隔符。改善后的代码如下：

{% highlight lua %}
local t = {}
for line in io.lines() do
    t[#t + 1] = line
end
local s = table.concat(t, '\n') .. '\n'
{% endhighlight %}

现在`concat`就会自动在string之间加上`\n`，但是我们还需要手动添加最后一个`\n`。最后这次的字符串移动是最长的，解决的方法是我们手动在表*t*的最后添加一个空字符串：

{% highlight lua %}
t[#t + 1] = ''
s = table.concat(t, '\n')
{% endhighlight %}

## 图

像大多数语言一样，Lua也可以实现图的多种实现方式，每种实现方式适合特定的算法。现在我们看一种简单的面向对象的实现，用对象代表节点，用节点之间的引用代表弧。

现在用table来表示节点，有两个域：*name*表示节点名称，*adj*表示和当前节点邻接的节点集合。由于我们要从文本文件中读取图，需要一个根据节点名称找到节点的方法。所以，我们用一个额外的table表示名称到节点的映射。函数`name2node`可以根据名称返回对应的节点：

{% highlight lua %}
local function name2node(graph, name)
    local node = graph[name]
    if not node then
        node = {name = name, adj = {}}
        graph[name] = node
    end
    return node
end
{% endhighlight %}

下面的函数用来创建一个图。它读取一个文件，文件中的每一行有两个节点名称，表示这两个节点之间有一条弧。对每一行，使用`string.match`来分隔两个名称，然后通过`name2node`来找到对应的节点（如果找不到则创建新节点），并且添加节点之间的连接。

{% highlight lua %}
function readgraph()
    local graph = {}
    for line in io.lines() do
        local namefrom, nameto = string.match(line, '(%S+)%s+(%S+)')
        local from = name2node(graph, namefrom)
        local to = name2node(graph, nameto)
        from.adj[to] = true
    end
    return graph
end
{% endhighlight %}

下面的函数`findpath`使用深度优先遍历来寻找两个节点之间的路径。它第一个参数是当前节点，第二个参数是目标节点，第三个参数保存了开始节点到当前节点的路径，第四个参数保存所有已经访问过的节点。注意算法怎样不通过节点名称来操作节点。如，*visited*是一个节点集合，而不是节点名称，同样，*path*是一个节点列表。

{% highlight lua %}
function findpath(curr, to, path, visited)
    path = path or {}
    visited = visited or {}
    if visited[curr] then
        return nil
    end
    visited[curr] = true
    path[#path + 1] = curr
    if curr == to then
        return path
    end
    for node in pairs(curr.adj) do
        local p = findpath(node, to, path, visited)
        if p then return p end
    end
    path[#path] = nil
end
{% endhighlight %}

现在我们添加一些代码来测试上面这几个函数：

{% highlight lua %}
function printpath(path)
    for i = 1, #path do
        print(path[i].name)
    end
end

g = readgraph()
a = name2node(g, 'a')
b = name2node(g, 'b')
p = findpath(a, b)
if p then printpath(p) end
{% endhighlight %}
