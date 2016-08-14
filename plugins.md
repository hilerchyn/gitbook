# 插件 / Plugins

Plugins are modules that you can build to inject the Iris' flow. Think it like a middleware for the Iris framework itself, not the requests. Middleware starts it's actions after the server listen and executes itself on every request, Plugin on the other hand starts working when you registered it, it has to do with framework's code, it has access to the *iris.Framework, so it can register routes, start a second server, read the iris' configs or edit them and all things you can do with Iris. Look how it's interface looks:

插件是你创建后可以注入到 Iris 工作流的模块。对于 Iris 框架自身而言它有点像中间件，但不是针对请求的。在服务器监听以后，每次请求都会执行中间件的动作。插件当你注册完它以后就开始工作了，它需要使用框架的代码，它有访问 *iris.Framework 的全新啊，所以它可以注册路由，开启另一台服务器，读取 iris 的配置 或编辑配置，来做所有你可以使用 Iris 来做的所有事情。来看看它的接口是什么样的：

```go
type (
	// Plugin just an empty base for plugins
	// Plugin 只是所有插件的空的基础
	// A Plugin can be added with: .Add(PreListenFunc(func(*Framework))) and so on... or
	// .Add(myPlugin{},myPlugin2{}) which myPlugin is  a struct with any of the methods below or
	//.PostListen(func(*Framework)) and so on...
	// Plugin 可使用如下方式添加:  .Add(PreListenFunc(func(*Framework))) 等 或者 
	// .Add(myPlugin{},myPlugin2{}) myPlugin 是带有下面任何方法的结构体 或者
	// .PostListen(func(*Framework) 等
	Plugin interface {
	}

	// pluginGetName implements the GetName() string method
	// pluginGetName 实现 GetName() string 方法
	pluginGetName interface {
		// GetName has to returns the name of the plugin, a name is unique
		// GetName 必须返回唯一的插件名
		// name has to be not dependent from other methods of the plugin,
		// 名字不能依赖于插件的其它方法
		// because it is being called even before the Activate
		// 因为在启动之前它就被调用了
		GetName() string
	}

	// pluginGetDescription implements the GetDescription() string method
	// pluginGetDescription 实现 GetDescription() string 方法
	pluginGetDescription interface {
		// GetDescription has to returns the description of what the plugins is used for
		// GetDescription() 返回插件使用目的的描述字符串
		GetDescription() string
	}

	// pluginActivate implements the Activate(pluginContainer) error method
	// pluginActivate 实现 Activate(pluginContainer) error 方法
	pluginActivate interface {
		// Activate called BEFORE the plugin being added to the plugins list,
		// Activate 在插件被添加到插件列表前被调用
		// if Activate returns none nil error then the plugin is not being added to the list
		// it is being called only one time
		// 如果 Active 返回 非nill 错误，那么插件没有被添加到列表且只被调用这一次。
		//
		// PluginContainer parameter used to add other plugins if that's necessary by the plugin
		// 如果插件需要其它插件，那么可以使用参数 PluginContainer 来添加
		Activate(PluginContainer) error
	}
	// pluginPreListen implements the PreListen(*Framework) method
	// pluginPreListen 实现 PreListen(*Framework) 方法
	pluginPreListen interface {
		// PreListen it's being called only one time, BEFORE the Server is started (if .Listen called)
		// PreListen 只在服务器被启动(调用 .Listen 方法)之前调用一次
		// is used to do work at the time all other things are ready to go
		// 当所有东西都准备好时才开始工作
		//  parameter is the station
		// 参数是整站对象 iris
		PreListen(*Framework)
	}
	// PreListenFunc implements the simple function listener for the PreListen(*Framework)
	// PreListenFunc 时间一个 简单的 PreListen(*Framework) 监听器函数
	PreListenFunc func(*Framework)
	// pluginPostListen implements the PostListen(*Framework) method
	// pluginPostListen 实现了 PostListen(*Framework) 方法
	pluginPostListen interface {
		// PostListen it's being called only one time, AFTER the Server is started (if .Listen called)
		// PostListen 只在服务器被启动(调用 .Listen)之后调用一次
		// parameter is the station
		// 参数是 iris 站点对象
		PostListen(*Framework)
	}
	// PostListenFunc implements the simple function listener for the PostListen(*Framework)
	// PostListenFunc 实现 PostListen(*Framework) 函数的监听器
	PostListenFunc func(*Framework)
	// pluginPreClose implements the PreClose(*Framework) method
	// pluginPreClose 实现了 PreClose(*Framework) 方法
	pluginPreClose interface {
		// PreClose it's being called only one time, BEFORE the Iris .Close method
		// PreClose 只在 Iris.Close 方法钱被调用一次
		// any plugin cleanup/clear memory happens here
		// 任何插件的内存清理工作在此处进行
		//
		// The plugin is deactivated after this state
		// 插件关闭
		PreClose(*Framework)
	}
	// PreCloseFunc implements the simple function listener for the PreClose(*Framework)
	// PreCloseFunc 实现了 PreClose(*Framework) 简单的监听器函数
	PreCloseFunc func(*Framework)

	// pluginPreDownload It's for the future, not being used, I need to create
	// pluginPreDownload 留待以后使用的，
	// and return an ActivatedPlugin type which will have it's methods, and pass it on .Activate
	// 我需要创建并返回一个拥有自己方法的 ActivatedPlugin 类型，将其传递给 .Activate ，但现在我们返回整个 pluginContainer
	// but now we return the whole pluginContainer, which I can't determinate which plugin tries to
	// download something, so we will leave it here for the future.
	// 我不能断定哪个插件会下载东西，所以我们把它留给以后再处理
	pluginPreDownload interface {
		// PreDownload it's being called every time a plugin tries to download something
		// PreDownload 当每次插件尝试下载东西时都会被调用
		//
		// first parameter is the plugin
		// 第2个参数是插件
		// second parameter is the download url
		// 第2个参数是下载 url
		// must return a boolean, if false then the plugin is not permmited to download this file
		// 必须返回一个布尔值，如果返回 false 那么这个插件不允许下载当前文件
		PreDownload(plugin Plugin, downloadURL string) // bool
	}

	// PreDownloadFunc implements the simple function listener for the PreDownload(plugin,string)
	// PreDownloadFunc 为 PreDownload(plugin,string) 函数实现一个简单的监听器函数
	PreDownloadFunc func(Plugin, string)

)
```

