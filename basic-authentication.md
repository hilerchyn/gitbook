# 基本身份认证 / Basic Authentication

This is a [middleware](https://github.com/iris-contrib/middleware/tree/master/basicuath).

这是一个[中间件](https://github.com/iris-contrib/middleware/tree/master/basicuath).

HTTP Basic authentication (BA) implementation is the simplest technique for enforcing access controls to web resources because it doesn't require cookies, session identifiers, or login pages; rather, HTTP Basic authentication uses standard fields in the HTTP header, obviating the need for handshakes. Read [more](https://en.wikipedia.org/wiki/Basic_access_authentication).

HTTP 基本身份认证(BA)是实施web资源强制访问控制最简单的技术方法，因为它不需要Cookie、会话标识符或登录页面；相反，HTTP基本身份认证使用标准HTTP头字段，避免了握手的需求。了解[更多](https://en.wikipedia.org/wiki/Basic_access_authentication)。



### 简单示例 / Simple example


```go

package main

import (
	"github.com/iris-contrib/middleware/basicauth"
	"github.com/kataras/iris"
)

func main() {
	authentication := basicauth.Default(map[string]string{"myusername": "mypassword", "mySecondusername": "mySecondpassword"})

	// to global iris.Use(authentication)
	// 用于全局 iris.Use(authentication)
	// to party: iris.Party("/secret", authentication) { ... }
	// 用于派别: iris.Party("/secret", authentication) { ... }

	// to routes
	// 用于路由
	iris.Get("/secret", authentication, func(ctx *iris.Context) {
		username := ctx.GetString("user") // this can be changed, you will see at the middleware_basic_auth_2 folder
		ctx.Write("Hello authenticated user: %s ", username)
	})

	iris.Get("/secret/profile", authentication, func(ctx *iris.Context) {
		username := ctx.GetString("user")
		ctx.Write("Hello authenticated user: %s from localhost:8080/secret/profile ", username)
	})

	iris.Get("/othersecret", authentication, func(ctx *iris.Context) {
		username := ctx.GetString("user")
		ctx.Write("Hello authenticated user: %s from localhost:8080/othersecret ", username)
	})

	iris.Listen(":8080")
}


```

### 可配置示例 / Configurable example

```go
package main

import (
	"time"

	"github.com/iris-contrib/middleware/basicauth"
	"github.com/kataras/iris"
)

func main() {
	authConfig := basicauth.Config{
		Users:      map[string]string{"myusername": "mypassword", "mySecondusername": "mySecondpassword"},
		Realm:      "Authorization Required", // if you don't set it it's "Authorization Required" // 默认为 "Authorization Required" 
		ContextKey: "mycustomkey",            // if you don't set it it's "user" // 默认为 "user"
		Expires:    time.Duration(30) * time.Minute,
	}

	authentication := basicauth.New(authConfig)

	// to global iris.Use(authentication)
	// 用于全局 iris.Use(authentication)
	// to routes
	// 用于路由
	/*
		iris.Get("/mysecret", authentication, func(ctx *iris.Context) {
			username := ctx.GetString("mycustomkey") //  the Contextkey from the authConfig // 配置中的 ContextKey
			ctx.Write("Hello authenticated user: %s ", username)
		})
	*/

	// to party
	// 用于派别

	needAuth := iris.Party("/secret", authentication)
	{
		needAuth.Get("/", func(ctx *iris.Context) {
			username := ctx.GetString("mycustomkey") //  the Contextkey from the authConfig // 配置中的 ContextKey
			ctx.Write("Hello authenticated user: %s from localhost:8080/secret ", username)
		})

		needAuth.Get("/profile", func(ctx *iris.Context) {
			username := ctx.GetString("mycustomkey") //  the Contextkey from the authConfig // 配置中的 ContextKey
			ctx.Write("Hello authenticated user: %s from localhost:8080/secret/profile ", username)
		})

		needAuth.Get("/settings", func(ctx *iris.Context) {
			username := ctx.GetString("mycustomkey") //  the Contextkey from the authConfig // 配置中的 ContextKey
			ctx.Write("Hello authenticated user: %s from localhost:8080/secret/settings ", username)
		})
	}

	iris.Listen(":8080")
}



```