# 子域名 / Subdomains

Subdomains are split into two categories, first is the static subdomain and second is the dynamic subdomain.

子域名被分成了两类，第1类是静态子域名，第2类是动态子域名。

* static : when you know the subdomain, usage: `controlpanel.mydomain.com` 
* 静态 : 当你知道子域名时，用例: `controlpanel.mydomain.com`
* dynamic : when you don't know the subdomain, usage: `user1993.mydomain.com`, `otheruser.mydomain.com` 
* 动态 : 当你不知道子域名时, 用例: `user1993.mydomain.com`, `otheruser.mydomain.com` 

Iris has the simplest known form for subdomains, simple as [Parties](party.md).

Iris 拥有已知最简单的子域名形式，就像 [派别](party.md)一样简单。

**Static**

**静态**

```go
package main

import (
    "github.com/kataras/iris"
)

func main() {
    api := iris.New()

    // first the subdomains.
    // 首先是子域名。
    admin := api.Party("admin.")
    {
        // admin.mydomain.com
        admin.Get("/", func(c *iris.Context) {
            c.Write("INDEX FROM admin.mydomain.com")
        })
        // admin.mydomain.com/hey
        admin.Get("/hey", func(c *iris.Context) {
            c.Write("HEY FROM admin.mydomain.com/hey")
        })
        // admin.mydomain.com/hey2
        admin.Get("/hey2", func(c *iris.Context) {
            c.Write("HEY SECOND FROM admin.mydomain.com/hey")
        })
    }

    // mydomain.com/
    api.Get("/", func(c *iris.Context) {
        c.Write("INDEX FROM no-subdomain hey")
    })

    // mydomain.com/hey
    api.Get("/hey", func(c *iris.Context) {
        c.Write("HEY FROM no-subdomain hey")
    })

    api.Listen("mydomain.com:80")
}

```

**Dynamic/Wildcard **

**动态/匹配**

```go

// Package main an example on how to catch dynamic subdomains - wildcard.
// main 包是一个如何抓取动态（匹配）子域名的例子。
// On the first example (subdomains_1) we saw how to create routes for static subdomains, subdomains you know that you will have.
// 在第一个例子中(subdomains_1)我们看到了如何为静态子域名创建路由，你了解这个将要使用的子域名。
// Here we will see an example how to catch unknown subdomains, dynamic subdomains, like username.mydomain.com:8080.
// 这里我们将看一个如何捕捉为止子域名的例子，动态子域名，就行 username.mydomain.com:8000 。

package main

import "github.com/kataras/iris"

// register a dynamic-wildcard subdomain to your server machine(dns/...) first, check ./hosts if you use windows.
// 首先在你的服务器上(dns/...)注册一个动态匹配的子域名，如果你用的是 windows 那么找到 ./hosts 文件。
// run this file and try to redirect: http://username1.mydomain.com:8080/ , http://username2.mydomain.com:8080/ , http://username1.mydomain.com/something, http://username1.mydomain.com/something/sadsadsa
// 打开这个文件并重定向下面的内容: http://username1.mydomain.com:8080/ , http://username2.mydomain.com:8080/ , http://username1.mydomain.com/something, http://username1.mydomain.com/something/sadsadsa

func main() {
    /* Keep note that you can use both of domains now (after 3.0.0-rc.1)
       admin.mydomain.com,  and for other the Party(*.) but this is not this example's purpose
       
       需要注意的是现在你可以同时使用静态和动态子域名了(3.0.0-rc.1以后的版本)
       admin.mydomain.com,  Party(*.) 用作其它子域名，单着不是此示例的目的。

    admin := iris.Party("admin.")
    {
        // admin.mydomain.com
        admin.Get("/", func(c *iris.Context) {
            c.Write("INDEX FROM admin.mydomain.com")
        })
        // admin.mydomain.com/hey
        admin.Get("/hey", func(c *iris.Context) {
            c.Write("HEY FROM admin.mydomain.com/hey")
        })
        // admin.mydomain.com/hey2
        admin.Get("/hey2", func(c *iris.Context) {
            c.Write("HEY SECOND FROM admin.mydomain.com/hey")
        })
    }*/

    dynamicSubdomains := iris.Party("*.")
    {
        dynamicSubdomains.Get("/", dynamicSubdomainHandler)

        dynamicSubdomains.Get("/something", dynamicSubdomainHandler)

        dynamicSubdomains.Get("/something/:param1", dynamicSubdomainHandlerWithParam)
    }

    iris.Get("/", func(ctx *iris.Context) {
        ctx.Write("Hello from mydomain.com path: %s", ctx.PathString())
    })

    iris.Get("/hello", func(ctx *iris.Context) {
        ctx.Write("Hello from mydomain.com path: %s", ctx.PathString())
    })

    iris.Listen("mydomain.com:8080")
}

func dynamicSubdomainHandler(ctx *iris.Context) {
    username := ctx.Subdomain()
    ctx.Write("Hello from dynamic subdomain path: %s, here you can handle the route for dynamic subdomains, handle the user: %s", ctx.PathString(), username)
    // if  http://username4.mydomain.com:8080/ prints:
    // 如果访问 http://username4.mydomain.com:8080/ 将输出:
    // Hello from dynamic subdomain path: /, here you can handle the route for dynamic subdomains, handle the user: username4
}

func dynamicSubdomainHandlerWithParam(ctx *iris.Context) {
    username := ctx.Subdomain()
    ctx.Write("Hello from dynamic subdomain path: %s, here you can handle the route for dynamic subdomains, handle the user: %s", ctx.PathString(), username)
    ctx.Write("THE PARAM1 is: %s", ctx.Param("param1"))
}


```

> You can still set unlimitted number of middleware\/handlers to the dynamic subdomains also
> 
> 你仍然也可以给动态子域名设置无数的中间件/处理器



You noticed the comments  'subdomains\_1' and so on, this is because almost all book's code shots, are running examples.

你注意到了 `'subdomains\_'` 等内容，这是因为几乎所有书中的代码段都是可执行的示例。

You can find them by pressing [here.](https://github.com/iris-contrib/examples)

你可以点击 [这里](https://github.com/iris-contrib/examples) 找到它们。

