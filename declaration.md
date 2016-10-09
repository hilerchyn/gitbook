# 声明 / Declaration

You might have asked yourself:

你也许已经问过自己这些问题:

* Q: Other frameworks need more lines to start a server, why is Iris different?
* 问：别的框架启动一个服务器需要很多行代码，为什么 Iris 如此不同？

* A: Iris gives you the freedom to choose between four ways to use Iris
* 答：Iris 给了你在4种方式中选择并使用Iris的自由

  1. global **iris.** ／ 全局 **iris.**
  2. declare a new iris station with default config: **iris.New\(\)** / 声明具有默认配置的 iris 站点: **iris.New\(\)**
  3. declare a new iris station with custom config: **api := iris.New\(iris.Configuration{...}\)** / 用自定义配置声明一个新的 iris 站点: **api := iris.New\(iris.Configuration{...}\)**
  4. declare a new iris station with custom options: **api := iris.New\(iris.OptionCharset\("UTF-8"\), iris.OptionSessionsCookie\("mycookie"\), ...\)** / 用自定义选项声明一个新的 iris 站点: **api := iris.New\(iris.OptionCharset\("UTF-8"\), iris.OptionSessionsCookie\("mycookie"\), ...\)**


Configuration is **OPTIONAL** and can change after declaration with`$instance.Config.`, \/ `$instance.Set(Option...)`

Configuration 是 **可选的** 而且可以在声明之后使用 `$instance.Config.` / `$instance.Set(Option...)`来变更配置。

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

// 3.
func thirdWay() {

   config := iris.Configuration{IsDevelopment: true}
   iris.New(config)
   iris.Get("/home", func(c*iris.Context){})
   iris.Listen(":8080")
}

// 4.
func forthWay() {

    api := iris.New()
    api.Set(iris.OptionCharset("UTF-8"))

    api.Get("/home",func(c *iris.Context){})
    api.Listen(":8080")
}

