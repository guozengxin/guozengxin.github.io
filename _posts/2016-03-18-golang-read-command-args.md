---
layout: post
title: "golang 获取命令行参数"
description: "go语言编程，获取命令行参数，io, Args"
category: golang
tags: [golang, io, Args, 命令行参数]
comments: true
---

## GO语言获取命令行参数

```golang
package main

import "os"

func main() {
	arg_num := len(os.Args)
	for i := 0; i < arg_num; i++ {
		println(os.Args[i])
	}
}
```

编译运行

```shell
$ go build args.go
$ ./args 1 2 3
./args
1
2
3
```

## 参考文档

1. os: <https://golang.org/pkg/os/>
