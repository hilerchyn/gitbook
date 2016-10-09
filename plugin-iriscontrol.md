# 控制面板 / Control panel

[This is a plugin](https://github.com/iris-contrib/plugin/tree/master/iriscontrol) which is working but still work in progress.

[这是个插件](https://github.com/iris-contrib/plugin/tree/master/iriscontrol) 可用但是仍旧在开发过程中。

It gives you access to information/stats about your iris server via a web interface.
通过 web 接口让你了解 iris 服务器信息／状态。
> You need an internet connection the first time you will run this plugin, because the assets don't exist in the repository (but [here](https://github.com/iris-contrib/iris-control-assets)). The plugin will install these for you at the first run.
> 
> 你第一次运行这个插件你需要网络连接，因为这个仓库不存在辅助资源，而是在[这里](https://github.com/iris-contrib/iris-control-assets)。第一次运行时插件会安装这些东西。

-----

How to use

如何使用

```go
iriscontrol.New(port int, authenticatedUsers map[string]string) iris.IPlugin
```

Example

示例

```go
package main

import (
    "github.com/kataras/iris"
    "github.com/iris-contrib/plugin/iriscontrol"
)

func main() {

    iris.Plugins.Add(iriscontrol.New(9090, map[string]string{
        "irisusername1": "irispassword1",
        "irisusername2": "irispassowrd2",
    }))
    //or
    // ....
    // iriscontrol.New(iriscontrol.Config{...})

    iris.Get("/", func(ctx *iris.Context) {
    })

    iris.Post("/something", func(ctx *iris.Context) {
    })

    iris.Listen(":8080")
}

```