// after .New, at runtime, also possible because Iris have default values, configuration is TOTALLY OPTIONAL DSESIRE
// .New 函数以后，在运行时，Iris 虽然拥有默认值，但配置仍然是可选操作
func main() {
    iris.Config.Websocket.Endpoind = "/ws"

   //...

   iris.Listen(":8080")
}
```



Let's take a quick look at the [**iris.Configuration**](configuration.md)

让我们快速看一下 [**iris.Configuration**](configuration.md):

```go
// Configuration the whole configuration for an iris instance ($instance.Config) or global iris instance (iris.Config)
// Configuration 是 iris 实例 ($instance.Config) 和 全局 iris 实例 (iris.Config) 的全部配置信息
// these can be passed via options also, look at the top of this file(configuration.go)
// 这些配置信息也能够通过选项传递，可以查看(configuration.go)文件中的定义。
//
// Configuration is also implements the OptionSet so it's a valid option itself, this is briliant enough
// Congiguration 也实现了 OptionSet 这是一个它自己的可用选项，这太机智了
type Configuration struct {
    // VHost is the addr or the domain that server listens to, which it's optional
    // VHost 是服务器监听的地址或域名，这是可选项
    // When to set VHost manually:
    // 何时手动设置 VHost:
    // 1. it's automatically setted when you're calling
    // 1. 它会被自动设置，当你调用以下函数时
    //     $instance.Listen/ListenUNIX/ListenTLS/ListenLETSENCRYPT functions or
    //     ln,_ := iris.TCP4/UNIX/TLS/LETSENCRYPT; $instance.Serve(ln)
    // 2. If you using a balancer, or something like nginx
    // 2. 如果你正在使用类似nginx的负载均衡
    //    then set it in order to have the correct url
    //    when calling the template helper '{{url }}'
    //    *keep note that you can use {{urlpath }}) instead*
    //    那么设置它可以在调用模版帮助器 '{{url }}' 时获得正确的url
    //    *需要注意的是你也可以使用 {{urlpath }}*
    //
    // Note: this is the main's server Host, you can setup unlimited number of fasthttp servers
    // 注意: 这是主服务器的 Host, 你可以设置无限台 fasthttp 服务器
    // listening to the $instance.Handler after the manually-called $instance.Build
    // 在手动调用 $instance.Build 后来监听  $instance.Handler
    //
    // Default comes from iris.Listen/.Serve with iris' listeners (iris.TCP4/UNIX/TLS/LETSENCRYPT)
    // 默认值来自 iris.Listen/.Serve 的 iris 监听器(iris.TCP4/UNIX/TLS/LETSENCRYPT)
    VHost string

    // VScheme is the scheme (http:// or https://) putted at the template function '{{url }}'
    // VScheme 是添加到模版函数 '{{url }}' 中的协议(http:// 或 https://)
    // It's an optional field,
    // 这是可选字段,
    // When to set VScheme manually:
    // 何时需要手动设置 VScheme:
    // 1. You didn't start the main server using $instance.Listen/ListenTLS/ListenLETSENCRYPT or $instance.Serve($instance.TCP4()/.TLS...)
    // 1. 你没有使用 $instance.Listen/ListenTLS/ListenLETSENCRYPT 或 $instance.Serve($instance.TCP4()/.TLS...) 启动主服务器
    // 2. if you're using something like nginx and have iris listening with addr only(http://) but the nginx mapper is listening to https://
    // 2. 如果你正在使用类似 nginx 的服务器，iris 只监听(http://)地址而nginx 代理监听的是 https://
    //
    // Default comes from iris.Listen/.Serve with iris' listeners (TCP4,UNIX,TLS,LETSENCRYPT)
    // 默认值来自 iris.Listen/.Serve 的 iris 监听器(TCP4,UNIX,TLS,LETSENCRYPT)
    VScheme string

    // MaxRequestBodySize Maximum request body size.
    // MaxRequestBodySize 最大的请求体尺寸。
    //
    // The server rejects requests with bodies exceeding this limit.
    // 服务器将拒绝请求体超过这个限制的请求。
    //
    // By default request body size is 8MB.
    // 默认请求体尺寸为 8MB。
    MaxRequestBodySize int

    // Per-connection buffer size for requests' reading.
    // 每个连接请求读取的缓存尺寸。
    // This also limits the maximum header size.
    // 这也限制了最大的头尺寸。
    //
    // Increase this buffer if your clients send multi-KB RequestURIs
    // and/or multi-KB headers (for example, BIG cookies).
    // 如果你的客户端发送多个KB以上大小的URI请求并且／或者发送了多个KB以上大小的头信息(如,大的cookie)
    //
    // Default buffer size is used if not set.
    // 如果没有设置那么使用默认缓存值。
    ReadBufferSize int

    // Per-connection buffer size for responses' writing.
    // 每个连接请求的写缓存尺寸。
    //
    // Default buffer size is used if not set.
    // 如果没有设置那么使用默认缓存值。
    WriteBufferSize int

    // Maximum duration for reading the full request (including body).
    // 读取请求(包括 请求体)的最大持续时间。
    //
    // This also limits the maximum duration for idle keep-alive
    // connections.
    // 这也限定了空闲存活连接的最大持续时间。
    //
    // By default request read timeout is unlimited.
    // 默认的请求读取超时没有限制。
    ReadTimeout time.Duration

    // Maximum duration for writing the full response (including body).
    // 应答完全写入(包括 应答体)的最大持续时间。
    //
    // By default response write timeout is unlimited.
    // 默认没有对写入应答超时做限制。
    WriteTimeout time.Duration

    // Maximum number of concurrent client connections allowed per IP.
    // 允许每个IP客户端的最大并发连接数。
    //
    // By default unlimited number of concurrent connections
    // 默认没有限制并发连接数。
    MaxConnsPerIP int

    // Maximum number of requests served per connection.
    // 每个连接的最大请求伺服数。
    //
    // The server closes connection after the last request.
    // 最后一个请求之后服务器关闭连接。
    // 'Connection: close' header is added to the last response.
    // 添加 'Connection: close' 头信息到最后一个应答中。
    //
    // By default unlimited number of requests may be served per connection.
    // 默认没有限制每个连接的请求数。
    MaxRequestsPerConn int

    // CheckForUpdates will try to search for newer version of Iris based on the https://github.com/kataras/iris/releases
    // CheckForUpdates 将从 https://github.com/kataras/iris/releases 检索新版本的 Iris
    // If a newer version found then the app will ask the he dev/user if want to update the 'x' version
    // 如果发现新的版本，那么应用程序将会询问 开发人员/用户 是否想要升级 'x' 版本
    // if 'y' is pressed then the updater will try to install the latest version
    // 如果按下并回复了 'y' 那么更新器将尝试安装最新的版本
    // the updater, will notify the dev/user that the update is finished and should restart the App manually.
    // 更新器会通知 开发人员/用户 更新已经完成且应该手痛重启应用程序。
    // Notes:
    // 注意：
    // 1. Experimental feature
    // 1. 实验性功能
    // 2. If setted to true, the app will start the server normally and runs the updater in its own goroutine,
    // 2. 如果设置为 true，应用程序将正常启动服务器且在单独的goroutine中运行更新器，
    //    for a sync operation see CheckForUpdatesSync.
    //    同步操作的话查看 CheckForUpdatesSync 。
    // 3. If you as developer edited the $GOPATH/src/github/kataras or any other Iris' Go dependencies at the past
    // 3. 如果你做为一个开发人员过去修改过 $GOPATH/src/github/kataras 或 任何 Iris 的任何Go依赖
    //    then the update process will fail.
    //    那么更新处理将会失败。
    //
    // Usage: iris.Set(iris.OptionCheckForUpdates(true)) or
    //        iris.Config.CheckForUpdates = true or
    //        app := iris.New(iris.OptionCheckForUpdates(true))
    // Default is false
    // 默认值为 false
    CheckForUpdates bool
    // CheckForUpdatesSync checks for updates before server starts, it will have a little delay depends on the machine's download's speed
    // CheckForUpdatesSync 在服务器启动前检查更新, 他会有一点儿延迟取决于服务器的下载速度
    // See CheckForUpdates for more
    // 更多内容查看 CheckForUpdates
    // Notes:
    // 注意:
    // 1. you could use the CheckForUpdatesSync while CheckForUpdates is false, set this or CheckForUpdates to true not both
    // 1. 当 CheckForUpdates 为false时，你可以使用 CheckForUpdatesSync, CheckForUpdatesSync 和 CheckForUpdates 设置其中一个为true
    // 2. if both CheckForUpdates and CheckForUpdatesSync are setted to true then the updater will run in sync mode, before server server starts.
    // 2. 如果 CheckForUpdates 和 CheckForUpatesSync 都设置为 true 那么在服务器启动前更新器会运行 sync 模式。
    //
    // Default is false
    // 默认是 false
    CheckForUpdatesSync bool

    // DisablePathCorrection corrects and redirects the requested path to the registed path
    // DisablePathCorrection 矫正并重置请求路径到注册的路径
    // for example, if /home/ path is requested but no handler for this Route found,
    // 如果 /home/ 路径被请求但没有此路由的处理器
    // then the Router checks if /home handler exists, if yes,
    // 那么路由器会检查 /home 处理器是否存在, 如果是,
    // (permant)redirects the client to the correct path /home
    // (永久)重置客户端到正确的路径 /home
    // 
    //
    // Default is false
    // 默认值为 false
    DisablePathCorrection bool

    // DisablePathEscape when is false then its escapes the path, the named parameters (if any).
    // DisablePathEscape 设置为 false 时不再过滤(url decode)该路径和命名参数(如果有的话)。
    // Change to true it if you want something like this https://github.com/kataras/iris/issues/135 to work
    // 如果你想像 https://github.com/kataras/iris/issues/135  这样的路径工作那么设置为 true。
    //
    // When do you need to Disable(true) it:
    // 何时需要禁用(true)它:
    // accepts parameters with slash '/'
    // 接收带有斜杠'/'的参数
    // Request: http://localhost:8080/details/Project%2FDelta
    // 请求: http://localhost:8080/details/Project%2FDelta
    // ctx.Param("project") returns the raw named parameter: Project%2FDelta
    // ctx.Param("project") 返回原始命名参数: Project%2FDelta
    // which you can escape it manually with net/url:
    // 你也可以不使用net/url手动不解析:
    // projectName, _ := url.QueryUnescape(c.Param("project").
    // Look here: https://github.com/kataras/iris/issues/135 for more
    // 更多信息查看这里: https://github.com/kataras/iris/issues/135
    //
    // Default is false
    // 默认值为 false
    DisablePathEscape bool

    // DisableBanner outputs the iris banner at startup
    // DisableBanner 启动时输出iris横幅
    //
    // Default is false
    // 默认值为 false
    DisableBanner bool

    // LoggerOut is the destination for output
    // LoggerOut 输出的目的地
    //
    // Default is os.Stdout
    // 默认为 os.Stdout
    LoggerOut io.Writer
    // LoggerPreffix is the logger's prefix to write at beginning of each line
    // LoggerPreffix 是日志器在输出每行日志的开始处添加的前缀
    //
    // Default is [IRIS]
    // 默认是 [IRIS]
    LoggerPreffix string

    // DisableTemplateEngines set to true to disable loading the default template engine (html/template) and disallow the use of iris.UseEngine
    // DisableTemplateEngines 设置为true 以禁止加载默认的模版引擎(html/templateP)并不允许使用iris.UseEngine
    // default is false
    // 默认值为 false
    DisableTemplateEngines bool

    // IsDevelopment iris will act like a developer, for example
    // IsDevelopment iris 将扮演一个开发人员的角色，例如
    // If true then re-builds the templates on each request
    // 如果设置为true那么每次请求都会重新构建模板
    // default is false
    // 默认值为false
    IsDevelopment bool

    // TimeFormat time format for any kind of datetime parsing
    // TimeFormat 解析任何日期时间类型的格式
    TimeFormat string

    // Charset character encoding for various rendering
    // Charset 各种渲染器的字符编码
    // used for templates and the rest of the responses
    // 用于模板和应答
    // defaults to "UTF-8"
    // 默认值为"UTF-8"
    Charset string

    // Gzip enables gzip compression on your Render actions, this includes any type of render, templates and pure/raw content
    // Gzip 在你的渲染器的动作中启用gzip压缩，这包裹任意类型的渲染器，模板和纯/原生内容
    // If you don't want to enable it globaly, you could just use the third parameter on context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
    // 如果你不想全局启用，你可以只使用渲染器的第3个参数 context.Render("myfileOrResponse", structBinding{}, iris.RenderOptions{"gzip": true})
    // defaults to false
    // 默认值为false
    Gzip bool

    // Sessions contains the configs for sessions
    // Sessions 包含所有会话(session)配置
    Sessions SessionsConfiguration {
        // Cookie string, the session's client cookie name, for example: "mysessionid"
        // Cookie string, 会话客户端的cookie名，如: "mysessionid"
        //
        // Defaults to "gosessionid"
        // 默认设定为 "gosessionid"
        Cookie string

        // DecodeCookie set it to true to decode the cookie key with base64 URLEncoding
        // DecodeCookie 设置为 true 用base64 URLEncoding 解码cookie键
        //
        // Defaults to false
        // 默认值为 false
        DecodeCookie bool

        // Expires the duration of which the cookie must expires (created_time.Add(Expires)).
        // Expires cookie必须过期(created_time.Add(Expires))的持续时间。
        // If you want to delete the cookie when the browser closes, set it to -1 but in this case, the server side's session duration is up to GcDuration
        // 如果你想在浏览器关闭时删除 cookie ，这时把它设置成-1，服务器端的session持续时间取决于GcDuration
        //
        // Defaults to infinitive/unlimited life duration(0)
        // 默认生命周期是无限(0) 
        Expires time.Duration

        // CookieLength the length of the sessionid's cookie's value, let it to 0 if you don't want to change it
        // CookieLength sessionid 的 cookie 值的长度，如果你不想改变的话那么设置成 0。
        //
        // Defaults to 32
        // 默认值为 32
        CookieLength int

        // GcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
        // GcDuration 每过多久应该清除内存中无用的 cookie
        // for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
        // 如: timme.Duration(2)*time.Hour 每过两个小时将会检查2小时内没有被使用的cookie
        // deletes it from backend memory until the user comes back, then the session continue to work as it was
        // 从后端内存汇总删除它，直到用户回来会话才会继续跟之前一样工作。
        //
        // Defaults to 2 hours
        // 默认值为2小时
        GcDuration time.Duration

        // DisableSubdomainPersistence set it to true in order dissallow your q subdomains to have access to the session cookie
        // DisableSubdomainPersistence 为了不允许q子域名访问cookie那那么将其设置为true
        //
        // Defaults to false
        // 默认值为 false
        DisableSubdomainPersistence bool

        // DisableAutoGC disables the auto-gc running when the session manager is created
        // DisableAutoGC 当session管理器被创建以后可以禁用自动垃圾回收运行
        // set to false to disable this behavior, you can call(onnce) manually the GC using the .GC functiona
        // 设置为false禁用垃圾回收行为，你可以使用.GC函数手动调用垃圾回收
        //
        // Defaults to false
        // 默认值为false
        DisableAutoGC bool
    }

    // Websocket contains the configs for Websocket's server integration
    // Websocket 包含集成websocket服务器的所有配置
    Websocket WebsocketConfiguration {
        // WriteTimeout time allowed to write a message to the connection.
        // WriteTimeout 允许向连接写入消息的时间。
        // Default value is 15 * time.Second
        // 默认值为 15 * time.Second
        WriteTimeout time.Duration
        // PongTimeout allowed to read the next pong message from the connection
        // PongTimeout 允许从连接读取下一个pong消息的超时时间
        // Default value is 60 * time.Second
        // 默认值为 60 * time.Second
        PongTimeout time.Duration
        // PingPeriod send ping messages to the connection with this period. Must be less than PongTimeout
        // PingPeriod 使用这个周期向连接发送 ping 消息, 必须小于PongTimeout
        // Default value is (PongTimeout * 9) / 10
        // 默认值为 (PongTimeout * 9) / 10
        PingPeriod time.Duration
        // MaxMessageSize max message size allowed from connection
        // MaxMessageSize 允许来自连接的最大消息尺寸
        // Default value is 1024
        // 默认值为 1024
        MaxMessageSize int64
        // BinaryMessages set it to true in order to denotes binary data messages instead of utf-8 text
        // BinaryMessages 设置为 true 意味着用二进制数据消息替代utf-8文本消息
        // see https://github.com/kataras/iris/issues/387#issuecomment-243006022 for more
        // 更多内容查看 https://github.com/kataras/iris/issues/387#issuecomment-243006022
        // defaults to false
        // 默认值为 false
        BinaryMessages bool
        // Endpoint is the path which the websocket server will listen for clients/connections
        // Endpoint websocket服务器监听客户端/连接的路径
        // Default value is empty string, if you don't set it the Websocket server is disabled.
        // 默认值是空字符串，如果你没有设置它那么Websocket服务器将被禁用。
        Endpoint string
        // ReadBufferSize is the buffer size for the underline reader
        // ReadBufferSize 后端读取器的缓存尺寸
        ReadBufferSize int
        // WriteBufferSize is the buffer size for the underline writer
        // WriteBufferSize 后端写入器的缓存尺寸
        WriteBufferSize int
    }

    // Tester contains the configs for the test framework, so far we have only one because all test framework's configs are setted by the iris itself
    // Tester 包含了测试框架的所有配置，到现在为止我们只有一个参数，因为所有测试框架的配置都由iris自己配置。
    // You can find example on the https://github.com/kataras/iris/glob/master/context_test.go
    // 你可以在 https://github.com/kataras/iris/glob/master/context_test.go 中找到例子
    Tester TesterConfiguration {
        // ExplicitURL If true then the url (should) be prepended manually, useful when want to test subdomains
        // ExplicitURL 如果为true那么可以手动添加前置url, 当你想要测试子域名时是有用的
        // Default is false
        // 默认值为 false
        ExplicitURL bool
        // Debug if true then debug messages from the httpexpect will be shown when a test runs
        // Debug 如果为true那么来自httpexpect的调试消息当测试时会被显示出来
        // Default is false
        // 默认值为 false
        Debug bool
    }

    // Other are the custom, dynamic options, can be empty
    // Other 是自定义的，动态选项，可以为空
    // this fill used only by you to set any app's options you want
    // 这个由你按照你应用程序选项来设定每一个 Iris 实例
    // for each of an Iris instance
    Other options.Options
}


```

> Note that with 2.,  3. & 4. you **can serve more than one Iris server** in the
> same app, when it's necessary.
> 
> 注意使用 2., 3. 和 4. 时，如果需要，你在同一个应用中 **能够定义并监听不止一台 Iris 服务器**

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



iris.New(iris.OptionCharset("UTF-8"), iris.OptionIsDevelopment(true), 
iris.OptionSessionsCookie("mycookie"), iris.OptionWebsocketEndpoint("/my_endpoint"))



// if you want to set configuration after the .New use the .Set:
// 如果你想要在 .New 之后设置配置信息那么使用 .Set

iris.Set(iris.OptionDisableBanner(true))

```

