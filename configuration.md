# 配置 / Configuration

All configurations have default values, things will work as you expected with `iris.New()` 

所有的配置都有默认值，使用'iris.New()'所有代码都会如你期望的正常运行。

but if you want to customize iris then pass `iris.New(iris.Configuration{})` or use the options-func, example: `iris.New(iris.OptionCharset("UTF-8"))`

如果你想要自定义iris那么使用选项函数如：iris.New(iris.OptionCharset("UTF-8")或iris.New(iris.Configuration{}) 

`.New` **by configuration**

`.New` **使用配置**
```go
import "github.com/kataras/iris"
//...

myConfig := iris.Configuration{Charset: "UTF-8", IsDevelopment:true, Sessions: iris.SessionsConfiguration{Cookie:"mycookie"}, Websocket: iris.WebsocketConfiguration{Endpoint: "/my_endpoint"}}
iris.New(myConfig)
```

`.New` **by options**

`.New` **使用选项**

```go
import "github.com/kataras/iris"
//...

iris.New(iris.OptionCharset("UTF-8"), iris.OptionIsDevelopment(true), iris.OptionSessionsCookie("mycookie"), iris.OptionWebsocketEndpoint("/my_endpoint"))

// if you want to set configuration after the .New use the .Set: 
// 如果你想在.New之后设定配置可以使用.Set：
iris.Set(iris.OptionDisableBanner(true))
```


### List of available options

### 可用的选项列表

```go
// OptionDisablePathCorrection corrects and redirects the requested path to the registed path
// OptionDisablePathCorrection 矫正并重定向请求路径到注册的路径
// for example, if /home/ path is requested but no handler for this Route found,
// 如果请求路径为 /home/ 但没有发现此路由个的处理器，
// then the Router checks if /home handler exists, if yes,
// 那么Router会检查 /home 处理器是否存在，如果是，
// (permant)redirects the client to the correct path /home
// 永久重定向客户端到正确的路径 /home
//
// Default is false
// 默认为 false
OptionDisablePathCorrection(val bool)

// OptionDisablePathEscape when is false then its escapes the path, the named parameters (if any).
// OptionDisablePathEscape 当为false时它不再解析路径、命名参数(如果有的话)
OptionDisablePathEscape(val bool)

// OptionDisableBanner outputs the iris banner at startup
// OptionDisableBanner 启动时输出iris横幅
//
// Default is false
// 默认为 false
OptionDisableBanner(val bool)

// OptionLoggerOut is the destination for output
// OptionLoggerOut 输出的目的地
//
// Default is os.Stdout
// 默认为 os.Stdout
OptionLoggerOut(val io.Writer)

// OptionLoggerPreffix is the logger's prefix to write at beginning of each line
// OptionLoggerPreffix 是日志器写入每行日之前的前缀
//
// Default is [IRIS]
// 默认为 [IRIS]
OptionLoggerPreffix(val string)

// OptionProfilePath a the route path, set it to enable http pprof tool
// OptionProfilePath 是路由路径，设置它可以启用http pprof工具
// Default is empty, if you set it to a $path, these routes will handled:
// 默认为空，如果你设置其为一个路径，这些路由将会被处理：
OptionProfilePath(val string)

// OptionDisableTemplateEngines set to true to disable loading the default template engine (html/template) and disallow the use of iris.UseEngine
// OptionDisableTemplateEngines 设置为true那么将禁用加载默认的模版引擎（html/template)，并且不允许使用iris.UseEngine
// Default is false
// 默认为 false
OptionDisableTemplateEngines(val bool)

// OptionIsDevelopment iris will act like a developer, for example
// OptionIsDevelopment iris 的动作像一个开发人员，
// If true then re-builds the templates on each request
// 如果为true那么每次请求将重建模版
// Default is false
// 默认为false
OptionIsDevelopment(val bool)

// OptionTimeFormat time format for any kind of datetime parsing
// OptionTimeFormat 任意日期时间解析用到的时间格式
OptionTimeFormat(val string)

// OptionCharset character encoding for various rendering
// OptionCharset 各种渲染的字符编码
// used for templates and the rest of the responses(via serializer)
// 用在模版和其它部分的应答（通过serializer)
// Default is "UTF-8"
// 默认为 "UTF-8"
OptionCharset(val string)

// OptionGzip enables gzip compression on your Render actions, this includes any type of render, templates and pure/raw content
// OptionGzip 在你的渲染动作中启用gzip压缩，这可以用在任意类型的渲染器、模版和纯／原生内容中
// If you don't want to enable it globaly, you could just use the third parameter on context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
// 如果你不想全局启用它，你可以在中context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})只使用第3个参数
// Default is false
// 默认为 false
OptionGzip(val bool)

// OptionOther are the custom, dynamic options, can be empty
// OptionOther 是动态、自定义选项，可以为空
// this fill used only by you to set any app's options you want
// 这个填充的选项只能由你自己在应用程序中使用
// for each of an Iris instance
// 在每一个Iris实例中
OptionOther(val ...options.Options) //map[string]interface{}, options is github.com/kataras/go-options

// OptionSessionsCookie string, the session's client cookie name, for example: "qsessionid"
// OptionSessionsCookie string, 客户端的cookie名称， 如："qsessionid"
OptionSessionsCookie(val string)

// OptionSessionsDecodeCookie set it to true to decode the cookie key with base64 URLEncoding
// OptionSessionsDecodeCookie 设置为true用以使用base64 URLEncoding解析cookie的键
// Defaults to false
// 默认为 false
OptionSessionsDecodeCookie(val bool)

// OptionSessionsExpires the duration of which the cookie must expires (created_time.Add(Expires)).
// OptionSessionsExpires 是cookie必需过期的持续时间（created_time.Add(Expires))。
// If you want to delete the cookie when the browser closes, set it to -1 but in this case, the server side's session duration is up to GcDuration
// 如果你想要在浏览器关闭时删除cookie，那么把它设置为 -1 ， 服务器端会话的持续时间取决于GcDuration
//
// Default infinitive/unlimited life duration(0)
// 默认为没有限制生命周期（0)
OptionSessionsExpires(val time.Duration)

// OptionSessionsCookieLength the length of the sessionid's cookie's value, let it to 0 if you don't want to change it
// OptionSessionsCookieLength 会话ID的cookie值的长度，如果你不想改变它的话那么这是为0
// Defaults to 32
// 默认为 32
OptionSessionsCookieLength(val int)

// OptionSessionsGcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
// OptionSessionsGcDuration 每间隔多久应该清除内容中未使用的cookie
// for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
// 如：time.Duration(2)*time.Hour, 它将每隔2个小时检查两小时内为使用的cookie，
// deletes it from backend memory until the user comes back, then the session continue to work as it was
// 从后段内存中删除它直至用户回来，会话才会跟之前一样继续工作
//
// Default 2 hours
// 默认为2小时
OptionSessionsGcDuration(val time.Duration)

// OptionSessionsDisableSubdomainPersistence set it to true in order dissallow your q subdomains to have access to the session cookie
// OptionSessionsDisableSubdomainPersistence 为了不允许你的子域名去访问会话cookie那么把它设置为true
// defaults to false
// 默认为 false
OptionSessionsDisableSubdomainPersistence(val bool)

// OptionWebsocketWriteTimeout time allowed to write a message to the connection.
// OptionWebsocketWriteTimeout 允许将消息写入到连接的时间。
// Default value is 15 * time.Second
// 默认值为 15 * time.Second
OptionWebsocketWriteTimeout(val time.Duration)

// OptionWebsocketPongTimeout allowed to read the next pong message from the connection
// OptionWebsocketPongTimeout 允许从连接读取下一个pong消息的超时时间
// Default value is 60 * time.Second
// 默认为 60 * time.Second
OptionWebsocketPongTimeout(val time.Duration)

// OptionWebsocketPingPeriod send ping messages to the connection with this period. Must be less than PongTimeout
// OptionWebsocketPingPeriod 发送ping消息到连接的周期，必需小于 PongTimeout
// Default value is (PongTimeout * 9) / 10
// 默认值为 （PongTimeout * 9) / 10
OptionWebsocketPingPeriod(val time.Duration)

// OptionWebsocketMaxMessageSize max message size allowed from connection
// OptionWebsocketMaxMessageSize 允许来自连接的最大消息尺寸
// Default value is 1024
// 默认值为 1024
OptionWebsocketMaxMessageSize(val int64)

// OptionWebsocketBinaryMessages set it to true in order to denotes binary data messages instead of utf-8 text
// OptionWebsocketBinaryMessages 设置为true那么意味着使用二进制数据消息替代utf-8文本消息
// see https://github.com/kataras/iris/issues/387#issuecomment-243006022 for more
// 查看 https://github.com/kataras/iris/issues/387#issuecomment-243006022 了解更多
// defaults to false
// 默认为 false
OptionWebsocketBinaryMessages(val bool)

// OptionWebsocketEndpoint is the path which the websocket server will listen for clients/connections
// OptionWebsocketEndpoint websocket服务器监听客户端/连接的路径
// Default value is empty string, if you don't set it the Websocket server is disabled.
// 默认值为空字符串，如果你不设置它那么Websocket服务器被禁用。
OptionWebsocketEndpoint(val string)

// OptionWebsocketReadBufferSize is the buffer size for the underline reader
// OptionWebsocketReadBufferSize 后端读取器的缓存尺寸
OptionWebsocketReadBufferSize(val int)

// OptionWebsocketWriteBufferSize is the buffer size for the underline writer
// OptionWebsocketWriteBufferSize 后端写入器的缓存尺寸
OptionWebsocketWriteBufferSize(val int)

// OptionTesterListeningAddr is the virtual server's listening addr (host)
// OptionTesterListeningAddr 虚拟服务器的监听地址（host)
// Default is "iris-go.com:1993"
// 默认为 "iris-go.cn:1993"
OptionTesterListeningAddr(val string)

// OptionTesterExplicitURL If true then the url (should) be prepended manually, useful when want to test subdomains
// OptionTesterExplicitURL 如果为true那么url会被手动前置追加，当你测试子域名时很有用
// Default is false
// 默认为 false
OptionTesterExplicitURL(val bool)

// OptionTesterDebug if true then debug messages from the httpexpect will be shown when a test runs
// OptionTesterDebug 如果设置为true那么在测试运行时来自httpexpect的debug消息会被显示出来
// Default is false
// 默认为 false
OptionTesterDebug(val bool)

```go


### ServerConfiguration

### 服务器配置

Note: One iris instance can have and listening to many iris' Servers, one iris' Server can be used/passed to many iris instance, with that in mind, the server configuration has its own options to set.

注意：一个iris实例可以拥有并监听多个iris服务器，一个iris服务器可以被传递给多个iris实例，需要注意的是，每个服务器得有它自己的配置选项。


```go
// examples:
iris.AddServer(iris.OptionServerCertFile("file.cert"),iris.OptionServerKeyFile("file.key"))
iris.ListenTo(iris.OptionServerReadBufferSize(42000))

// or 
iris.AddServer(iris.ServerConfiguration{ListeningAddr: "mydomain.com", CertFile: "file.cert", KeyFile: "file.key"})
iris.ListenTo(iris.ServerConfiguration{ReadBufferSize:42000, ListeningAddr: "mydomain.com"})

// both are valid
```

**List** of all Server's options:

所有服务器选项**列表**：

```go
OptionServerListeningAddr(val string)

OptionServerCertFile(val string)

OptionServerKeyFile(val string)

// AutoTLS enable to get certifications from the Letsencrypt
// AutoTLS 启用从Letsencrypt获取证书
// when this configuration field is true, the CertFile & KeyFile are empty, no need to provide a key.
// 当这个配置字段为true，CertFile 和 KeyFile 都为空，则不需要提供Key。
//
// example: https://github.com/iris-contrib/examples/blob/master/letsencyrpt/main.go
// 例子: https://github.com/iris-contrib/examples/blob/master/letsencyrpt/main.go
OptionServerAutoTLS(val bool)

// Mode this is for unix only
// Mode只用在unix类系统下
OptionServerMode(val os.FileMode)
// OptionServerMaxRequestBodySize Maximum request body size.
// OptionServerMaxRequestBodySize 最大的请求体尺寸。
//
// The server rejects requests with bodies exceeding this limit.
// 如果请求体尺寸超过这个限制，那么服务器会拒绝该请求。
//
// By default request body size is 8MB.
// 默认请求体尺寸为 8MB。
OptionServerMaxRequestBodySize(val int)

// Per-connection buffer size for requests' reading.
// 每个连接的请读取缓存尺寸。
// This also limits the maximum header size.
// 这也限制了最大的头尺寸。
//
// Increase this buffer if your clients send multi-KB RequestURIs
// 如果你的客户端发送多个KB级的请求URI或头信息那么可以增加这个缓存
// and/or multi-KB headers (for example, BIG cookies).
//
// Default buffer size is used if not set.
// 如果没有设置的话将使用默认尺寸。
OptionServerReadBufferSize(val int)

// Per-connection buffer size for responses' writing.
// 每个连接的应答体写入尺寸。
//
// Default buffer size is used if not set.
// 如果没有设置那么使用默认缓存。
OptionServerWriteBufferSize(val int)

// Maximum duration for reading the full request (including body).
// 读取全部请求(包含请求体)的最大执行周期。
//
// This also limits the maximum duration for idle keep-alive
// connections.
// 这也限制了空闲长连接的最大周期。
//
// By default request read timeout is unlimited.
// 默认没有限制请求读取超时时间。
OptionServerReadTimeout(val time.Duration)

// Maximum duration for writing the full response (including body).
// 写入全部应答（包括应答体）的最大周期。
//
// By default response write timeout is unlimited.
// 默认没有限制应答写入的超时时间。
OptionServerWriteTimeout(val time.Duration)

// RedirectTo, defaults to empty, set it in order to override the station's handler and redirect all requests to this address which is of form(HOST:PORT or :PORT)
// 默认为空，设置它是为了重载站点的处理器并重定型所有的请求到这个地址即（HOST:PORT 或 :PORT)
//
// NOTE: the http status is 'StatusMovedPermanently', means one-time-redirect(the browser remembers the new addr and goes to the new address without need to request something from this server
// 注意：http状态为 'StatusMovedPermanently', 意味着一次跳转（浏览器记住新的地址，跳转到该地址不需要从服务器端请求任何东西）
// which means that if you want to change this address you have to clear your browser's cache in order this to be able to change to the new addr.
// 如果你想要改变这个地址的话，你需要清除浏览器的缓存。
//
// example: https://github.com/iris-contrib/examples/tree/master/multiserver_listening2
// 如: https://github.com/iris-contrib/examples/tree/master/multiserver_listening2
OptionServerRedirectTo(val string)

// OptionServerVirtual If this server is not really listens to a real host, it mostly used in order to achieve testing without system modifications
// OptionServerVirtual 如果这个服务器没有监听到真实的host，它通常是用作不修改系统配置的情况下完成测试
OptionServerVirtual(val bool)

// OptionServerVListeningAddr, can be used for both virtual = true or false,
// OptionServerVListeningAddr, 设置为 true 或 false 都可以使用，
// if it's setted to not empty, then the server's Host() will return this addr instead of the ListeningAddr.
// 如果它被设置为非空，那么服务器的Host()函数将返回这个设置的地址而非ListeningAddr。
// server's Host() is used inside global template helper funcs
// 服务器的 Host() 函数被用在全局模版的帮助函数中
// set it when you are sure you know what it does.
// 当你确定它作什么时可以设定它。
//
// Default is empty ""
// 默认为空 ""
OptionServerVListeningAddr(val string)

// OptionServerVScheme if setted to not empty value then all template's helper funcs prepends that as the url scheme instead of the real scheme
// OptionServerVScheme 如果设置为非空值那么所有的模版帮助函数将使用这个协议替换真实的url协议
// server's .Scheme returns VScheme if  not empty && differs from real scheme
// 如果非空且不同于真实的协议的话，服务器端的 .Scheme 将返回VScheme
//
// Default is empty ""
// 默认值为空 ""
OptionServerVScheme(val string)

// OptionServerName the server's name, defaults to "iris".
// OptionServerName 服务器的名称，默认为 "iris"。
// You're free to change it, but I will trust you to don't, this is the only setting whose somebody, like me, can see if iris web framework is used
// 你可以随意改变它，但我相信你不会的，这是唯一标识是否正在使用iris框架的设定
OptionServerName(val string)

```    

### The main Configuration

### 主要配置
```go
type Configuration struct {
	// DisablePathCorrection corrects and redirects the requested path to the registed path
	// DisablePathCorrection 矫正并重定向请求路径到注册的路径
	// for example, if /home/ path is requested but no handler for this Route found,
	// 如果没有法相请求 /home/ 路径的处理器 
	// then the Router checks if /home handler exists, if yes,
	// 路由器检查 /home 处理器是否存在，如果是，
	// (permant)redirects the client to the correct path /home
	// 永久重定向客户端到指定的路径 /home
	//
	// Default is false
	// 默认为 false
	DisablePathCorrection bool

	// DisablePathEscape when is false then its escapes the path, the named parameters (if any).
	// DisablePathEscape 当设定为 fals 时那么它不再解析路径、命名参数（如果有的话）。
	// Change to true it if you want something like this https://github.com/kataras/iris/issues/135 to work
	// 如果你想要完成一些像 https://github.com/kataras/iris/issues/135 问题中描述的工作，那么设置为true
	//
	// When do you need to Disable(true) it:
	// 何时你需要禁用(true)它：
	// accepts parameters with slash '/'
	// 接受带有斜杠 '/' 的参数
	// Request: http://localhost:8080/details/Project%2FDelta
	// 请求：http://localhost:8080/details/Project%2FDelta
	// ctx.Param("project") returns the raw named parameter: Project%2FDelta
	// ctx.Param("project") 返回原生命名参数：Project%2FDelta
	// which you can escape it manually with net/url:
	// 你可以使用 net/url 手动解析
	// projectName, _ := url.QueryUnescape(c.Param("project").
	// Look here: https://github.com/kataras/iris/issues/135 for more
	// 查看这里：https://github.com/kataras/iris/issues/135 了解更多
	//
	// Default is false
	// 默认为 false
	DisablePathEscape bool

	// DisableBanner outputs the iris banner at startup
	// DisableBanner 在启动时输出iris横幅
	//
	// Default is false
	// 默认为 false
	DisableBanner bool

	// LoggerOut is the destination for output
	// LoggerOut 是输出的目的文件
	//
	// Default is os.Stdout
	// 默认为 os.Stdout
	LoggerOut io.Writer
	// LoggerPreffix is the logger's prefix to write at beginning of each line
	// LoggerPreffix 是写入到每行日志开始处的前缀文本
	//
	// Default is [IRIS]
	// 默认为 [IRIS]
	LoggerPreffix string

	// ProfilePath a the route path, set it to enable http pprof tool
	// ProfilePath 是一个路由路径，设置它可以启用http pprof工具
	// Default is empty, if you set it to a $path, these routes will handled:
	// 默认为空，如果你设置为一个路径，这些路由的路由会处理：
	// $path/cmdline
	// $path/profile
	// $path/symbol
	// $path/goroutine
	// $path/heap
	// $path/threadcreate
	// $path/pprof/block
	// for example if '/debug/pprof'
	// 如 '/debug/pprof'
	// http://yourdomain:PORT/debug/pprof/
	// http://yourdomain:PORT/debug/pprof/cmdline
	// http://yourdomain:PORT/debug/pprof/profile
	// http://yourdomain:PORT/debug/pprof/symbol
	// http://yourdomain:PORT/debug/pprof/goroutine
	// http://yourdomain:PORT/debug/pprof/heap
	// http://yourdomain:PORT/debug/pprof/threadcreate
	// http://yourdomain:PORT/debug/pprof/pprof/block
	// it can be a subdomain also, for example, if 'debug.'
	// 它也可以时一个子域名，如 'debug.'
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
	// DisableTemplateEngines 设置为true可以禁用加载默认的模版引擎 (html/template) 并不允许使用iris.UseEngine
	// default is false
	// 默认为 false
	DisableTemplateEngines bool

	// IsDevelopment iris will act like a developer, for example
	// IsDevelopment 使iris的动作像一个开发者，如
	// If true then re-builds the templates on each request
	// 如果为 true 那么每次请求都会重建模版
	// default is false
	// 默认为 false
	IsDevelopment bool

	// TimeFormat time format for any kind of datetime parsing
	// TimeFormat 解析任何日期时间字符串的时间格式
	TimeFormat string

	// Charset character encoding for various rendering
	// Charset 各种渲染的字符编码
	// used for templates and the rest of the responses
	// 用在模版和应答中
	// defaults to "UTF-8"
	// 默认为 "UTF-8"
	Charset string

	// Gzip enables gzip compression on your Render actions, this includes any type of render, templates and pure/raw content
	// Gzip 在你的渲染动作中启用gzip压缩，这可以用在任意类型的渲染器、模版和纯／原生内容中
	// If you don't want to enable it globaly, you could just use the third parameter on context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
	// 如果你不想全局启用它，你可以使用context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})的第3个参数。
	// defaults to false
	// 默认为 false
	Gzip bool

	// Sessions contains the configs for sessions
	// Sessions 包含会话的配置
	Sessions SessionsConfiguration

	// Websocket contains the configs for Websocket's server integration
	// Websocket 包含Websocket服务器集成的配置
	Websocket WebsocketConfiguration

	// Tester contains the configs for the test framework, so far we have only one because all test framework's configs are setted by the iris itself
	// Tester 含有测试框架的配置，迄今为止我们只有这一个配置，因为所有的测试框架参数都由iris自己设定
	// 你可以在 https://github.com/kataras/iris/glob/master/context_test.go 中找到示例
	Tester TesterConfiguration

	// Other are the custom, dynamic options, can be empty
	// Other 是动态、自定义的选项，可以为空
	// this fill used only by you to set any app's options you want
	// 这个选项只是由你设定为任意应用程序需要的内容
	// for each of an Iris instance
	// 每个Iris实例
	Other options.Options
}
```

View all configuration fields and options by navigating to the [kataras/iris/configuration.go source file](https://github.com/kataras/iris/blob/master/configuration.go)

在 [kataras/iris/configuration.go 源文件](https://github.com/kataras/iris/blob/master/configuration.go) 中可以查看所有选项和字段
