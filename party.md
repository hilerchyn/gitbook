# 派别 / Party

Let's party with Iris web framework!

让我们用Iris网站框架开个派对！

```go
package main

import "github.com/kataras/iris"

func main() {
    // 需要在中间件中添加 ctx.Next() 才能继续后续的逻辑
    admin := iris.Party("/admin", func(ctx *iris.Context){ ctx.Write("Middleware for all party's routes!") })
    {
        // add a silly middleware
        // 加个笨点儿的中间件
        admin.UseFunc(func(c *iris.Context) {
            //your authentication logic here...
            //这儿是你的验证逻辑
            println("from ", c.PathString())
            authorized := true
            if authorized {
                c.Next()
            } else {
                c.Text(401, c.PathString()+" is not authorized for you")
            }

        })
        admin.Get("/", func(c *iris.Context) {
            c.Write("from /admin/ or /admin if you pathcorrection on")
        })
        admin.Get("/dashboard", func(c *iris.Context) {
            c.Write("/admin/dashboard")
        })
        admin.Delete("/delete/:userId", func(c *iris.Context) {
            c.Write("admin/delete/%s", c.Param("userId"))
        })
    }


    beta := admin.Party("/beta")
    beta.Get("/hey", func(c *iris.Context) { c.Write("hey from /admin/beta/hey") })

    iris.Listen(":8080")
}
```

