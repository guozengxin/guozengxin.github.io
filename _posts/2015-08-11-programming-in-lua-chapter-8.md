---
layout: post
title: "lua学习笔记(8)--编译、运行和异常"
description: "lua学习笔记，第八章，编译、运行和异常"
category: lua
tags: [programming in lua, 编译, 运行, 异常]
comments: true
---

虽然Lua是解释型语言，但是Lua总是把代码编译成中间代码然后执行。解释型语言的特征不在于它是否被编译，而是编译器是语言运行时的一部分。Lua执行编译产生的中间代码运行速度会更快。

## 编译

前面讲过`dofile`可以运行chunk，但是`dofile`只是一个辅助操作，真正运行的是`loadfile`。`dofile`加载并运行chunk，但是`loadfile`只是从文件中加载chunk，并不执行代码，仅仅是对chunk进行编译并作为一个函数返回。`dofile`和`loadfile`的区别是：`loadfile`不抛出异常，但是返回错误码。相反，`dofile`会抛出异常，我们可以这样定义`dofile`：

```lua
function dofile(filename)
    local f = assert(loadfile(filename))
    return f()
end
```

对于普通任务，`dofile`足够使用，因为当加载失败时它会告诉我们错误信息。但是`loadfile`更灵活，如果加载错误，我们可以定义自己的方式来处理错误。并且，如果我们需要多次执行一个文件，可以用`loadfile`加载一次，然后多次调用结果。但是每次调用`dofile`都需要编译。

<!-- more -->

`loadstring`函数和`loadfile`函数很相似，不同的是`loadstring`从一个字符串中加载一个chunk，不是从一个文件。如：

```lua
f = loadstring('i = i + 1')
i = 0
f()
print(i)            --> 1
f()
print(i)            --> 2
```

`loadstring`函数很强大，但是需要多加小心，它可能会带来很多诡异的问题。所以如果你确定没有更多简单的方法时再使用它。

如果需要快速加载并运行一个字符串（像`dostring`函数一样）,可以如下使用：

```lua
loadstring(s)()
```

然而，在上面的代码中，如果加载的string中有语法错误，`loadstring`将返回一个空值，并且错误信息会变成**attempt to call a nil value**，下面的代码可以打印出详细的错误：

```lua
assert(loadstring(s))()
```

通常，使用`loadstring`来加载字符串没有什么意义，例如：

```lua
f = loadstring('i = i + 1')

f = function()
    i = i + 1
end
```

这两种表示方式功能上没有什么区别，但是第二种函数定义更快，因为使用`loadstring`多调用了一次编译。

由于`loadstring`没有使用词法定界来编译，上述两种表示方式并不是完全相同：

```lua
i = 32
local i = 0
f = loadstring('i = i + 1; print(i)')
g = function() i = i + 1; print(i); end
f()             --> 33
g()             --> 1
```

函数`g`操作了局部变量，但是`f`却使用的是全局变量，是因为`loadstring`始终在全局环境下编译chunk。

`loadstring`函数返回的是一个普通函数，所以你可以多次调用它：

```lua
print 'enter function to be plotted (with variable "x"):'
local l = io.read()
local f = assert(loadstring('return ' .. l))
for i = 1, 20 do
    x = i         -- global 'x' (to be visible from the chunk)
    print(string.rep('*', f()))
end
```

`loadstring`, `loadfile`函数没有副作用，它们仅仅编译源数据，并将结果作为匿名函数返回。一个通常的错误是认为加载chunk定义了函数, 在Lua中，函数定义是赋值操作，它发生在运行阶段，而不是编译阶段。假设有如下文件`foo.lua`：

```lua
-- file 'foo.lua'
function foo(x)
    print(x)
end
```

然后我们执行代码：

```lua
f = loadfile('foo.lua')
```

执行这段代码之后，`foo`被编译了，但是没有被定义，我们只有执行chunk之后才会被定义：

```lua
print(foo)      --> nil
f()
foo('ok')       --> ok
```

## 预编译代码

