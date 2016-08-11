# 记录器 / Logger

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/logger)

[这是一个中间件](https://github.com/iris-contrib/middleware/tree/master/logger)

Logs the incoming requests

记录传入的请求

```go
 New(theLogger *logger.Logger, config ...Config) iris.HandlerFunc
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

errorLogger := logger.New(iris.Logger, logger.Config{
		EnableColors: false, //enable it to enable colors for all, disable colors by iris.Logger.ResetColors(), defaults to false
		// 所有记录启用颜色标记，通过 iris.Logger.ResetColors() 禁用颜色，默认为 false
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

iris.Use(logger.New(iris.Logger))
*/
func main() {

	iris.Use(logger.New(iris.Logger))

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
	errorLogger := logger.New(iris.Logger)


	iris.OnError(iris.StatusNotFound, func(ctx *iris.Context) {
		errorLogger.Serve(ctx)
		ctx.Write("My Custom 404 error page ")
	})
	//

	iris.Listen(":8080")

}


```

You can create your **own [Logger instance](https://github.com/iris-contrib/logger)** to use

你可以创建 **自己的[Logger实例](https://github.com/iris-contrib/logger)** 来用

```go

import (
    "github.com/iris-contrib/logger"
     mLogger "github.com/iris-contrib/middleware/logger"
)

theLogger := logger.New(logger.DefaultConfig())

iris.Use(mLogger.New(theLogger))
```

>Note that: The logger middleware uses the ColorBgOther and ColorFgOther fields.
>注意: logger 中间件用到了 ColorBgOther 和 ColorFgOther 的字段

The configuration struct for the `iris-contrib/logger ` is the `iris-contrib/logger/config.go`  

`iris-contrib/logger` 的配置结构在 `iris-contrib/logger/config.go`

```go
	Logger struct {
		// Out the (file) writer which the messages/logs will printed to
		// Out 将 消息或日志 输出到的 (文件)写入器
		// Default is os.Stdout
		// 默认是 os.Stdout
		Out *os.File
		// Prefix the prefix for each message
		// Prefix 每个消息的前缀
		// Default is ""
		// 默认为 ""
		Prefix string
		// Disabled default is false
		// Disabled 默认为 false
		Disabled bool

		// foreground colors single SGR Code
		// 前端颜色信息 SGR Code

		// ColorFgDefault the foreground color for the normal message bodies
		// ColorFgDefault 正常消息体的前端颜色
		ColorFgDefault int
		// ColorFgInfo the foreground  color for info messages
		// ColorFgInfo 信息消息的前端颜色
		ColorFgInfo int
		// ColorFgSuccess the foreground color for success messages
		// ColorFgSuccess 成功消息的前端颜色
		ColorFgSuccess int
		// ColorFgWarning the foreground color for warning messages
		// ColorFgWarning 警告消息的前端颜色
		ColorFgWarning int
		// ColorFgDanger the foreground color for error messages
		// ColorFgDanger 错误消息的前端颜色
		ColorFgDanger int
		// OtherFgColor the foreground color for the rest of the message types
		// OtherFgColor 其它类型消息的前端颜色
		ColorFgOther int

		// background colors single SGR Code
		// 后端颜色信号 SGR Code

		// ColorBgDefault the background color for the normal messages
		// ColorBgDefault 正常消息的后端颜色
		ColorBgDefault int
		// ColorBgInfo the background  color for info messages
		// ColorBgInfo 信息消息的后端颜色
		ColorBgInfo int
		// ColorBgSuccess the background color for success messages
		// ColorBgSuccess 成功消息的后端颜色
		ColorBgSuccess int
		// ColorBgWarning the background color for warning messages
		// ColorBgWarning 警告信息的后端颜色
		ColorBgWarning int
		// ColorBgDanger the background color for error messages
		// ColorBgDanger 错误信息的后端颜色
		ColorBgDanger int
		// OtherFgColor the background color for the rest of the message types
		// OtherFgColor 其它消息类型的后端颜色
		ColorBgOther int

		// banners are the force printed/written messages, doesn't care about Disabled field
		
		// Iris 标识是强制打印的消息，Disabled 字段不起作用

		// ColorFgBanner the foreground color for the banner
		// ColorFgBanner 标识的前端颜色
		ColorFgBanner int
	}


```


The `logger.DefaultConfig()`  returns `logger.Config` : 

`logger.DefaultConfig()` 返回 `logger.Config` : 


```go
	return Config{
		Out:      os.Stdout,
		Prefix:   "",
		Disabled: false,
		// foreground colors
		// 前端颜色
		ColorFgDefault: int(color.FgHiWhite),
		ColorFgInfo:    int(color.FgHiCyan),
		ColorFgSuccess: int(color.FgHiGreen),
		ColorFgWarning: int(color.FgHiMagenta),
		ColorFgDanger:  int(color.FgHiRed),
		ColorFgOther:   int(color.FgHiYellow),
		// background colors
		// 后端颜色
		ColorBgDefault: 0,
		ColorBgInfo:    0,
		ColorBgSuccess: 0,
		ColorBgWarning: 0,
		ColorBgDanger:  0,
		ColorBgOther:   0,
		// banner color
		// 标识颜色
		ColorFgBanner: int(color.FgHiBlue),
	}


```
