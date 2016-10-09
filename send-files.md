# 发送文件 / Send files

Send a file, force-download to the client

发送一个文件，强制下载到客户端

```go
// You can define your own "Content-Type" header also, after this function call
// 调用此函数后，你也可以定义自己的 "Content-Type" 头
// for example: ctx.Response.Header.Set("Content-Type","thecontent/type")
// 例如: ctx.Response.Header.Set("Content-Type","thecontent/type")
SendFile(filename string, destinationName string)
```

```go
package main

import "github.com/kataras/iris"

func main() {

    iris.Get("/servezip", func(c *iris.Context) {
        file := "./files/first.zip"
        c.SendFile(file, "saveAsName.zip")
    })

    iris.Listen(":8080")
}
```



You can also send bytes manually, which will be downloaded by the user:

你也可以手动发送字节数据，会由用户自己下载:

```go
package main

import "github.com/kataras/iris"

func main() {

    iris.Get("/servezip", func(c *iris.Context) {
        // read your file or anything
        // 读取你的文件或任何数据
        var binary data[]
        ctx.Data(iris.StatusOK, data)
    })

    iris.Listen(":8080")
}

```
