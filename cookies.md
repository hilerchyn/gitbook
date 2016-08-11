# Cookies

Cookie management, even your little brother can do this!

管理饼干(Cookie), 即使你弟弟也可以做到！

```go
// SetCookie adds a cookie
// SetCookie 添加一个 cookie
SetCookie(cookie *fasthttp.Cookie)

// SetCookieKV adds a cookie, receives just a key(string) and a value(string)
// SetCookieKV 添加一个cookie, 只接收 key(string) 和 value(string)
SetCookieKV(key, value string)

// GetCookie returns cookie's value by it's name
// GetCookie 通过名称返回cookie的值
// returns empty string if nothing was found
// 如果没有找到的话返回空字符串
GetCookie(name string) string 

// RemoveCookie removes a cookie by it's name/key
// RemoveCookie 通过名称/key 移除 cookie
RemoveCookie(name string) 


// VisitAllCookies takes a visitor which loops on each (request's) cookie key and value
// VisitAllCookies 使用一个访问器循环每个请求的 cookie 的 key 和 value
//
// Note: the method ctx.Request.Header.VisitAllCookie by fasthttp, has a strange bug which I cannot solve at the moment.
// 注意: ctx.Request.Header.VisitAllCookie 方法来自 fasthttp, 现在有一个我无法解决的奇怪bug。
// This is the reason which this function exists and should be used instead of fasthttp's built'n.
// 这就是为什么这个函数存在的原因，应该用它替代 fasthttp 内建的函数
VisitAllCookies(visitor func(key string, value string))

```
How to use 

如何使用

```go

iris.Get("/set", func(c *iris.Context){
    c.SetCookieKV("name","iris")
    c.Write("Cookie has been setted.")
})

iris.Get("/get", func(c *iris.Context){
    name := c.GetCookie("name")
    c.Write("Cookie's value: %s", name)
})

iris.Get("/remove", func(c *iris.Context){
    if name := c.GetCookie("name"); name != "" {
       c.RemoveCookie("name")  
    }
    c.Write("Cookie has been removed.")
})

```