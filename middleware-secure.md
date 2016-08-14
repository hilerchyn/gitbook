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
        AllowedHosts:            []string{"ssl.example.com"},                                                                                                                         
        // AllowedHosts is a list of fully qualified domain names
        //that are allowed. Default is empty list, 
        //which allows any and all host names.
        // AllowedHosts 是一个被允许的完全限定域名列表。默认为空允许所有人和主机名。
        SSLRedirect:             true,     
        
        // If SSLRedirect is set to true, then only allow HTTPS requests.
        // 如果 SSLRedirect 设置为 true， 那么只允许 HTTPS 请求
        //Default is false.
        // 默认为 false。
        SSLTemporaryRedirect:    false,    
        
        // If SSLTemporaryRedirect is true, 
        //the a 302 will be used while redirecting.
        // 如果 SSLTemporaryRedirect 为 true，当重定向时使用 302 状态码。
        //Default is false (301).
        // 默认为 false (301)
        SSLHost:                 "ssl.example.com",
        
        // SSLHost is the host name that is used to 
        //redirect HTTP requests to HTTPS.
        // SSLHost 是用来将 HTTP 请求重定向后的 HTTPS 主机名
        //Default is "", which indicates to use the same host.
        // 默认为 "" 表明使用相同的主机名。
        SSLProxyHeaders:         map[string]string{"X-Forwarded-Proto": "https"},
        
        // SSLProxyHeaders is set of header keys with associated values 
        //that would indicate a 
        //valid HTTPS request. Useful when using Nginx: 
        //`map[string]string{"X-Forwarded-
        //Proto": "https"}`. Default is blank map.
        // SSLProxyHeaders 是 键－值 联合集 以表明有效的 HTTPS 请求。当使用NGINX很有用:`map[string]string{"X-Forwarded-Proto": "https"}`。 默认是空 map。
        
        STSSeconds:              315360000,                                                                                                                                           
        // STSSeconds is the max-age of the Strict-Transport-Security header.
        //Default is 0, which would NOT include the header.
        // STSSeconds 是 Strict-Transport-Security 头信息的最大声明周期。
        // 默认为 0， 头信息中不会包含此项设置。
        STSIncludeSubdomains:    true,                                                                                                                                                
        // If STSIncludeSubdomains is set to true, 
        //the `includeSubdomains`
        //will be appended to the Strict-Transport-Security header. Default is false.
        // 如果 STSIncludeSubdomains 设置为 true，那么 `includeSubdomains` 将被追加到 Strict-Transport-Security 头信息后面。默认为 false
        STSPreload:              true,    
        
        // If STSPreload is set to true, the `preload`
        //flag will be appended to the Strict-Transport-Security header.
        // 如果 STSPreload 设置为 true, 那么 `preload` 标记会被追加到 Strict-Transport-Security 头信息后面。
        //Default is false.
        // 默认为 false。
        ForceSTSHeader:          false,  
        
        // STS header is only included when the connection is HTTPS. 
        //If you want to force it to always be added, set to true. 
        //`IsDevelopment` still overrides this. Default is false.
        // STS 头信息只有 HTTPS 请求时才会包含，如果你希望强制添加这个头信息，将其设置为 true。
        // `IsDevelopmen` 仍然会重载这个设置，默认为 false。
        FrameDeny:               true,    
        // If FrameDeny is set to true, adds the X-Frame-Options header with
        //the value of `DENY`. Default is false.
        // 如果 FrameDeny 设置为 true, 添加值为 `DENY` 的 X-Frame-Options 头信息。 默认为 false
        CustomFrameOptionsValue: "SAMEORIGIN",
        // CustomFrameOptionsValue allows the X-Frame-Options header 
        //value to be set with
        //a custom value. This overrides the FrameDeny option.
        // CustomFrameOptionsValue 允许使用自定义值的 X-Frame-Options 的头信息重载 FrameDeny 选项。
        ContentTypeNosniff:      true,  
        // If ContentTypeNosniff is true, adds the X-Content-Type-Options
        //header with the value `nosniff`. Default is false.
        // 如果 ContentTypeNosniff 设置为 true，添加值为 `nosniff` 的 X-Content-Type-Options 头信息。默认为 false
        BrowserXSSFilter:        true,
        // If BrowserXssFilter is true, adds the X-XSS-Protection header 
        //with the value `1;mode=block`. Default is false.
        // 如果 BrowserXssFilter 设置为 true， 添加值为 `1;mode=block` 的 X-XSS-Protection 头信息。默认值为 false。
        ContentSecurityPolicy:   "default-src 'self'",   
        // ContentSecurityPolicy allows the Content-Security-Policy
        //header value to be set with a custom value. Default is "".
        // ContentSecurityPolicy 允许 使用自定义值的 Content-Security-Policy 头信息。默认为 ""。
        PublicKey:               `pin-sha256="base64+primary=="; pin-sha256="base64+backup=="; max-age=5184000; includeSubdomains; report-uri="https://www.example.com/hpkp-report"`,
        // PublicKey implements HPKP to prevent 
        //MITM attacks with forged certificates. Default is "".
        // PublicKey 实现了 HPKP 防止伪造证书的 MITM 攻击。默认为 ""。

        IsDevelopment: true,
        // This will cause the AllowedHosts, SSLRedirect, 
        //..and STSSeconds/STSIncludeSubdomains options to be 
        //ignored during development. 
        // 这个设置将造成 AllowedHosts, SSLRedirect,和  STSSeconds/STSIncludeSubdomains 等选项在开发期间被忽略。
        //When deploying to production, be sure to set this to false.
        // 当部署到产品环境时，一定要设置成 false。
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