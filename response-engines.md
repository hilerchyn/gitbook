## 安装 / Install

Install one response engine and all will be installed.

安装一种应答引擎所有的都会被安装上。

```sh
$ go get -u github.com/iris-contrib/response/json
```

## Iris 的站点配置 ／ Iris' Station configuration


Remember, when 'station' we mean the default `iris.$CALL ` or `api:= iris.New(); api.$CALL` 

记住， 当使用 '站点' 时我们是指 `iris.$CALL` 或 `api:=iris.New(); api.$CALL`

```go
iris.Config.Gzip = true // 压缩后的内容发送给客户端，对模版引擎也一样，默认为 false ／ compressed gzip contents to the client, the same for Template Engines also, defaults to false
iris.Config.Charset = "UTF-8" // 模版引擎也一样默认值为 "UTF-8" / defaults to "UTF-8", the same for Template Engines also
```

They can be overriden for specific `Render` action: 

为特定的 `Render` 动作它们可以被重载: 

```go
func(ctx *iris.Context){
 ctx.Render("any/contentType", anyValue{}, iris.RenderOptions{"gzip":false, "charset": "UTF-8"})
}
```

## How to use

First of all don't be scary about the 'big' article here, a response engine works very simple and is easy to understand how.

首选不被这篇长篇文章吓到，应答引擎工作非常简单，且很容易理解它是如何工作的。

Let's see what are the built'n response by content-type context's methods using the defaults only, unchanged, response engines.

让我们看一下 content-type 的上下文方法使用默认配置，未做任何更改的内建响应。


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


Bellow you will, propably, see how 'good' are my english (joke...), but at the end we're coders and some of us programmers too, so I hope you will be able to understand at least, the code snippets ( a lot of them, you will be tired from this simplicity ).

下面你可能会知道我的英语如何之好（开个玩笑）, 但我们终究是个码农而且有些人还是程序员，我希望你们至少能够通过代码片段来理解(它们大部分都很简单你可能会觉得烦)。




** Text Response Engine**

**文本响应引擎**

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

** Custom response engine**

**自定义响应引擎**

You can create a custom response engine using a func or an interface which implements the

你可以使用实现了下面接口的方法或接口来创建自定义响应引引擎

` iris.ResponseEngine`  包含一个简单的函数: ` Response(val interface{}, options ...map[string]interface{}) ([]byte, error)` 

A custom engine can be used to register a totally new content writer for a known ContentType or for a custom ContentType  

一个自定义引擎可以用来注册一个已知ContentType 或 自定义ContentType 的全新的内容作者

You can imagine its useful, I will show you one right now.

假设这是有用的，我这就给你展示一个。

Let's do a 'trick' here, which works for all other response engines, custom or not:

我们来个对所有其它引擎都起作用的戏法，自定义或不是自定的:

say for example, that you want a static'footer/suffix' on your content.

假如，你想要加一个 'footer/suffix' 到你的内容里。

IF a response engine has the same key and the same content type then the contents are appended and the final result will be rendered to the client.

如果一个响应引擎有一个形同的key和相同的内容类型，那么内容会被追加到后面，并且渲染到客户端。

Let's do this with ` text/plain` content type, because you can see its results easly, the first engine will use this "text/plain" as key, the second & third will use the same, as firsts, key, which is the ContentType also.

我们使用 `text/plain` 内容类型，因为你可以很容易的看到结果，第1个引擎使用 "text/plain" 最为 key， 第2个和第3个也一样，只是第1个的key同样也是ContentType。

```go

package main

import (
	"github.com/iris-contrib/response/text"
	"github.com/kataras/iris"
)

func main() {
	// here we are registering the default text/plain,  and after we will register the 'appender' only
	// 我们注册默认的 text/plain ， 之后我们将只注册 'appender'
	// we have to register the default because we will 
	// 我们得注册默认的因为
    // add more response engines with the same content,
    // 要使用相同的内容添加更多应答引擎，
	// iris will not register this by-default if 
	// iris 不会使用默认的注册
    // other response engine with the corresponding ContentType already exists
    // 如果其它应答引擎相应的ContentType已经存在的话。

	iris.UseResponse(text.New(), text.ContentType) // it's the key which happens to be a valid content-type also, "text/plain" so this will be used as the ContentType header

	// register a response engine: iris.ResponseEngine 
	// 注册一个应答引擎: iris.ResponseEngine
	iris.UseResponse(&CustomTextEngine{}, text.ContentType)
	
    // register a response engine with func
    // 使用函数注册一个应答引擎
	iris.UseResponse(iris.ResponseEngineFunc(func(val interface{}, options ...map[string]interface{}) ([]byte, error) {
		return []byte("\nThis is the static SECOND AND LAST suffix!"), nil
	}), text.ContentType)

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Text(iris.StatusOK, "Hello!") // or ctx.Render(text.ContentType," Hello!")
	})

	iris.Listen(":8080")
}

// This is the way you create one with raw iris.ResponseEngine implementation:
// 这是你使用原生 iris.ResponseEngine 实现一个应答引擎的方式:

// CustomTextEngine the response engine which appends a simple string on the default's text engine
// CustomTextEngine 应答引擎在默认的text引擎后追加一个简单的字符串
type CustomTextEngine struct{}

// Implement the iris.ResponseEngine
// 实现 iris.ResponseEngine
func (e *CustomTextEngine) Response(val interface{}, options ...map[string]interface{}) ([]byte, error) {
	// we don't need the val, because we want only to append, so what we should do?
	// 我们不需要值，因为我们只想追加内容，那么我们应该做什么？
	// just return the []byte we want to be appended after the first registered text/plain engine
	// 只需在第1个注册的 text/plain 引擎之后，返回想要追加的 []byte 内容

	return []byte("\nThis is the static FIRST suffix!"), nil
}

```

