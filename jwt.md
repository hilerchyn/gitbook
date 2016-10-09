## JSON 网站令牌 / JSON Web Tokens


This is a [middleware](https://github.com/iris-contrib/middleware/tree/master/jwt).

这是一个[中间件](https://github.com/iris-contrib/middleware/tree/master/jwt).

## JSON 网站令牌 是什么? / What is it?

[JWT.io](https://jwt.io) has a great [introduction](https://jwt.io/introduction/) to JSON Web Tokens.

[JWT.io](https://jwt.io) 有关于 JSON 网站令牌的一个很好的 [说明](https://jwt.io/introduction/)

In short, it's a signed JSON object that does something useful (for example: authentication). 
It's commonly used for Bearer tokens in Oauth 2. A token is made of three parts, separated by .'s. 
The first two parts are JSON objects, that have been base64url encoded. The last part is the signature, encoded the same way.

简单来说，它是一个带签名的 JSON 对象，做些有用的事(如，身份认证)。
在 Oauth2 中它通常用做持票人(Bearer)令牌。一个令牌由三部份构成，由 "." 符号分割。
前两部分是 被 base64url 加密的 JSON 对象。最后一部分是签名，使用同样的加密方式。


The first part is called the header. It contains the necessary information for verifying the last part, the signature. 
For example, which encryption method was used for signing and what key was used.

第一部分被称作头，包含有验证最后一部分的必要信息 —— 签名。
例如，哪一个加密啊方法被用做签名，使用了哪个key。

The part in the middle is the interesting bit. It's called the Claims and contains the actual stuff you care about. 
Refer to the RFC for information about reserved keys and the proper way to add your own.

中间部分有点意思，被称作声明，含有你实际关心的东西。
可参考 RFC 上关于保留关键字和添加自己信息的合适方式的信息，


## 示例 / Example

```go
package main

import (
	"github.com/dgrijalva/jwt-go"
	jwtmiddleware "github.com/iris-contrib/middleware/jwt"
	"github.com/kataras/iris"
)

func main() {

	myJwtMiddleware := jwtmiddleware.New(jwtmiddleware.Config{
		ValidationKeyGetter: func(token *jwt.Token) (interface{}, error) {
			return []byte("My Secret"), nil
		},
		SigningMethod: jwt.SigningMethodHS256,
	})

	iris.Get("/ping", PingHandler)

	iris.Get("/secured/ping", myJwtMiddleware.Serve, SecuredPingHandler)
	iris.Listen(":8080")

}

type Response struct {
	Text string `json:"text"`
}

func PingHandler(ctx *iris.Context) {
	response := Response{"All good. You don't need to be authenticated to call this"}
	ctx.JSON(iris.StatusOK, response)
}

func SecuredPingHandler(ctx *iris.Context) {
	response := Response{"All good. You only get this message if you're authenticated"}
	// get the *jwt.Token which contains user information using:
	// 获取含有用户信息的 *jwt.Token 使用如下方法:
	// user := myJwtMiddleware.Get(ctx) or context.Get("jwt").(*jwt.Token)
	ctx.JSON(iris.StatusOK, response)
}
```