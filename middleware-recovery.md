# 恢复机制 ／ Recovery

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/recovery).

[这是个中间件](https://github.com/iris-contrib/middleware/tree/master/recovery).

Safety recover the server from panic.

使服务器从恐慌错误中安全恢复。

```go
recovery.New(...*logger.Logger)
```

```go

package main

import (
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/recovery"
)

func main() {

	iris.Use(recovery.New(iris.Logger)) // optional parameter is the logger which the stack of the panic will be printed, here we're using the default station's Logger.
	// 可以打印恐慌错误的记录器是可选参数，这里我们使用的是默认的站点记录器。

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Write("Hi, let's panic")
		panic("errorrrrrrrrrrrrrrr")
	})

	iris.Listen(":8080")
}
```