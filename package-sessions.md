# 会话 / Sessions
If you notice a bug or issue [post it here](https://github.com/kataras/iris/issues)

如果你发现错误或问题[在这里提出](https://github.com/kataras/iris/issues)


- Cleans the temp memory when a sessions is iddle, and re-allocate it , fast, to the temp memory when it's necessary. Also most used/regular sessions are going front in the memory's list.

- 当会话空闲时清理缓存，当需要时重新快速分配临时内存。另外最常用的／常规会话在内存列表的最前面

- Supports any type of database, currently only [redis](https://github.com/iris-contrib/sessiondb/).

- 支持任意类型的数据库，当前只有 [redis](https://github.com/iris-contrib/sessiondb/)。


**A session can be defined as a server-side storage of information that is desired to persist throughout the user's interaction with the web site** or web application.

**会话可以定义为服务器端的信息存储，贯穿在整个用户和网站(网页应用程序)之间的交互过程**

Instead of storing large and constantly changing information via cookies in the user's browser, **only a unique identifier is stored on the client side** (called a "session id"). This session id is passed to the web server every time the browser makes an HTTP request (ie a page link or AJAX request). The web application pairs this session id with it's internal database/memory and retrieves the stored variables for use by the requested page.

替代了在用户浏览器中通过 cookie 存储 大量并不断变更的信息，**只有一个唯一标识符存储在客户端**(被称作 "会话ID")。每次HTTP请求会话ID都会被传递给网站服务器(如:页面链接或AJAX请求)。网页应用程序使用内部的数据库/内存来匹配会话ID,并获取请求页面所存储的变量。

----



You will see two different ways to use the sessions, I'm using the first. No performance differences.

我会先试用，你将看到使用会话的两种不同方式。没有性能不同。

## 如何使用 / How to use

```go
package main

import	"github.com/kataras/iris"

func main() {
	/*  These are the optionally fields to configurate the sessions, using the station's Config field (iris.Config.Sessions)
	// 这些是配置会话的可选字段，使用站点的 Config 字段 (iris.Config.Sessions)

	// Cookie string, the session's client cookie name, for example: "irissessionid"
	// Cookie 字符串，会话在客户端的 cookie 名，如:"irissessionid"
	
	Cookie string
	// DecodeCookie set it to true to decode the cookie key with base64 URLEncoding
	// DecodeCookie 设置为 true 用来 使用 base64 URLEncoding 解码 cookie
	// Defaults to false
	// 默认为 false
	DecodeCookie bool
	// Expires the duration of which the cookie must expires (created_time.Add(Expires)).
	// Expires cookie 必须过期的持续时间 (created_time.Add(Expires))。
	// Default infinitive/unlimited life duration(0)
	// 默认为无限声明周期 0
	Expires time.Duration
	// GcDuration every how much duration(GcDuration) the memory should be clear for unused cookies (GcDuration)
	// GcDuration 每隔多久应该清理内存中为使用cookie
	// for example: time.Duration(2)*time.Hour. it will check every 2 hours if cookie hasn't be used for 2 hours,
	// deletes it from backend memory until the user comes back, then the session continue to work as it was
	// 如：time.Duration(2)*time.Hour ， 每隔2个小时会检测2个小时未使用的cookie，从后端内存中删除。直到用户回来，session才会继续跟之前一样工作。
	//
	// Default 2 hours
	// 默认为 2小时
	GcDuration time.Duration
	// DisableSubdomainPersistence set it to true in order dissallow your iris subdomains to have access to the session cookie
	// DisableSubdomainPersistence 为了禁用 iris 子域名访问session cookie 的权限，将其设置为 true
	// defaults to false
	// 默认为 false
	DisableSubdomainPersistence bool
	*/
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
		// get a specific key, as string, if no found returns just an empty string
		// 获取指定键的会话值，返回字符串，如果没有返现的话那么返回一个空字符串。
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
		//destroy, removes the entire session and cookie
		// 销毁、移除整个 session 和 cookie
		c.SessionDestroy()
		c.Log("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\n ame: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
		c.Write("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\nName: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
	})

	iris.Listen(":8080")
	//iris.ListenTLS("0.0.0.0:443", "mycert.cert", "mykey.key")
}


```

Example with **redis session database**, which located [here](https://github.com/iris-contrib/sessiondb/tree/master/redis)

**redis 会话数据库** 示例在 [这里](https://github.com/iris-contrib/sessiondb/tree/master/redis)

```go
package main

import (
	"github.com/iris-contrib/sessiondb/redis"
	"github.com/iris-contrib/sessiondb/redis/service"
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

		//set session values
		// 设置 session 值
		c.Session().Set("name", "iris")

		//test if setted here
		// 如果设置成功的话，在这里测试一下
		c.Write("All ok session setted to: %s", c.Session().GetString("name"))
	})

	iris.Get("/get", func(c *iris.Context) {
		// get a specific key, as string, if no found returns just an empty string
		// 获取指定键的会话值，返回字符串，如果没有返现的话那么返回一个空字符串。
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
		//destroy, removes the entire session and cookie
		// 销毁、移除整个 session 和 cookie
		c.SessionDestroy()
		c.Log("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\n ame: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))
		c.Write("You have to refresh the page to completely remove the session (on browsers), so the name should NOT be empty NOW, is it?\nName: %s\n\nAlso check your cookies in your browser's cookies, should be no field for localhost/127.0.0.1 (or what ever you use)", c.Session().GetString("name"))

	})

	iris.Listen(":8080")
}

```
