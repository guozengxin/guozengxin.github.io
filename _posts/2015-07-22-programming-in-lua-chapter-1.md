---
layout: post
title: "lua学习笔记(1)--入门"
description: "lua学习笔记，第一章"
category: lua
tags: [programming in lua, lua, chunk]
comments: true
---

## 入门示例

### Hello World

在lua交互模式下输入代码，或者将以下代码保存到文件中，然后用`lua a.lua`执行就可以输出结果。

```lua
print('Hello World!')
```

### 求阶乘

这是一个稍微复杂点的例子，输入一个数字然后求阶乘：

```lua
#!/usr/bin/env lua

function fact(n)
	if n < 0 then
		return 0
	elseif n == 0 then
		return 1
	else 
		return n * fact(n - 1)
	end
end

print("enter a number: ")
a = io.read('*n')
print(fact(a))
```

<!-- more -->

## Chunk

在lua中，chunk是指一个代码片段，每个lua文件是一个chunk，或者在交互模式下每行是一个chunk。

在lua中，语句可以用分号分隔，也可以不用。但是推荐单行中的多个语句用分号分隔。如：

```lua
a = 1
b = a * 2
a = 1 b = a * 2    -- ugly
a = 1; b = a * 2   -- recommend
```

在命令行下，python一样，输入`lua`可进入交互模式。在交互模式下每输入一行代码是一个chunk, 会被立即执行：

```lua
$ lua
Lua 5.1.4  Copyright (C) 1994-2008 Lua.org, PUC-Rio
> print('hello')
hello
> 
```

用`-i`命令可以执行一段代码后进行交互模式，这种方法在调试和测试的时候非常有用:

```bash
$ lua -i my.lua
```

用`dofile()`函数也可以达到同样的效果：

```lua
> dofile('my.lua')
```

`dofile()`函数在编写代码时很有用，打开两个窗口，一个窗口用编辑器写代码，另一个打开lua的交互模式，当编辑器中保存文件之后，可以在交互模式下执行`dofile('a.lua')`加载最新的代码用于调试。

## 语法

### 变量
lua的变量命名和c差不多，可以用数字、字母、下划线来做为变量名，开头不能为数字。

下面的单词是语言预留的，不能用做变量：

```lua
and		break	do		else	elseif
end		false	goto	for	function
if		in		local	nil	not
or		repeat	return	then 	true
until	while
```

### 注释

- 用双连接号`--`作为单行注释
- 用`--[[`和`--]]`作为双待注释

如：

```lua
-- comment
--[[
print(10);
--]]
```

## 全局变量

在lua中，全局变量不需要声明，只要去使用就可以了。而且直接使用一个没有初始化的变量是不会报错的：

```lua
print(b)  -- nil
b = 10
print(b)  -- 10
```

将`nil`赋值给变量可以销毁这个变量，lua后续会回收它占用的内存：

```lua
b = nil
print(b)  -- nil
```

## 运行

### 解释器

像其他脚本语言一样，lua也可以声明解释器，从而让lua程序可以像可执行程序一样自己执行。在文件的开头加上`#!/usr/bin/env lua`即可。

### lua命令

* 直接输入`lua`进入到交互模式

* `-e`选项可以执行单行程序：

		$ lua -e "print(2+3)"

* `-i`选项进入交互模式

* `-l`选项可以加载一个库

	如下面的命令将会加载lib库和执行`x = 10`这个语句，然后进入交互模式
		
		$ lua -i -llib -e "x = 10"

* 在交互模式下可以用以下的方式打印变量	
	
		> = 5     --> 5
		> a = 30
		> = a     --> 30 注意等号前面不能有空格

* lua脚本中获取命令行参数

	在lua脚本中，可以用`arg`变量获取命令行参数，索引0是程序名，索引1是第1个参数，等等。负的索引代码程序名前面的选项：
	
		$ lua -e "sin=math.sin" script a b
	
	lua会用如下方式获取上面的命令：

		arg[-3] = "lua"
		arg[-2] = "-e"
		arg[-1] = "sin=math.sin"
		arg[0] = "script"
		arg[1] = "a"
		arg[2] = "b"

## 小结

最近在学习《Programming in Lua》这本书，本文是第一章的学习笔记。后续会把每章的学习情况记录下来，加深理解。
