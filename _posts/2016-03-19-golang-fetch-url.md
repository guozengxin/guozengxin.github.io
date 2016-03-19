---
layout: post
title: "Golang根据URL下载文件"
description: "Golang从根据Url下载文件"
category: golang
tags: [golang, url, ioutil, http]
comments: true
---

## Golang下载文件

```golang
package main

import (
    "io/ioutil"
    "net/http"
    "os"
    "fmt"
)

func main() {
    url := "http://www.baidu.com"
    resp, err := http.Get(url)          // 根据URL获取响应
    if err != nil {
        fmt.Fprintf(os.Stderr, "fetch: %v\n", err)    // 输出到标准错误输出
        os.Exit(1)            // 退出，错误码为1
    }
    b, err := ioutil.ReadAll(resp.Body)      //从Body中读取全部数据
    resp.Body.Close()
    if err != nil {
        fmt.Fprintf(os.Stderr, "reading: %v\n", err)
        os.Exit(1)
    }
    fmt.Printf("%s", b)
}
```


## 参考

1. ioutil: <https://golang.org/pkg/io/ioutil/>
2. http: <https://golang.org/pkg/net/http/>