** iris.ResponseString **


ResponseString gives you the result of the response engine's work, it doesn't renders to the client but you can use
this function to collect the end result and send it via e-mail to the user, or anything you can imagine.

ResponseString 给你应答引擎工作的结果，它不渲染到客户端，但你可以使用这个函数收集最终结果并通过邮件发送给用户，或任何你能设想到的情况。


```go
package main

import "github.com/kataras/iris"

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
		// let's see
		// 让我们看一下
		// convert markdown string to html and print it to the logger
		// 转换 markdown 字符串为 html 并且将其打引导日志里
		// THIS WORKS WITH ALL RESPONSE ENGINES, but I am not doing the same example for all engines again :) (the same you can do with templates using the iris.TemplateString)
		// 可以和所有的应答引擎工作，但我没有为所有的引擎做示例 (你也可以使用 iris.TemplateString 对模版做同样的事)
		htmlContents := iris.ResponseString("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"}) // default is the iris.Config.Charset, which is UTF-8

		ctx.Log(htmlContents)
		ctx.Write("The Raw HTML is:\n%s", htmlContents)
	})

	iris.Listen(":8080")
}


```


Now we can continue to the rest of the default & built'n response engines

现在我们可以继续介绍 默认和内建 应答引擎的剩余内容了。


** JSON Response Engine **

**JSON 应答引擎**


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
	"github.com/iris-contrib/response/json"
	"github.com/kataras/iris"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// 第1个例子 使用 json 包的配置
	// use the json's Config, we need the import of the json response engine in order to change its internal configs
	// 为了改变它的内部配置，我们需要导入 json 应答引擎
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	// 这是你需要导入默认引擎(模版引擎 或 应答引擎)的原因
	/*
		type Config struct {
			Indent        bool
			UnEscapeHTML  bool
			Prefix        []byte
			StreamingJSON bool
		}
	*/
	iris.UseResponse(json.New(json.Config{
		Prefix: []byte("MYPREFIX"),
	}), json.ContentType) // you can use anything as the second parameter, the json.ContentType is the string "application/json", the context.JSON renders with this engine's key.
	// 你可以使用任何东西做为第2个参数，json.ContentType 就是字符串 "application/json", context.JSON 使用这个引擎key来渲染

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
	// imagine that we need the context.JSON to be listening to our "application/json" response engine with a custom prefix (we did that before)
	// 假设我们需要 context.JSON 被监听到一个带有自定义的前缀 "applicatin/json" 应答引擎
	// but we also want a different renderer, but again application/json content type, with Indent option setted to true:
	// 但我们也想要一个不同的渲染器，但还是 application/json 内容类型，那么把 Indent 选项设置为 true:
	iris.UseResponse(json.New(json.Config{Indent: true}), "json2")("application/json")
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	// 是的 UseResponse 返回一个如果没有声明 key 就可以做内容类型映射
	json2Handler := func(ctx *iris.Context) {
		ctx.Render("json2", myjson{Name: "My iris"})
	}

	iris.Get("/", jsonHandlerSimple)

	iris.Get("/render", jsonHandlerWithRender)

	iris.Get("/json2", json2Handler)

	iris.Listen(":8080")
}

```


** JSONP Response Engine **

**JSONP 应答引擎**

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
	"github.com/iris-contrib/response/jsonp"
	"github.com/kataras/iris"
)

type myjson struct {
	Name string `json:"name"`
}

func main() {
	iris.Config.Charset = "UTF-8" // this is the default, which you can change

	//first example
	// 第1个示例
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	// 这是你需要导入默认引擎(模版引擎 或 应答引擎)的其中一个原因
	/*
		type Config struct {
			Indent   bool
			Callback string // the callback can be override by the context's options or parameter on context.JSONP ／ 回调可以使用 context 选项 或 context.JSONP 参数重载
		}
	*/
	iris.UseResponse(jsonp.New(jsonp.Config{
		Indent: true,
	}), jsonp.ContentType)
	// you can use anything as the second parameter,
	// 你可以使用任何内容类型座位第2个参数，
	// the jsonp.ContentType is the string "application/javascript",
	// jsonp.ContentType 是字符串 "application/javascript"，
	// the context.JSONP renders with this engine's key.
	// context.JSONPY 使用引擎key渲染

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
	iris.UseResponse(jsonp.New(jsonp.Config{Callback: "callbackName"}), "jsonp2")("application/javascript")
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	// 是的 UseResponse 返回一个 如果没有声明key就可以映射内容类型的 函数
	handlerJsonp2 := func(ctx *iris.Context) {
		ctx.Render("jsonp2", myjson{Name: "My iris"})
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/jsonp2", handlerJsonp2)

	iris.Listen(":8080")
}


```



