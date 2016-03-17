---
layout: post
title: "python语言logging模块的进阶用法"
description: "python语言logging模块的进阶用法，handler, filter, logger, formatter等类的用法"
category: python
tags: [python, logging]
comments: true
---

### OverView

在上一篇文章中，介绍了[logging模块的基本用法]({% post_url 2013-07-27-python-logging-module %})，本篇中，介绍有关logging模块的进阶用法。

### 四个基本组件

logging模块包括四个基本基本的组件：loggers, handlers, filters和formatters.

- Logger: 日志类，应用程序往往通过调用它提供的api来记录日志。
- Handler: 对日志信息处理，可以将日志发送(保存)到不同的目标域中。
- Filter: 对日志信息进行过滤，决定哪些日志信息可以被输出。
- Formatter: 日志的格式化。

<!-- more -->

### 日志流

下图展示了日志信息在loggers和handlers中的输出流，图片来源是[logging HOTTO -- Logging Flow](http://docs.python.org/2/howto/logging.html#logging-flow):

<img src="/assets/images/python/logging_flow.png" />

对上图作一些说明：

#### 打印一条日志的处理流程：

1. 由用户代码调用打印日志函数（`logging.info()`, `logging.debug()`等）开始。
2. 如果要打印的日志级别不够，则流程停止。否则继续下一步。
3. 建立一个`LogRecord`对象（该对象表示一条日志），并判断本条日志是否被filter过滤掉，如果被过滤，流程停止，否则进行下一步。
4. 当前logger会将当前`LogRecord`**传递到它所定义的handlers进行处理**(handler的处理流程后面说明)，**注意：handlers可以设置多个，会对一条日志进行多次处理**。并且，如果当前logger的propagate属性为假，则流程停止，否则继续下一步。
5. 判断当前logger有无父logger，如果没有，则流程停止，否则设置当前logger为它的父logger，继续执行步骤4。

#### 日志在handler中的处理过程：
1. 如果当前`LogRecord`的级别小于handler所设置的LogLevel，则停止流程，否则执行一步。
2. 判断当前`LogRecord`是否被handler是否被它所设置的filter过滤，如果被过滤，则流程停止，否则继续执行下一步。
3. 进行日志打印（打印之前应用formatters所设置的格式）。

### Loggers

Logger类常用的方法可以分为两类：配置函数和输出日志函数

#### Logger类的配置函数

- [**Logger.setLevel()**](http://docs.python.org/2/library/logging.html#logging.Logger.setLevel)函数设置了logger处理的最低日志级别。
- [**Logger.addHandler()**](http://docs.python.org/2/library/logging.html#logging.Logger.addHandler)和[**Logger.removeHandler()**](http://docs.python.org/2/library/logging.html#logging.Logger.removeHandler)函数负责为logger添加handler和从logger移除handler. 
- [**Logger.addFilter()**](http://docs.python.org/2/library/logging.html#logging.Logger.addFilter)和[**Logger.removeFilter()**](http://docs.python.org/2/library/logging.html#logging.Logger.removeFilter)函数负责为logger添加filter和从logger移除filter. 

#### logger类的输出日志函数

- [**Logger.debug()**](http://docs.python.org/2/library/logging.html#logging.Logger.debug), [**Logger.info()**](http://docs.python.org/2/library/logging.html#logging.Logger.info), [**Logger.warning()**](http://docs.python.org/2/library/logging.html#logging.Logger.warning), [**Logger.error()**](http://docs.python.org/2/library/logging.html#logging.Logger.error)和[**Logger.critical()**](http://docs.python.org/2/library/logging.html#logging.Logger.critical)用来打印日志。参数支持传入字符串及不定参数的传入(\*\*kwargs)。
- [**Logger.exception()**](http://docs.python.org/2/library/logging.html#logging.Logger.exception)输出的日志和[**Logger.error()**](http://docs.python.org/2/library/logging.html#logging.Logger.error)相同，它们的区别在于前者会丢出一个stack trace.
- [**Logger.log()**](http://docs.python.org/2/library/logging.html#logging.Logger.log)输入一个明确的参数表示日志级别，其它参数和输出日志的函数相同。

### Handlers

通过handler对象可以把日志内容写到不同的地方。比如简单的StreamHandler就是把日志写到类似文件的地方。python提供了十几种实用handler，比较常用和比较有意思的我列举一下：

- StreamHandler 写入类文件的流。
- BaseRotatingHandler 可以按时间写入到不同的日志中。比如将日志按天写入不同的日期结尾的文件文件。
- SocketHandler 用TCP网络连接写LOG
- DatagramHandler 用UDP网络连接写LOG
- SMTPHandler 把LOG写成EMAIL邮寄出去

#### 同时把日志输出到控制台和文件


```python
import logging

# 创建一个logger
logger = logging.getLogger('mylogger')
logger.setLevel(logging.DEBUG)

# 创建一个handler，用于写入日志文件
fh = logging.FileHandler('test.log')
fh.setLevel(logging.DEBUG)

# 再创建一个handler，用于输出到控制台
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)

# 定义handler的输出格式
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
fh.setFormatter(formatter)
ch.setFormatter(formatter)

# 给logger添加handler
logger.addHandler(fh)
logger.addHandler(ch)

# 记录一条日志
logger.info('foorbar')
```

### Formatters

Formatter对象用来设置最终输出的日志的格式、结构和内容。实例化Formatter类并带有两个参数可以创建一个Formatter对象：

```python
logging.Formatter.__init__(fmt=None, datefmt=None)
```

日志格式的定义中可以使用的格式符号可以参考：[**LogRecord attributes**](http://docs.python.org/2/library/logging.html#logrecord-attributes).
时间格式(datefmt)的定义与time模块的[**time.strftime()**](http://docs.python.org/2/library/time.html#time.strftime)相同。

### 配置日志的几种方式

python可以通过以下三种方式配置logging:

1. 在python源码中定义loggers, handlers和fomatters，如上面示例所示。
2. 创建一个日志配置文件，然后调用[**fileConfig()**](http://docs.python.org/2/library/logging.config.html#logging.config.fileConfig)函数加载。
3. 创建一个日志配置字典，然后调用[**dictConfig()**](http://docs.python.org/2/library/logging.config.html#logging.config.dictConfig)函数加载。

#### 通过python源码配置logging

```python
import logging

# 创建logger
logger = logging.getLogger('simple_example')
logger.setLevel(logging.DEBUG)

# 创建控制台handler并设置level为logging.DEBUG
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)

# 创建formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# 将formatter添加到handler
ch.setFormatter(formatter)

# 为logger设置handler
logger.addHandler(ch)

# 输出日志
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')
```

执行这段代码，会输出如下日志：

	2005-03-19 15:10:26,618 - simple_example - DEBUG - debug message
	2005-03-19 15:10:26,620 - simple_example - INFO - info message
	2005-03-19 15:10:26,695 - simple_example - WARNING - warn message
	2005-03-19 15:10:26,697 - simple_example - ERROR - error message
	2005-03-19 15:10:26,773 - simple_example - CRITICAL - critical message

#### 通过配置文件配置logging

下面的配置文件**logging.conf**可以产生与上面代码配置相同的效果：

	[loggers]
	keys=root,simpleExample

	[handlers]
	keys=consoleHandler

	[formatters]
	keys=simpleFormatter

	[logger_root]
	level=DEBUG
	handlers=consoleHandler

	[logger_simpleExample]
	level=DEBUG
	handlers=consoleHandler
	qualname=simpleExample
	propagate=0

	[handler_consoleHandler]
	class=StreamHandler
	level=DEBUG
	formatter=simpleFormatter
	args=(sys.stdout,)

	[formatter_simpleFormatter]
	format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
	datefmt=

用下面的代码加载logging.conf：

```python
import logging
import logging.config

logging.config.fileConfig('logging.conf')

# create logger
logger = logging.getLogger('simpleExample')

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warn('warn message')
logger.error('error message')
logger.critical('critical message')
```

#### 使用dictionary配置logging

从python2.7开始，增加了用dictionary来配置logging. 下面的示例可以产生与以上两种方式相同的配置效果。(logging.conf)

	version: 1
	formatters:
		simple:
			format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
	handlers:
		console:
			class: logging.streamhandler
			level: debug
			formatter: simple
			stream: ext://sys.stdout
	loggers:
		simpleexample:
			level: debug
			handlers: [console]
			propagate: no
	root:
		level: debug
		handlers: [console]

这段配置是用yaml格式书写的，可以这样加载：

```python
import logging
#读取logging.conf文件
D = yaml.load(open('logging.conf', 'r'))
#加载配置
logging.config.dictConfig(D)
```

### 参考

1. Python HOWTO -- Advanced Logging Tutorial: <http://docs.python.org/2/howto/logging.html#advanced-logging-tutorial>
2. Python logging module: <http://docs.python.org/2/library/logging.html>
