---
layout: post
title: "golang应用thrift框架"
description: "Golang 最适合的领域就是服务器端程序，结合thrift框架大大增加了它的能力。"
category: golang
tags: [golang, thrift, rpc]
comments: true
---

Golang 在服务端有性能优势，与 thrift 框架结合，增强了与各语言之间的通讯，使 Golang 不仅自身之间可以使用 rpc，与其它语言之间也可以方便的调用 rpc。

## 安装及创建接口

安装 thrift 库：

```
go get git.apache.org/thrift.git/lib/go/thrift
```

创建一个 thrift 文件 RpcService.thrift：

```
namespace go demo.rpc
namespace py demo.rpc

service RpcService {
    list<string> funCall(1:i64 callTime, 2:string funCode, 3:map<string, string> paramMap),
}
```

文件中定义了一个函数，有4个参数。

生成代码：

```
thrift -gen go RpcService.thrift
```

会生成四个文件：`constants.go rpc_service.go ttypes.go`是协议文件，`rpc_service-remote.go`是示例文件。


## 创建服务端程序

```go
package main

import (
    "demo/rpc"
    "fmt"
    "git.apache.org/thrift.git/lib/go/thrift"
    "os"
)

const (
    NetworkAddr = "127.0.0.1:38000"
)

type RpcServiceImpl struct {
}

func (rpc *RpcServiceImpl) FunCall(callTime int64, funCode string, paramMap map[string]string) (r []string, err   error) {
    fmt.Println("-->FunCall", callTime, funCode, paramMap)
    for k, v := range paramMap {
        r = append(r, k+v)
    }
    return
}

func main() {
    transportFactory := thrift.NewTFramedTransportFactory(thrift.NewTTransportFactory())
    protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()
    serverTransport, err := thrift.NewTServerSocket(NetworkAddr)
    if err != nil {
        fmt.Println("Error!", err)
        os.Exit(1)
    }

    handler := &RpcServiceImpl{}
    processor := rpc.NewRpcServiceProcessor(handler)

    server := thrift.NewTSimpleServer4(processor, serverTransport, transportFactory, protocolFactory)
    fmt.Println("thrift server in", NetworkAddr)
    server.Serve()
}
```

服务端的代码逻辑比较简单，每次调用的时候打出函数的参数。


## 客户端代码

```go
package main

import (
    "demo/rpc"
    "fmt"
    "git.apache.org/thrift.git/lib/go/thrift"
    "net"
    "os"
    "time"
)

func main() {
    startTime := currentTimeMillis()
    transportFactory := thrift.NewTFramedTransportFactory(thrift.NewTTransportFactory())
    protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()

    transport, err := thrift.NewTSocket(net.JoinHostPort("127.0.0.1", "38000"))
    if err != nil {
        fmt.Fprintln(os.Stderr, "error resolving address:", err)
        os.Exit(1)
    }

    useTransport := transportFactory.GetTransport(transport)
    client := rpc.NewRpcServiceClientFactory(useTransport, protocolFactory)
    if err := transport.Open(); err != nil {
        fmt.Fprintln(os.Stderr, "Error opening socket to 127.0.0.1:38000", " ", err)
        os.Exit(1)
    }
    defer transport.Close()

    for i := 0; i < 1000; i++ {
        paramMap := make(map[string]string)
        paramMap["name"] = "qinerg"
        paramMap["passwd"] = "123456"
        r1, e1 := client.FunCall(currentTimeMillis(), "login", paramMap)
        fmt.Println(i, "Call->", r1, e1)
    }

    endTime := currentTimeMillis()
    fmt.Println("Program exit. time->", endTime, startTime, (endTime - startTime))
}

func currentTimeMillis() int64 {
    return time.Now().UnixNano()
}
```

客户端的代码逻辑是循环 1000 次，调用服务端的 FunCall 函数。

然后启动服务端和客户端，可以看到服务端有打印输出，证明 rpc 调用正常。

## 参考

1. <https://thrift.apache.org/tutorial/go>
2. <https://m.oschina.net/blog/165285>
