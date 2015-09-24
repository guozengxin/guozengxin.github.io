---
layout: post
title: "lua学习笔记(9)--协程"
description: "lua学习笔记，第九章，协程"
category: lua
tags: [Programming in Lua, 协程, Coroutines, 线程]
comments: true
---

协程和多线程的概念类似：拥有自己的堆栈，自己的局部变量，自己的指令指针，并且和其它协程共享全局变量等信息。线程和协程的区别在于：在多处理器的情况下，理论上多线程程序是在多个处理上并行运行；而协程是协作的：同一时间只有一个协程在运行，并且这个协程只有明确要求被挂起时，它才会挂起。

协程是一个强大的概念，但是它的应用场景可能会很复杂。python语言也可以实现协程。

## 协程基础

Lua把所有协程相关的函数都放在`coroutine`表中。`create`函数用于创建新的协程，它有一个参数，传递一个用于执行协程的函数。它返回一个`thread`类型的值，代表新的协程。通常，`create`函数的参数都是匿名函数：

{% highlight lua %}
co = coroutine.create(function()
        print('hi')
    end)
print(co)           --> thread: 0x1214180
{% endhighlight %}

<!-- more -->

协程拥有4个状态：挂起(suspended)、运行(running)、停止(dead)和正常(normal)。我们可以用`coroutine.status`来检测状态：

{% highlight lua %}
print(coroutine.status(co))   --> suspended
{% endhighlight %}

创建一个协程时，它不会自动开始运行，处于**挂起**状态。函数`coroutine.resume`可以开始（或者恢复）协程的运行，将状态从**挂起**转为**运行**：

{% highlight lua %}
coroutine.resume(co)            --> hi
{% endhighlight %}

在这个示例中，协程简单的打印"hi"之后就结束了，协程从**运行**状态转为**停止**：

{% highlight lua %}
print(coroutine.status(co))     --> dead
{% endhighlight %}

到现在为止，协程看上去只是简单的函数调用。它真正的威力从`coroutine.yield`函数开始，一个正在运行的协程可以用这个函数让自己从**运行**状态转为**挂起**状态。看这个示例：

{% highlight lua %}
co = coroutine.create(function()
        for i = 1, 10 do
            print('co', i)
            coroutine.yield()
        end
    end)
{% endhighlight %}

现在当我们恢复这个协程时，它开始运行直到遇到`yield`函数：

{% highlight lua %}
coroutine.resume(co)        --> co   1
{% endhighlight %}

现在我们检测状态，它的状态变为**挂起**，那么它就可以用`resume`函数恢复了：

{% highlight lua %}
print(coroutine.status(co))           --> suspended
{% endhighlight %}

从协程的观点看：协程使用`yield`函数可以使自己挂起，当我们激活被挂起的程序时，`yield`返回并且继续执行直到遇到下一个`yield`：

{% highlight lua %}
coroutine.resume(co)        --> co   2
coroutine.resume(co)        --> co   3
...
coroutine.resume(co)        --> co   10
coroutine.resume(co)        -- print nothing
{% endhighlight %}

在最后一次调用`resume`时，协程函数完成循环并返回，没有任何输出。但如果我们再一次调用`resume`，将会返回**false**并打印错误信息：

{% highlight lua %}
print(coroutine.resume(co))      --> false     cannot resume dead coroutine
{% endhighlight %}

注意`resume`函数在保护下运行。因此，如果协程函数内部出错，Lua将不会抛出错误，而是将错误返回给`resume`调用。

当在一个协程(协程1)中`resume`另一个协程(协程2)时，协程1没有转为挂起态，毕竟我们不能`resume`协程1。然而，协程1也不是运行状态，因为正在运行的是协程2。因此，协程1的状态称为正常(normal)状态。

Lua中的一对resume-yield可以交换数据。对一个协程第一次调用`resume`函数时，可以给协程的主函数传递参数：

{% highlight lua %}
co = coroutine.create(function(a, b, c)
        print('co', a, b, c + 2)
    end)
coroutine.resume(co, 1, 2, 3)       --> co  1  2  5
{% endhighlight %}

在`resume`函数结束时，除了返回true以外，`yield`的其它参数将作为返回值传给`resume`，作为`resume`的返回值：

{% highlight lua %}
co = coroutine.create(function(a, b)
        coroutine.yield(a + b, a - b)
    end)
print(coroutine.resume(co, 20, 10))             --> true  30  10
{% endhighlight %}

