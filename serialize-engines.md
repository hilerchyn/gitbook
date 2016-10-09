## 安装 / Install

Default Serializers[*](https://github.com/kataras/go-serializers/) are already installed when Iris has been installed.   

默认的序列化器[*](https://github.com/kataras/go-serializers/) 在 Iris 安装时就已经安装好了。

## Iris 的站点配置 ／ Iris' Station configuration


Remember, by 'station' we mean the default `iris.$CALL ` or `api:= iris.New(); api.$CALL` 

记住， 当使用 '站点' 时我们是指 `iris.$CALL` 或 `api:=iris.New(); api.$CALL`

```go
iris.Config.Gzip = true // compresses/gzips response content to the client (same for Template Engines), defaults to false
						// 压缩应答到客户端的内容(Template Engines 也一样), 默认为 false。
iris.Config.Charset = "UTF-8" // defaults to "UTF-8" (same for Template Engines also)
							  // 默认 "UTF-8" (Template Engines 也一样)

// or
iris.Set(iris.OptionGzip(true), iris.OptionCharset("UTF-8"))
// or
iris.New(iris.OptionGzip(true), iris.OptionCharset("UTF-8"))
// or 
iris.New(iris.Configuration{ Gzip:true, Charset: "UTF-8" })
```

They can be overriden for specific `Render` actions: 

他们可以被指定的 'Render' 动作重载:

```go
func(ctx *iris.Context){
 ctx.Render("any/contentType", anyValue{}, iris.RenderOptions{"gzip":false, "charset": "UTF-8"})
}
```

## 如何使用 / How to use

First of all don't be scared about the 'big' article, a serialize engine(serializer, old: Serializer) is very simple and is easy to understand.
Let's see what built-in response types are available in `iris.Context`.

首先不要被这篇长篇大论吓到，序列化引擎 (serializer, old: Serializer) 是非常简单且容易理解的。
让我们看一下 'iris.Context' 中有哪些可用的内建应答类型。


```go
package main

import (
	"encoding/xml"

	"github.com/kataras/iris"
)

type ExampleXml struct {
	XMLName xml.Name `xml:"example"`
	One     string   `xml:"one,attr"`
	Two     string   `xml:"two,attr"`
}

func main() {
	iris.Get("/data", func(ctx *iris.Context) {
		ctx.Data(iris.StatusOK, []byte("Some binary data here."))
	})

	iris.Get("/text", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, "Plain text here")
	})

	iris.Get("/json", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, map[string]string{"hello": "json"}) // or myjsonStruct{hello:"json}
	})

	iris.Get("/jsonp", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", map[string]string{"hello": "jsonp"})
	})

	iris.Get("/xml", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, ExampleXml{One: "hello", Two: "xml"}) // or iris.Map{"One":"hello"...}
	})

	iris.Get("/markdown", func(ctx *iris.Context) {
		ctx.Markdown(iris.StatusOK, "# Hello Dynamic Markdown Iris")
	})

	iris.Listen(":8080")
}


```

**Text Serializer**

**文本序列器**

```go
package main

import "github.com/kataras/iris"

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, you don't have to set it manually / 这是默认值，你没有必要手动设置

	myString := "this is just a simple string which you can already render with ctx.Write"

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, myString)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.Render("text/plain", myString)
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/plain", myString)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.Render("text/plain", myString, iris.RenderOptions{"charset": "UTF-8"}) // default & global charset is UTF-8
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		ctx.MustRender("text/plain", myString)
	})

	iris.Listen(":8080")
}

```

**Custom Serializer**

**自定义序列器**

You can create a custom Serializer using a func or an interface which implements the
` serializer.Serializer`  which contains a simple function: ` Serialize(val interface{}, options ...map[string]interface{}) ([]byte, error)
` 

你可以使用函数或接口创建一个自定义的序列器
只要实现 ' serializeer.Serializer' 包含的一个简单的函数 : ` Serialize(val interface{}, options ...map[string]interface{}) ([]byte, error)
` 


A custom engine can be used to register a totally new content writer for a known ContentType or for a custom ContentType.  

一个自定义引擎可以用来注册一个已知ContentType 或 自定义ContentType 的全新的内容作者

You can imagine its useful, I will show you one right now.

假设这是有用的，我这就给你展示一个。

Let's do a 'trick' here, which works for all other Serializers, custom or not:
say for example, that you want a static'footer/suffix' on your content.

我们来个对所有其它序列器都起作用的戏法，自定义或不是自定的:
假如，你想要加静态文本 'footer/suffix' 到你的内容里。

If a Serializer has the same key and the same content type then the contents are appended and the final result will be rendered to the client
.

如果一个序列器有相同的键和相同的内偶然那个类型，那么这些内容会被追加到一起，且最终结果被渲染到客户端。

Let's do this with the `text/plain` content type, because you can see its results easly.

让我们以 'text/plain' 内容类型演示，因为你可以很容易的看到结果。

```go
// You can create a custom serialize engine(serializer) using a func or an interface which implements the
// 你可以使用函数或接口创建一个自定义的序列化引擎(serializer)
// serializer.Serializer which contains a simple function: Serialize(val interface{}, options ...map[string]interface{}) ([]byte, error)
// 只要实现 serializeer.Serializer 包含的一个简单的函数 : Serialize(val interface{}, options ...map[string]interface{}) ([]byte, error)

// A custom engine can be used to register a totally new content writer for a known ContentType or for a custom ContentType
//一个自定义引擎可以用来注册一个已知ContentType 或 自定义ContentType 的全新的内容作者

// Let's do a 'trick' here, which works for all other serialize engine(serializer)s, custom or not:
// 我们来个对所有其它序列器引擎(serializer)都起作用的戏法，自定义或不是自定的:

// say for example, that you want a static'footer/suffix' on your content, without the need to create & register a middleware for that, per route or globally
// 假如，你想要在你的内容里加上静态的 'footer/suffix', 不需要为每个路由或全局路由创建或注册中间件
// you want to be even more organised.
// 你想要更有组织化的。
//
// IF a serialize engine(serializer) has the same key and the same content type then the contents are appended and the final result will be rendered to the client.
// 如果序列化引擎 （serializer) 用相同的键和相同的内容类型，那么内容将被追加到渲染到客户端的最终结果中。

// Enough with my 'bad' english, let's code something small:
// 真实够了我这差劲的英语，让我们写点代码:

package main

import (
	"github.com/kataras/go-serializer"
	"github.com/kataras/go-serializer/text"
	"github.com/kataras/iris"
)

// Let's do this with ` text/plain` content type, because you can see its results easly, the first engine will use this "text/plain" as key,
// 让我们用 'text/plain' 内容类型来做演示，因为你能很容易的看到结果，第一个引擎将使用 'text/plain' 作为键，
// the second & third will use the same, as firsts, key, which is the ContentType also.
// 第2和第3个使用相同的内容类型作为键。
func main() {
	// we are registering the default text/plain,  and after we will register the 'appender' only
	// 我们注册默认的 text/plain， 然后我们将只注册 'appender'
	// we have to register the default because we will add more serialize engine(serializer)s with the same content,
	// 我们得注册到默认的引擎中，因为我们将使用相同的内容注册更多序列化引擎(serializer)，
	// iris will not register this by-default if other serialize engine(serializer) with the corresponding ContentType already exists
	// 如果已经存在使用相同的ContentType的其它序列化引擎(serializer)那么iris默认将不会注册该类型
	iris.UseSerializer(text.ContentType, text.New())

	// register a serialize engine(serializer) serializer.Serializer
	// 注册一个序列化引擎(serializer) serializer.Serializer
	iris.UseSerializer(text.ContentType, &CustomTextEngine{})
	// register a serialize engine(serializer) with func
	// 使用函数注册一个序列化引擎(serializer)
	iris.UseSerializer(text.ContentType, serializer.SerializeFunc(func(val interface{}, options ...map[string]interface{}) ([]byte, error) {
		return []byte("\nThis is the static SECOND AND LAST suffix!"), nil
	}))

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, "Hello!") // or ctx.Render(text.ContentType," Hello!")
	})

	iris.Listen(":8080")
}

// This is the way you create one with raw serialiser.Serializer implementation:
// 这是你使用原生 serializer.Serializer 创建一个序列化引擎的实现：

// CustomTextEngine the serialize engine(serializer) which appends a simple string on the default's text engine
// CustomTextEngine 追加一个简单的字符串到默认文本引擎的序列化引擎(serializer)
type CustomTextEngine struct{}

// Implement the serializer.Serializer
// 实现 serializer.Serializer
func (e *CustomTextEngine) Serialize(val interface{}, options ...map[string]interface{}) ([]byte, error) {
	// we don't need the val, because we want only to append, so what we should do?
	// 我们不需要值，因为我们只想追加内容，那么我们应该做什么？
	// just return the []byte we want to be appended after the first registered text/plain engine
	// 只需在第1个注册的 text/plain 引擎之后，返回想要追加的 []byte 内容
	return []byte("\nThis is the static FIRST suffix!"), nil
}


```

**iris.SerializeToString**


SerializeToString gives you the result of the Serializer's work, it doesn't renders to the client but you can use
this function to collect the end result and send it via e-mail to the user, or anything you can imagine.

SerializeToString 给你应答引擎工作的结果，它不渲染到客户端，
但你可以使用这个函数收集最终结果并通过邮件发送给用户，或任何你能设想到的情况。


```go
package main

import "github.com/kataras/iris"

func main() {

	// SerializeToString gives you the result of the serialize engine(serializer)'s work, it doesn't renders to the client but you can use
	// SerializeToString 给你应答引擎工作的结果，它不渲染到客户端，
	// this function to collect the end result and send it via e-mail to the user, or anything you can imagine.
	// 但你可以使用这个函数收集最终结果并通过邮件发送给用户，或任何你能设想到的情况。

	// Note that: iris.SerializeToString is called outside of the context, using your iris $instance (iris. is the default)
	// 注意: iris.SerializeToString 是在context之外调用的，使用你的 iris 实例 $instance (默认是 iris.)

	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself against JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Thread safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.
	
	[this is a link](https://github.com/kataras/iris) `

	iris.Get("/", func(ctx *iris.Context) {
		// let's see
		// 让我们看一下
		// convert markdown string to html and print it to the logger
		// 转换 markdown 字符串为 html 并且将其打引导日志里
		// THIS WORKS WITH ALL serialize engine(serializer)S, but I am not doing the same example for all engines again :) (the same you can do with templates using the iris.TemplateString)
		// 所有的应答引擎都可以这样使用，但我没有为所有的引擎写示例 (你也可以使用 iris.TemplateString 对模版做同样的事)
		htmlContents := iris.SerializeToString("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"}) // default is the iris.Config.Charset, which is UTF-8 / 默认是 iris.Config.Charset, 其值为UTF-8

		ctx.Log(htmlContents)
		ctx.Write("The Raw HTML is:\n%s", htmlContents)
	})

	iris.Listen(":8080")
}

