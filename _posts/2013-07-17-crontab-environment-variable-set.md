---
layout: post
title: "crontab中环境变量设置"
description: "crontab中的环境变量是与当前用户环境不一致的，因此当前可以执行的脚本不一定能在crontab中执行"
category: linux
tags: [linux, crontab, 环境变量]
---
{% include JB/setup %}

### Question

今天写的一个脚本，直接在shell中可以正常运行，但是放到crontab中不能正常运行，出现了**“pig: command not found”**错误。判断应该是crontab中的`$PATH`变量和当前环境不一致的问题。

<!-- more -->
### Test

写一个简单脚本来测试crontab中的环境变量：

	#!/bin/sh
	echo $PATH

在shell中直接运行会输出：

	/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/lib/hadoop/bin:/usr/lib/hbase/bin:/usr/lib/hive/bin:/usr/lib/pig-0.9.1/bin:/root/bin:/usr/lib/hadoop/bin:/usr/lib/hbase/bin:/usr/lib/hive/bin:/usr/lib/pig-0.9.1/bin:/usr/lib/hadoop/bin:/usr/lib/hbase/bin:/usr/lib/hive/bin:/usr/lib/pig-0.9.1/bin

但是在crontab中运行上面脚本，输出：

	/usr/bin:/bin

证实crontab中的环境变量和当前linux环境中的不同。

### Solution

在网上查资料，发现直接将path写在crontab中就可以设置crontab中的环境变量。在crontab文件的开头设置以下选项：
	
{% highlight bash %}
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/
{% endhighlight %}

- `SHELL`设置当前bash
- `PATH`设置当前环境变量
- `MAILTO`设置通知
- `HOME`设置crontab任务执行时使用的主目录。

因此，我将pig的路径添加到当前环境变量中就可以解决问题。设置环境时注意要在当前用户的crontab任务中设置。



