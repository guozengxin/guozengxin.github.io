---
layout: post
title: "python语言中struct模块pack和unpack介绍"
description: "python语言中可以使用struct模块对二进制数据进行操作"
category: python
tags: [python, struct, pack, unpack]
---
{% include JB/setup %}

### OverView

Python语言风格比较简洁，在数据类型的定义上也只有六种：**字符串(str)**，**整数(int)**，**浮点数(float)**，**元组(tuple)**，**列表(list)**，**字典(dict)**。通常使用中，我们可以用这六种类型完成大部分工作。但是有时候需要进行与其它语言的交互，或者存取文件，网络传输时，需要对二进制数据进行操作，这时候，可以使用**struct**模块来完成。

struct模块中有三个比较重要的函数：

1. `struct.pack(fmt, v1, v2, ...)`
> 按照给定的格式(fmt)，把数据封装成字符串(实际上是类似于c结构体的字节流)

2. `struct.unpack(unpack(fmt, string))`
> 按照给定的格式(fmt)解析字节流string，返回解析出来的tuple

3. `calcsize(fmt)`
> 计算给定的格式(fmt)占用多少字节的内存

<!-- more -->

### struct中支持的format格式

<style type="text/css">
	table,td,th {
		border:2px solid gray;
	}
</style>

<table >
   <tr>
      <th>Format</th>
      <th>C Type</th>
      <th>Python type</th>
      <th>Standard size</th>
   </tr>
   <tr>
      <td>x</td>
      <td>pad byte</td>
      <td>no value</td>
      <td> </td>
   </tr>
   <tr>
      <td>c</td>
      <td>char</td>
      <td>string of length 1</td>
      <td>1</td>
   </tr>
   <tr>
      <td>b</td>
      <td>signed char</td>
      <td>integer</td>
      <td>1</td>
   </tr>
   <tr>
      <td>B</td>
      <td>unsigned char</td>
      <td>integer</td>
      <td>1</td>
   </tr>
   <tr>
      <td>?</td>
      <td>_Bool</td>
      <td>bool</td>
      <td>1</td>
   </tr>
   <tr>
      <td>h</td>
      <td>short</td>
      <td>integer</td>
      <td>2</td>
   </tr>
   <tr>
      <td>H</td>
      <td>unsigned short</td>
      <td>integer</td>
      <td>2</td>
   </tr>
   <tr>
      <td>i</td>
      <td>int</td>
      <td>integer</td>
      <td>4</td>
   </tr>
   <tr>
      <td>I</td>
      <td>unsigned int</td>
      <td>integer</td>
      <td>4</td>
   </tr>
   <tr>
      <td>l</td>
      <td>long</td>
      <td>integer</td>
      <td>4</td>
   </tr>
   <tr>
      <td>L</td>
      <td>unsigned long</td>
      <td>integer</td>
      <td>4</td>
   </tr>
   <tr>
      <td>q</td>
      <td>long long</td>
      <td>integer</td>
      <td>8</td>
   </tr>
   <tr>
      <td>Q</td>
      <td>unsigned long long</td>
      <td>integer</td>
      <td>8</td>
   </tr>
   <tr>
      <td>f</td>
      <td>float</td>
      <td>float</td>
      <td>4</td>
   </tr>
   <tr>
      <td>d</td>
      <td>double</td>
      <td>float</td>
      <td>8</td>
   </tr>
   <tr>
      <td>s</td>
      <td>char[]</td>
      <td>string</td>
      <td> </td>
   </tr>
   <tr>
      <td>p</td>
      <td>char[]</td>
      <td>string</td>
      <td> </td>
   </tr>
   <tr>
      <td>P</td>
      <td>void *</td>
      <td>integer</td>
      <td> </td>
   </tr>
</table>
> 注1.q和Q只在机器支持64位操作时有意思

> 注2.每个格式前可以有一个数字，表示个数

> 注3.s格式表示一定长度的字符串，4s表示长度为4的字符串，但是p表示的是pascal字符串

> 注4.P用来转换一个指针，其长度和机器字长相关

> 注5.最后一个可以用来表示指针类型的，占4个字节

一个format字符串是用数字和上述定义的字符来表示的。如`5i`表示5个**int**型数，`H`表示一个**unsigned short**类型的数，`0Q`则表示0个**unsigned long long**类型的数。在上面的表格中，如果表示了字符代表的位数，则这个字符前面再加数字，表示多个变量，如`3Q`表示3个变量；如果没有指明字符代表的位数，则前面加数据代表一个变量，如`3s`表示一个长度为3的字符串，在pack或者unpack的时候需要注意这一点。

### struct支持的数据对齐方式

在与其它语言进行数据交互时，通常还要考虑的事情是：编译器是否有字节对齐？大端存储还是小端存储？在format字段串的第一个字符可以用来定义数据的对齐方式：

<table>
   <tr>
      <td>Character</td>
      <td>Byte order</td>
      <td>Size</td>
      <td>Alignment</td>
   </tr>
   <tr>
      <td>@</td>
      <td>native</td>
      <td>native</td>
      <td>native</td>
   </tr>
   <tr>
      <td>=</td>
      <td>native</td>
      <td>standard</td>
      <td>none</td>
   </tr>
   <tr>
      <td><</td>
      <td>little-endian</td>
      <td>standard</td>
      <td>none</td>
   </tr>
   <tr>
      <td>></td>
      <td>big-endian</td>
      <td>standard</td>
      <td>none</td>
   </tr>
   <tr>
      <td>!</td>
      <td>network (= big-endian)</td>
      <td>standard</td>
      <td>none</td>
   </tr>
</table>
<br/>
<br/>

### 使用示例

#### 示例1: C结构体打包和解包

一个C结构体：

{% highlight c++ %}
struct Header {
    unsigned short id;
    char[4] tag;
    unsigned int version;
    unsigned int count;
}
{% endhighlight %}

现在接收到一个上述结果体数据，可以用unpack函数解析：

{% highlight python %}
import struct
(id, tag, version, count) = struct.unpack('!H4s2I', s)
{% endhighlight %}

在上述代码中，H对应id，4s对应tag，2I对应version和count。!表示是由网络字节顺序传输。

这样，通过unpack，就解出对应的信息。同样，通过pack也可以将信息打包也对应的二进制格式：

{% highlight python %}
import struct
ss = struct.pack("!H4s2I", id, tag, version, count);
{% endhighlight %}

pack函数将信息打包成一个字符串，实际上是类C结果体字节流，表示的就是一个Header结构体。

#### 示例2: 格式化类型和二进制的转换

{% highlight python %}
import struct
a=12.34
#将a变为二进制
bytes=struct.pack('i',a)
{% endhighlight %}

pack后，bytes就是一个str字符串，内容与a的二进制存储内容相同

可以用如下代码进行反转换:

{% highlight python %}
(a,) = struct.unpack('i', bytes)
{% endhighlight %}

注意：unpack返回的是一个tuple。

### 可能遇到的错误

1. struct.pack时可能会遇到：
> struct.error: pack requires exactly 2 arguments

这一错误说明format中的参数个数与实际输入的参数个数不符。如`bytes = struct.pack('2I', a)`需要2个参数而只输入了一个

2. struct.unpack时可能会遇到：
> struct.error: unpack requires a string argument of length \*

这一错误说明format中参数代表的数据占用内存数，与输入的数据占用的内存长度不符。这时候可以用`struct.calcsize(fmt)`函数检查format字符代表的长度，及用len(string)函数来检测数据的长度。

### 参考

1. python struct: <http://docs.python.org/2/library/struct.html>
