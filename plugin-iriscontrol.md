# 控制面板 / Control panel

[This is a plugin](https://github.com/iris-contrib/plugin/tree/master/iriscontrol) which is working but not finished yet.

[这是个插件](https://github.com/iris-contrib/plugin/tree/master/iriscontrol) 正在实现但还没有完成。


Which gives  access to your iris server's information via a web interface.

通过 web 接口给你访问 iris 服务器信息的权限。

> You need internet connection the first time you will run this plugin, because the assets don't exists to this repository but [here](https://github.com/iris-contrib/iris-control-assets). The plugin will install these for you at the first run.
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