`resume`的额外参数也会传递给`yield`，作为`yield`的返回值。

{% highlight lua %}
co = coroutine.create(function(x)
        print('co1', x)
        print('co2', coroutine.yield())
    end)
coroutine.resume(co, 'hi')        --> co1  hi
coroutine.resume(co, 4, 5)        --> co2  4  5
{% endhighlight %}

当一个协程结束时，它的主函数返回的任何值都会传递给`resume`：

{% highlight lua %}
co = coroutine.create(function()
        return 6, 7
    end)
print(coroutine.resume(co))       --> true  6  7
{% endhighlight %}

通常不会在同一个协程中应用这些特性，但是它们都有其用处。

现在我们深入介绍一点协程的概念。Lua提供的是一个**非对称协程**，意思是它用一个函数来挂起协程，然后用一个不同的函数来恢复协程。一些其它的语言提供了**对称协程**，意味着只有一个函数控制挂起和运行状态。

有人称不对称的协程为**半协程**，然而另一些人用**半协程**表示严格意义上的协程，即只有在协程主函数中才能挂起。Python中的**产生器(generator)**就是一种**半协程**。

除了不对称协程和对称协程的区别，协程和产生器的区别更大。产生器相对简单，它不能完成真正的协程能完成的一些任务。我们熟练使用不对称的协程之后，可以它实现比较优越的协程。

## 管道和过滤器

协程的一个常用示例是“生产者-消费者”问题。假设有一个函数持续的产生值，另一个函数持续的消费值。这两个函数如下：

{% highlight lua %}
function producer()
    while true do
        local x = io.read()
        send(x)
    end
end

function consumer()
    while true do
        local x = receive()
        io.write(x, '\n')
    end
end
{% endhighlight %}

这里的问题是怎么是协同`send`和`receive`。生产者和消费者同时在运行，它们都自己主循环，并且都认为另一方是可调用的服务。对于这种情况，可以改变一个函数的结构解除循环，使其被动接受。但是这看起来很别扭，不符合逻辑。

协程提供了一种方法协调生产者和消费者，因为调用者和被调用者之间的resume-yield关系会不断颠倒。当协程调用*yield*，它不会进入一个新函数，反而会返回一个等待的*resume*；类似的，当调用*resume*时也不会进入一个新的函数，而是返回到*yield*的调用。这种特征正是生产者消费者问题中需要的，它可以使`send`和`receive`函数很的协作。`receive`唤醒生产者产生新值，`send`把产生的值送给消费者消费。

{% highlight lua %}
function receive()
    local status, value = coroutine.resume(producer)
    return value
end

function send()
    coroutine.yield(x)
end

producer = coroutine.create(
    function()
        while true do
            local x = io.read()
            send(x)
        end
    end)
{% endhighlight %}

在这个设计中，调用consumer来使程序开始。当消费者需要一个值时，调用*resume*恢复生产者，生产者产生一个值，然后直到消费者再一次恢复它。因此，我们得到一个**消费者驱动**的设计。另一种方式是**生产者驱动**的设计，那时消费者是一个协程。

现在我们用过滤器来扩展这个设计。过滤器可以在生产者和消费者之间，对数据进行一些转换处理。过滤器在同一时间既是生产者又是消费者，它恢复生产者来获取新值并且将转换过后的值传给消费者。下面看一下带过滤器的生产者和消费者示例：

{% highlight lua %}
function receive(prod)
    local status, value = coroutine.resume(prod)
    return value
end

function send(x)
    coroutine.yield(x)
end

function producer()
    return coroutine.create(function()
        while true do
            local x = io.read()
            send(x)
        end
    end)
end

function filter(prod)
    return coroutine.create(function()
        local line = 1
        while true do 
            local x = receive(prod)
            x = string.format('%5d %s', line, x)
            send(x)
            line = line + 1
        end
    end)
end

function consumer(prod)
    while true do
        local x = receive(prod)
        io.write(x, '\n')
    end
end

p = producer()
f = filter(p)
consumer(f)
{% endhighlight %}

在这个示例中，有两个协程，分别是生产者和过滤器。仔细分析这个示例，可以对*resume-yield*对之间的调用关系有更加深刻的理解。

## 迭代器