```go
package main

import (
	"fmt"

	"github.com/kataras/iris"
)

func main() {
	// first way:
	// 第1种方式:
	// simple way for simple things
	// 简单的事情用简单的方式来做
	// PreListen before a station is listening ( iris.Listen/TLS...)
	// PreListen 在站点启动监听前 ( iris.Listen/TLS... )
	iris.Plugins.PreListen(func(s *iris.Framework) {
		for _, route := range s.Lookups() {
			fmt.Printf("Func: Route Method: %s | Subdomain %s | Path: %s is going to be registed with %d handler(s). \n", route.Method(), route.Subdomain(), route.Path(), len(route.Middleware()))
		}
	})

	// second way:
	// 第2种方式:
	// structured way for more things
	// 更复杂的事情使用结构体的方式
	plugin := myPlugin{}
	iris.Plugins.Add(plugin)

	iris.Get("/first_route", aHandler)

	iris.Post("/second_route", aHandler)

	iris.Put("/third_route", aHandler)

	iris.Get("/fourth_route", aHandler)

	iris.Listen(":8080")
}

func aHandler(ctx *iris.Context) {
	ctx.Write("Hello from: %s", ctx.PathString())
}

type myPlugin struct{}

// PostListen after a station is listening ( iris.Listen/TLS...)
// PostListen 在站点启动监听后 ( iris.Listen/TLS... )
func (pl myPlugin) PostListen(s *iris.Framework) {
	fmt.Printf("myPlugin: server is listening on host: %s", s.HTTPServer.Host())
}

//list:
//列表:
/*
	Activate(iris.PluginContainer)
	GetName() string
	GetDescription() string
	PreListen(*iris.Framework)
	PostListen(*iris.Framework)
	PreClose(*iris.Framework)
	PreDownload(thePlugin iris.Plugin, downloadUrl string)
	// for custom events
	On(string,...func())
	Call(string)
*/

```

An example of one plugin which is under development is the Iris control, a web interface that gives you control to your server remotely. You can find it's code [here](https://github.com/kataras/iris/tree/master/plugins/iriscontrol).

Iris control 是一个正在开发的插件示例，是一个你可以远程控制你的服务器的web 接口。你可以在[这里](https://github.com/kataras/iris/tree/master/plugins/iriscontrol)找到它的代码。

Take a look at [the real plugins](https://github.com/iris-contrib/plugin), easy to make your own.

看一下[已有的插件](https://github.com/iris-contrib/plugin),你可以很容易创建自己的插件。
