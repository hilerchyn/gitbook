# 处理器 Handlers

Handlers, as the name implies, handle requests.   
Each of the handler registration methods described in the following subchapters returns a [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type.

Handlers (处理器), 正如名字的所暗示的，用来处理请求。每一个处理器注册的方法都在随后的子章节中做了描述，都返回一个 [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) 类型数据。

Handlers must implement the Handler interface:

所有的处理器必须实现 Handler 接口:

```go
type Handler interface {
	Serve(*Context)
}
```

Once the handler is registered, we can use the returned [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type (which is actually just a `func` type) to give a name to the handler registration for easier lookup in code or in templates.
For more information, checkout the [Routing and reverse lookups](routing.md) section.

一旦处理器被注册，我们就能使用返回的 [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) 类型 (实际上是一个 `func` 类型)，用于给这个注册的处理器起一个名字，便于在代码或模版中使用。
更多信息，请查阅 [路由和反转查询](routing.md) 章节。
