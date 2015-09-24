---
layout: post
title: "lua学习笔记(10)--完整程序示例"
description: "lua学习笔记，第十章，完整程序示例"
category: lua
tags: [Programming in Lua]
comments: true
---

在这一篇里，将展示三个完整的程序示例。**八皇后问题**，**词频问题**，**马尔可夫链算法**。

## 八皇后问题

第一个示例是一个简单的程序来解决八皇后问题：目标是将八个皇后放到棋盘中，并且互相之间不能攻击。

解决这个问题的第一步是：注意问题的每一个解都是每行只能有一个皇后。因此，我们可以用一个8个数字的数组表示问题的解，每个数字代表皇后放到那一行的哪个位置。例如，*{3, 7, 2, 1, 8, 6, 5, 4}*表示一个解（但是不合法，斜的路线可以攻击）。注意每个合法的解都是整数1到8的置换，因为同样每一列都只能有一个皇后。

整个程序如下所示。

{% highlight lua %}
local N = 8

local function isplaceok(a, n, c)
    for i = 1, n-1 do
        if (a[i] == c) or
                (a[i] - i == c - n) or
                (a[i] + i == c + n) then
            return false
        end
    end
    return true
end

local function printsolution(a)
    for i = 1, N do
        for j = 1, N do
            io.write(a[i] == j and 'X' or '-', ' ')
        end
        io.write('\n')
    end
    io.write('\n')
end

local function addqueen(a, n)
    if n > N then
        printsolution(a)
    else
        for c = 1, N do
            if isplaceok(a, n, c) then
                a[n] = c
                addqueen(a, n+1)
            end
        end
    end
end

addqueen({}, 1)
{% endhighlight %}

函数`isplaceok`检查给出的一个数组是否是问题的解。问题的解必须保证没有两个皇后在同一列或者在同一条对角线上。

函数`printsolution`将结果打印到屏幕上。*X*表示皇后，`-`表示空白。

函数`addqueen`是程序的核心函数。它用回溯的方法搜索合法的解。首先，它检查搜索是否完成，如果完成，打印出结果。否则，循环所有列，并给当前行当前列放置一个皇后，并检查已经放置的皇后是否合法，如果合法，递归调用`addqueen`继续搜索下一行，否则，搜索下一列。

<!-- more -->

## 词频问题

下一个示例是读取一个文本文件然后打印出词频最高的单词。

程序的主要数据结构是一个*table*，存储了单词和它的词频。围绕这个变量，程序有三个任务：

* 读取文本文件，为每个单词的出现计数。
* 将单词列表按词频递减排序。
* 打印排序后的列表的前n项。

下面是完整的程序：

{% highlight lua %}
local function allwords()
    local auxwords = function()
        for line in io.lines() do
            for word in string.gmatch(line, '%w+') do
                coroutine.yield(word)
            end
        end
    end
    return coroutine.wrap(auxwords)
end

local counter = {}
for w in allwords() do
    counter[w] = (counter[w] or 0) + 1
end

local words = {}
for w in pairs(counter) do
    words[#words+1] = w
end

table.sort(words, function(w1, w2)
    return counter[w1] > counter[w2] or
            counter[w1] == counter[w2] and w1 < w2
    end)

for i = 1, (tonumber(arg[1]) or 10) do
    print(words[i], counter[words[i]])
end
{% endhighlight %}

## 马尔可夫链算法

最后一个示例是**马尔可夫链算法**的实现，程序以原文的前n个单词串为基础，产生一个伪随机的文本串。这里假设n等于2.

程序的第一部分读取原文，并且为每两个单词的前缀构建一个table，这个表给出了具有那些前缀的单词的一个顺序。建表完成后，这个程序利用这张表生成一个随机的文本。在此文本中，每个单词都随着它的前两个单词，这两个单词在文本中有相同的概率。这样，我们就产生了一个非常随机，介并不完全随机的文本.

下面的函数用来将两个单词用空格连接起来：

{% highlight lua %}
function prefix(w1, w2)
    return w1 .. ' ' .. w2
end
{% endhighlight %}

我们用*NOWORD*（一个新行）表示文件的结尾并且初始化前缀单词。例如，对文本“the more we try the more we do”，构造的表如下：

{% highlight lua %}
{  ['\n \n'] = {'the'},
   ['\n the'] = {'more'},
   ['the more'] = {'we', 'we'}
   ['more we'] = {'try', 'do'}
   ['we try'] = {'the'}
   ['try the'] = {'more'}
   ['we do'] = {'\n'}
{% endhighlight %}

程序用变量`statetab`保存这个表。在这个表的前缀列表中添加一个单词，用下面的函数：

{% highlight lua %}
function insert(index, value)
    local list = statetab[index]
    if list == nil then
        statetab[index] = {value}
    else
        list[#list + 1] = value
    end
end
{% endhighlight %}

它首先检查该前缀是否在这个列表中，如果没有，它用新的值创建一个新项。否则，它把新的值添加到列表尾部。

我们使用两个变量w1和w2来保存最后读入的两个单词的值，对于每一个前缀，我们保存紧跟其后的单词列表。例如上面示例中初始化构造的表。

下面看完整的代码：

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
                line = io.read()
                pos = 1
            end
        end
        return nil
    end
end

function prefix(w1, w2)
    return w1 .. ' ' .. w2
end

local statetab = {}

function insert(index, value)
    local list = statetab[index]
    if list == nil then
        statetab[index] = {value}
    else
        list[#list + 1] = value
    end
end

local N = 2
local MAXGEN = 10000
local NOWORD = '\n'

local w1, w2 = NOWORD, NOWORD
for w in allwords() do
    insert(prefix(w1, w2), w)
    w1 = w2; w2 = w;
end

w1 = NOWORD; w2 = NOWORD
for i = 1, MAXGEN do
    local list = statetab[prefix(w1, w2)]
    local r = math.random(#list)
    local nextword = list[r]
    if nextword == NOWORD then return end
    io.write(nextword, ' ')
    w1 = w2; w2 = nextword
end
{% endhighlight %}