我们将迭代器看成是一种特殊的生产者-消费者示例：迭代器函数产生值，循环主体消费值。所以可以使用协程来实现迭代器。协程的一个关键特征是可以不断颠倒调用者和被调用者之间的关系。这样我们可以实现迭代器而不用担心怎么去保存迭代函数的状态。

为了说明迭代器，首先我们写一个迭代器来打印一个数组的所有排列。直接写这样一个迭代器并不容易，但是用递归来实现就比较简单。思路很简单：轮流把每个元素放到最后一个位置，然后递归的产生剩余元素的所有排列。代码如下所示：

{% highlight lua %}
function printResult()
    for i = 1, #a do
        io.write(a[i], ' ')
    end
    io.write('\n')
end

function permgen(a, n)
    if n == 0 then
        printResult(a)
    else
        for i = 1,n do
            a[n], a[i] = a[i], a[n]
            permgen(a, n-1)
            a[n], a[i] = a[i], a[n]
        end
    end
end

permgen({1, 2, 3, 4})
--> 2 3 4 1
--> 3 2 4 1
    ...
--> 1 2 3 4
{% endhighlight %}

有了上面的生成器之后，可以通过下面的步骤把它转换为一个迭代器。

第一，我们把`printResult`函数转换为`yield`:

{% highlight lua %}
function permgen(a, n)
    n = n or #a
    if n <= 1 then
        coroutine.yield(a)
    else
        ...
{% endhighlight %}

第二，我们定义一个工厂函数，让生成器在协程中运行，并且创建迭代器函数。迭代器只要*resume*协程就可以产生下一个置换：

{% highlight lua %}
function permutations(a)
    local co = coroutine.create(function() permgen(a) end)
    return function()
        local code, res = coroutine.resume(co)
        return res
    end
end
{% endhighlight %}

现在，`permutations`将返回一个迭代器函数，我们可以在for循环中来使用它：

{% highlight lua %}
for p in permutations({'a', 'b', 'c'}) do
    printResult(p)
end

--> b c a
    ...
--> a b c
{% endhighlight %}

`permutations`函数使用了Lua中常见的模式，将*resume*的调用封装到一个函数中。这种方式中Lua很常见，所以Lua中专门提供了一个函数`coroutine.wrap`。与*create*相同的是，*wrap*创建一个新的协程，与*create*不同的是，*wrap*不返回协程本身，而是返回一个函数，调用这个函数可以*resume*这个协程。和直接调用*resume*不同，`wrap`函数不会返回错误代码，而且抛出一个错误。我们使用`wrap`重写`permutations`函数：

{% highlight lua %}
function permutations(a)
    return coroutine.wrap(function() permgen(a) end)
end
{% endhighlight %}

通常，使用`coroutine.wrap`比使用`cotoutine.create`更简单，因为它直接给出我们需要从协程中得到的功能：一个可以*resume*的函数。但是缺少灵活性，不能知道协程的状态，也不能进行错误检查。

## 非抢占式多线程

从上面的示例可以看出，协程是一种协作式的多线程。每个协程相当于一个线程，每个*yield-resume*对相当于在线程间相互切换。然而，与常规的多线程不同，协程是非抢占式的。当协程正在运行时，不能从外部来停止它，只能协程自己需要挂起时，调用*yield*来挂起自己。在大多数应用中，这些没有任何问题,但是有些应用就不能忍受这样的问题。非抢占式的多线程程序容易编写，因为不用考虑复杂线程间的同步，因为线程的切换都是用*yield-resume*显式调用。

然而，对于非抢占式的线程来说，当调用一个阻塞操作时，整个程序都会被阻塞直到调用完成。在大多数应用中，这是不可接受的，这也是大多数人不愿意用协程的原因。这里，我们将提出一个有趣的方案来解决这个问题。

我们假设一个典型的多线程情景：通过HTTP协议下载远程文件。下载远程文件用到*LuaSocket*库。为了下载文件，我们必须新建一个连接到远程站点，然后发送一个请求，接收数据，最后关闭连接。在Lua中，我们可以用下面的代码：

首先，加载*LuaSocket*连接：

{% highlight lua %}
local socket = require 'socket'
{% endhighlight %}

然后，定义host和需要下载的文件：

{% highlight lua %}
host = 'www.w3.org'
file = '/TR/REC-html32.html'
{% endhighlight %}

然后，打开一个TCP连接：

{% highlight lua %}
c = assert(socket.connect(host, 80))
{% endhighlight %}

然后用上面建立的连接发送请求：

