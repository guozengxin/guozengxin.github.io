---
layout: post
title: "pig cookbook学习"
description: "pic cookbook学习"
category: hadoop
tags: [pig, hadoop]
comments: true
---

### Overview

近期需要用pig做一些统计，由于没有系统学习，总是出现一些问题，且不容易调试，执行效率也不高。所以打算看一些官方文档，在此做些笔记。

### pig性能提升


#### 指定类型

如果在load文件时不指定类型，pig在计算时会指定为**double**类型，而在很多时候，数据本应是整形等，指定为**double**类型会增加广计算量。另外，指定类型也会使错误提早暴露出来。

	--Query 1
	A = load 'myfile' as (t, u, v);
	B = foreach A generate t + u;

	--Query 2
	A = load 'myfile' as (t: int, u: int, v);
	B = foreach A generate t + u;

第二个query会比第一个更有效率，有时候会增加一倍。

<!-- more -->

#### 提早去除没用的field

pig不用自动判定某一个field没用，并且去掉它。例如，下面一段代码：

	A = load 'myfile' as (t, u, v);
	B = load 'myotherfile' as (x, y, z);
	C = join A by t, B by x;
	D = group C by u;
	E = foreach D generate group, COUNT($1);

**v**,**y**,**z**在代码的执行过程中并没有用到，而且在join之后，没有必要同时保留**t**和**x**，将上述代码修改为下面的形式，将会很大程度上减小在map和reduce阶段所pig携带的数据量。

	A = load 'myfile' as (t, u, v);
	A1 = foreach A generate t, u;
	B = load 'myotherfile' as (x, y, z);
	B1 = foreach B generate x;
	C = join A1 by t, B1 by x;
	C1 = foreach C generate t, u;
	D = group C1 by u;
	E = foreach D generate group, COUNT($1);

根据数据的不同，这一策略会很大程序上节省时间。对上面的数据而言，可以减小50%的时间。

#### 提早过滤不必要的行

大多数情况下，提早使用filter会减少通过管道传输的数据量。

	-- Query 1
	A = load 'myfile' as (t, u, v);
	B = load 'myotherfile' as (x, y, z);
	C = filter A by t == 1;
	D = join C by t, B by x;
	E = group D by u;
	F = foreach E generate group, COUNT($1);

	-- Query 2
	A = load 'myfile' as (t, u, v);
	B = load 'myotherfile' as (x, y, z);
	C = join A by t, B by x;
	D = group C by u;
	E = foreach D generate group, COUNT($1);
	F = filter E by C.t == 1;

上面代码中，query1明显比query2高效，因为query1减少了进入join之前的数据量。

但是如果过早的执行filter代价高，且过滤的数据量少的话，就不值得过早的执行filter语句。

#### 减少操作管道

为了使脚本清晰易读，你可能会将任务分为多个操作来执行：

	A = load 'data' as (in: map[]);
	-- get key out of the map
	B = foreach A generate in#k1 as k1, in#k2 as k2;
	-- concatenate the keys
	C = foreach B generate CONCAT(k1, k2);
	.......

上述代码容易理解，但是将后面两步合并会使效率更高：

	A = load 'data' as (in: map[]);
	-- concatenate the keys from the map
	B = foreach A generate CONCAT(in#k1, in#k2);
	....

同样的逻辑也适用于filter操作

#### 使用PARALLEL语句

使用PARALLEL语句可以增加job的并行度:

- PARALLEL设置了一个job的reduce数量，如果不设置，默认为1。
- PARALLEL只负责设置job的reduce数量，map数量是由输入数据决定的，每一个输入块是生成一下map。
- 如果不设置PARALLEL，将只会生成一个reduce任务。

可以在任务一个会产生reduce任务的操作中使用PARALLEL语句，包括：[COGROUP](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html#COGROUP), [CROSS](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html#CROSS), [DISTINCT](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html#DISTINCT), [GROUP](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html#GROUP), [JOIN (inner)](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html#JOIN+%28inner%29), [JOIN (outer)](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html#JOIN+%28outer%29), and [ORDER](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html#ORDER).

下面示例PARALLEL应用于GROUP语句：

	A = LOAD 'myfile' AS (t, u, v);
	B = GROUP A BY t PARALLEL 18;
	.....

还可以使用[set default parallel](http://pig.apache.org/docs/r0.7.0/piglatin_ref2.html#set)来设置：

	SET DEFAULT_PARALLEL 20;
	A = LOAD ‘myfile.txt’ USING PigStorage() AS (t, u, v);
	B = GROUP A BY t;
	C = FOREACH B GENERATE group, COUNT(A.t) as mycount;
	D = ORDER C BY mycount;
	STORE D INTO ‘mysortedcount’ USING PigStorage();


### 参考
1. Pig Cookbook: <http://pig.apache.org/docs/r0.7.0/cookbook.html>
