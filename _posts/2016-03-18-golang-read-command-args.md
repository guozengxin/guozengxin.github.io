---
layout: post
title: "golang 获取命令行参数"
description: "go语言编程，获取命令行参数，io, Args"
category: golang
tags: [golang, io, Args, 命令行参数]
comments: true
---

## os包获取命令行参数

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

## flag包获取命令行参数

```go
package main

import "flag"  // flag包负责命令行解析的代码
import "fmt"

func main() {
    // 定义参数i接收int型，注意返回的是指针
    i := flag.Int("i", 10, "input a number")
    // 定义参数s接收string，返回指针
    s := flag.String("s", "ss", "input a string")
    var a int
    // 定义参数a，将解析的结果放到变量a
    flag.IntVar(&a, "a", 20, "input a number")
    // 执行解析
    flag.Parse()

    // 输出其余的参数
    fmt.Printf("%v\n", flag.Args())
    fmt.Printf("i=%d\n", *i)
    fmt.Printf("s=%s\n", *s)
    fmt.Printf("a=%d\n", a)
}
```

## 参考文档

1. os: <https://golang.org/pkg/os/>
2. flag: <https://golang.org/pkg/flag/>
