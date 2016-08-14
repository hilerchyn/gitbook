# Editor

[This is a plugin](https://github.com/iris-contrib/plugin/tree/master/editor).

[这是一个插件](https://github.com/iris-contrib/plugin/tree/master/editor)。

Editor Plugin is just a bridge between Iris and [alm-tools](http://alm.tools).

编辑器插件只是 Iris 和 [alm-tools](http://alm.tools)


[alm-tools](http://alm.tools) is a typescript online IDE/Editor, made by [@basarat](https://twitter.com/basarat) one of the top contributors of the [Typescript](http://www.typescriptlang.org).

[alm-tools](http://alm.tools) 是一个 typescript 在线 IDE/Editor, 由[Typescript](http://www.typescriptlang.org)的顶级贡献者之一[@basarat](https://twitter.com/basarat)创建。

Iris gives you the opportunity to edit your client-side using the alm-tools editor, via the editor plugin.

Iris 给你使用编辑器插件 alm-tools 编辑器来编辑客户端内容的机会。


This plugin starts it's own server, if Iris server is using TLS then the editor will use the same key and cert.

这个插件启动它自己的服务器，如果Iris 服务器使用TLS 那么编辑器将使用同样的key 和 cert。

## 如何使用 / How to use


```go
package main

import (
	"github.com/kataras/iris"
	"github.com/iris-contrib/plugin/editor"
)

func main(){
	e := editor.New() 
   // editor.Config{ Username: "admin", Password: "admin!123", Port: 4444, WorkingDir: "/public/scripts"}

	iris.Plugins.Add(e)

	iris.Get("/", func (ctx *iris.Context){})

	iris.Listen(":8080")
}


```

**Note for username, password**: The Authorization specifies the authentication mechanism (in this case Basic) followed by the username and password.
Although, the string aHR0cHdhdGNoOmY= may look encrypted it is simply a base64 encoded version of username:password.
Would be readily available to anyone who could intercept the HTTP request. [Read more here](https://www.httpwatch.com/httpgallery/authentication).

**注意username 和 password**: 身份认证指定了使用 username 和 password 的授权机制(示例中是最基本的)。最然字符串 aHR0cHdhdGNoOmY= 看起来像加密过的，它其实是使用 base64 加密的 username:password 。对任何能够拦截HTTP请求的过程而言是随时可用的。[了解更多](https://www.httpwatch.com/httpgallery/authentication).

> The editor can't work if the directory doesn't contains a [tsconfig.json](http://www.typescriptlang.org/docs/handbook/tsconfig.json.html).
> 
> 如果目录不含有[tsconfig.json](http://www.typescriptlang.org/docs/handbook/tsconfig.json.html)那么编辑器不能工作。

> If you are using the [typescript plugin](https://github.com/iris-contrib/plugin/tree/master/typescript) you don't have to call the .Dir(...)
> 
> 如果你正在使用[typescript 插件](https://github.com/iris-contrib/plugin/tree/master/typescript)，你没有必要调用 .Dir(...)
> 


