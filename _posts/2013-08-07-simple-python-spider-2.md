---
layout: post
title: "用python开发一个简易的spider(二)"
description: "用python语言开发一个简单的spider系统。本篇介绍下载器及相关的东西。"
category: python
tags: [python, spider, 网页抓取, 爬虫, downloader]
comments: true
---

### 简介

在上一篇文章[python开发一个简易的spider(一)]({% post_url 2013-08-06-simple-python-spider-1 %})中从总体上介绍了simpleSpider系统的开发场景及各个模块的简单介绍。本篇文章介绍下载器(downloader)的开发及所需要注意的事项。

### python标准库urllib2

urllib2是python中的一个标准模块，用于获取url。 它提取一系列简单的接口，可以方便的设置及下载网络上的数据。downloader模块就用urllib2来实现，下面是downloader模块所需要设置一些内容。

<!-- more -->

#### Timeout设置

在python2.6之前，urllib2模块的API不支持Timeout设置，如果确实需要设置Timeout，只能通过下面的办法：更新socket的全局超时设置。

```python
import urllib2
import socket

socket.setdefaulttimeout(10) # 10 秒钟后超时
urllib2.socket.setdefaulttimeout(10) # 另一种方式
```

从python2.6开始，[**urllib2.urlopen()**](http://docs.python.org/2/library/urllib2.html#urllib2.urlopen)函数可以直接用参数设置Timeout

```python
import urllib2
response = urllib2.urlopen('http://www.sogou.com', timeout=10) # 设置超时为10秒
```

#### 设置HTTP Header

可以用[**urllib2.Request**](http://docs.python.org/2/library/urllib2.html#request-objects)对象来设置发起HTTP请示时的HTTP header.

直接在构造函数中设置，用一个dict类型的变量表示headers:

```python
import urllib2

headers = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.22 (KHTML, like Gecko) Chrome/25.0.1364.97 Safari/537.22'}
request = urllib2.Request('http://www.sogou.com', headers=headers)
response = urllib2.urlopen(request, timeout=10)
```

也可以用[**urllib2.Request.add_header()**](http://docs.python.org/2/library/urllib2.html#urllib2.Request.add_header)函数来设置：

```python
import urllib2

ua = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.22 (KHTML, like Gecko) Chrome/25.0.1364.97 Safari/537.22'
request = urllib2.Request('http://www.sogou.com')
request.add_header('User-Agent', ua)
response = urllib2.urlopen(request, timeout=10)
```

#### 自动解压编码

有些网页为了减少网络传输的数据，会将网页的内容进行压缩传输。此时可以继承[**urllib2.BaseHandler**](http://docs.python.org/2/library/urllib2.html#basehandler-objects)类编写一个自动处理编码的handler，可以在下载完成后自动解码网页内容。

处理编码的类为ContentEncodingProcessor(具体实现后面介绍)，用如下方式来加入到下载器：

```python
import urllib2

opener = urllib2.build_opener(ContentEncodingProcessor, urllib2.HTTPHandler)
request = urllib2.Request('http://www.sogou.com')
response = opener.open(request, timeout=10)
```

其中，[**urllib2.build_opener**](http://docs.python.org/2/library/urllib2.html#urllib2.build_opener)函数返回接受一个或多个handler参数，返回一个[**OpenerDirector**](http://docs.python.org/2/library/urllib2.html#urllib2.OpenerDirector)实例。用该实例创建的连接会应用所设置的handler.

### downloader模块代码

```python
#!/usr/bin/env python
#encoding=utf-8

import urllib2
import sys
import logging
import settings

from gzipSupport import ContentEncodingProcessor
from urllib2 import HTTPError, URLError
from StringIO import StringIO

logging.basicConfig(level = logging.INFO, format = '[%(asctime)s %(levelname)s] [%(module)s] %(message)s', datefmt='%Y%m%d %H:%M:%S')
logger = logging.getLogger('downloader')

class Downloader:

	def __init__(self):
		self.__headers = self.__getHeaders()
		self.__timeout = self.__getTimeout()

	def __getHeaders(self):
		header = {
				'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.22 (KHTML, like Gecko) Chrome/25.0.1364.97 Safari/537.22'
				}
		ua = getattr(settings, 'USER_AGENT', None)
		if ua is not None:
			header['User-Agent'] = ua
		return header

	def __getTimeout(self):
		timeout = getattr(settings, 'TIMEOUT', None)
		if timeout is not None:
			timeout = int(timeout)
		else:
			timeout = 10

		return timeout

	def httpResponse(self, url, referer=None):
		'''get html response from web'''
		response = None
		encoding_support = ContentEncodingProcessor
		opener = urllib2.build_opener(encoding_support, urllib2.HTTPHandler)
		try:
			headers = self.__headers
			if referer is not None:
				headers['Referer'] = referer
			request = urllib2.Request(url, headers = headers)
			response = opener.open(request, timeout=self.__timeout)
		except HTTPError, e:
			logger.error(str(e))
		except URLError, e:
			logger.error(str(e))
		except IOError, e:
			logger.error(str(e))
		return response

	def httpGet(self, url, referer=None):
		'''get html file from web'''
		data = None
		response = self.httpResponse(url, referer)
		if response:
			try:
				data = response.read()
			except:
				logger.error('unknown error occured while read response')
		return data
```

上面的代码中，settings模块是一个全局的设置文件，用于设置一些常用的参数。

### gzipSupport模块代码

```python
#!/usr/bin/env python

from StringIO import StringIO
from gzip import GzipFile
import zlib
import urllib2

class ContentEncodingProcessor(urllib2.BaseHandler):
	"""A handler to add gzip capabilities to urllib2 requests """

	# add headers to requests
	def http_request(self, req):
		req.add_header("Accept-Encoding", "gzip, deflate")
		return req

	# decode
	def http_response(self, req, resp):
		old_resp = resp
		# gzip
		if resp.headers.get("content-encoding") == "gzip":
			gz = GzipFile(fileobj=StringIO(resp.read()), mode="r")
			resp = urllib2.addinfourl(gz, old_resp.headers, old_resp.url, old_resp.code)
			resp.msg = old_resp.msg
		# deflate
		if resp.headers.get("content-encoding") == "deflate":
			gz = StringIO( deflate(resp.read()) )
			resp = urllib2.addinfourl(gz, old_resp.headers, old_resp.url, old_resp.code)	# 'class to add info() and
			resp.msg = old_resp.msg
		return resp

def deflate(data):	# zlib only provides the zlib compress format, not the deflate format;
	try:			# so on top of all there's this workaround:
		return zlib.decompress(data, -zlib.MAX_WBITS)
	except zlib.error:
		return zlib.decompress(data)
```

在http_request函数中，设置Accpet-Encoding为可接受的网页类型。在http_response函数中，根据返回的content-encoding的编码方式来解压数据。

### 参考

1. urllib2 HOWTO: <http://docs.python.org/2/howto/urllib2.html>.
2. python urllib2模块: <http://docs.python.org/2/library/urllib2.html>.
3. urllib2的使用细节: <http://zhuoqiang.me/python-urllib2-usage.html>.