{% highlight lua %}
c:send('GET ' .. file .. ' HTTP/1.0\r\n\r\n')
{% endhighlight %}

接下来，我们按块接收文件，每次读1KB，将结果输出：

{% highlight lua %}
while true do
    local s, status, partial = c:receive(2^10)
    io.write(s or partial)
    if status == 'closed' then break end
end
{% endhighlight %}

`receive`函数会返回接收到的字符串（或者出错时返回nil和错误代码`status`）。当连接关闭时，跳出循环。

最后，关闭这个连接：

{% highlight lua %}
c:close()
{% endhighlight %}

现在我们知道了如何下载一个文件，现在让我们回到下载多个文件的问题。我们当然不是顺序下载。实际上，我们发送请求之后，大部分的时间都是在等待`receive`。如果可以同时下载多个文件，效率将会大大提高。当一个连接没有数据到达时，可以从另一个连接读取数据。显然，协程为这种同时下载提供了很方便的支持，我们为每一个下载任务创建一个线程，当一个线程没有数据到达时，它将控制权交给一个分配器，由分配器唤起另外的线程读取数据。

使用协程重写上面的代码：

{% highlight lua %}
function download(host, file)
    local c = assert(socket.connect(host, 80))
    local count = 0
    c:send('GET ' .. file .. ' HTTP/1.0\r\n\r\n')
    while true do
        local s, status = receive(c)
        count = count + #s
        if status == 'closed' then break end
    end
    c:close()
    print(file, count)
end
{% endhighlight %}

在这个代码中，我们使用了一个辅助函数`receive`从连接接收数据。串行接收数据方法如下：

{% highlight lua %}
function receive(connection)
    local s, status, partial = connection:receive(2^10)
    return s or partial, status
end
{% endhighlight %}

在协程的实现中，这个函数必须非阻塞的接收数据。在没有数据可接收时，调用yield:

{% highlight lua %}
function receive(connection)
    connection:settimeout(0)         -- 非阻塞
    local s, status, partial = connection:receive(2^10)
    if status == 'timeout' then
        coroutine.yield(connection)
    end
    return s or partial, status
end
{% endhighlight %}

调用`settimeout(0)`使连接的任何操作都为非阻塞的。当返回状态是*timeout*时，意味着操作没结束就返回了。在这种情况下，线程挂起(yield)。*yield*的参数非负，告诉分配器接收任务还没有完成。注意在timeout模式下，连接依然返回它接收到timeout为止，因此`receive`会一直返回s给它的调用者。

下面的代码是分配器加上一个辅助代码。`threads`表为分配器保存所有活动的线程。函数`get`保证了每个下载在一个单独的线程运行。分配器本身维持了一个循环，轮流*resume*每个线程，并且还要负责移除已经完成线程。当所有任务结束后，分配器结束。

{% highlight lua %}
threads = {}

function get(host, file)
    local co = coroutine.create(funciton()
        download(host, file)
    end)
    table.insert(threads, co)
end

funciton dispatch()
    local i = 1
    while true do
        if threads[i] == nil then
            if threads[1] == nil then break end
            i = 1
        end
        local status, res = coroutine.resume(threads[i])
        if not res then
            table.remove(threads, i)
        else
            i = i + 1
        end
    end
end
{% endhighlight %}

下面是主程序：

{% highlight lua %}
host = 'www.w3.org'

get(host, '/TR/html401/html40.txt')
get(host, '/TR/2002/REC-xhtml1-20020801/xhtml1.pdf')
get(host, '/TR/REC-html32.html')

dispatch()
{% endhighlight %}

在上面的函数中，当没有数据要接收时，分配器将进入忙等待状态，在线程之间不停的判断是否有数据可接收。上面的代码还可以进行优化，用*LoaSocket*库中的`select`函数。当在一组socket中不断的循环等待状态改变时，它使程序被阻塞。我们只需要修改分配器，使用`select`函数修改后的代码如下：

{% highlight lua %}
function dispatcher()
    while true do
        local n = table.getn(threads)
        if n == 0 then break end
        local connections = {}
        for i = 1, n do
            local status, res = coroutine.resume(threads[i])
            if not res then
                table.remove(threads, i)
                break
            else
                table.insert(connections, res)
            end
        end
        if table.getn(connections) == n then
            socket.select(connections)
        end
    end
end
{% endhighlight %}

这个分配器不会使程序一直循环检查连接，消耗CPU较小。
