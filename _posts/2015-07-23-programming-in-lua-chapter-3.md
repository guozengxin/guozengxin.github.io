---
layout: post
title: "lua学习笔记(3)--表达式"
description: "lua学习笔记，第三章，表达式"
category: lua
tags: [programming in lua, lua, 表达式, 表的构造, table]
comments: true
---

## 算术运算符

Lua支持以下算术运算符：

* `+-*/`: 加减乘除
* `^`: 幂运算
* `%`: 模运算
* `-`: 负号

### 实数的模运算

Lua中，所有的算术操作符都是用于实数，所以求模操作`%`就会有特殊的含义。例如：`x%1`的结果是x的小数部分，`x - x%1`就是x的整数部分，类似的，`x - x%0.01`的结果是将x精确到2位小数：

```lua
x = math.pi
print(x - x%0.01)          --> 3.14
```

<!-- more -->

## 关系运算符

Lua提供了以下关系运算符：

	<    >    <=    >=    ==    ~=

所有这些运算符的结果都是Boolean类型。

`==`和`~=`这两个运算符的操作数可以是任何类型，如果两个值的类型不同，那么Lua认为是不相等的。另外，nil只和它自己相等。

Lua用引用来比较table和userdata，即如果两个引用指向同一个对象，Lua认为它们相等，否则它们不相等。

```lua
a = {}; a.x = 1; a.y = 0
b = {}; b.x = 1; b.y = 0
c = a
print(c == a)              --> true
print(b == a)              --> false
```

## 逻辑运算

Lua中的逻辑运算符是`and`, `or`和`not`。逻辑运算符两边的值也可以是任意类型，Lua中认为只有`false`和`nil`这两个值是False，其他的都是真。

三个运算符的逻辑：

* `and`: 如果第一个值为false, 则返回第一个值；否则，返回第二个值。
* `or`: 如果第一个值不是false, 则返回第一个值。否则，返回第二个值。
* `not`: 返回true或者false。

像C语言一样(大多数语言都是这样)，`and`和`or`这两个操作符使用短路计算，即只有在需要的时候才会计算第二个值。如`true or 3 > "3"`会返回true，而`false or 3 > "3"`会报类型不匹配的错误，因为第一个式子没有计算and右边的表达式。

一个常用的表达是`x = x or v`，这个式子和下面的逻辑相等：

```lua
if not x then x = v end
```

另一个常用的表达式是：`a and b or c`(在python中也可以这么用)，在**b**不为false的时候，它和C语言中的`a ? b : c`具有相同的作用。在`a and b or c`中，如果**a**为true，那么`a and b`的值是**b**，而**b**不为false，所以`b or c`的值是**b**; 当**a**为false, 那么`a and b`的值是**a**，当然`a or c`的值就为**c**.

## 连接运算符

连接运算符是`..`，用于连接两个字符串，如果操作数是数字，Lua会将数字转换为string：

```lua
print("Hello" .. "World")     --> Hello World
print(0 .. 1)                 --> 01
print(000 .. 01)              --> 01
```

## 长度操作

长度操作符`#`用于string和table，如果是string，它得出string长度，如果是table，它给出table中**序列**的长度（**序列**是指table中以下标1开始，到第一个值不为nil的串）：

```lua
print(a[#a])            --> 打印序列a的最后一个元素
a[#a] = nil             --> 删除最后一个元素
a[#a + 1] = v           --> 在a最后添加值v
```

## 优先级

Lua中操作符的优先级如下(从上到下优先级降低)：

	^
	not    #    -(负号)
	*      /    %
	+      -
	..
	<      >    <=    >=    ~=    ==
	and
	or

除了`^`和`..`外，所有的二元运算符都是左连接的。

## 表的构造

在Lua中，表是用构造函数来构造的，最简单的构造器是`{}`，它创建了一个空表。也可以直接初始化数组：

```lua
days = {"Sunday", "Monday", "Tuesday", "Wednesday",
        "Thusday", "Friday", "Saturday"}
```

Lua中默认数组下标是从1开始的。

Lua也提供了一个特殊的语法来将table作为record使用：

```lua
a = {x = 0, y = 0}
```

不管用什么方构造的table，都可以对表进行添加和删除元素：

```lua
w = {x = 0, y = 0, label="console"}
x = {math.sin(0), math.sin(1), math.sin(2)}
w[1] = "another field"                   --> add key 1
x.f = w                                  --> add key "f"
print(w['x'])                            --> 0
print(w[1])                              --> another field
print(x.f[1])                            --> another field
w.x = nil                                --> remove field 'x'
```

也可以在同一个构造函数中同时初始化list和record：

```lua
polyline = {color = "blue",
            thickness = 2,
            p = 4,
            {x = 0, y = 0},
            {x = -10, y = 0},
            {x = -10, y = 1},
            {x = 0, y = 1}
}
print(polyline[2].x)                 --> -10
print(polyline[4].y)                 --> 1
```

上面的初始化方式还有限制，如不能使用负索引来初始化一个表中的元素，有些字符串也不能用做索引。下面看一种通用的方式，用`[expression]`来初始化：

```lua
opnames = {['+'] = 'add', ['-'] = 'sub',
           ['*'] = 'mul', ['/'] = 'div'}
a = {[-1] = '-1'}
print(a[-1])              --> -1
```

还可以在构造的时候用分号代替逗号，通常用分号也分隔不同的数据段：

```lua
{x = 10, y = 45; 'one', 'two', 'three'}
```
