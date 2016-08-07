# 声明 / Declaration

You have wondered this:

你想知道这些:

* Q: Other frameworks need more lines to start a server, why is Iris different?
* 问：别的框架启动一个服务器需要很多行代码，为什么 Iris 如此不同？
* A: Iris gives you the freedom to choose between three ways to use Iris
* 答：Iris 给了你在3种方式中选择并使用Iris的自由

  1. global **iris.** ／ 全局 **iris.**
  2. declare a new iris station with default config: **iris.New\(\)** / 声明具有默认配置的 iris 站点: **iris.New\(\)**
  3. declare a new iris station with custom config: **api := iris.New\(config.Iris{...}\)** / 声明具有自定义配置的 iris 站点: **api := iris.New\(config.Iris{...}\)**


Config can change after declaration with 1&2. `iris.Config.` 3. \/ `api.Config.`

声明以后，可以使用 1 和 2 的 `iris.Config.` / 3 的 `api.Config.` 改变配置

```go
import "github.com/kataras/iris"

// 1.
func firstWay() {

    iris.Get("/home",func(c *iris.Context){})
    iris.Listen(":8080")
}
// 2.
func secondWay() {

    api := iris.New()
    api.Get("/home",func(c *iris.Context){})
    api.Listen(":8080")
}
```

Before looking at the 3rd way, let's take a quick look at the [**config**](configuration.md)**\*\***.Iris\*\*:

开始第3种方式前，让我们快速看一下 [**config**](configuration.md)**\*\***.Iris\*\*:

