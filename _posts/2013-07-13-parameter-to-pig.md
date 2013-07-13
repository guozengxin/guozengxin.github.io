---
layout: post
title: "向pig脚本传递参数"
description: ""
category: hadoop
tags: [pig, hadoop, 传参]
---
{% include JB/setup %}

### 直接传参

采用-p传递参数，每一个变量前都要加-p:

{% highlight bash %}
$ pig -p DATE=`date +%Y-%m-%d` daily.pig
{% endhighlight %}

<!-- more -->
pig引用参数的方式和shell相同：

	daily     = load 'NYSE_daily' as (exchange:chararray, symbol:chararray,
					date:chararray, open:float, high:float, low:float, close:float,
					volume:int, adj_close:float);
	yesterday = filter daily by date == '$DATE';
	grpd      = group yesterday all;
	minmax    = foreach grpd generate MAX(yesterday.high), MIN(yesterday.low);

### 由文件传参


{% highlight bash %}
#Param file
YEAR=2009-
MONTH=12-
DAY=17
DATE=$YEAR$MONTH$DAY
{% endhighlight %}

调用方式：

{% highlight bash %}
pig -param_file daily.params daily.pig
{% endhighlight %}

pig脚本中调用参数和直接传参的调用方式相同
