# 安全 / Secure

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/secure)

这是一个[中间件](https://github.com/iris-contrib/middleware/tree/master/secure)

Secure is an HTTP middleware for Go that facilitates some quick security wins.

Secure 只一个为Go而写的有助于在安全性方面快速获得成绩的HTTP中间件。

```go
import "github.com/iris-contrib/middleware/secure"

secure.New(secure.Options{}) // options here / 这里是选项

```

Example

示例

```go
package main

import (
    "github.com/kataras/iris"
    "github.com/iris-contrib/middleware/secure"
)

func main() {
    s := secure.New(secure.Options{
        // AllowedHosts is a list of fully qualified domain names
        // that are allowed. Default is empty list, 
        // which allows any and all host names.
        // AllowedHosts 是一个被允许的完全限定域名列表。默认为空允许所有人和主机名。
        AllowedHosts:            []string{"ssl.example.com"},                                                                                                                         
        // If SSLRedirect is set to true, then only allow HTTPS requests.
        // 如果 SSLRedirect 设置为 true，那么只允许 HTTPS 请求。
        // Default is false.
        // 默认为 false。
        SSLRedirect:             true,     
        // If SSLTemporaryRedirect is true, 
        // 如果 SSLTemporaryRedirect 为 true，
        // then a 302 will be used while redirecting.
        // 当重定向时使用 302 状态码。
        // Default is false (301).
        // 默认为 false (301)。
        SSLTemporaryRedirect:    false,    
        // SSLHost is the host name that is used to 
        // redirect HTTP requests to HTTPS.
        // SSLHost 是用来将 HTTP 请求重定向到 HTTPS 主机名
        // Default is "", which indicates to use the same host.
        // 默认为 "", 表明使用相同的主机。
        SSLHost:                 "ssl.example.com",
        // SSLProxyHeaders is set of header keys with associated values 
        // that would indicate a valid HTTPS request. 
        // SSLProxyHeaders 是 键－值 联合集 以表明有效的 HTTPS 请求。
        // Useful when using Nginx: `map[string]string{"X-Forwarded-Proto": "https"}`.
        // 当使用Nginx时有用的：`map[string]string{"X-Forwarded-Proto": "https"}`。 
        // Default is blank map.
        // 默认为空map。
        SSLProxyHeaders:         map[string]string{"X-Forwarded-Proto": "https"},
        // STSSeconds is the max-age of the Strict-Transport-Security header.
        // STSSeconds 是 Strict-Transport-Security 头信息的最大生命周期。
        // Default is 0, which would NOT include the header.
        // 默认为 0， 不会包含此头信息。
        STSSeconds:              315360000,                                                                                                                                                
        // If STSIncludeSubdomains is set to true,
        // 如果 STSIncludeSubdomains 设置为 true, 
        // the `includeSubdomains`
        // will be appended to the Strict-Transport-Security header. Default is false.
        // `includeSubdomains` 将被追加到 Strict-Transport-Security 头中。 默认为 false。
        STSIncludeSubdomains:    true,                                                                                                                                           
        // If STSPreload is set to true, the `preload`
        // 如果 STSPreload 设置为 true,
        // flag will be appended to the Strict-Transport-Security header.
        // `preload` 标记将被追加到 Strict-Transport-Security 头信息中。
        // Default is false.
        // 默认为 false。
        STSPreload:              true,    
        // STS header is only included when the connection is HTTPS. 
        // STS 头只有当连接为HTTPS时才会包含。
        // If you want to force it to always be added, set to true.
        // 如果你想强制带有话，设置为 true。 
        // `IsDevelopment` still overrides this. Default is false.
        // `IsDevelopment` 仍旧会重载这个参数。默认为 false。
        ForceSTSHeader:          false,  
        // If FrameDeny is set to true, adds the X-Frame-Options header with
        // 如果 FrameDeny 设置为 true，添加值为 `DENY` 的 X-Frame-Options 头信息
        // the value of `DENY`. Default is false.
        // 默认为 false。
        FrameDeny:               true,    
        // CustomFrameOptionsValue allows the X-Frame-Options header 
        // value to be set with a custom value. 
        // CustomFrameOptionsValue 允许设置带有自定义值的 X-Frame-Options 头信息。
        // This overrides the FrameDeny option.
        // 这个参数重载了 FrameDeny 选项。
        CustomFrameOptionsValue: "SAMEORIGIN",
        // If ContentTypeNosniff is true, adds the X-Content-Type-Options
        // 如果 ContentTypeNosniff 设置为 true, 将添加值为 `nosniff` 的 X-Content-Type-Options 头信息
        // header with the value `nosniff`. Default is false.
        // 默认为 false。
        ContentTypeNosniff:      true,  
        // If BrowserXssFilter is true, adds the X-XSS-Protection header 
        // 如果 BrowserXSSFilter 设置为 true, 添加值为 `1;mode=block` 的 X-XSS-Protection 头信息
        // with the value `1;mode=block`. Default is false.
        // 默认为 false。
        BrowserXSSFilter:        true,
        // ContentSecurityPolicy allows the Content-Security-Policy
        // ContentSecurityPolicy 允许设置自定义值的 Content-Security-Policy 头信息。
        // header value to be set with a custom value. Default is "".
        // 默认为 ""。
        ContentSecurityPolicy:   "default-src 'self'",   
        // PublicKey implements HPKP to prevent 
        // PublicKey 实现了HPKP以组织伪造证书的MITM攻击。
        // MITM attacks with forged certificates. Default is "".
        // 默认为 ""。
        PublicKey:               `pin-sha256="base64+primary=="; pin-sha256="base64+backup=="; max-age=5184000; includeSubdomains; report-uri="https://www.example.com/hpkp-report"`,
        // This will cause the AllowedHosts, SSLRedirect, 
        // 这个选项会造成 AllowedHosts, SSLRedirect,
        //..and STSSeconds/STSIncludeSubdomains options to be ignored during development.
        // 和 STSSeconds/STSIncludeSubdomains 等选项在开发期间被忽略掉。 
        // When deploying to production, be sure to set this to false.
        // 当部署到产品环境下时，一定要确定这个选项设置为false。
        IsDevelopment: true,
    })

    iris.UseFunc(func(c *iris.Context) {
        err := s.Process(c)

        // If there was an error, do not continue.
        // 如果有错误，不再继续
        if err != nil {
            return
        }

        c.Next()
    })

    iris.Get("/home", func(c *iris.Context) {
        c.Write("Hello from /home")
    })

    iris.Listen(":8080")
}


```