** XML Response Engine **

**XML 应答引擎**


```go
package main

import "github.com/kataras/iris"

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

	"github.com/iris-contrib/response/xml"
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
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	// 这是你需要导入默认引擎(模版引擎 或 应答引擎)的其中一个原因
	/*
		type Config struct {
			Indent bool
			Prefix []byte
		}
	*/
	iris.UseResponse(xml.New(xml.Config{
		Indent: true,
	}), xml.ContentType)
	// you can use anything as the second parameter,
	// 你可以使用任何内容类型座位第2个参数，
	// the jsonp.ContentType is the string "text/xml",
	// jsonp.ContentType 是字符串 "text/xml",
	// the context.XML renders with this engine's key.
	// context.XML 使用这个key渲染。

	handlerSimple := func(ctx *iris.Context) {
		ctx.XML(iris.StatusOK, myxml{First: "first attr", Second: "second attr"})
	}

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		// 你可以使用 RenderOptions 改变某一特定的渲染动作字符编码
		ctx.Render("text/xml", myxml{First: "first attr", Second: "second attr"}, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// 第2个示例，
	// but we also want a different renderer, but again "text/xml" as content type, with prefix option setted by configuration:
	// 虽然我们想要一个不同的渲染器，但为了演示还是使用 "text/xml" 做为内容类型，通过配置来设定 Prefix选项:
	iris.UseResponse(xml.New(xml.Config{Prefix: []byte("")}), "xml2")("text/xml") // if you really use a PREFIX it will be not valid xml, use it only for special cases / 如果你真的用了前缀，那么会变成不可用的xml，只在特殊示例中使用它。
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	// 是的 UseResponse 返回一个 如果没有声明key就可以映射内容类型的 函数
	handlerXML2 := func(ctx *iris.Context) {
		ctx.Render("xml2", myxml{First: "first attr", Second: "second attr"})
	}

	iris.Get("/", handlerSimple)

	iris.Get("/render", handlerWithRender)

	iris.Get("/xml2", handlerXML2)

	iris.Listen(":8080")
}


```


** Markdown Response Engine **

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

	// text/markdown is just the key which the markdown response engine and ctx.Markdown communicate,
	// text/markdown 只是 markdown 应答引擎和ctx.Markdown 交互的key
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
	"github.com/iris-contrib/response/markdown"
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
	// 第1个示例
	// this is one of the reasons you need to import a default engine,(template engine or response engine)
	// 这是你需要导入默认引擎(模版引擎 或 应答引擎)的其中一个原因
	/*
		type Config struct {
			MarkdownSanitize bool
		}
	*/
	iris.UseResponse(markdown.New(), markdown.ContentType)
	// you can use anything as the second parameter,
	// 你可以使用任何内容类型座位第2个参数
	// the markdown.ContentType is the string "text/markdown",
	// markdown.ContentType 是字符串 "text/markdown"，
	// the context.Markdown renders with this engine's key.
	// context.Markdown 渲染器是使用引擎的key。

	handlerWithRender := func(ctx *iris.Context) {
		// you can also change the charset for a specific render action with RenderOptions
		// 你可以使用 RenderOptions 改变某一特定的渲染动作字符编码
		ctx.Render("text/markdown", markdownContents, iris.RenderOptions{"charset": "8859-1"})
	}

	//second example,
	// 第2个示例
	// but we also want a different renderer, but again "text/markdown" as 'content type' (this is converted to text/html behind the scenes), with MarkdownSanitize option setted to true:
	// 虽然我们想使用不同的渲染器，但是为了展示还是使用 "text/markdown" 做为内容类型 (在后台被转换成了 text/html), 将 MarkdownSanitize 选项一起设置为 true:
	iris.UseResponse(markdown.New(markdown.Config{MarkdownSanitize: true}), "markdown2")("text/markdown")
	// yes the UseResponse returns a function which you can map the content type if it's not declared on the key
	// 是的 UseResponse 返回一个 如果没有声明key就可以映射内容类型的 函数
	handlerMarkdown2 := func(ctx *iris.Context) {
		ctx.Render("markdown2", markdownContents, iris.RenderOptions{"gzip": true})
	}

	iris.Get("/", handlerWithRender)

	iris.Get("/markdown2", handlerMarkdown2)

	iris.Listen(":8080")
}


```

** Data(Binary) Response Engine **
**数据(二进制) 应答引擎**


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

 - examples are located [here](https://github.com/iris-contrib/examples/tree/master/response_engines/) 

 - 示例都被放置在 [这里](https://github.com/iris-contrib/examples/tree/master/response_engines/) 

- You can contribute to create more response engines for Iris, click [here](https://github.com/iris-contrib/response) to navigate to the reository.
- 你可以继续为Iris创建更多应答引擎, 点击[这里](https://github.com/iris-contrib/response)导航到相应的源码仓库。
