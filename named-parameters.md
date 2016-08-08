# 命名参数 / Named Parameters

Named parameters are just custom paths to your routes, you can access them for each request using context's **c.Param("nameoftheparameter")**. Get all, as array (**{Key,Value}**) using **c.Params** property.

命名参数只是路由的自定义路径，你可以使用 context 的 **c.Param("参数的名字")** 来访问每次请求中的内容。获取数组(**{Key,Value}**)形式的所有参数，可以使用 **c.Params** 属性。

No limit on how long a path can be.

路径的长度没有限制。

Usage:

用例：


```go
package main

import (
	"strconv"

	"github.com/kataras/iris"
)

func main() {
	// Match to /hello/iris,  (if PathCorrection:true match also /hello/iris/)
	// 匹配 /hello/iris，(如果 PathCorrection:true 也会匹配 /hello/iris/)
	// Not match to /hello or /hello/ or /hello/iris/something
	// 不会匹配 /hello 或者 /hello/ 或 /hello/iris/somehing
	iris.Get("/hello/:name", func(c *iris.Context) {
		// Retrieve the parameter name
		// 获取参数 name
		name := c.Param("name")
		c.Write("Hello %s", name)
	})

	// Match to /profile/iris/friends/1, (if PathCorrection:true match also /profile/iris/friends/1/)
	// 匹配 /profile/iris/friends/1, (如果 PathCorrection:true 也会匹配 /profile/iris/friends/1/)
	// Not match to /profile/ , /profile/iris ,
	// 不会匹配 /profile/ 、 ／profile/iris ,
	// Not match to /profile/iris/friends,  /profile/iris/friends ,
	// 不会匹配 /profile/iris/friends 、/profile/iris/friends ,
	// Not match to /profile/iris/friends/2/something
	// 不会匹配 /profile/iris/friends/2/something
	iris.Get("/profile/:fullname/friends/:friendID", func(c *iris.Context) {
		// Retrieve the parameters fullname and friendID
		// 获取参数 fullname 和 friendID
		fullname := c.Param("fullname")
		friendID, err := c.ParamInt("friendID")
		if err != nil {
			// Do something with the error
			// 处理error
		}
		c.HTML(iris.StatusOK, "<b> Hello </b>"+fullname+"<b> with friends ID </b>"+strconv.Itoa(friendID))
	})

	/* Example: /posts/:id and /posts/new (dynamic value conficts with the static 'new') for performance reasons and simplicity
	   but if you need to have them you can do that: */
	/* 例如: /posts/:id 和 /posts/new (动态值与静态值 'new' 冲突) 为了性能和简单性，如果你需要这样的路径可以这样做: */

	iris.Get("/posts/*action", func(ctx *iris.Context) {
		action := ctx.Param("action")
		if action == "/new" {
			// it's posts/new page
			// 这是 psots/new 页面
			ctx.Write("POSTS NEW")
		} else {
			ctx.Write("OTHER POSTS")
			// it's posts/:id page
			// 这是 psots/:id 页面
			//doSomething with the action which is the id
			//处理 action 是 id 的内容
		}
	})

	iris.Listen(":8080")
}


```

### 匹配任何东西 / Match anything

```go
// Will match any request which url's preffix is "/anything/" and has content after that
// 会匹配任何带有 "/anything/" 前缀的 url 请求并带有后续内容
iris.Get("/anything/*randomName", func(c *iris.Context) { } )
// Match: /anything/whateverhere/whateveragain , /anything/blablabla
// 匹配: /anything/whateverhere/whateveragain , /anything/blablabla
// c.Param("randomName") will be /whateverhere/whateveragain, blablabla
// c.Param("randomName") 会是 /whateverhere/whateveragain, /blablabla
// Not Match: /anything , /anything/ , /something
// 不会匹配: /anything, /anything/, /something
```
