---
layout: post
title: "Linux终端的英汉词典(1)"
description: "编写一个轻量级的词典程序，数据来源于iciba.com，python语言编写，在linux平台上运行."
category: python
tags: [python, ldict, spider, lxml, linux, 终端]
comments: true
---

作为码农，怎能忍受离开终端和键盘来操作......

比如查词典，本程序实现了一个简单的工具，可以在终端界面下查词典。

## 1. ldict简介

这篇文章介绍我编写的一个小程序，一个简单的英汉词典[ldict][]。从[iciba.com][iciba]上获取翻译的内容，通过[lxml][]进行解析，然后展示。

程序将会用到[lxml][], [urllib2][]等python包。

本项目的github地址: <https://github.com/guozengxin/ldict>

**安装方法**：

{% highlight python %}
python setup.py install
{% endhighlight %}

**依赖的库**: [lxml][]

**适用系统**：Linux, 装有Python运行环境的系统

## 2. 下载翻译结果

本项目的关键之一就是从iciba上下载词语的翻译结果，分2步。1. 用词语拼成iciba的URL；2. 用urllib2下载结果。

### 拼URL

英文词语: iciba的URL格式很简单，比如`every`的翻译结果URL就是<http://www.iciba.com/every>，所以，直接拼接即可。

中文词语: iciba的中文词语可以和英文词语一样，即<http://www.iciba.com/每个>，但是为了避免编码问题，我将中文的编码都统一转为`utf-8`:

{% highlight python %}
url = 'http://www.iciba.com/' + word.decode(outputEncoding).encode('utf8')
{% endhighlight %}

### 下载翻译结果

下载页面用`urllib2`来实现很简单，下面的代码可以实现最简单的下载功能：

{% highlight python %}
import urllib2
response = urllib2.urlopen('http://www.iciba.com/every')
data = response.read()
{% endhighlight %}

我的代码里添加了超时、异常处理、解压等功能：[源文件github地址](https://github.com/guozengxin/ldict/blob/master/ldutil/htmlfetcher.py)

{% highlight python %}
#!/usr/bin/env python
# encoding=utf-8

import urllib2
from urllib2 import HTTPError, URLError
import sys
from gzipSupport import ContentEncodingProcessor


def get_headers():
    header = {'User-agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.22 (KHTML, like Gecko) Chrome/25.0.1364.97 Safari/537.22'}
    return header


def http_get_response(url, referer=None):
    '''get html response from web'''
    response = None
    encoding_support = ContentEncodingProcessor
    opener = urllib2.build_opener(encoding_support, urllib2.HTTPHandler)
    try:
        headers = get_headers()
        if referer is not None:
            headers['Referer'] = referer
        request = urllib2.Request(url, headers=headers)
        response = opener.open(request, timeout=20)
    except HTTPError, e:
        sys.stderr.write(str(e) + '\n')
    except URLError, e:
        sys.stderr.write(str(e) + '\n')
    except IOError, e:
        sys.stderr.write(str(e) + '\n')
    return response


def http_get(url, referer=None):
    '''get html file from web'''
    data = None
    response = http_get_response(url, referer)
    if response:
        try:
            data = response.read()
        except:
            sys.stderr.write('unnkown error in http_get')
    return data
{% endhighlight %}

## 3. 从html文件解析翻译结果

这个项目用了python的[lxml][]库来解析html数据，lxml可以解析html和xml的数据，对xml结构进行操纵，简单实用。在提取数据时，还可以很好的和[xpath][]进行结合。

如下的简单示例即可简单的从网页中提取数据：

{% highlight python %}
import StringIO
from lxml import etree
import urllib2

parser = etree.HTMLParser()
data = urllib2.urlopen('http://www.baidu.com').read()
tree = etree.parse(StringIO.StringIO(data), parser)
print tree.xpath('//title')[0].text.encode('gbk')
{% endhighlight %}

上述代码可以提取到百度首页的title:

{% highlight text %}
百度一下，你就知道
{% endhighlight %}

在[ldict][]项目中我写了一个用于提取解析html的类: [htmlparser](https://github.com/guozengxin/ldict/blob/master/ldutil/htmlparser.py).

## 4. 后记

下一篇文章将会介绍本项目中用到的其他技术: 终端输出彩色文字，简单的交互式输入实现等。


[iciba]: http://www.iciba.com/
[lxml]: http://lxml.de/
[urllib2]: https://docs.python.org/2/library/urllib2.html
[ldict]: https://github.com/guozengxin/ldict
[xpath]: http://www.w3school.com.cn/xpath/
