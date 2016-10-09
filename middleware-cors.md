# HTTP 访问控制 / HTTP access control
[This is a middleware](https://github.com/kataras/iris/tree/master/middleware/cors).

[这是一个中间件](https://github.com/kataras/iris/tree/master/middleware/cors).

Some security work for between you and the requests.

在请求之间给你一些安全性的功能。


Options

选项


```go
    // AllowedOrigins is a list of origins a cross-domain request can be executed from.
    // AllowedOrigins 是一个可以执行跨域请求的源列表
    // If the special "*" value is present in the list, all origins will be allowed.
    // 如果列表中有 "*" 值，那么所有的源都将被允许。
    // An origin may contain a wildcard (*) to replace 0 or more characters
    // 一个源可以包含通配符(*) 来代替0个或多个字符
    // (i.e.: http://*.domain.com). Usage of wildcards implies a small performance penality.
    // (如: http://*.domain.com). 通配符的使用意味着有一点性能惩罚。
    // Only one wildcard can be used per origin.
    // 每个源只能用一个通配符
    // Default value is ["*"]
    // 默认值是 ["*"]
    AllowedOrigins []string
    
    // AllowOriginFunc is a custom function to validate the origin. It take the origin
    // AllowOriginFunc 是用来验证源的自定义韩式。
    // as argument and returns true if allowed or false otherwise. If this option is
    // 使用源作为参数，如果允许的话返回true，否则为 false。
    // set, the content of AllowedOrigins is ignored.
    // 如果这个选项设置了，那么 AllowedOrigins 的内容就会被忽略掉。
    AllowOriginFunc func(origin string) bool
    
    // AllowedMethods is a list of methods the client is allowed to use with
    // AllowedMethods 是允许客户端跨域请求使用的方法列表
    // cross-domain requests. Default value is simple methods (GET and POST)
    // 默认值是简单方法 (GET 和 POST)
    AllowedMethods []string
    
    // AllowedHeaders is list of non simple headers the client is allowed to use with
    // cross-domain requests.
    // AllowedHeaders  是允许客户端跨域请求使用的非简单的 HTTP headers 列表
    // If the special "*" value is present in the list, all headers will be allowed.
    // 如果 "*" 值存在列表中，那么所有的 headers 将被允许
    // Default value is [] but "Origin" is always appended to the list.
    // 默认值是 [] 但 "Origin" 总是会被追加到列表中。
    AllowedHeaders []string

    AllowedHeadersAll bool

    // ExposedHeaders indicates which headers are safe to expose to the API of a CORS
    // ExposedHeaders 用来指明哪些 headers 公开给 符合 跨源HTTP请求（CORS : cross-origin HTTP request）API规范的API是安全的
    // API specification
    ExposedHeaders []string
    
    // AllowCredentials indicates whether the request can include user credentials like
    // cookies, HTTP authentication or client side SSL certificates.
    // AllowCredentials 指明是否请求可以包含像cookies、HTTP 授权 或 客户端SSL证书 这样的 用户凭据。
    AllowCredentials bool
    
    // MaxAge indicates how long (in seconds) the results of a preflight request
    // can be cached
    // MaxAge 指明预期检查的请求结果可以被缓存多久(秒)
    MaxAge int
    
    // OptionsPassthrough instructs preflight to let other potential next handlers to
    // process the OPTIONS method. Turn this on if your application handles OPTIONS.
    // OptionsPassthrough 指导预检让其它潜在的下一个处理器来处理 OPTIONS 方法。
    // 开启这个选项如果你的程序能够处理 OPTIONS
    OptionsPassthrough bool
    
    // Debugging flag adds additional output to debug server side CORS issues
    // Debug 标记 为 调试服务器端的 CORS 问题添加附加输出
    Debug bool

```

```go
import "github.com/iris-contrib/middleware/cors"

cors.New(cors.Options{})
```

Example:

示例


```go
package main

import (
    "github.com/kataras/iris"
    "github.com/iris-contrib/middleware/cors"
)

func main() {

    crs := cors.New(cors.Options{}) // options here / 选项在这里

    iris.Use(crs) // register the middleware ／ 注册中间件

    iris.Get("/home", func(c *iris.Context) {
        // ...
    })

    iris.Listen(":8080")
}

```
