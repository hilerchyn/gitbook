# 使用处理器 API / Using Handler API

HandlerAPI is any custom struct which has an `*iris.Context` field and known methods signatures.

处理器 API 是任何有一个 `*iris.Context` 字段和已知方法签名的自定义结构体。



Before continue I will liked to notice you that this method is slower than `iris.Get, Post..., Handle, HandleFunc`.

在继续之前我得提醒你一下，这个方法比 `iris.Get, Post..., Handle, HandleFunc` 慢。


I know maybe sounds awful but I, my self not using it, I did it because developers used to use frameworks with the 'MVC' pattern, so think it like the 'C\|Controller'. If you don't care about routing performance\(~ms\) and you like to spent some code time, you're free to use it.

我知道它也许听起来很棒，但是我自己不用它，我实现它是因为开发人员曾经以 'MVC' 模式使用框架，所有我想它更像 'C | Controller' 层。如果你不关心路由性能\(~ms\)， 而且你愿意耗费一些代码执行时间，你可以随意使用它。


Instead of writing Handlers\/HandlerFuncs for eachone API routes, you can use the `iris.API` function.

你可以使用 `iris.API` 函数，替代为每个 API 路由而写的 Handlers\/HandlerFuncs 。

```go
API(path string, api HandlerAPI, middleware ...HandlerFunc) error
```

**For example**, for a user API you need some of these routes:

**例如**, 为了实现一个用户 API ，你需要这些路由:

* GET `/users` , for selecting all / 选出所有用户
* GET`/users/:id` , for selecting specific / 选出特定用户
* PUT `/users` , for inserting / 查处新用户
* POST `/users/:id` , for updating / 更新指定用户
* DELETE `/users/:id` , for deleting / 删除指定用户



Normally, with HandlerFuncs you should do something like this:

正常情况下，使用 HandlerFuncs  你应该这样做：

```go
iris.Get("/users", func(ctx *iris.Context){})
iris.Get("/users/:id", func(ctx *iris.Context){ id := ctx.Param("id) })

iris.Put("/users",...)

iris.Post("/users/:id", ...)

iris.Delete("/users/:id", ...)
```

**But** with API you can do this instead:

**然而** 使用 API 你可以这样做:

```go
package main

import (
    "github.com/kataras/iris"
)

type UserAPI struct {
    *iris.Context
}

// GET /users
func (u UserAPI) Get() {
    u.Write("Get from /users")
    // u.JSON(iris.StatusOK,myDb.AllUsers())
}

// GET /:param1 which its value passed to the id argument
// GET /:param1 它的值传递给了 id
func (u UserAPI) GetBy(id string) { // id equals to u.Param("param1") / id 等于 u.Param("param1")
    u.Write("Get from /users/%s", id)
    // u.JSON(iris.StatusOK, myDb.GetUserById(id))

}

// PUT /users
func (u UserAPI) Put() {
    name := u.FormValue("name")
    // myDb.InsertUser(...)
    println(string(name))
    println("Put from /users")
}

// POST /users/:param1
func (u UserAPI) PostBy(id string) {
    name := u.FormValue("name") // you can still use the whole Context's features! / 你仍旧可以使用 Context 的所有特性!
    // myDb.UpdateUser(...)
    println(string(name))
    println("Post from /users/" + id)
}

// DELETE /users/:param1
func (u UserAPI) DeleteBy(id string) {
    // myDb.DeleteUser(id)
    println("Delete from /" + id)
}

func main() {
    iris.API("/users", UserAPI{})
    iris.Listen(":8080")
}

```

As you saw you can still get other request values via the \*iris.Context, API has all the  flexibility of handler\/handlerfunc.

如你所见，你仍然可以通过 \*iris.Context 获取其它请求的值。API 拥有 handler/handlerFunc 的所有灵活性。

If you want to use **more than one named parameter**, simply do this:

如果你想使用 **多有一个的命名参数**，很简单:


```go
// users/:param1/:param2
func (u UserAPI) GetBy(id string, otherParameter string) {}
```

API receives a third parameter which are the middlewares, is optional parameter:

API 接收第3个参数及后续参数作为中间件，这是可选参数：

```go
func main() {
    iris.API("/users", UserAPI{}, myUsersMiddleware1, myUsersMiddleware2)
    iris.Listen(":8080")
}

func myUsersMiddleware1(ctx *iris.Context) {
    println("From users middleware 1 ")
    ctx.Next()
}
func myUsersMiddleware2(ctx *iris.Context) {
    println("From users middleware 2 ")
    ctx.Next()
}

```

Available methods: "GET", "POST", "PUT", "DELETE", "CONNECT", "HEAD", "PATCH", "OPTIONS", "TRACE" should use this **naming conversion**:  **Get\/GetBy, Post\/PostBy, Put\/PutBy** and so on...

可用的方法："GET", "POST", "PUT", "DELETE", "CONNECT", "HEAD", "PATCH", "OPTIONS", "TRACE" 应该使用这样的 **命名转换**: **Get\GetBy, Post/PostBy, Put/PutBy** 等...