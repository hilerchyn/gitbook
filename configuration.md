# 配置 / Configuration

Configuration is a relative package `github.com/kataras/iris/config`

配置有一个相应的包 `github.com/kataras/iris/config`

> No need to download it separately, it's downloaded automatically when you install Iris.
> 
> 不需要单独下载，当你安装 Iris 时，它已经被自动下载了。

### 为什么? / Why?

I took this decision after a lot of thought and I ensure you that this is the best and easiest
architecture:

考虑了很多并确定这是最好且最简单的架构，做才拿定这个主意：

* change the configs without needing to re-write all of their fields.

* 改变配置不需要重写所有的字段。

  ```go
    irisConfig := config.Iris{ DisablePathCorrection: true }
    api := iris.New(irisConfig)
  ```

* **easy to remember**: `iris` type takes `config.Iris`, sessions takes `config.Sessions`, `iris.Config.Render` is of type `config.Render`, `iris.Config.Render.Template` is the type `config.Template`, `Logger` takes `config.Logger` and so on...

* **记忆简单**: `iris` 类型使用 `config.Iris`, 会话使用 `config.Sessions`, `iris.Config.Render` 是 `config.Render` 类型, `iris.Config.Render.Template` 是 `config.template` 类型, `Loger` 使用 `config.Logger` 等等...

* **easy to search & find out what features exists and what you can change**: just navigate to the config folder and open the type you want to learn about, for example `/websocket.go /iris.Websocket 's`configuration is inside `/config/websocket.go`

* **容易查询、找出已存在的特性和你能改变的内容**: 找到源码中的 config 文件夹，打开你想要了解的类型，例如 `/websocket.go /iris.Websocket's` 配置信息在 `/config/websocket.go`

* Enables you to do this **without setting up a config yourself**: `iris.Config.Gzip = true` or `iris.Config.Charset = "UTF-8"`.

* 使你可以 **不需要设置自己的配置**

* **\(Advanced usage\) merge configs**:

* **\(高级用例\) 合并配置**:


```go
//...
import "github.com/kataras/iris/config"
//...
websocketFromDefault:= config.DefaultWebsocket()
//....
websocketManual:= config.Websocket{  Endpoint: "/ws"}

websocketConfig := websocketFromDefault.MergeSingle(websocketManual)

```

## [Click here to view all station's configs](https://github.com/kataras/iris/tree/master/config)

## [点击这里查看所有站点的配置](https://github.com/kataras/iris/tree/master/config)

