# 记录器 / Logger

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/logger)

[这是一个中间件](https://github.com/iris-contrib/middleware/tree/master/logger)

Logs the incoming requests

记录传入的请求

```go
 New(config ...Config) iris.HandlerFunc
```

How to use

如何使用

```go
package main

import (
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/logger"
)

/*
With configs:
使用配置:

errorLogger := logger.New(logger.Config{
	// Status displays status code
	// Status 显示状态码
	Status: true,
	// IP displays request's remote address
	// IP 显示请求的远程地址
	IP: true,
	// Method displays the http method
	// Method 显示 HTTP 方法 (GET/POST/PUT/DELETE ... 等)
	Method: true,
	// Path displays the request path
	// Path 显示请求的路径
	Path: true,
})

iris.Use(errorLogger)

With default configs:
使用默认配置:

iris.Use(logger.New())
*/
func main() {

	iris.Use(logger.New())

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Write("hello")
	})

	iris.Get("/1", func(ctx *iris.Context) {
		ctx.Write("hello")
	})

	iris.Get("/2", func(ctx *iris.Context) {
		ctx.Write("hello")
	})

	// log http errors
	// 记录 http 错误
	errorLogger := logger.New()


	iris.OnError(iris.StatusNotFound, func(ctx *iris.Context) {
		errorLogger.Serve(ctx)
		ctx.Write("My Custom 404 error page ")
	})
	//

	iris.Listen(":8080")

}
```
