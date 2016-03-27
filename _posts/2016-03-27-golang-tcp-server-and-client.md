---
layout: post
title: "golang 创建 TCP 的服务端与客户端"
description: "go语言中如何建立一个TCP服务端，如何用客户端连接服务端"
category: golang
tags: [golang, tcp, server, client, 服务端]
comments: true
---

## 创建 TCP 服务端

```golang
package main

import (
    "fmt"
    "net"
)

func main() {
    fmt.Println("Starting the server ...")

    // 创建一个监听器监听本机的38000端口，注明是tcp连接
    listener, err := net.Listen("tcp", "127.0.0.1:38000")
    // 监听失败，直接退出
    if err != nil {
        fmt.Println("Listener err!", err.Error())
        return
    }

    for {
        // 建立一个客户端的连接
        conn, err := listener.Accept()

        if err != nil {
            // 建立连接错误，直接退出
            fmt.Println("Error accept", err.Error())
            return
        }

        // 建立一个协程来执行连接
        go doStuff(conn)
    }
}

func doStuff(conn net.Conn) {
    for {
        // 创建 512 字节的缓冲区
        buff := make([]byte, 512)
        // 从连接中读数据
        _, err := conn.Read(buff)
        if err != nil {
            // 读取错误，退出
            fmt.Println("Error reading", err.Error())
            return
        }
        fmt.Printf("Received data: %v\n", string(buff))
    }
}
```

服务端的代码首先创建了一个 TCP 的监听器，然后为每个客户端的连接建立一个协程来读客户端传送的数据。

## 建立 TCP 客户端

<!-- more -->

```golang
package main

import (
    "net"
    "fmt"
    "strings"
    "bufio"
    "os"
)

func main() {
    // 创建一个与TCP服务器的连接
    conn, err := net.Dial("tcp", "127.0.0.1:38000")
    if err != nil {
        // 建立连接失败，直接退出
        fmt.Println("Error Dail", err.Error())
        return
    }

    // 创建一个读标准输入的reader
    reader := bufio.NewReader(os.Stdin)

    // 输入客户端名称
    fmt.Println("Input your name:")
    clientName, _ := reader.ReadString('\n')
    clientName = strings.Trim(clientName, "\n")

    for {
        fmt.Println("What to send? Input Q to quit")

        // 输入传输的数字
        input, _ := reader.ReadString('\n')
        input = strings.Trim(input, "\n")
        if input == "Q" {
            return
        }

        // 向服务端发送数据
        _, err = conn.Write([]byte(clientName + " says: " + input))
    }
}
```

客户端的代码首先建立了一个与服务端的连接，然后从标准输入读取数据发送到客户端。可以建立多个客户端的连接，根据不同的客户端的名字来发送数据，服务端会同时处理各个客户端连接：

```
Starting the server ...
Received data: xiaoxin1 says: Hello, there
Received data: xiaoxin2 says: Hello, Hello
Received data: xiaoxin3 says: Nothing to say
```

## 参考

1. net: <https://golang.org/pkg/net/>
2. bufio: <https://golang.org/pkg/bufio/>
3. strings: <https://golang.org/pkg/strings/>
