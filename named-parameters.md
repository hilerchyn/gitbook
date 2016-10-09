# 命名参数 / Named Parameters

Named parameters are just custom paths for your routes, you can access them for each request 
using context's `c.Param("nameoftheparameter")`. Use `c.Params` to get all values as an array ({K,V}).

There's no limit on how long a path can be.

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
	// Matches /hello/iris,  (if PathCorrection:true match also /hello/iris/)
	// 匹配 /hello/iris，(如果 PathCorrection:true 也会匹配 /hello/iris/)
	// Doesn't match /hello or /hello/ or /hello/iris/something
	// 不会匹配 /hello 或者 /hello/ 或 /hello/iris/somehing
	iris.Get("/hello/:name", func(c *iris.Context) {
		// Retrieve the parameter name
		// 获取参数 name
		name := c.Param("name")
		c.Write("Hello %s", name)
	})

	// Matches /profile/iris/friends/1, (if PathCorrection:true match also /profile/iris/friends/1/)
	// 匹配 /profile/iris/friends/1, (如果 PathCorrection:true 也会匹配 /profile/iris/friends/1/)
	// Doesn't match /profile/ or /profile/iris
	// 不会匹配 /profile/ 、 ／profile/iris ,
	// Doesn't match /profile/iris/friends or  /profile/iris/friends
	// 不会匹配 /profile/iris/friends 、/profile/iris/friends ,
	// Doesn't match /profile/iris/friends/2/something
	// 不会匹配 /profile/iris/friends/2/something
	iris.Get("/profile/:fullname/friends/:friendID", func(c *iris.Context) {
		// Retrieve the parameters fullname and friendID
		// 获取参数 fullname 和 friendID
		fullname := c.Param("fullname")
		friendID, err := c.ParamInt("friendID")
		if err != nil {
			// Do something with the error
			// 处理error
			return
		}
		c.HTML(iris.StatusOK, "<b> Hello </b>"+fullname+"<b> with friends ID </b>"+strconv.Itoa(friendID))
	})

	// Route Example: 
	// 路由示例:
	// /posts/:id and /posts/new conflict with each other for performance reasons and simplicity (dynamic value conficts with the static 'new').   
	// 因为性能和简单性原因 /posts/:id 和 /posts/new (动态值与静态值 'new' 冲突) 相互冲突
	// but if you need to have them you can do following: 
	// 但是如果你需要这样的话可以如下操作：
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
// Will match any request which's url prefix is "/anything/" and has content after that
// 会匹配任何带有 "/anything/" 前缀的 url 请求以及前缀后面的内容
// Matches /anything/whateverhere/whateveragain or /anything/blablabla
// 匹配: /anything/whateverhere/whateveragain 或 /anything/blablabla
// c.Param("randomName") will be /whateverhere/whateveragain, blablabla
// c.Param("randomName") 将是 /whateverhere/whateveragain, blablabla
// Doesn't match /anything or /anything/ or /something
// 不会匹配: /anything 或 /anything/ 或 /something
iris.Get("/anything/*randomName", func(c *iris.Context) { } )
```