```go
type (
    // Iris configs for the station
    // 站点的 Iris 配置
    Iris struct {

        // DisablePathCorrection corrects and redirects the requested path to the registed path
        // DisablePathCorrection 纠正并重定向路径为注册的路径
        // for example, if /home/ path is requested but no handler for this Route found,
        // 例如，如果 /home/ 路径被请求，但是没有路由器被发现，
        // then the Router checks if /home handler exists, if yes,
        // 那么路由器会检测 /home 处理器是否存在，如果是的话，
        // (permant)redirects the client to the correct path /home
        // (永久)重定向客户端到正确的路径 /home
        //
        // Default is false
        // 默认值为 false
        DisablePathCorrection bool

        // DisablePathEscape when is false then its escapes the path, the named parameters (if any).
        // DisablePathEscape 为 false 时那么对路径、命名参数（如果有）进行编码。
        // Change to true it if you want something like this https://github.com/kataras/iris/issues/135 to work
        // 如果你想像 https://github.com/kataras/iris/issues/135 此问题描述的内容工作起来的话，那么把它改成 true。
        //
        // When do you need to Disable(true) it:
        // 何时你需要禁用(true)它：
        // accepts parameters with slash '/'
        // 接收带有分割线 '/' 的参数
        // Request: http://localhost:8080/details/Project%2FDelta
        // 请求: http://localhost:8080/details/Project%2FDelta
        // ctx.Param("project") returns the raw named parameter: Project%2FDelta
        // ctx.Param("project") 返回原始命名参数：Project%2FDelta
        // which you can escape it manually with net/url:
        // 你可以使用 net/url 对其手动编码：
        // projectName, _ := url.QueryUnescape(c.Param("project").
        // Look here: https://github.com/kataras/iris/issues/135 for more
        // 更多内容看这里：https://github.com/kataras/iris/issues/135
        //
        // Default is false
        // 默认 false
        DisablePathEscape bool

        // DisableBanner outputs the iris banner at startup
        // DisableBanner 在启动时输出 iris 旗子（标识）
        //
        // Default is false
        // 默认为 false
        DisableBanner bool

        // ProfilePath a the route path, set it to enable http pprof tool
        // ProfilePath 是个路由路径，设置以后会启用 http pprof 工具
        // Default is empty, if you set it to a $path, these routes will handled:
        // 默认为空，如果你把它设置成了一个路径，下面的路由将被处理：
        // $path/cmdline
        // $path/profile
        // $path/symbol
        // $path/goroutine
        // $path/heap
        // $path/threadcreate
        // $path/pprof/block
        // for example if '/debug/pprof'
        // 如以 '/debug/pprof' 为示例
        // http://yourdomain:PORT/debug/pprof/
        // http://yourdomain:PORT/debug/pprof/cmdline
        // http://yourdomain:PORT/debug/pprof/profile
        // http://yourdomain:PORT/debug/pprof/symbol
        // http://yourdomain:PORT/debug/pprof/goroutine
        // http://yourdomain:PORT/debug/pprof/heap
        // http://yourdomain:PORT/debug/pprof/threadcreate
        // http://yourdomain:PORT/debug/pprof/pprof/block
        // it can be a subdomain also, for example, if 'debug.'
        // 也可以是一个子域名，如以 'debug.' 为示例
        // http://debug.yourdomain:PORT/
        // http://debug.yourdomain:PORT/cmdline
        // http://debug.yourdomain:PORT/profile
        // http://debug.yourdomain:PORT/symbol
        // http://debug.yourdomain:PORT/goroutine
        // http://debug.yourdomain:PORT/heap
        // http://debug.yourdomain:PORT/threadcreate
        // http://debug.yourdomain:PORT/pprof/block
        ProfilePath string
        // DisableTemplateEngines set to true to disable loading the default template engine (html/template) and disallow the use of iris.UseEngine
        // DisableTemplateEngines 设置为 true 可以禁用家在默认模版引擎 (html/template) 并且不允许使用 iris.UseEngine
        // default is false
        // 默认为 false
        DisableTemplateEngines bool
        // IsDevelopment iris will act like a developer, for example
        // IsDevelopment iris 会像一个开发者，例如
        // If true then re-builds the templates on each request
        // 如果设置为 true 那么每次请求都会重新构建模版
        // default is false
        // 默认为 false
        IsDevelopment bool

        // Charset character encoding for various rendering
        // Charset 各种渲染引擎的字符编码
        // used for templates and the rest of the responses
        // 用在模版和响应的其它部分
        // defaults to "UTF-8"
        // 默认 "UTF-8"
        Charset string

        // Gzip enables gzip compression on your Render actions, this includes any type of render, templates and pure/raw content
        // Gzip 在 Render 动作中启用 gzip 压缩，包括所有人和类型的 渲染器、模版 和 纯／原始内容（此处指代不理解，估计是指直接返回的文本内容）
        // If you don't want to enable it globaly, you could just use the third parameter on context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
        // 如果你不想全局启用，你只要使用渲染器的第3个参数 context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
        // defaults to false
        // 默认值为 false
        Gzip bool

        // Sessions contains the configs for sessions
        // Sessions 包含所有会话的配置
        Sessions Sessions

        // Websocket contains the configs for Websocket's server integration
        // Websocket 包含了集成 Websocket 服务器的所有配置
        Websocket *Websocket

        // Tester contains the configs for the test framework, so far we have only one because all test framework's configs are setted by the iris itself
        // Tester 包含了测试框架的所有配置，现在为止我们只有一个，因为所有的测试框架的配置都由 iris 自己设置。
        // You can find example on the https://github.com/kataras/iris/glob/master/context_test.go
        // 你可以在  https://github.com/kataras/iris/glob/master/context_test.go 上找到相关示例
        Tester Tester
    }
)
```

```go
// 3.
package main 

import (
  "github.com/kataras/iris"
  "github.com/kataras/iris/config"
)

func main() {
    c := config.Iris{
        ProfilePath:        "/mypath/debug",
    }
    // to get the default: c := config.Default()
    // 获取默认配置: c := config.Default()

    api := iris.New(c)
    api.Listen(":8080")
}

```

> Note that with 2. & 3. you **can define and Listen with more than one Iris server** in the
> same app, when it's necessary.
> 
> 注意使用 2. 和 3. 时，如果需要，你在同一个应用中 **能够定义并监听不止一个 Iris 服务器**

For profiling there are eight \(8\) generated routes with pages filled with info:

对于性能分析有 \(8\) 个生成的路由信息页面:

* \/mypath\/debug\/
* \/mypath\/debug\/cmdline
* \/mypath\/debug\/profile
* \/mypath\/debug\/symbol
* \/mypath\/debug\/goroutine
* \/mypath\/debug\/heap
* \/mypath\/debug\/threadcreate
* \/mypath\/debug\/pprof\/block

* More about configuration [here](configuration.md)
* 更多的配置信息在 [这里](configuration.md)


