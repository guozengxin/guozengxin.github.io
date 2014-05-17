---
layout: post
title: "python语言的logging模块介绍"
description: "python语言自带了一个logging模块，简单易用，功能强大。"
category: python
tags: [python, logging]
comments: true
---

### OverView

在进行软件开发时，一个重要的工作就是打日志。在其他语言中，会有各种第三方日志组件，如log4net，log4cpp等。但python语言自带有一个强大的日志模块: logging。支持日志的分发，设置日志级别，设置日志格式等。现在python的日志模块做简要介绍并说明简单用法。

先看一个简单示例。我在最初了解logging模块时，都是这样简单进行设置。

{% highlight python %}
#logging_example1.py
import logging
logging.basicConfig(level = logging.INFO, format = '[%(asctime)s %(levelname)s] %(message)s', datefmt='%Y%m%d %H:%M:%S')
logging.debug('debug log')
logging.info('info log')
logging.warning('warning log')
logging.error('error log')
logging.critical('critial log')
{% endhighlight %}

其实，在程序规模较小时，只需要这样简单设置即可。上面的代码会输出：

	[20130727 21:31:17 INFO] info log
	[20130727 21:31:17 WARNING] warning log
	[20130727 21:31:17 ERROR] error log
	[20130727 21:31:17 CRITICAL] critial log

**注意**：**debug log**这一行没有输出，是因为设置的日志级别为`INFO`，关于日志级别后面介绍。

<!-- more -->

### 打印日志到文件

下面的代码可以将日志输出到文件中
#logging_example2.py
{% highlight python %}
import logging
logging.basicConfig(filename='example.log',level=logging.DEBUG)
logging.debug('This message should go to the log file')
logging.info('So should this')
logging.warning('And this, too')
{% endhighlight %}

会在文件中显示内容：

	DEBUG:root:This message should go to the log file
	INFO:root:So should this
	WARNING:root:And this, too

### 日志级别介绍

logging模块中的日志分为5个级别, 从高到低依次是：CRITICAL > ERROR > WARNING > INFO > DEBUG.设置了日志级别之后，低于该级别的日志将会被忽略。

通常以下面两种方式(不止这两种)来指定日志的级别。

* 通过命令行参数指定：

{% highlight python %}
python --log=INFO xxx.py
{% endhighlight %}

* 如上面示例程序，通过`basicConfig()`函数指定

{% highlight python %}
logging.basicConfig(filename='example.log',level=logging.DEBUG)
{% endhighlight %}

### 改变日志格式

`basicConfig()`函数可以设置日志的显示格式：

{% highlight python %}
import logging
logging.basicConfig(format='%(levelname)s:%(message)s', level=logging.DEBUG)
logging.debug('This message should appear on the console')
logging.info('So should this')
logging.warning('And this, too')
{% endhighlight %}

可以得到输出：

	DEBUG:This message should appear on the console
	INFO:So should this
	WARNING:And this, too

上面`%(levelname)s`和`%(message)s`分别指的是日志级别名和日志内容。在格式字符串，可以指定的属性见下表：

<table>
   <tr>
      <th>属性名</th>
      <th>格式</th>
      <th>说明</th>
   </tr>
   <tr>
      <td>asctime</td>
      <td>%(asctime)s</td>
      <td>打印当前时间，默认会展示2003-07-08 16:49:45,896，精度会精确到千分之一秒。</td>
   </tr>
   <tr>
      <td>created</td>
      <td>%(created)f</td>
      <td>打印当前的时间戳（time.time()函数的返回值）</td>
   </tr>
   <tr>
      <td>filename</td>
      <td>%(filename)s</td>
      <td>打印当前文件名。</td>
   </tr>
   <tr>
      <td>funcName</td>
      <td>%(funcName)s</td>
      <td>打印当前函数名。</td>
   </tr>
   <tr>
      <td>levelname</td>
      <td>%(levelname)s</td>
      <td>本条日志的日志级别 ('DEBUG',&#160;'INFO',&#160;'WARNING',&#160;'ERROR','CRITICAL').</td>
   </tr>
   <tr>
      <td>levelno</td>
      <td>%(levelno)s</td>
      <td>日志级别的整型值 (DEBUG,&#160;INFO,&#160;WARNING,&#160;ERROR,CRITICAL).</td>
   </tr>
   <tr>
      <td>lineno</td>
      <td>%(lineno)d</td>
      <td>打印日志的代码在文件中的行号 (if available).</td>
   </tr>
   <tr>
      <td>module</td>
      <td>%(module)s</td>
      <td>打印当前的模块名。</td>
   </tr>
   <tr>
      <td>msecs</td>
      <td>%(msecs)d</td>
      <td>打印当前时间的毫秒部分。</td>
   </tr>
   <tr>
      <td>message</td>
      <td>%(message)s</td>
      <td>当前的日志内容。</td>
   </tr>
   <tr>
      <td>name</td>
      <td>%(name)s</td>
      <td>当前日志的对象的name。</td>
   </tr>
   <tr>
      <td>pathname</td>
      <td>%(pathname)s</td>
      <td>打印当前文件的完整路径名。</td>
   </tr>
   <tr>
      <td>process</td>
      <td>%(process)d</td>
      <td>进程号</td>
   </tr>
   <tr>
      <td>processName</td>
      <td>%(processName)s</td>
      <td>进程名</td>
   </tr>
   <tr>
      <td>relativeCreated</td>
      <td>%(relativeCreated)d</td>
      <td>当前时间与日志对象创建的相对时间，显示为毫秒值</td>
   </tr>
   <tr>
      <td>thread</td>
      <td>%(thread)d</td>
      <td>线程ID</td>
   </tr>
   <tr>
      <td>threadName</td>
      <td>%(threadName)s</td>
      <td>线程名</td>
   </tr>
</table>

### 设置时间格式

logging默认的时间格式是ISO8601, 如果你对默认的时间格式不满意，可以在配置函数中自己设置：

{% highlight python %}
import logging
logging.basicConfig(format='%(asctime)s %(message)s', level=logging.DEBUG, datefmt='%Y%m%d %H:%M:%S')
logging.debug('show time')
{% endhighlight %}

将会显示：

	20130728 21:04:20 show time

时间格式的定义与time模块的[**time.strftime()**](http://docs.python.org/2/library/time.html#time.strftime)相同。

logging模块的基础用法就介绍的这儿，相信在小规模的程序中已经足够使用了。下一篇将介绍logging模块的进阶用法。

### 参考：

1. Python Logging HOWTO: <http://docs.python.org/2/howto/logging.html>.