```


Now we can continue to the rest of the default & built'n Serializers

现在我们可以继续介绍 默认和内建 序列化引擎的剩余内容了。


**JSON Serializer**

**JSON 序列化引擎**


```go
package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, iris.Map{"name": "iris"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, myjson{Name: "iris"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("application/json", myjson{Name: "iris"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/json", myjson{Name: "iris"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		// 如果有错误就写日志 并 向客户端发送 http 状态 '500 internal server error'
		ctx.MustRender("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}

```


```go
package main

import (
	"github.com/kataras/go-serializer/json"
	"github.com/kataras/iris"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// 第1个示例
	// use the json's Config, we need the import of the json serialize engine(serializer) in order to change its internal configs
	// 使用 json 的配置，为了改变内部配置我们需要导入json序列化引擎(serializer)
	// this is one of the reasons you need to import a default engine,(template engine or serialize engine(serializer))
	// 这是你需要导入默认引擎(模版引擎或序列化引擎serializer)的一个原因
	/*
		type Config struct {
			Indent        bool
			UnEscapeHTML  bool
			Prefix        []byte
			StreamingJSON bool
		}
	*/
	iris.UseSerializer(json.ContentType, json.New(json.Config{
		Prefix: []byte("MYPREFIX"),
	})) // you can use anything as the second parameter, the json.ContentType is the string "application/json", the context.JSON renders with this engine's key.
	// 你可以使用任何数据做为第2个参数，json.ContentType 就是字符串 "application/json", context.JSON 使用这个引擎key来渲染

	jsonHandlerSimple := func(ctx *iris.Context) {
		ctx.JSON(iris.StatusOK, myjson{Name: "iris"})
	}

	jsonHandlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		// 你可以使用 RenderOptions 为某一特定的渲染动作改变字符编码
		ctx.Render("application/json", myjson{Name: "iris"}, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// 第2个示例，
	// imagine that we need the context.JSON to be listening to our "application/json" serialize engine(serializer) with a custom prefix (we did that before)
	// 假设我们需要 context.JSON 被监听到一个带有自定义的前缀(我们之前添加的) "applicatin/json" 序列化引擎
	// but we also want a different renderer, but again application/json content type, with Indent option setted to true:
	// 但是我们也想要一个不同的渲染器，但不同与 application/json 内容类型， 把 Indent 选项设置为 true:
	iris.UseSerializer("json2", json.New(json.Config{Indent: true}))
	json2Handler := func(ctx *iris.Context) {
		ctx.Render("json2", myjson{Name: "My iris"})
		ctx.SetContentType("application/json")
	}

	iris.Get("/", jsonHandlerSimple)

	iris.Get("/render", jsonHandlerWithRender)

	iris.Get("/json2", json2Handler)

	iris.Listen(":8080")
}


```


**JSONP Serializer**

**JSONP 序列化引擎**

```go
package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", iris.Map{"name": "iris"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", myjson{Name: "iris"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		// 如果有错误就写日志 并 向客户端发送 http 状态 '500 internal server error'
		ctx.MustRender("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName", "charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}


```


```go
package main

import (
	"github.com/kataras/go-serializer/jsonp"
	"github.com/kataras/iris"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change / 这是默认值，你可以修改它

	//first example
	// 第1个示例
	// this is one of the reasons you need to import a default engine,(template engine or serialize engine(serializer))
	// 这是你需要导入默认引擎(模版引擎 或 序列化引擎)的其中一个原因
	/*
		type Config struct {
			Indent   bool
			Callback string // the callback can be override by the context's options or parameter on context.JSONP ／ 回调可以使用 context 选项 或 context.JSONP 参数重载
		}
	*/
	iris.UseSerializer(jsonp.ContentType, jsonp.New(jsonp.Config{
		Indent: true,
	}))
	// you can use anything as the second parameter,
	// 你可以使用任何数据作为第2个参数，
	// the jsonp.ContentType is the string "application/javascript",
	// jsonp.ContentType 是字符串 "application/javascript"，
	// the context.JSONP renders with this engine's key.
	// context.JSONP 使用引擎的key渲染

	handlerSimple := func(ctx *iris.Context) {
		ctx.JSONP(iris.StatusOK, "callbackName", myjson{Name: "iris"})
	}

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		// 你可以使用 RenderOptions 改变某一特定的渲染动作字符编码
		ctx.Render("application/javascript", myjson{Name: "iris"}, iris.RenderOptions{"callback": "callbackName", "charset": "8859-1"})
	}

	//second example,
	// 第2个示例，
	// but we also want a different renderer, but again "application/javascript" as content type, with Callback option setted globaly:
	// 虽然我们也想有一个不同的渲染器，但是为了示例再用 "application/javascript" 做为内容类型，使用全局设定的 Callback 选项:
	iris.UseSerializer("jsonp2", jsonp.New(jsonp.Config{Callback: "callbackName"}))
	// yes the UseSerializer returns a function which you can map the content type if it's not declared on the key
	// 是的 UseSerializer 返回一个函数，这个函数如果没有定义键值的话，你可以映射内容类型
	handlerJsonp2 := func(ctx *iris.Context) {
		ctx.Render("jsonp2", myjson{Name: "My iris"})
		ctx.SetContentType("application/javascript")
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/jsonp2", handlerJsonp2)

	iris.Listen(":8080")
}

```



**XML Serializer**

**XML 序列化引擎**


```go
package main

import (
	"encoding/xml"
	"github.com/kataras/iris"
)

type myxml struct {
	XMLName xml.Name `xml:"xml_example"`
	First   string   `xml:"first,attr"`
	Second  string   `xml:"second,attr"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, iris.Map{"first": "first attr ", "second": "second attr"})
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/xml", myxml{First: "first attr", Second: "second attr"})
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		// 如果有错误就写日志 并 向客户端发送 http 状态 '500 internal server error'
		ctx.MustRender("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "UTF-8"})
	})

	iris.Listen(":8080")
}


```


```go
package main

import (
	encodingXML "encoding/xml"

	"github.com/kataras/go-serializer/xml"
	"github.com/kataras/iris"
)

type myxml struct {
	XMLName encodingXML.Name `xml:"xml_example"`
	First   string           `xml:"first,attr"`
	Second  string           `xml:"second,attr"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// 第1个示例
	// this is one of the reasons you need to import a default engine,(template engine or serialize engine(serializer))
	// 这是你需要导入默认引擎(模版引擎 或 序列化引擎)的其中一个原因
	/*
		type Config struct {
			Indent bool
			Prefix []byte
		}
	*/
	iris.UseSerializer(xml.ContentType, xml.New(xml.Config{
		Indent: true,
	}))
	// you can use anything as the second parameter,
	// 你可以使用任何数据作为第2个参数，
	// the jsonp.ContentType is the string "text/xml",
	// jsonp.ContentType 是字符串 "text/xml",
	// the context.XML renders with this engine's key.
	// context.XML 使用这个引擎的key渲染。

	handlerSimple := func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, myxml{First: "first attr", Second: "second attr"})
	}

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		// 你可以使用 RenderOptions 改变某一特定的渲染动作的字符编码
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// 第2个示例，
	// but we also want a different renderer, but again "text/xml" as content type, with prefix option setted by configuration:
	// 虽然我们想要一个不同的渲染器，但为了演示再次使用 "text/xml" 做为内容类型，通过配置来设定 Prefix选项:
	iris.UseSerializer("xml2", xml.New(xml.Config{Prefix: []byte("")})) // if you really use a PREFIX it will be not valid xml, use it only for special cases ／ 如果你真的使用前缀的话xml将会不可用，只在特殊情况下使用它，
	handlerXML2 := func(ctx *iris.Context) {
		ctx.Render("xml2", myxml{First: "first attr", Second: "second attr"})
		ctx.SetContentType("text/xml; charset=" + iris.Config.Charset)
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/xml2", handlerXML2)

	iris.Listen(":8080")
}

```


**Markdown Serializer**

**Markdown 应答引擎**


```go
package main

import "github.com/kataras/iris"

type myjson struct {
	Name string `json:"name"`
}

func main() {
	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself against JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Thread safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.
	
	[this is a link](https://github.com/kataras/iris) `

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Markdown(iris.StatusOK, markdownContents)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		htmlContents := ctx.MarkdownString(markdownContents)
		ctx.HTML(iris.StatusOK, htmlContents)
	})

	// text/markdown is just the key which the markdown serialize engine(serializer) and ctx.Markdown communicate,
	// text/markdown 只是 makrdown 序列化引擎(serializer) 和ctx.Markdown 交互的key
	// it's real content type is text/html
	// 它真实的内容类型是 text/html
	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.Render("text/markdown", markdownContents)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "text/markdown", markdownContents)
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		ctx.Render("text/markdown", markdownContents, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Get("/alternative_5", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		// 如果有错误就写日志 并 向客户端发送 http 状态 '500 internal server error'
		ctx.MustRender("text/markdown", markdownContents, iris.RenderOptions{"charset": "UTF-8"}) // UTF-8 is the default.
	})

	iris.Listen(":8080")
}


```


```go
package main

import (
	"github.com/kataras/go-serializer/markdown"
	"github.com/kataras/iris"
)

func main() {
	markdownContents := `## Hello Markdown from Iris

This is an example of Markdown with Iris



Features
--------

All features of Sundown are supported, including:

*   **Compatibility**. The Markdown v1.0.3 test suite passes with
    the --tidy option.  Without --tidy, the differences are
    mostly in whitespace and entity escaping, where blackfriday is
    more consistent and cleaner.

*   **Common extensions**, including table support, fenced code
    blocks, autolinks, strikethroughs, non-strict emphasis, etc.

*   **Safety**. Blackfriday is paranoid when parsing, making it safe
    to feed untrusted user input without fear of bad things
    happening. The test suite stress tests this and there are no
    known inputs that make it crash.  If you find one, please let me
    know and send me the input that does it.

    NOTE: "safety" in this context means *runtime safety only*. In order to
    protect yourself against JavaScript injection in untrusted content, see
    [this example](https://github.com/russross/blackfriday#sanitize-untrusted-content).

*   **Fast processing**. It is fast enough to render on-demand in
    most web applications without having to cache the output.

*   **Thread safety**. You can run multiple parsers in different
    goroutines without ill effect. There is no dependence on global
    shared state.

*   **Minimal dependencies**. Blackfriday only depends on standard
    library packages in Go. The source code is pretty
    self-contained, so it is easy to add to any project, including
    Google App Engine projects.

*   **Standards compliant**. Output successfully validates using the
    W3C validation tool for HTML 4.01 and XHTML 1.0 Transitional.
	
	[this is a link](https://github.com/kataras/iris) `

	//first example
	// this is one of the reasons you need to import a default engine,(template engine or serialize engine(serializer))
	/*
		type Config struct {
			MarkdownSanitize bool
		}
	*/
	iris.UseSerializer(markdown.ContentType, markdown.New())
	// you can use anything as the second parameter,
	// the markdown.ContentType is the string "text/markdown",
	// the context.Markdown renders with this engine's key.

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		ctx.Render("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// but we also want a different renderer, but again "text/html" as 'content type' (this is the real content type we want to render with, at the first ctx.Render the text/markdown key is converted automatically to text/html without need to call SetContentType), with MarkdownSanitize option setted to true:
	iris.UseSerializer("markdown2", markdown.New(markdown.Config{MarkdownSanitize: true}))
	handlerMarkdown2 := func(ctx *iris.Context) {
		ctx.Render("markdown2", markdownContents, iris.RenderOptions{"gzip": true})
		ctx.SetContentType("text/html")
	}

	iris.Get("/", handlerWithRender)

	iris.Get("/markdown2", handlerMarkdown2)

	iris.Listen(":8080")
}

```

**Data(Binary) Serializer**

**数据(二进制) 序列化引擎**


```go
package main

import "github.com/kataras/iris"

func main() {
	myData := []byte("some binary data or a program here which will not be a simple string at the production")

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Data(iris.StatusOK, myData)
	})

	iris.Get("/alternative_1", func(ctx *iris.Context) {
		ctx.Render("application/octet-stream", myData)
	})

	iris.Get("/alternative_2", func(ctx *iris.Context) {
		ctx.RenderWithStatus(iris.StatusOK, "application/octet-stream", myData)
	})

	iris.Get("/alternative_3", func(ctx *iris.Context) {
		ctx.Render("application/octet-stream", myData, iris.RenderOptions{"gzip": true}) // gzip is false by default
	})

	iris.Get("/alternative_4", func(ctx *iris.Context) {
		// logs if any error and sends http status '500 internal server error' to the client
		// 如果有错误就写日志 并 向客户端发送 http 状态 '500 internal server error'
		ctx.MustRender("application/octet-stream", myData)
	})

	iris.Listen(":8080")
}


```


 ----- 

 - examples are located [here](https://github.com/iris-contrib/examples/tree/master/serialize_engines/).
 - 示例都被放置在 [这里](https://github.com/iris-contrib/examples/tree/master/serialize_engines/).
 - You can contribute to Serializers, click [here](https://github.com/kataras/go-serializer) to navigate to the reository.
 - 你向序列化引擎Serializer共享代码，点击[这里](https://github.com/kataras/go-serializer) 跳转到代码库。