Lua会在执行代码之前进行预编译，也允许我们发布预编译的代码。

最简单的方式创建预编译文件（也叫二进制chunk）的方法是用`luac`程序，这个是在Lua安装中自带命令。例如，下面的命令将会创建一个新的文件`prog.lc`，这个文件是`prog.lua`的预编译版本：

```bash
$ luac -o prog.lc prog.lua
```

像普通的lua文件一样，预编译文件也可以用**lua**命令执行：

```bash
$ lua prog.lc
```

Lua在任何可以加载源码的地方都可以加载预编译代码，如，`loadfile`函数.

也可以直接把luac代码直接写入文件：

```lua
p = loadfile(arg[1])
f = io.open(arg[2], 'wb')
f:write(string.dump(p))
f:close()
```

`string.dump`函数接收一个Lua函数然后以string的方式返回它的预编译代码。

预编译代码不一定比源代码小，但是加载更快。另一个好处是可以避免对源代码的意外改动。然而，恶意的二进制代码有可能会造成Lua编译器崩溃，所以在使用不信任的预编译代码时需要谨慎。

## C代码

C代码在运行前需要加载应用并链接，在大部分的操作系统上，常见的链接方式是动态链接机制。然而，这一机制不是`ANSI C`的规范，因此没有简单的方式莱实现。

通常，Lua不会包含那些在`ANSI C`中无法实现的机制。然而，动态链接机制是例外，可以把动态链接看成是其他所有机制之母，我们可以用它来动态的加载那些原来在Lua不存在的机制。因此Lua打破了它的规范实现了动态链接机制。

Lua可以用一个函数`package.loadlib`实现了所有的动态链接功能，它有两个字符串参数：lib的完整路径和lib中的一个函数名称。它的调用如下所示：

```lua
local path = '/usr/local/lib/lua/5.1/socket.so'
local f = package.loadlib(path, 'luaopen_socket')
```

`package.loadlib`函数加载指定的库并链接到Lua，但是它没有调用指定的函数。相反它返回这个C函数作为一个Lua函数。如果加载出错，则会返回nil和一个错误信息。

`loadlib`函数是一个非常底层的函数，必须提供库的全路径和正确的函数名称（有时候需要包含编译器加上的下划线等字符）。通常，我们使用`require`函数来加载C库。关于`require`函数的用法，这里不做讨论。

## 错误

由于Lua经常会作为扩展嵌入在别的语言中，因此当错误发生时不能简单的退出或者崩溃。相反，当错误发生需要结束当前chunk然后返回到应用中，必要时提供错误信息。

当Lua遇到不期望的情形时就会抛出一个错误，如对非数字进行相加，尝试调用一个非函数类型的值，等等。你也可以调用`error`函数明确的产生一个错误信息：

```lua
print 'enter a number:'
n = io.read('*n')
if not n then error('invalid input') end
```

Lua有另一个内置的函数`assert`来更方便的实现上面的逻辑：

```lua
print 'enter a number:'
n = assert(io.read('*n'), 'invalid input')
```

`assert`函数检测它的第一个参数，如果不为false，则返回这个参数，如果为false，则assert抛出一个错误。它的第二个参数是可选的，表示错误信息。同样的，Lua也是一个普通函数，它总是在执行函数之前计算它的参数的表达式的值，因此：

```lua
n = io.read()
assert(tonumber(n), 'invalid input:' .. n .. 'is not a number')
```

Lua将总是执行这个连接操作。所以在这个示例中，先进行明确判断n的值是一个好的方法。

当一个函数遇到异常情况时，有两种基本的方法可以处理：一是返回一个错误代码（通常是nil），二是抛出一个错误。然而没有明确的规定来指明该用哪种方法，但是有一个通用的判断方法：当一个异常可以轻易避免时，抛出一个错误，否则，则返回一个错误代码。

例如，我们考虑`sin`函数，如果用一个table作为参数，假定我们返回错误代码，我们需要手动检测错误的发生：

```lua
local res = math.sin(x)
if not res then                -- error
    ...
```

我们也可以在调用函数前检测参数：

