# TLS

```go
// Listen starts the standalone http server
// Listen 启动独立的 HTTP 服务器
// which listens to the addr parameter which as the form of
// 以如下的地址参数形式开始监听
// host:port
//
// It panics on error if you need a func to return an error, use the ListenTo
// Listen 发生错误时会引发恐慌，如果你需要一个返回错误的 func 那么请使用 ListenTo
// ex: err := iris.ListenTo(config.Server{ListeningAddr:":8080"})
// 如: err := iris.ListenTo(config.Server{ListeningAddr:":8080"})
Listen(addr string) 

// ListenTLS Starts a https server with certificates,
// ListenTLS 使用证书启动HTTPS服务器,
// if you use this method the requests of the form of 'http://' will fail
// 如果你使用这个方法时 'http://' 形式的请求将会失败
// only https:// connections are allowed
// 只有 https:// 连接被允许
// which listens to the addr parameter which as the form of
// 以如下的地址参数形式开始监听
// host:port
//
// It panics on error if you need a func to return an error, use the ListenTo
// ListenTLS 发生错误时会引发恐慌，如果需要一个返回错误的函数，那么请使用 ListenTo
// ex: err := iris.ListenTo(":8080","yourfile.cert","yourfile.key")
// 如: err := iris.ListenTo(":8080","yourfile.cert","yourfile.key")
ListenTLS(addr string, certFile string, keyFile string)

// ListenTLSAuto starts a server listening at the specific nat address
// ListenTLSAuto 启动一个监听指定NAT地址的服务器
// using key & certification taken from the letsencrypt.org 's servers
// 使用 letsencrypt.org 服务器的 key 和 certification
// it also starts a second 'http' server to redirect all 'http://$ADDR_HOSTNAME:80' to the' https://$ADDR'
// 同时启动另一台 'http'服务器，用来将 'http://$ADDR_HOSTNAME:80' 的所有请求重置到 'https://$ADDR'
//
// Notes:
// 注意:
// if you don't want the last feature you should use this method:
// 如果你不想用最新的特性，应该使用下面的方法：
// iris.ListenTo(config.Server{ListeningAddr: "mydomain.com:443", AutoTLS: true})
// it's a blocking function
// 它是个阻塞函数
// Limit : https://github.com/iris-contrib/letsencrypt/blob/master/lets.go#L142
// 限制 : https://github.com/iris-contrib/letsencrypt/blob/master/lets.go#L142
//
// example: https://github.com/iris-contrib/examples/blob/master/letsencyrpt/main.go
// 示例: https://github.com/iris-contrib/examples/blob/master/letsencyrpt/main.go
ListenTLSAuto(addr string)

// ListenUNIX starts the process of listening to the new requests using a 'socket file', this works only on unix
// ListenUNIX 用来启动监听 'socket file' 请求的进程，只在 unix 上工作
//
// It panics on error if you need a func to return an error, use the ListenTo
// 此函数发生错误时会引发恐慌，如果你需要返回错误的函数，请使用 ListenTo
// ex: err := iris.ListenTo(":8080", Mode: os.FileMode)
// 如: err := iris.ListenTo(config.Server{":8080", Mode: os.FileMode})
ListenUNIX(addr string, mode os.FileMode)

// ListenVirtual is useful only when you want to test Iris, it doesn't starts the server but it configures and returns it
// ListenVirtual 只有当你想测试 Iris 时才有用，它不启动服务器只是配置并返回服务器
// initializes the whole framework but server doesn't listens to a specific net.Listener
// 初始化整个框架，而服务器不坚定任何指定的 net.Listener
// it is not blocking the app
// 它不是阻塞应用
ListenVirtual(optionalAddr ...string) *Server

// ListenTo listens to a server but acceots the full server's configuration
// ListenTo 启动监听服务器，且接受所有服务器配置
// returns an error, you're responsible to handle that
// 返回错误，由你负责处理
// or use the iris.Must(iris.ListenTo(config.Server{}))
// 或使用 iris.Must(iris.ListenTo(config.Server{})) 来处理
//
// it's a blocking func
// 它是个阻塞函数
ListenTo(cfg config.Server) (err error) 

// Close terminates all the registered servers and returns an error if any
// 任何 Close 函数终止所有注册的服务器并返回错误
// if you want to panic on this error use the iris.Must(iris.Close())
// 如果你想要这个错误引发恐慌，请使用 iris.Must(iris.Close())
Close() error 

```

**上面代码中的ListenTo示例有错误，遗漏了 config.Server。typo: acceots -> accepts**

```go
iris.Listen(":8080")
err := iris.ListenTo(config.Server{ListeningAddr: ":8080"})
```
```go
iris.ListenTLS(":8080", "myCERTfile.cert", "myKEYfile.key")
err := iris.ListenTo(config.Server{ListeningAddr: ":8080", CertFile: "myCERTfile.cert", KeyFile: "myKEYfile.key"})
```

```go
// Package main provide one-line integration with letsencrypt.org
// main 包单行整合了 letsencrypt.org
package main

import "github.com/kataras/iris"

func main() {
	iris.Get("/", func(ctx *iris.Context) {
		ctx.Write("Hello from SECURE SERVER!")
	})

	iris.Get("/test2", func(ctx *iris.Context) {
		ctx.Write("Welcome to secure server from /test2!")
	})

	// This will provide you automatic certification & key from letsencrypt.org's servers
	// 这将自动提供给你来自 letsencrypt.org 服务器的 certification & key 
	// it also starts a second 'http://' server which will redirect all 'http://$PATH' requests to 'https://$PATH'
	// 它启动另一台 'http://' 服务器 用来将 'http://$PATH' 的所有请求重新指向为 'https://$PATH'
	iris.ListenTLSAuto("127.0.0.1:443")
}


```

