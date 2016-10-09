# 使用 HandlerFuncs / Using HandlerFuncs

HandlerFuncs should implement the Serve\(*Context) func.
HandlerFunc is the most simple method to register a route or a middleware, but under the hood it acts like a Handler. It implements the Handler interface as well:

HandlerFuncs 应该实现 Serve\(\*Context\) 函数.
HandlerFunc 是注册路由或中间件的最简单方法，而且在幕后其行为像一个 Handler。它也实现了 Handler 接口:

```go
type HandlerFunc func(*Context)

func (h HandlerFunc) Serve(c *Context) {
    h(c)
}
```

HandlerFuncs shoud have this function signature:

HandlerFuncs 应该有这样的函数签名:

```go
func handlerFunc(c *iris.Context)  {
    c.Write("Hello")
}

iris.HandleFunc("GET","/letsgetit",handlerFunc)
//OR
//或
iris.Get("/letsgetit", handlerFunc)
iris.Post("/letspostit", handlerFunc)
iris.Put("/letputit", handlerFunc)
iris.Delete("/letsdeleteit", handlerFunc)
```