```lua
if not tonumber(x) then           -- x is not a number
    ...
```

但是通常我们在使用`sin`函数的时候既不判断参数也不判断返回值，这时很可能我们就不会发现错误的发生。所以在`sin`函数中抛出一个错误，并结束程序的运行，是一个很好的方式。

下面，我们看另一个函数`io.open`，用于打开文件，但是文件不存在时会怎么样呢？很多系统中，我们通常都用open函数来判断文件是否存在。所以如果`io.open`不能打开文件时，返回nil和错误信息（“文件不存在”或者“没有权限”）。这种情况下，我们可以手动的来处理错误：

```lua
local file, msg
repeat
    print 'enter a file name:'
    local name = io.read()
    if not name then return end           --no input
    file, msg = io.open(name, 'r')
    if not file then print(msg) end
until file
```

或者：

```lua
file = assert(io.open(name, 'r'))
```

## 错误处理和异常

在大多数Lua应用中，我们不需要进行错误处理；应用程序已经做了这样的处理。通常应用调用Lua来执行一个chunk，如果发生异常，应用根据Lua返回的错误进行处理。在控制台应用中，执行Lua代码遇到错误时，打印出错误并继续等待下一个指令。

如果需要在Lua中处理错误，需要使用`pcall`函数封装代码。

假如你想执行一段Lua代码，在这段代码中可以处理遇到的所有错误和异常。第一步是将这段代码封装到一个函数中；通常使用一个匿名函数。然后用`pcall`调用这个函数：

```lua
local ok, msg = pcall(function()
        <some code>
        if unexpected_condition then error() end
        <some code>
        print(a[i])                          -- a可能不是一个table
        <some code>
    end)
if ok then           -- 没有错误
    ...
else                 -- 有错误
    ...
end
```

`pcall`函数在**保护模式**下调用函数，它能捕捉运行时的任何错误。当没有错误发生时，`pcall`返回**true**，加上函数调用返回的值；否则，返回**false**，加上错误信息。

还有一点，**错误信息**并不一定是字符串，它可以是任何值：

```lua
local status, err = pcall(function()
        error({code = 121})
    end)
print(err.code)              --> 121
```

## 错误信息和回溯

虽然错误信息可以是任何类型的值，但是通常情况我们都用字符串来描述发生了什么错误。如果遇到内部错误，Lua会自己产生错误信息；否则，使用**error**函数的参数作为错误信息：

```lua
local status, err = pcall(function()
        a = 'a' + 1
    end)
print(err)   --> err3.lua:2: attempt to perform arithmetic on a string value
local status,err = pcall(function()
        error('my error')
    end)
print(err)   --> err3.lua:7: my error
```

可以看到，打印出的错误信息给出了文件名（err3.lua）和行号。

`error`函数还有第二个参数，表示错误的运行级别。这个参数用来指明应该为这个错误负责的代码（即应该改正的代码）：

```lua
function foo(str)
    if type(str) ~= 'string' then
        error('string expected')
    end
end
```

当调用`foo({x=1})`时，函数会报错并指明在**foo**中出错，然而真正的应该肇事者应该是调用者。为了指明真正出错的地方，可以在`error`函数用**level 2**（level 1是默认值，表示当前函数）：

```lua
function foo(str)
    if type(str) ~= 'string' then
        error('string expected', 2)
    end
end
```

当程序出错时，我们通常需要更多的信息，而不仅仅是出错的位置。至少我们需要一个回溯，来显示出错之前的整个调用栈。而*pcall*返回错误信息时会破坏调用栈。所以，如果我们想回溯，就必须在*pcall*返回之前来实现。Lua提供了另一个函数来实现这个功能：`xpcall`。这个函数可以接收第二个参数：一个错误处理函数，Lua会在调用栈释放之前执行这个函数，因此可以使用debug库收集错误相关的信息。有两个常用的错误处理函数：`debug.debug`，可以在此处中断并打开Lua提示符，然后手动查看错误发生时的情况；`debug.traceback`，可以创建更多的错误信息。
