## 安装 / Install

Install one template engine and all will be installed.

安装一个模版引擎所有的都会被安装上。

```sh
$ go get -u github.com/iris-contrib/template/html
```

## Iris 的站点配置 / Iris' Station configuration 

Remember, when 'station' we mean the default `iris.$CALL ` or `api:= iris.New(); api.$CALL`

记住当使用 '站点' 时我们默认是指 `iris.$CALL`  或者 `api:=iris.New(); api.$CALL`

```go
iris.Config.IsDevelopment = true // reloads the templates on each request, defaults to false / 每次请求都重新加载模版，默认值为 false
iris.Config.Gzip  = true // compressed gzip contents to the client, the same for Response Engines also, defaults to false / 向客户端发送gzip压缩的内容, 对于 应答引擎来说同样如此设置， 默认值为 false
iris.Config.Charset = "UTF-8" // defaults to "UTF-8", the same for Response Engines also / 默认字符编码为 "UTF-8", 应答引擎也同样如此
```

The last two options (Gzip, Charset) can be overriden for specific 'Render' action:

// 最后两个配置选项 (Gzip, Charset) 可以被特定的 'Render' 动作重载：

```go
func(ctx *iris.Context){
    ctx.Render("templateFile.html", anyBindingStruct{}, iris.RenderOptions{"gzip":false, "charset": "UTF-8"})
}
```

## How to use

## 如何使用

