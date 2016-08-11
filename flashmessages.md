# 即显消息 / Flash messages

**A flash message is used in order to keep a message in session through one or several requests of the same user**. By default, it is removed from session after it has been displayed to the user. Flash messages are usually used in combination with HTTP redirections, because in this case there is no view, so messages can only be displayed in the request that follows redirection.

**即显信息 通常被用做在同一用户的一个或几个请求会话中保存一个消息**。默认消息被显示以后就从会话中移除。即显消息通常跟 HTTP 重定向组合使用，因为在这种情况下不需要重新查看，所以消息职能在随后的重定向请求中显示。

**A flash message has a name and a content (AKA key and value). It is an entry of a map**. The name is a string: often "notice", "success", or "error", but it can be anything. The content is usually a string. You can put HTML tags in your message if you display it raw. You can also set the message value to a number or an array: it will be serialized and kept in session like a string.

**即显消息 有自己的名称和内容(可以看作 键 和 值)。它其实就是一个映射（map）**。名称是字符串：通常用 "noticee", "success", 或 "error"， 当然也可以是其它任何字符串。内容通常也是个字符串。如果你原本显示内容的话，可以在内容里使用 HTML 标签。你也可以把消息的内容设置为一个数字或一个数组：会被序列化后像字符串一样保存在会话中。

----


```go

// SetFlash sets a flash message, accepts 2 parameters the key(string) and the value(string)
// SetFlash 设置即显消息，接收2个参数 key(string) 和 value(string)
// the value will be available on the NEXT request
// 设置内容在下个请求中可用
SetFlash(key string, value string)

// GetFlash get a flash message by it's key
// returns the value as string and an error
// GetFlash 通过key获取即显消息，返回内容和一个error
//
// if the cookie doesn't exists the string is empty and the error is filled
// 如果 cookie 不存在，那么返回的内容字符串为空且有error信息
// after the request's life the value is removed
// 请求结束后内容会被移除
GetFlash(key string) (value string, err error)

// GetFlashes returns all the flash messages for available for this request 
// GetFlashes 返回本次请求可用的所有即显信息
GetFlashes() map[string]string
```

Example

示例

```go

package main

import (
	"github.com/kataras/iris"
)

func main() {

	iris.Get("/set", func(c *iris.Context) {
		c.SetFlash("name", "iris")
		c.Write("Message setted, is available for the next request")
	})

	iris.Get("/get", func(c *iris.Context) {
		name, err := c.GetFlash("name")
		if err != nil {
			c.Write(err.Error())
			return
		}
		c.Write("Hello %s", name)
	})

	iris.Get("/test", func(c *iris.Context) {

		name, err := c.GetFlash("name")
		if err != nil {
			c.Write(err.Error())
			return
		}

		c.Write("Ok you are comming from /set ,the value of the name is %s", name)
		c.Write(", and again from the same context: %s", name)

	})

	iris.Listen(":8080")
}


```