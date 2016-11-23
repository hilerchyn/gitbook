# 中间件 / Middleware

**Quick view**

**快速浏览**

```go
// First mount static files
// 首先挂载静态文件目录
iris.Static("/assets", "./public/assets", 1)

// Then declare which middleware to use (custom or not)
// 然后使用声明的中间件（自定义的或不是自定义的）
iris.Use(myMiddleware{})
iris.UseFunc(func(ctx *iris.Context){})

// declare any finish middleware/ runs always at the end of the request using .Done/.DoneFunc
iris.DoneFunc(executeLast)

// Now declare routes
// 现在声明路由
iris.Get("/myroute", func(c *iris.Context) {
    // do stuff
    // 处理点东西
})
iris.Get("/secondroute", myMiddlewareFunc, myRouteHandlerfunc)

// Now run the server
// 现在运行服务器
iris.Listen(":8080")


// executeLast func middleware 
// executeLast 中间件函数

func executeLast(ctx *iris.Context){
    println("before close the http request")
}

// myMiddleware will be like that
// myMiddleware 看起来像这样

type myMiddleware struct {
  // your 'stateless' fields here
  // 这儿是你的 'stateless' 字段
}

func (m myMiddleware) Serve(ctx *iris.Context){
  // ...
}

```

Middlewares in Iris are not complicated to implement, they are similar to simple Handlers.

They implement the Handler interface as well:

Iris 中的中间件实现起来并不复杂，与简单的 Handlers 相似。它们也实现了 Handler 接口:


```go
type Handler interface {
    Serve(*Context)
}
type Middleware []Handler
```

Handler middleware example:

Handler 中间件示例:

```go

type myMiddleware struct {}

func (m myMiddleware) Serve(c *iris.Context){
    shouldContinueToTheNextHandler := true

    if shouldContinueToTheNextHandler {
        c.Next()
    }else{
        c.Text(403,"Forbidden !!")
    }

}

iris.Use(&myMiddleware{})

iris.Get("/home", func (c *iris.Context){
    c.HTML(iris.StatusOK,"<h1>Hello from /home </h1>")
})

iris.Listen(":8080")
```

HandlerFunc middleware example:

HandlerFunc 中间件示例:

```go

func myMiddleware(c *iris.Context){
    c.Next()
}

iris.UseFunc(myMiddleware)

```

HandlerFunc middleware for a specific route:

指定路由的 HandlerFunc 中间件:

```go

func mySecondMiddleware(c *iris.Context){
    c.Next()
}

iris.Get("/dashboard", func(c *iris.Context) {
    loggedIn := true
    if loggedIn {
        c.Next()
    }
}, mySecondMiddleware, func (c *iris.Context){
    c.Write("The last HandlerFunc is the main handler, everything before that is middleware for this route /dashboard")
})

iris.Listen(":8080")

```

> Note that middlewares must come before route declarations.
> 注意中间件必须在路由声明之前

Make use of the [middleware](https://github.com/iris-contrib/middleware) package, view practical [examples here](https://github.com/iris-contrib/examples).

如何使用 [中间件](https://github.com/iris-contrib/middleware)包, 可以在这里查看实际 [示例](https://github.com/iris-contrib/examples)

```go
package main

import (
 "github.com/kataras/iris"
 "github.com/iris-contrib/middleware/logger"
)

type Page struct {
    Title string
}

iris.Use(logger.New())

iris.Get("/", func(c *iris.Context) {
    c.Render("index.html", Page{"My Index Title"})
})

iris.Listen(":8080")
```

**Done\/DoneFunc**

```go
package main

import "github.com/kataras/iris"

func firstMiddleware(ctx *iris.Context) {
    ctx.Write("1. This is the first middleware, before any of route handlers \n")
    ctx.Next()
}

func secondMiddleware(ctx *iris.Context) {
    ctx.Write("2. This is the second middleware, before the '/' route handler \n")
    ctx.Next()
}

func thirdMiddleware(ctx *iris.Context) {
    ctx.Write("3. This is the 3rd middleware, after the '/' route handler \n")
    ctx.Next()
}

func lastAlwaysMiddleware(ctx *iris.Context) {
    ctx.Write("4. This is ALWAYS the last Handler \n")
}

func main() {

    iris.UseFunc(firstMiddleware)
    iris.DoneFunc(lastAlwaysMiddleware)

    iris.Get("/", secondMiddleware, func(ctx *iris.Context) {
        ctx.Write("Hello from / \n")
        ctx.Next() // .Next because we 're using the third middleware after that, and lastAlwaysMiddleware also
        // 因为在这之后要使用 thirdMiddleware 和 lastAlwaysMiddleware 所以使用 .Next
    }, thirdMiddleware)

    iris.Listen(":8080")

}


```

**Done / DoneFunc with Parties**

**Done / DoneFunc 与 派别 一起使用**

```go
// Package main same as middleware_2 but with party
// main 包与上面的示例一样，只是使用了派别
package main

import "github.com/kataras/iris"

func firstMiddleware(ctx *iris.Context) {
    ctx.Write("1. This is the first middleware, before any of route handlers \n")
    ctx.Next()
}

func secondMiddleware(ctx *iris.Context) {
    ctx.Write("2. This is the second middleware, before the '/' route handler \n")
    ctx.Next()
}

func thirdMiddleware(ctx *iris.Context) {
    ctx.Write("3. This is the 3rd middleware, after the '/' route handler \n")
    ctx.Next()
}

func lastAlwaysMiddleware(ctx *iris.Context) {
    ctx.Write("4. This is ALWAYS the last Handler \n")
}

func main() {

    // with parties:
    // 使用派别:
    myParty := iris.Party("/myparty", firstMiddleware).DoneFunc(lastAlwaysMiddleware)
    {
        myParty.Get("/", secondMiddleware, func(ctx *iris.Context) {
            ctx.Write("Hello from /myparty/ \n")
            ctx.Next() // .Next because we 're using the third middleware after that, and lastAlwaysMiddleware also
        }, thirdMiddleware)

    }

    iris.Listen(":8080")

}


```

> Done\/DoneFuncs are just last-executed handlers, like Use\/UseFunc the children party inheritates these 'done\/last' handlers too.
> Done/DoneFuncs 只是最后执行的处理器，像 Use/UseFunc 子 派别 也继承了 'done/last' 处理器。

