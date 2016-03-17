---
layout: post
title: "Hadoop Streaming添加自定义Counter"
description: "怎样在Hadoop Streaming应用中添加自定义Counter，就像在Java API中一样."
category: hadoop
tags: [hadoop, hadoop streaming, Counter]
comments: true
---

## 如何在Hadoop Streaming中添加自定义Counter

在[hadoop][]的[Java API][javaapi]中，我们很容易可以通过下面的代码，来给task增加一个Counter:

```java
// 添加一个groupName为"group" counterName为"counter"的Counter
context.getCounter("group", "counter").increment(1l);
```

但是在[Hadoop Streaming][streaming]中怎么去添加Counter呢？在[Hadoop Streaming的介绍][streaming]中，可以找到：

> A streaming process can use the stderr to emit counter information. reporter:counter:<group>,<counter>,<amount> should be sent to stderr to update the counter.

即我们只要身标准错误输出打印这种格式的日志就可以添加Counter了。

#### C++版

在map或者reduce程序的任意位置输出如下代码，就可以添加一个`groupName`为`group`，counterName为`counter`的Counter，每次打印时其值会加1.

```cpp
// groupName: group  counterName: counter
fprintf(stderr, "reporter:counter:group,counter,1"); 
```

#### Python版

```python
// groupName: group  counterName: counter
print >> sys.stderr, 'reporter:counter:group,counter,1'
```

其他编程语言的操作类似。

[hadoop]: http://hadoop.apache.org/ "http://hadoop.apache.org/"
[javaapi]: http://hadoop.apache.org/docs/r2.3.0/api/ "Hadoop Java API"
[streaming]: http://hadoop.apache.org/docs/r1.2.1/streaming.html "Hadoop Streaming"
