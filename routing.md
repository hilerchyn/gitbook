# 路由 / Routing

As mentioned in the [Handlers](handlers.md) chapter, Iris provides several handler registration methods, each of which returns a [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) type.

正如 [处理器/Handlers](handlers.md) 那章中提到的，Iris 提供了几种注册器的注册方法，他们每一个返回一个 [`RouteNameFunc`](https://godoc.org/github.com/kataras/iris#RouteNameFunc) 类型

## 路由命名 / Route naming

Route naming is easy, since we just call the returned `RouteNameFunc` with a string parameter to define a name:

路由命名很简单，因为我们只需要传递一个字符串参数来调用返回的 `RouteNameFunc`函数来定义一个名字：

```go
package main

import (
	"github.com/kataras/iris"
)

func main() {

	// define a function
	// 定义一个函数
	render := func(ctx *iris.Context) {
		ctx.Render("index.html", nil)
	}

	// handler registration and naming
	// 处理注册和命名
	iris.Get("/", render)("home")
	iris.Get("/about", render)("about")
	iris.Get("/page/:id", render)("page")

	iris.Listen(":8080")
}
```

## 路由反转亦称作从路由名生成 URLs / Route reversing AKA generating URLs from the route name

When we register the handlers for a specific path, we get the ability to create URLs based on the structured data we pass to Iris. In the example above, we've named three routers, one of which even takes parameters. If we're using the default `html/template` templating engine, we can use a simple action to reverse the routes (and generae actual URLs):

当我们为一个特定的路径注册处理器时，我们获得了基于传递给 Iris 结构化数据来创建 URLs 的能力。在下面的例子中，我们命名了3个路由器，他们中的一个还带有参数。如果我们正在使用默认的 `html/template` 模版引擎，可用使用简单的动作来反转路由 (并生成实际的 URLs):

```
Home: {{ url "home" }}
About: {{ url "about" }}
Page 17: {{ url "page" "17" }}
```

Above code would generate the following output:

上面的代码会生成下面的输出内容:

```
Home: http://0.0.0.0:8080/ 
About: http://0.0.0.0:8080/about
Page 17: http://0.0.0.0:8080/page/17
```

## 在代码中使用路由名 / Using route names in code

We can use the following methods/functions to work with named routes (and their parameters):

我们可以使用下面 方法/函数 与命名路由(还有它们的参数)配合:

* global [`Lookups`](https://godoc.org/github.com/kataras/iris#Lookups) function to get all registered routes
* global [`Lookups`](https://godoc.org/github.com/kataras/iris#Lookups) 函数用于获取所有注册的路由
* [`Lookup(routeName string)`](https://godoc.org/github.com/kataras/iris#Framework.Lookup) framework method to retrieve a route by name
* [`Lookup(routeName string)`](https://godoc.org/github.com/kataras/iris#Framework.Lookup) 框架方法用来通过名称获取路由
* [`URL(routeName string, args ...interface{})`](https://godoc.org/github.com/kataras/iris#Framework.URL) framework method to generate url string based on supplied parameters
* [`URL(routeName string, args ...interface{})`](https://godoc.org/github.com/kataras/iris#Framework.URL) 框架方法用来基于提供的参数生成url字符串
* [`Path(routeName string, args ...interface{}`](https://godoc.org/github.com/kataras/iris#Framework.Path) framework method to generate just the path (without host and protocol) portion of the URL based on provided values
* [`Path(routeName string, args ...interface{}`](https://godoc.org/github.com/kataras/iris#Framework.Path) 框架方法用来基于提供的值生成URL的路径(没有主机和协议)部分
* [`RedirectTo(routeName string, args ...interface{})`](https://godoc.org/github.com/kataras/iris#Context.RedirectTo) context method to return a redirect response to a URL defined by the named route and optional parameters
* [`RedirectTo(routeName string, args ...interface{})`](https://godoc.org/github.com/kataras/iris#Context.RedirectTo) 上下文方法用来返回一个由命名路由和可选参数定义的URL作为重定向应答
## 示例 / Examples

Check out the [`template_engines/template_html_4`](https://github.com/iris-contrib/examples/blob/master/template_engines/template_html_4/) example in the `iris-contrib/examples` repository.

检出 [`template_engines/template_html_4`](https://github.com/iris-contrib/examples/blob/master/template_engines/template_html_4/) 例子， 在 `iris-contrib/examples` 代码库中.