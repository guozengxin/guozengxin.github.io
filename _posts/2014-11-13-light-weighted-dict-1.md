---
layout: post
title: "一个轻量级的英汉词典(1)"
description: "编写一个轻量级的词典程序，数据来源于iciba.com，python语言编写，在linux平台上运行."
category: python
tags: [python, ldict, spider, lxml]
comments: true
---

## 1. 简介

这篇文章介绍我编写的一个小程序，一个简单的英汉词典。从[iciba.com][iciba]上获取翻译的内容，通过[lxml][]进行解析，然后展示。

程序将会用到[lxml][], [urllib2][]等python包。

本项目的github地址: <https://github.com/guozengxin/ldict>

## 2. 下载翻译结果

本项目的关键之一就是从iciba上下载词语的翻译结果，分2步。1. 用词语拼成iciba的结果；2. 用urllib2下载结果。

### 拼URL

英文词语: iciba的URL格式很简单，比如`every`的翻译结果URL就是<http://www.iciba.com/every>，所以，直接拼接即可。

中文词语: iciba的中文词语可以和英文词语一样，即<http://www.iciba.com/每个>，但是为了避免编码问题，我将中文的编码都统一转为`utf-8`:

{% highlight python %}
url = 'http://www.iciba.com/' + word.decode(outputEncoding).encode('utf8')
{% endhighlight %}

### 下载翻译结果

下载页面用`urllib2`来实现很简单，下面的代码可以实现最简单的下载功能：

```python
import urllib2
response = urllib2.urlopen('http://www.iciba.com/every')
data = response.read()
```

我的代码里添加了超时、异常处理、解压等功能：

{% highlight 

[iciba]: http://www.iciba.com/
[lxml]: http://lxml.de/
[urllib2]: https://docs.python.org/2/library/urllib2.html
