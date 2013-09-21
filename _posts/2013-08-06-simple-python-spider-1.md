---
layout: post
title: "用python开发一个简易的spider(一)"
description: "用python语言开发一个简单的spider系统，并暴露出一些方便易用的接口，可以在有业务需求时导入spider模块，快速构建抓取流程。"
category: python
tags: [python, spider, 网页抓取, 爬虫]
---
{% include JB/setup %}

### 前言

我平时的工作之一就是做一些数据积累工作，具体就是从特定的垂直网站上抓取一些结构化的数据，以文件或者数据库的形式保存下来。平时抓取用python语言较多，零零散散的写过一些抓取网页分析网页的程序，但是编码时，存在着很多重复的工作，很多代码都是复制粘贴的，耦合度很高，不利于维护及修改需求。

因此想把抓取和解析网页的程序分离出来，做成一个单独的模块，模块中暴露出一些方便易用的接口或者类，使可以在有业务需求时，导入模块并配置一些解析特定网页的项，编写和业务相关的代码，就可以快速构建一个抓取流程。


<!-- more -->

### 各个模块及各自的功能

#### 调度(Selector)

Selector从抓取队列中选择一个URL，交由下载器去下载页面。Selector的另外一个任务就是控制抓取速度，保持spider的友好性。

#### 下载器(Downloader)

Downloader的任务是抓取网页，并将结果返回。

#### 蜘蛛(SimpleSpider)

SimpleSpider的任务是接收抓取结果并解析，然后将解析结果分发。它至少需要完成以下任务：

1. 设置抓取的种子文件，即从哪些页面开始抓取。
2. 设置解析程序，为不同类型的页面设置不同的解析程序。
3. 将解析结果分发。解析结果主要分为两类：我们需要的页面中的元素(Item)和需要抓取的URL集合。将Item发送给存储中间件进行保存；将URL集合发送给抓取队列进行后续的抓取。

#### 解析器(SimpleParser)

解析器的任务是解析抓取到的HTML文件

1. 平时开发时发现用xslt模板进行解析比较方便管理，SimpleParser需要支持这一点。
2. 一些特殊的抓取结果是json形式的字符串，也需要支持json的解析。
3. 需要支持自己编写解析函数。

#### 存储中间件(Handler)

这个模块的作用是可以接收从页面中提取的结构化结果，按照配置的信息保存到指定位置。

### spider所能完成的工作

根据平时工作的需求，spider的设计需要具有如下功能：

1. 从一个站点抓取所有符合某个URL模式的链接，并将提取的结构化内容保存。在这一任务中需要自己配置的项有：种子URL（一般为站点首页）、需要提取数据URL模式、解析网页的方法和保存网页的方式。
2. 解析某个页面的时候，需要二次提取才能保证信息的完整性。spider需要支持二次提取并合并数据，合并之后发给存储模块。
3. 业务开发的代码要尽量和spider模块正交，只需要导入spider模块进行后续开发就可以完成任务。
4. 控制抓取速度。尽量能支持配置同域名的抓取速度，这个需要Selector模块支持。

### 其他说明
1. 在本系列文章中，我把编写代码的过程用日志记录下来，可以促进我进行良好的设计及逻辑的完整性，同时可以锻炼写作能力。
2. 设计过程中一部分是根据平时工作需要及经验，另外模块的组织参考了著名的python spider工具[scrapy](http://scrapy.org/).

### 参考

1. 开源python网络爬虫框架Scrapy: <http://blog.csdn.net/zbyufei/article/details/7554322>.