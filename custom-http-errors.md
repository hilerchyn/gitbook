# 自定义HTTP错误 / Custom HTTP Errors

You can define your own handlers when http error occurs.

当发生 HTTP 错误时，你可以定义自己的处理器。

```go
package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.OnError(iris.StatusInternalServerError, func(ctx *iris.Context) {
        ctx.Write("CUSTOM 500 INTERNAL SERVER ERROR PAGE")
		iris.Logger.Printf("http status: 500 happened!")
	})

	iris.OnError(iris.StatusNotFound, func(ctx *iris.Context) {
		ctx.Write("CUSTOM 404 NOT FOUND ERROR PAGE")
		iris.Logger.Printf("http status: 404 happened!")
	})

	// emit the errors to test them
	// 发出错误测试
	iris.Get("/500", func(ctx *iris.Context) {
		ctx.EmitError(iris.StatusInternalServerError) // ctx.Panic()
	})

	iris.Get("/404", func(ctx *iris.Context) {
		ctx.EmitError(iris.StatusNotFound) // ctx.NotFound()
	})

	println("Server is running at: 80")
	iris.Listen(":80")

}


```