Most examples are written for the HTML Template Engine(default and built'n template engine for iris) but works for the rest of the engines also.

大部分示例是为 HTML 模版引擎而写的 (对 iris 而言是默认、内建的模版引擎)，对于其它模版引擎而言也可以参照使用。


You will see first the template file's code, after the main.go code

你既那个在 main.go 代码后，看到第1个模版文件的代码。


** HTML Template Engine, and general **

**HTML模版引擎和常规用法**


```html
<!-- ./templates/hi.html -->

<html>
<head>
<title>Hi Iris [THE TITLE]</title>
</head>
<body>
	<h1>Hi {{.Name}}
</body>
</html>

```

```go
// ./main.go
package main

import "github.com/kataras/iris"

// nothing to do, defaults to ./templates and .html extension, no need to import any template engine because HTML engine is the default
// 对于默认的文件夹 ./templates 和 文件后缀 .html 而言，没有什么额外需要做的，因为是HTML是默认配置所有不需要导入人和模版引擎
// if anything else has been registered
// 即使任何其它模版引擎已经被注册
func main() {
	iris.Config.IsDevelopment = true // this will reload the templates on each request, defaults to false / 每次请求会重新加载模版，默认值为 false
	iris.Get("/hi", hi)
	iris.Listen(":8080")
}

func hi(ctx *iris.Context) {
	ctx.MustRender("hi.html", struct{ Name string }{Name: "iris"})
}


```

```html
<!-- ./templates/layout.html -->
<html>
<head>
<title>My Layout</title>

</head>
<body>
	<h1>Body is:</h1>
	<!-- Render the current template here -->
	<!-- 在这里渲染当前模版 -->
	{{ yield }}
</body>
</html>
```

```html
 <!-- ./templates/mypage.html --> 
<h1>
	Title: {{.Title}}
</h1>
<h3>Message : {{.Message}} </h3>
```

```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/html"
	"github.com/kataras/iris"
)

type mypage struct {
	Title   string
	Message string
}

func main() {

	iris.UseTemplate(html.New(html.Config{
		Layout: "layout.html",
	})).Directory("./templates", ".html") // the .Directory() is optional also, defaults to ./templates, .html
	// Note for html: this is the default iris' templaet engine, if zero engines added, then the template/html will be used automatically
	// HTML模版引擎需要注意: 这是 iris 的默认模版引擎，如果没有添加任何引擎，那么会自动使用 template/html
	// These lines are here to show you how you can change its default configuration
	// 这儿的这几行代码用来展示如何改变默认配置

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("mypage.html", mypage{"My Page title", "Hello world!"}, iris.RenderOptions{"gzip": true})
		// Note that: you can pass "layout" : "otherLayout.html" to bypass the config's Layout property or iris.NoLayout to disable layout on this render action.
		// 注意：你可以通过配置 layout 属性来 传递 "layout" : "otherLayout.html" , 或者使用 iris.NoLayout 来禁用渲染动作的布局.
		// RenderOptions is an optional parameter
		// RenderOptions 是一个可选参数
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/layouts/layout.html -->
<html>
<head>
<title>Layout</title>

</head>
<body>
	<h1>This is the global layout</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```

```html
<!-- ./templates/layouts/mylayout.html -->
<html>
<head>
<title>my Layout</title>

</head>
<body>
	<h1>This is the layout for the /my/ and /my/other routes only</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```

```html
<!-- ./templates/partials/page1_partial1.html -->
<div style="background-color: white; color: red">
	<h1>Page 1's Partial 1</h1>
</div>


```

```html
<!-- ./templates/page1.html -->
<div style="background-color: black; color: blue">

	<h1>Page 1</h1>

	{{ render "partials/page1_partial1.html"}}

</div>


```

```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/html"
	"github.com/kataras/iris"
)

func main() {
	// directory and extensions defaults to ./templates, .html for all template engines
	// 所有模版引擎的默认文件夹为 ./templates 后缀为 .html
	iris.UseTemplate(html.New(html.Config{Layout: "layouts/layout.html"}))
	//iris.Config.Render.Template.Gzip = true
	iris.Get("/", func(ctx *iris.Context) {
		if err := ctx.Render("page1.html", nil); err != nil {
			println(err.Error())
		}
	})

	// remove the layout for a specific route
	// 移除指定路由的布局
	iris.Get("/nolayout", func(ctx *iris.Context) {
		if err := ctx.Render("page1.html", nil, iris.RenderOptions{"layout": iris.NoLayout}); err != nil {
			println(err.Error())
		}
	})

	// set a layout for a party, .Layout should be BEFORE any Get or other Handle party's method
	// 给派别设置部署, .Layout() 方法应该在 Get 或其它处理派别方法之前调用
	my := iris.Party("/my").Layout("layouts/mylayout.html")
	{
		my.Get("/", func(ctx *iris.Context) {
			ctx.MustRender("page1.html", nil)
		})
		my.Get("/other", func(ctx *iris.Context) {
			ctx.MustRender("page1.html", nil)
		})
	}

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/layouts/layout.html -->

<html>
<head>
<title>My Layout</title>

</head>
<body>
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>

```

```html
<!-- ./templates/partials/page1_partial1.html -->
<div style="background-color: white; color: red">
	<h1>Page 1's Partial 1</h1>
</div>


```

```html
<!-- ./templates/page1.html -->
<div style="background-color: black; color: blue">

	<h1>Page 1</h1>

	{{ render "partials/page1_partial1.html"}}

</div>


```

```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/html"
	"github.com/kataras/iris"
)

func main() {
	// directory and extensions defaults to ./templates, .html for all template engines
	// 所有模版引擎的默认文件夹为 ./templates 后缀为 .html
	iris.UseTemplate(html.New(html.Config{Layout: "layouts/layout.html"}))

	iris.Get("/", func(ctx *iris.Context) {
		s := iris.TemplateString("page1.html", nil)
		ctx.Write("The plain content of the template is: %s", s)
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/page.html -->
<a href="{{url "my-page1"}}">http://127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{url "my-page2" "theParam1" "theParam2"}}">http://127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{url "my-page3" "theParam1" "theParam2AfterStatic"}}">http://127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{url "my-page4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}">http://127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />
<a href="{{url "my-page5" "theParam1" "theParam2AfterStatic" "otherParam" "matchAnythingAfterStatic"}}">http://127.0.0.1:8080/mypath5/:param1/statichere/:param2/:otherparam/anything/*anything</a>
<br />
<br />
<a href="{{url "my-page6" .ParamsAsArray }}">http://127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic</a>


```

```go
// ./main.go
// Package main an example on how to naming your routes & use the custom 'url' HTML Template Engine, same for other template engines
// 此事例用来展示如何命名你的路由，并且使用自定义 'url' 的 HTML 模版引擎，其它模版引擎可以此为参考
// we don't need to import the iris-contrib/template/html because iris uses this as the default engine if no other template engine has been registered.
// 我们不需要导入 iris-contrib/template/html 因为如果没有其它模版引擎被注册的话，那么 iris 会使用 HTML 模版引擎座位默认引擎。
package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.Get("/mypath", emptyHandler)("my-page1")
	iris.Get("/mypath2/:param1/:param2", emptyHandler)("my-page2")
	iris.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("my-page3")
	iris.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("my-page4")

	// same with Handle/Func
	// 使用 HandlerFunc 也一样
	iris.HandleFunc("GET", "/mypath5/:param1/statichere/:param2/:otherparam/anything/*anything", emptyHandler)("my-page5")

	iris.Get("/mypath6/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("my-page6")

	iris.Get("/", func(ctx *iris.Context) {
		// for /mypath6...
		paramsAsArray := []string{"theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.html", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")

		println("The full uri of " + routeName + "is: " + iris.URL(routeName))
		// if routeName == "my-page1"
		// 如果 routeName == "my-page1"
		// prints: The full uri of my-page1 is: http://127.0.0.1:8080/mypath
		// 打印 my-page1 路由的全 url: https://127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName)
		// http://127.0.0.1:8080/redirect/my-page1 will redirect to -> http://127.0.0.1:8080/mypath
		// http://127.0.0.1:8080/redirect/my-page1 将重定向到 -> http://127.0.0.1:8080/mypath
	})

	iris.Listen(":8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("Hello from %s.", ctx.PathString())

}


```


```html
<!-- ./templates/page.html -->
<!-- the only difference between normal named routes and dynamic subdomains named routes is that the first argument of  url
is the subdomain part instead of named parameter-->
<!-- 动态子域名命名参数 与 正常的命名路由 唯一的不同是 url 的第一个参数用 子域名的部分内容 代替了 命名参数 -->

<a href="{{url "dynamic-subdomain1" "username1"}}">username1.127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{url "dynamic-subdomain2" "username2" "theParam1" "theParam2"}}">username2.127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{url "dynamic-subdomain3" "username3" "theParam1" "theParam2AfterStatic"}}">username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{url "dynamic-subdomain4" "username4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}">username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />
<a href="{{url "dynamic-subdomain5" .ParamsAsArray }}" >username5.127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic</a>


```
I will add hosts files contens only once, here, you can imagine the rest.

我只在此添加一次 hosts 内容，其它地方如果用到的话请自行添加

**File location is Windows: Drive:/Windows/system32/drivers/etc/hosts, on Linux: /etc/hosts **

**Windows hosts 文件的位置: 系统磁盘分区:/Windows/system32/drivers/etc/hosts, Linux 上的: /etc/hosts**

```sh
# localhost name resolution is handled within DNS itself.
127.0.0.1       localhost
::1             localhost
#-IRIS-For development machine, you have to configure your dns also for online, search google how to do it if you don't know
# 此处在 Iris 本地开发机器上的设定，线上的话你得配置你的DNS，如果不知道怎么弄问问搜索引擎吧

127.0.0.1		username1.127.0.0.1
127.0.0.1		username2.127.0.0.1
127.0.0.1		username3.127.0.0.1
127.0.0.1		username4.127.0.0.1
127.0.0.1		username5.127.0.0.1
# note that you can always use custom subdomains
# 注意你可以一直使用自定义子域名
#-END IRIS-

```

```go
// ./main.go
// Package main same example as template_html_4 but with wildcard subdomains
// 跟上一个示例相同只是使用通配符作为子域名
package main

import (
	"github.com/kataras/iris"
)

func main() {

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
		wildcard.Get("/mypath5/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("dynamic-subdomain5")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part
		// 第1个参数总是子域名
		paramsAsArray := []string{"username5", "theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.html", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// 如果 routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri of dynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		// 打印 dynamic-subdomain1 的全 url: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		// 第2个参数才是真正的参数，第1个参数是动态子域名的子域名，而后面的内容是命名参数
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
		// http://127.0.0.1:8080/redirect/my-subdomain1 将重定向到 ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}


```


** Django Template Engine **

**Django 模版引擎**


```html
<!-- ./templates/mypage.html -->
<html>
<head>
<title>Hello Django from Iris</title>

</head>
<body>
	{% if is_admin %}
	<p>{{username}} is an admin!</p>
	{% endif %}
</body>
</html>

```

```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/django"
	"github.com/kataras/iris"
)

func main() {

	iris.UseTemplate(django.New()).Directory("./templates", ".html")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("mypage.html", map[string]interface{}{"username": "iris", "is_admin": true}, iris.RenderOptions{"gzip": true})
	})

	iris.Listen(":8080")
}


```

```html
<!-- ./templates/page.html -->
<!-- the only difference between normal named routes and dynamic subdomains named routes is that the first argument of  url
is the subdomain part instead of named parameter-->
<!-- 动态子域名命名路由 与 正常命名路由 唯一不通是 url 函数的第1个参数使用子域名替换了命名参数-->
<a href="{{ url("dynamic-subdomain1","username1") }}">username1.127.0.0.1:8080/mypath</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain2","username2","theParam1","theParam2") }}">username2.127.0.0.1:8080/mypath2/:param1/:param2</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain3","username3","theParam1","theParam2AfterStatic") }}" >username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2</a>
<br />
<br />
<a href="{{ url("dynamic-subdomain4","username4","theParam1","theparam2AfterStatic","otherParam","matchAnything") }}" >username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something</a>
<br />
<br />



```

```go
// ./main.go
// Package main same example as template_html_5 but for django/pongo2
// 此例子跟 HTML模版引擎 的例子是一样的，这是针对 django/pongo2 而已
package main

import (
	"github.com/iris-contrib/template/django"
	"github.com/kataras/iris"
)

func main() {
	iris.UseTemplate(django.New())

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// 对于 dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part
		// 第1个参数总是子域名

		if err := ctx.Render("page.html", nil); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// 如果 routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri ofd ynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		// 打印出 dynamic-subdomain1 的全 url: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		// 第2个参数是真正的参数，第1个参数是动态子域名的子域名，之后的是命名参数
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
		// http://127.0.0.1:8080/redirect/my-subdomain1 将重定向到 ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}


```

> Note that, you can see more django examples syntax by navigating [here](https://github.com/flosch/pongo2)
> 
> 注意，你可以到 [这里](https://github.com/flosch/pongo2) 了解更多django模版引擎的示例语法


** Handlebars Template Engine **

**Handlebars 模版引擎**

```html
<!-- ./templates/layouts/layout.html -->

<html>
<head>
<title>Layout</title>

</head>
<body>
	<h1>This is the global layout</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>

```

```html
<!-- ./templates/layouts/mylayout.html -->
<html>
<head>
<title>my Layout</title>

</head>
<body>
	<h1>This is the layout for the /my/ and /my/other routes only</h1>
	<br />
	<!-- Render the current template here -->
	{{ yield }}
</body>
</html>


```


```html
<!-- ./templates/partials/home_partial.html -->
<div style="background-color: white; color: red">
	<h1>Home's' Partial here!!</h1>
</div>


```

```html
<!-- ./templates/home.html -->
<div style="background-color: black; color: white">

	Name: {{boldme Name}} <br /> Type: {{boldme Type}} <br /> Path:
	{{boldme Path}} <br />
	<hr />

	The partial is: {{ render "partials/home_partial.html"}}

</div>

```


```go
// ./main.go
package main

import (
	"github.com/aymerick/raymond"
	"github.com/iris-contrib/template/handlebars"
	"github.com/kataras/iris"
)

type mypage struct {
	Title   string
	Message string
}

func main() {
	// set the configuration for this template engine  (all template engines has its configuration)
	// 给这个模版引擎设置配置信息 (所有的模版引擎都有自己的配置信息)
	config := handlebars.DefaultConfig()
	config.Layout = "layouts/layout.html"
	config.Helpers["boldme"] = func(input string) raymond.SafeString {
		return raymond.SafeString("<b> " + input + "</b>")
	}

	// set the template engine
	// 设置模版引擎
	iris.UseTemplate(handlebars.New(config)).Directory("./templates", ".html") // or .hbs , whatever you want / 或者使用 .hbs，不论你想用什么都可以

	iris.Get("/", func(ctx *iris.Context) {
		// optionally, set a context  for the template
		// 也可以给模版设置一个 context
		ctx.Render("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/"})

	})

	// remove the layout for a specific route using iris.NoLayout
	// 使用 iris.NoLayout 为特定的路由移除布局
	iris.Get("/nolayout", func(ctx *iris.Context) {
		if err := ctx.Render("home.html", nil, iris.RenderOptions{"layout": iris.NoLayout}); err != nil {
			ctx.Write(err.Error())
		}
	})

	// set a layout for a party, .Layout should be BEFORE any Get or other Handle party's method
	// 给派别设置布局，应该在 Get 或 其它处理派别的方法前调用 .Layout
	my := iris.Party("/my").Layout("layouts/mylayout.html")
	{
		my.Get("/", func(ctx *iris.Context) {
			// .MustRender -> same as .Render but logs the error if any and return status 500 on client
			// .MustRender 跟 .Render 一样 只是如果有错误的话就写到日志并返回客户端 500 状态码
			ctx.MustRender("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/my/"})
		})
		my.Get("/other", func(ctx *iris.Context) {
			ctx.MustRender("home.html", map[string]interface{}{"Name": "Iris", "Type": "Web", "Path": "/my/other"})
		})
	}

	iris.Listen(":8080")
}

// Note than you can see more handlebars examples syntax by navigating to https://github.com/aymerick/raymond


```

>  Note than you can see more handlebars examples syntax by navigating [here](https://github.com/aymerick/raymond)
> 
> 你可以在[这里](https://github.com/aymerick/raymond)了解跟多 handlebars 示例语法


** Pug/Jade Template Engine **

**Pug/Jade 模版引擎**


```html
<!-- ./templates/partials/page1_partial1.jade -->
#footer
  p Copyright (c) foobar


```

```html
<!-- ./templates/page.jade -->
doctype html
html(lang=en)
	head
		meta(charset=utf-8)
		title Title
	body
		p ads
		ul
			li The name is {{bold .Name}}.
			li The age is {{.Age}}.

		range .Emails
			div An email is {{.}}

		with .Jobs
			range .
				div.
				 An employer is {{.Employer}}
				 and the role is {{.Role}}

		{{ render "partials/page1_partial1.jade"}}


```

```go
// ./main.go
package main

import (
	"html/template"

	"github.com/iris-contrib/template/pug"
	"github.com/kataras/iris"
)

type Person struct {
	Name   string
	Age    int
	Emails []string
	Jobs   []*Job
}

type Job struct {
	Employer string
	Role     string
}

func main() {
	// set the configuration for this template engine  (all template engines has its configuration)
	// 给这个模版引擎设置配置信息 (所有的模版引擎都有自己的配置信息)
	cfg := pug.DefaultConfig()
	cfg.Funcs["bold"] = func(content string) (template.HTML, error) {
		return template.HTML("<b>" + content + "</b>"), nil
	}

	iris.UseTemplate(pug.New(cfg)).
		Directory("./templates", ".jade")

	iris.Get("/", func(ctx *iris.Context) {

		job1 := Job{Employer: "Super Employer", Role: "Team leader"}
		job2 := Job{Employer: "Fast Employer", Role: "Project managment"}

		person := Person{
			Name:   "name1",
			Age:    50,
			Emails: []string{"email1@something.gr", "email2.anything@gmail.com"},
			Jobs:   []*Job{&job1, &job2},
		}
		ctx.MustRender("page.jade", person)

	})

	iris.Listen(":8080")
}


```



```html
<!-- ./templates/page.jade -->
a(href='{{url "dynamic-subdomain1" "username1"}}') username1.127.0.0.1:8080/mypath
p.
 a(href='{{url "dynamic-subdomain2" "username2" "theParam1" "theParam2"}}') username2.127.0.0.1:8080/mypath2/:param1/:param2

p.
 a(href='{{url "dynamic-subdomain3" "username3" "theParam1" "theParam2AfterStatic"}}') username3.127.0.0.1:8080/mypath3/:param1/statichere/:param2

p.
 a(href='{{url "dynamic-subdomain4" "username4" "theParam1" "theparam2AfterStatic" "otherParam" "matchAnything"}}') username4.127.0.0.1:8080/mypath4/:param1/statichere/:param2/:otherparam/*something

p.
 a(href='{{url "dynamic-subdomain5" .ParamsAsArray }}') username5.127.0.0.1:8080/mypath6/:param1/:param2/staticParam/:param3AfterStatic


```

```go
// ./main.go
// Package main same example as template_html_5 but for pug/jade
package main

import (
	"github.com/iris-contrib/template/pug"
	"github.com/kataras/iris"
)

func main() {
	iris.UseTemplate(pug.New()).Directory("./templates", ".jade")

	wildcard := iris.Party("*.")
	{
		wildcard.Get("/mypath", emptyHandler)("dynamic-subdomain1")
		wildcard.Get("/mypath2/:param1/:param2", emptyHandler)("dynamic-subdomain2")
		wildcard.Get("/mypath3/:param1/statichere/:param2", emptyHandler)("dynamic-subdomain3")
		wildcard.Get("/mypath4/:param1/statichere/:param2/:otherparam/*something", emptyHandler)("dynamic-subdomain4")
		wildcard.Get("/mypath5/:param1/:param2/staticParam/:param3AfterStatic", emptyHandler)("dynamic-subdomain5")
	}

	iris.Get("/", func(ctx *iris.Context) {
		// for dynamic_subdomain:8080/mypath5...
		// 如 dynamic_subdomain:8080/mypath5...
		// the first parameter is always the subdomain part
		// 第1个参数总是子域名
		paramsAsArray := []string{"username5", "theParam1", "theParam2", "theParam3"}

		if err := ctx.Render("page.jade", iris.Map{"ParamsAsArray": paramsAsArray}); err != nil {
			panic(err)
		}
	})

	iris.Get("/redirect/:namedRoute/:subdomain", func(ctx *iris.Context) {
		routeName := ctx.Param("namedRoute")
		subdomain := ctx.Param("subdomain")
		println("The full uri of " + routeName + "is: " + iris.URL(routeName, subdomain))
		// if routeName == "dynamic-subdomain1" && subdomain == "username1"
		// 如果 routeName == "dynamic-subdomain1" && subdomain == "username1"
		// prints: The full uri ofd ynamic-subdomain1 is: http://username1.127.0.0.1:8080/mypath
		// 打印 dynamic-subdomain1 的全uri 路径: http://username1.127.0.0.1:8080/mypath
		ctx.RedirectTo(routeName, subdomain) // the second parameter is the arguments, the first argument for dynamic subdomains is the subdomain part, after this, the named parameters
		//第2个参数是真正的参数，第1个参数为了动态子域名而存在的是子域名，这个参数之后的都是命名参数
		// http://127.0.0.1:8080/redirect/my-subdomain1 will redirect to ->  http://username1.127.0.0.1:8080/mypath
		// http://127.0.0.1:8080/redirect/my-subdomain1 将重定向到 ->  http://username1.127.0.0.1:8080/mypath
	})

	iris.Listen("127.0.0.1:8080")
}

func emptyHandler(ctx *iris.Context) {
	ctx.Write("[SUBDOMAIN: %s]Hello from Path: %s.", ctx.Subdomain(), ctx.PathString())
}

// Note than you can see more Pug/Jade syntax examples by navigating to https://github.com/Joker/jade


```

>  Note than you can see more Pug/Jade syntax examples by navigating [here](https://github.com/Joker/jade)
> 
> 你可以在[这里](https://github.com/Joker/jade)了解更多 Pug/Jade 的语法示例

```html
<!-- ./templates/basic.amber -->
!!! 5
html
    head
        title Hello Amber from Iris

        meta[name="description"][value="This is a sample"]

        script[type="text/javascript"]
            var hw = "Hello #{Name}!"
            alert(hw)

        style[type="text/css"]
            body {
                background: maroon;
                color: white
            }

    body
        header#mainHeader
            ul
                li.active
                    a[href="/"] Main Page
                        [title="Main Page"]
            h1
                 | Hi #{Name}

        footer
            | Hey
            br
            | There


```


```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/amber"
	"github.com/kataras/iris"
)

type mypage struct {
	Name string
}

func main() {

	iris.UseTemplate(amber.New()).Directory("./templates", ".amber")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("basic.amber", mypage{"iris"}, iris.RenderOptions{"gzip": true})
	})

	iris.Listen(":8080")
}


```


** Custom template engine** 

**自定义 模版引擎**

Simply, you have to implement only **3  functions**, for load and execute the templates. One optionally (**Funcs() map[string]interface{}**) which is used to register the iris' helpers funcs like `{{ url }}` and `{{ urlpath }}`.

很简单，为了加载并执行模版，你至少得实现 **3 个函数**。可选函数 (**Funcs()map[string]interface{}**) 用来注册像 `{{ url }}` 和 `{{ urlpath }}` 这样的 iris 的帮助函数。

```go

type (
	// TemplateEngine the interface that all template engines must implement
	// TemplateEngine 是所有模版引擎必须实现的接口
	TemplateEngine interface {
		// LoadDirectory builds the templates, usually by directory and extension but these are engine's decisions
		// LoadDirectory 构建模版，通常通过目录和扩展名来确定，但这都由引擎来决定
		LoadDirectory(directory string, extension string) error
		// LoadAssets loads the templates by binary
		// LoadAssets 通过二进制加载模版
		// assetFn is a func which returns bytes, use it to load the templates by binary
		// assetFn 是一个返回子节的函数，使用它来加载二进制模版
		// namesFn returns the template filenames
		// namesFn 返回模版文件名
		LoadAssets(virtualDirectory string, virtualExtension string, assetFn func(name string) ([]byte, error), namesFn func() []string) error

		// ExecuteWriter finds, execute a template and write its result to the out writer
		// ExecuteWrite 查找、执行一个模版，并将结果写到 out 的 writer 中
		// options are the optional runtime options can be passed by user
		// options 是可选的运行时选项，可以由用户传入
		// an example of this is the "layout" or "gzip" option
		// 这个最好的例子就是 "layout" 或 "gzip" 选项
		ExecuteWriter(out io.Writer, name string, binding interface{}, options ...map[string]interface{}) error
	}

	// TemplateEngineFuncs is optional interface for the TemplateEngine
	// TemplateEngineFuncs 是 TemplateEngine 可选的接口
	// used to insert the Iris' standard funcs, see var 'usedFuncs'
	// 用来插入  Iris 的标准函数，可以看一下 'usedFuncs' 变量
	TemplateEngineFuncs interface {
		// Funcs should returns the context or the funcs,
		// Funcs 应该返回 context 或 一些函数
		// this property is used in order to register the iris' helper funcs
		// 这个属性是用来注册 iris 的帮助函数
		Funcs() map[string]interface{}
	}
)

```

The simplest implementation, which you can look as example, is the Markdown Engine, which is located [here](https://github.com/iris-contrib/template/tree/master/markdown/markdown.go).

你可以作为示例看的最简单实现就是 Markdown 引擎，在 [这里](https://github.com/iris-contrib/template/tree/master/markdown/markdown.go)。



** iris.TemplateString **

**iris.TemplateString**


Executes and parses the template but instead of rendering to the client, it returns the contents. Useful when you want to send a template via e-mail or anything you can imagine.

执行并解析模版而不渲染到客户端，它只返回内容。当你想要通过电子邮件发送一个模版时或你能想到的任何其它情况下时用用的

```html
<!-- ./templates/mypage.html -->
<html>
<head>
<title>Hello Django from Iris</title>

</head>
<body>
	{% if is_admin %}
	<p>{{username}} is an admin!</p>
	{% endif %}
</body>
</html>

```


```go
// ./main.go
package main

import (
	"github.com/iris-contrib/template/django"
	"github.com/kataras/iris"
)

func main() {
	iris.UseTemplate(django.New()).Directory("./templates", ".html")

	iris.Get("/", func(ctx *iris.Context) {
		// THIS WORKS WITH ALL TEMPLATE ENGINES, but I am not doing the same example for all engines again :) (the same you can do with templates using the iris.ResponseString)
		// 所有的模版引擎都可以工作，但我没有给所有的引擎做同样的示例 :)(你同样可以使用 iris.Response)
		rawHtmlContents := iris.TemplateString("mypage.html", map[string]interface{}{"username": "iris", "is_admin": true}, iris.RenderOptions{"charset": "UTF-8"}) // defaults to UTF-8 already
		ctx.Log(rawHtmlContents)
		ctx.Write("The Raw HTML is:\n%s", rawHtmlContents)
	})

	iris.Listen(":8080")
}


```

 > Note that: iris.TemplateString can be called outside of the context also 
 >
 > 注意: iris.TemplateString 可以在外面调用




-----


 - examples are located [here](https://github.com/iris-contrib/examples/tree/master/template_engines/) 
 - 例子在 [这里](https://github.com/iris-contrib/examples/tree/master/template_engines/) 

- You can contribute to create more template engines for Iris, click [here](https://github.com/iris-contrib/template) to navigate to the reository. 
- 你可以为 Iris 创建更多模版引擎，点击［这里］调转到代码库。