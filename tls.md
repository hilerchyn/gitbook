# 监听与伺服函数 / Listen & Serve functions

```go
// Serve serves incoming connections from the given listener.
// Serve 函数 伺服来自给定监听器的传入连接
//
// Serve blocks until the given listener returns permanent error.
// Serve 函数会一直阻塞直至给定的监听器返回永久性错误
Serve(ln net.Listener) error

// Listen starts the standalone http server
// Listen 启动独立的 HTTP 服务器
// which listens to the addr parameter which as the form of
// 以如下的地址参数形式开始监听
// host:port
//
// It panics on error if you need a func to return an error, use the Serve
// Listen 发生错误时会引发恐慌，如果你需要一个返回错误的 func 那么请使用 Serve 函数
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
// It panics on error if you need a func to return an error, use the Serve
// 发生错误时它会引发恐慌，如果你需要一个返回错误的 func 那么请使用 Serve 函数
// ex: iris.ListenTLS(":8080","yourfile.cert","yourfile.key")
ListenTLS(addr string, certFile, keyFile string)

// ListenLETSENCRYPT starts a server listening at the specific nat address
// ListenLETSENCRYPT 启动一台监听指定NAT地址的服务器
// using key & certification taken from the letsencrypt.org 's servers
// 使用取自 letsencrypt.org 服务器的 键 和 证书
// it's also starts a second 'http' server to redirect all 'http://$ADDR_HOSTNAME:80' to the' https://$ADDR'
// 它同时也启动了第二个 'http' 服务器用于将所有 'http://$ADDR_HOSTNAME:80' 连接跳转到 'https://$ADDR'
// example: https://github.com/iris-contrib/examples/blob/master/letsencyrpt/main.go
// 示例: https://github.com/iris-contrib/examples/blob/master/letsencyrpt/main.go
ListenLETSENCRYPT(addr string)

// ListenUNIX starts the process of listening to the new requests using a 'socket file', this works only on unix
// ListenUNIX 用来启动监听 'socket file' 请求的进程，只在 unix 上工作
//
// It panics on error if you need a func to return an error, use the Serve
// 此函数发生错误时会引发恐慌，如果你需要返回错误的函数，请使用 Serve 函数
// ex: iris.ListenUNIX(":8080", Mode: os.FileMode)
ListenUNIX(addr string, mode os.FileMode)

// Close terminates all the registered servers and returns an error if any
// 任何 Close 函数终止所有注册的服务器并返回错误
// if you want to panic on this error use the iris.Must(iris.Close())
// 如果你想要这个错误引发恐慌，请使用 iris.Must(iris.Close())
Close() error


// Reserve re-starts the server using the last .Serve's listener
// Reserve 函数使用最后一个 .Serve 函数调用的监听器重启服务器
Reserve() error

// IsRunning returns true if server is running
// IsRunning 函数 如果服务器正在运行就返回 true
IsRunning() bool
```


```go
// Serve
ln, err := net.Listen("tcp4", ":8080")
if err := iris.Serve(ln); err != nil {
   panic(err)
}
// same as api := iris.New(); api.Serve(ln), iris. contains a default iris instance, this exists for
// api := iris.New(); api.Serve(ln) 与上面的用法相同, iris. 含有一个默认的 iris 实例,
// any function or field you will see at the rest of the gitbook.
// 它是为任意函数和字段而存在的，你会在此文档的剩余内容中看到。

// Listen
iris.Listen(":8080")

// ListenTLS
iris.ListenTLS(":8080", "./ssl/mycert.cert", "./ssl/mykey.key")

// and so on...
// 和其它...
```

```go
// Package main provide one-line integration with letsencrypt.org
// main包中提供了单行整合 letsencrypt.org 的示例
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
    // it also starts a second 'http://' server which will redirect all 'http://$PATH'
    // requests to 'https://$PATH'
    iris.ListenLETSENCRYPT("127.0.0.1:443")
}
```

Examples:

示例:

* [Listen using a custom fasthttp server](https://github.com/iris-contrib/examples/tree/master/custom_fasthttp_server)
* [Listen 使用自定义的 fasthttp 服务器](https://github.com/iris-contrib/examples/tree/master/custom_fasthttp_server)
* [Serve content using a custom router](https://github.com/iris-contrib/examples/tree/master/custom_fasthttp_router)
* [Serve 内容使用自定义路由器](https://github.com/iris-contrib/examples/tree/master/custom_fasthttp_router)
* [Listen using a custom net.Listener](https://github.com/iris-contrib/examples/tree/master/custom_net_listener)
* [Listen 使用自定义的 net.Listener](https://github.com/iris-contrib/examples/tree/master/custom_net_listener)
* [Redirect all http://$HOST to https://$HOST using a custom net.Listener](https://github.com/iris-contrib/examples/tree/master/listentls)
* [使用自定义的 net.Listener 将所有 http://$HOST  重定向到 https://$HOST](https://github.com/iris-contrib/examples/tree/master/listentls)
* [Listen using automatic ssl](https://github.com/iris-contrib/examples/tree/master/letsencrypt)
* [Listen 自动使用 ssl](https://github.com/iris-contrib/examples/tree/master/letsencrypt)




