---
layout: post
title: "python语言logging模块的进阶用法"
description: "python语言logging模块的进阶用法，handler, filter, logger, formatter等类的用法"
category: python
tags: [python, logging]
---
{% include JB/setup %}

## OverView

在上一篇文章中，介绍了[logging模块的基本用法](/python/2013/07/27/python-logging-module/)，本篇中，介绍有关logging模块的进阶用法。

## 四个基本组件

logging模块包括四个基本基本的组件：loggers, handlers, filters和formatters.

- Logger: 日志类，应用程序往往通过调用它提供的api来记录日志。
- Handler: 对日志信息处理，可以将日志发送(保存)到不同的目标域中。
- Filter: 对日志信息进行过滤，决定哪些日志信息可以被输出。
- Formatter: 日志的格式化。

<!-- more -->

## 日志流

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

## Loggers

Logger类常用的方法可以分为两类：配置函数和输出日志函数

#### Logger类的配置函数

- [**Logger.setLevel()**](http://docs.python.org/2/library/logging.html#logging.Logger.setLevel)函数设置了logger处理的最低日志级别。
- [**Logger.addHandler()**](http://docs.python.org/2/library/logging.html#logging.Logger.addHandler)和[**Logger.removeHandler()**](http://docs.python.org/2/library/logging.html#logging.Logger.removeHandler)函数负责为logger添加handler和从logger移除handler. 
- [**Logger.addFilter()**](http://docs.python.org/2/library/logging.html#logging.Logger.addFilter)和[**Logger.removeFilter()**](http://docs.python.org/2/library/logging.html#logging.Logger.removeFilter)函数负责为logger添加filter和从logger移除filter. 

#### logger类的输出日志函数

- [**Logger.debug()**](http://docs.python.org/2/library/logging.html#logging.Logger.debug), [**Logger.info()**](http://docs.python.org/2/library/logging.html#logging.Logger.info), [**Logger.warning()**](http://docs.python.org/2/library/logging.html#logging.Logger.warning), [**Logger.error()**](http://docs.python.org/2/library/logging.html#logging.Logger.error)和[**Logger.critical()**](http://docs.python.org/2/library/logging.html#logging.Logger.critical)用来打印日志。参数支持传入字符串及不定参数的传入(\*\*kwargs)。
- [**Logger.exception()**](http://docs.python.org/2/library/logging.html#logging.Logger.exception)输出的日志和[**Logger.error()**](http://docs.python.org/2/library/logging.html#logging.Logger.error)相同，它们的区别在于前者会丢出一个stack trace.
- [**Logger.log()**](http://docs.python.org/2/library/logging.html#logging.Logger.log)输入一个明确的参数表示日志级别，其它参数和输出日志的函数相同。


