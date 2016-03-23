---
layout: post
title: "Golang格式化json数据"
description: "Go语言对json数据进行编码和解码，利用encoding/json包"
category: golang
tags: [golang, json, 序列化, encoding/json]
comments: true
---

## 直接把结构体编码成json数据

```golang
package main

import (
    "encoding/json"
    "fmt"
    _ "os"
)

type Address struct {
    Type string
    City string
    Country string
}

type Card struct {
    Name string
    Age int
    Addresses []*Address
}

func main() {
    pa := &Address{"private", "Shanghai", "China"}
    pu := &Address{"work", "Beijing", "China"}
    c := Card{"Xin", 32, []*Address{pa, pu}}

    js, _ := json.Marshal(c)
    fmt.Printf("Json: %s", js)
}
```

利用[`json.Marshal`](https://golang.org/pkg/encoding/json/#Marshal)，可将结构体转为JSON数据：

```json
Json: {"Name":"Xin","Age":32,"Addresses":[{"Type":"private","City":"Shanghai","Country":"China"},{"Type":"work","City":"Beijing","Country":"China"}]}
```

而`json.MarshalIndent(c, "", "  ")`可以将json代码格式化：

```json
Json: {
  "Name": "Xin",
  "Age": 32,
  "Addresses": [
    {
      "Type": "private",
      "City": "Shanghai",
      "Country": "China"
    },
    {
      "Type": "work",
      "City": "Beijing",
      "Country": "China"
    }
  ]
}
```

### Tips

1. 在 web 应用中最好使用`json.MarshalforHTML()`函数，会对数据执行HTML转码。
2. map 的 key 必须是字符串类型。
3. Channel，复杂类型和函数类型不能被编码。
4. 指针可以被编码，实际上是对指针指向的值进行编码.

## 解码到数据结构

如果事先知道 JSON 数据的结构，可以事先定义一个结构来存储反序列化后的结果。如，对这样一段 json:

```json
b := []byte(`{"Name": "Wednesday", "Age": 6, "Parents": ["Gomez", "Morticia"]}`)
```

我们定义这样一个数据结构：

```golang
type FamilyMember struct {
	Name    string
	Age     int
	Parents []string
}
```

然后可以将其反序列化：

```golang
var m FamilyMember
err := json.Unmarshal(b, &m)
```

完整的程序：

```golang
package main

import (
    "encoding/json"
    "fmt"
)

type FamilyMember struct {
    Name    string
    Age     int
    Parents []string
}

func main() {
    b := []byte(`{"Name": "Wednesday", "Age": 6, "Parents": ["Gomez", "Morticia"]}`)
    var m FamilyMember
    // json.Unmarshal 用于解码 json 串
    err := json.Unmarshal(b, &m)
    if err == nil {
        fmt.Printf("name: %s\nAge: %d\nParents: %v\n", m.Name, m.Age, m.Parents)
    }
}
```

执行后输出：

```
name: Wednesday
Age: 6
Parents: [Gomez Morticia]
```

## 解码任意数据

json 包使用 map[string]interface{} 和 []interface{} 储存任意的 JSON 对象和数组。同样是上面的 JSON 串：

```json
b := []byte(`{"Name": "Wednesday", "Age": 6, "Parents": ["Gomez", "Morticia"]}`)
```

用下面的代码进行解码：

```golang
var f interface{}
err := json.Unmarshal(b, &f)
```

会得到：

```golang
map[string]interface{} {
    "Name": "Wednesday",
    "Age":  6,
    "Parents": []interface{} {
        "Gomez",
        "Morticia",
    },
}
```

对于这类的数据，可以使用`switch`来判断节点的类型来遍历数据。

## 参考

1. encoding/json: <https://golang.org/pkg/encoding/json/>
