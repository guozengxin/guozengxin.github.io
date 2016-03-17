---
layout: post
title: "golang - 按行读取文件"
description: "Golang按行读取文件, Golang逐行读文件, bufio"
category: golang
tags: [golang, bufio]
comments: true
---

#### Golang逐行读取一个文件示例：

```go
package main

import (
	"os"
	"io"
	"bufio"
	"strings"
)

func readLine(fileName string) {
	f, _ := os.Open(fileName)
	reader := bufio.NewReader(f)
	for {
		line, err := reader.ReadString('\n')
		if err == io.EOF {
			break
		}
		line = strings.TrimSpace(line)
		println(line)
	}
}

func main() {
	readLine("i.txt")
}
```

#### GO相关文档

1. bufio: <https://golang.org/pkg/bufio/>
2. strings: <https://golang.org/pkg/strings/>
