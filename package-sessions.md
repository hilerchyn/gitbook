# 会话 / Sessions
If you notice a bug or issue [post it here](https://github.com/kataras/go-sessions).

如果你发现错误或问题[在这里提出](https://github.com/kataras/go-sessions).

- Cleans the temp memory when a session is idle, and re-allocates it to the temp memory when it's necessary. 
The most used sessions are optimized to be in the front of the memory's list.

- 当回话空闲时清除临时内存，当需要时将它重新分配为临时内存。
使用的最多的回话都被优化且放置在内存列表的前端。

- Supports any type of database, currently only [Redis](https://github.com/kataras/go-sessions/sessiondb/).

- 支持任意类型的数据库，当前只有[Redis](https://github.com/kataras/go-sessions/sessiondb/)。

**A session can be defined as a server-side storage of information that is desired to persist throughout the user's interaction with the web application**.

**会话可以定义为服务器端的信息存储，贯穿在整个用户和网站(网页应用程序)之间的交互过程**

Instead of storing large and constantly changing data via cookies in the user's browser (i.e. CookieStore), 
**only a unique identifier is stored on the client side** called a "session id". 
This session id is passed to the web server on every request. 
The web application uses the session id as the key for retrieving the stored data from the database/memory. The session data is then available inside the iris.Context.

替代了在浏览器中(如  (i.e. CookieStore))使用cookie储存大量、持续变化的数据。
**只有一个唯一标识符存储在客户端**被称作 "会话ID"。
每次请求这个会话ID都被传递给网站服务器。
网站应用程序使用会话ID作为键从数据库／内存中获取存储的数据。在iris.Context中可以使用这些回话数据。

----



Here you see two different ways to use the sessions, we are using the first in this example. There are no performance differences.

这里你将看到两种使用会话的不同方式，我们将使用第一种方式在这个例子中，没有什么性能上的不同。

## How to use

## 如何使用

```go
package main

import	"github.com/kataras/iris"

func main() {

	// These are the optional fields to configurate sessions, 
	// 这些是配置会话的可选字段，
	// using the station's Config field (iris.Config.Sessions)
	// 使用站点的 Config 字段 (iris.Config.Sessions)

	// Cookie string, the session's client cookie name, for example: "qsessionid"
	// Cookie string, 会话的客户端cookie名称，若: "qsessionid"
	Cookie string
	// DecodeCookie set it to true to decode the cookie key with base64 URLEncoding
	// DecodeCookie 设置为 true 用来 使用 base64 URLEncoding 解码 cookie
	// Defaults to false
	// 默认为 false
	DecodeCookie bool

	// Expires the duration of which the cookie must expires (created_time.Add(Expires)).
	// Expires cookie 必须过期的持续时间 (created_time.Add(Expires))。
	// If you want to delete the cookie when the browser closes, set it to -1 but in this case, the server side's session duration is up to GcDuration
	// 如果你想要在浏览器关闭的时候删除cookie，那么设置为 -1。但是在此示例中，服务器端的持续时间取决于 GcDuration。
	//
	// Default infinitive/unlimited life duration(0)
	// 默认为无限生命周期(0)
	Expires time.Duration

	// CookieLength the length of the sessionid's cookie's value, let it to 0 if you don't want to change it
	// CookieLength 是会话ID的cookie的值的长度，如果你不想改变它的话那么将其设置为0。
	// Defaults to 32
	// 默认为 32
	CookieLength int

	// GcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
	// GcDuration 是每隔多久应该清楚未使用cookie的时间周期
	// for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
	// deletes it from backend memory until the user comes back, then the session continue to work as it was
	// 如：time.Duration(2)*time.Hour ， 每隔2个小时会检测2个小时未使用的cookie，
	// 从后端内存中删除。直到用户回来，session才会继续跟之前一样工作。
	//
	// Default 2 hours
	// 默认为 2小时
	GcDuration time.Duration

	// DisableSubdomainPersistence set it to true in order dissallow your q subdomains to have access to the session cookie
	// DisableSubdomainPersistence 为了禁用 iris 子域名访问session cookie 的权限，将其设置为 true
	// defaults to false
	// 默认为 false
	DisableSubdomainPersistence bool

	iris.Get("/", func(c *iris.Context) {
		c.Write("You should navigate to the /set, /get, /delete, /clear,/destroy instead")
	})

	iris.Get("/set", func(c *iris.Context) {

		//set session values
		// 设置 session 值
		c.Session().Set("name", "iris")

		//test if setted here
		// 如果设置成功的话，在这里测试一下
		c.Write("All ok session setted to: %s", c.Session().GetString("name"))
	})

	iris.Get("/get", func(c *iris.Context) {
		// get a specific key as a string.
		// 获取指定键的会话值的字符串，
		// returns an empty string if the key was not found.
		// 如果没有发现该键的话返回一个空字符串。
		name := c.Session().GetString("name")

		c.Write("The name on the /set was: %s", name)
	})

	iris.Get("/delete", func(c *iris.Context) {
		// delete a specific key
		// 删除指定的键
		c.Session().Delete("name")
	})

	iris.Get("/clear", func(c *iris.Context) {
		// removes all entries
		// 删除所有会话实体
		c.Session().Clear()
	})

	iris.Get("/destroy", func(c *iris.Context) {
		// destroy/removes the entire session and cookie
		// 销毁、移除整个 session 和 cookie
		c.SessionDestroy()
		c.Log("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\n ame: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or whatever you use)", c.Session().GetString("name"))
		c.Write("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\nName: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or whatever you use)", c.Session().GetString("name"))
	})

	iris.Listen(":8080")
	//iris.ListenTLS("0.0.0.0:443", "mycert.cert", "mykey.key")
}


```

Example with **Redis session database**, which is located [here]

**redis 会话数据库** 示例在 [这里](https://github.com/kataras/go-sessions/tree/master/sessiondb/redis).

```go
package main

import (
	"github.com/kataras/go-sessions/sessiondb/redis"
	"github.com/kataras/go-sessions/sessiondb/redis/service"
	"github.com/kataras/iris"
)

func main() {
	db := redis.New(service.Config{Network: service.DefaultRedisNetwork,
		Addr:          service.DefaultRedisAddr,
		Password:      "",
		Database:      "",
		MaxIdle:       0,
		MaxActive:     0,
		IdleTimeout:   service.DefaultRedisIdleTimeout,
		Prefix:        "",
		MaxAgeSeconds: service.DefaultRedisMaxAgeSeconds}) // optionally configure the bridge between your redis server / redis 服务器之间的可选桥接配置

	iris.UseSessionDB(db)

	iris.Get("/set", func(c *iris.Context) {

		// set session values
		// 设置 session 值
		c.Session().Set("name", "iris")

		// test if set here
		// 如果设置成功的话，在这里测试一下
		c.Write("All ok session set to: %s", c.Session().GetString("name"))
	})

	iris.Get("/get", func(c *iris.Context) {
		// get a specific key as a string.
		// 获取指定键的会话值的字符串,
		// returns an empty string if the key was not found.
		// 如果没有发现键的话则返回空字符串。
		name := c.Session().GetString("name")

		c.Write("The name on the /set was: %s", name)
	})

	iris.Get("/delete", func(c *iris.Context) {
		// delete a specific key
		// 删除指定的键
		c.Session().Delete("name")
	})

	iris.Get("/clear", func(c *iris.Context) {
		// removes all entries
		// 删除所有会话实体
		c.Session().Clear()
	})

	iris.Get("/destroy", func(c *iris.Context) {
		// destroy/removes the entire session and cookie
		// 销毁、移除整个 session 和 cookie
		c.SessionDestroy()
		c.Log("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\n ame: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
		c.Write("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\nName: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
	})

	iris.Listen(":8080")
}

```
