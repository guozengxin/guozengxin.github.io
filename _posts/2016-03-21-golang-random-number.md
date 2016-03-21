---
layout: post
title: "Golang 获取随机数"
description: "Go语言中如何获取随机数，以及设置随机数种子"
category: golang
tags: [golang, Go语言, 随机数, rand]
comments: true
---

## Go语言获取随机数

下面的代码展示了Golang中获取随机数的几种方法

```golang
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	// 获取随机整数
	for i := 0; i < 5; i++ {
		fmt.Printf("%v ", rand.Int())
	}
	fmt.Println()

	// 获取随机的32位整数
	for i := 0; i < 5; i++ {
		fmt.Printf("%v ", rand.Int31())
	}
	fmt.Println()

	// 获取指定范围内的随机数
	for i := 0; i < 5; i ++ {
		fmt.Printf("%v ", rand.Intn(10))
	}
	fmt.Println()

	// 获取浮点型数[0.0, 1.0)之间
	for i := 0; i < 5; i ++ {
		fmt.Printf("%v ", rand.Float32())
	}
	fmt.Println()
}
```

## 设置随机数种子

注意，如果上面的代码重复执行两遍，会得到相同的随机数，这时候需要根据时间设置随机数的种子。

Golang中如何设置呢？

```golang
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    // 根据时间设置随机数种子
    rand.Seed(int64(time.Now().Nanosecond()))
    // 获取指定范围内的随机数
    for i := 0; i < 5; i ++ {
        fmt.Printf("%v ", rand.Intn(100))
    }
    fmt.Println()
}
```

这时重复执行将会得到不同的随机数。

## 相关文档

1. rand: <https://golang.org/pkg/math/rand/>.
2. time: <https://golang.org/pkg/time/>.
