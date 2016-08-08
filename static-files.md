# 静态文件 / Static files

Serve a static directory

服务于一个静态目录

```go

// StaticHandler returns a HandlerFunc to serve static system directory
// StaticHandler 返回一个 HandlerFunc 用来服务于静态系统目录
// Accepts 5 parameters
// 接受 5 个参数
//
// first is the systemPath (string)
// 第1个是 systemPath (string)
// Path to the root directory to serve files from.
// 服务的文件来自于路径的跟目录。
//
// second is the  stripSlashes (int) level
// 第2个是 stripSlashed (int) 级别
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 0, 原始路径: "/foo/bar", 结果: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 1, 原始路径: "/foo/bar", 结果: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
// * stripSlashes = 2, 原始路径: "/foo/bar", 结果: ""
//
// third is the compress (bool)
// 第3个是 compress (bool)
// Transparently compresses responses if set to true.
// 如果设置为true的话将透明压缩响应。
//
// The server tries minimizing CPU usage by caching compressed files.
// 服务器通过缓存压缩后的文件来尝试最小化CPU的使用率。
// It adds FSCompressedFileSuffix suffix to the original file name and
// 它在原始文件名上添加了 FSCompressedFileSuffix 后缀，
// tries saving the resulting compressed file under the new file name.
// 并尝试用新的文件名来保存压缩后的文件结果。
// So it is advisable to give the server write access to Root
// 所以建议给服务器开放向Root目录操作的写权限,
// and to all inner folders in order to minimze CPU usage when serving
// compressed responses.
// 包括根目录下的所有文件夹，这样当压缩应答服务时可以最小化 CPU 使用率。
//
// fourth is the generateIndexPages (bool)
// 第4个是 generateIndexPages (bool)
// Index pages for directories without files matching IndexNames
// 目录索引页，当目录中没有文件匹配到索引文件名时
// are automatically generated if set.
// 如果设置了的话将自动生成。
//
// Directory index generation may be quite slow for directories
// 目录索引生成也许相当慢当目录中
// with many files (more than 1K), so it is discouraged enabling
// 有许多文件(多于1K)时，所以不鼓励对这样的目录启用索引页生成。
// index pages' generation for such directories.
//
// fifth is the indexNames ([]string)
// 第5个是索引名 ([]string)
// List of index file names to try opening during directory access.
// 当尝试打开访问目录时的索引文件名列表。
//
// For example:
// 例如：
//
//     * index.html
//     * index.htm
//     * my-super-index.xml
//
StaticHandler(systemPath string, stripSlashes int, compress bool,
                  generateIndexPages bool, indexNames []string) HandlerFunc 

// Static registers a route which serves a system directory
// Static 注册一个用于服务系统目录的路由
// this doesn't generates an index page which list all files
// 这不会生成列出所有文件的索引页
// no compression is used also, for these features look at StaticFS func
// 也不会用到压缩, 可以在 StaticFS 函数中查看这些特性
// accepts three parameters
// 接受3个参数
// first parameter is the request url path (string)
// 第1个参数是请求url的路径 (string)
// second parameter is the system directory (string)
// 第2个参数是系统目录 (string)
// third parameter is the level (int) of stripSlashes
// 第3个参数是带路径分割线的层级 (int)
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 0, 原始路径: "/foo/bar", 结果: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 1, 原始路径: "/foo/bar", 结果: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
// * stripSlashes = 2, 原始路径: "/foo/bar", 结果: ""
Static(relative string, systemPath string, stripSlashes int)

// StaticFS registers a route which serves a system directory
// StaticFS 注册一个服务于系统目录的路由
// generates an index page which list all files
// 生成一个所有文件的列表索引页
// uses compression which file cache, if you use this method it will generate compressed files also
// 使用压缩做文件缓存，如果你使用该方法的话它也会生成压缩文件
// think this function as small fileserver with http
// 可以把这个函数做为为一个小的http文件服务器使用
// accepts three parameters
// 接受3个参数
// first parameter is the request url path (string)
// 第1个参数是url请求的路径 (string)
// second parameter is the system directory (string)
// 第2个参数是系统目录 (string)
// third parameter is the level (int) of stripSlashes
// 第3个参数是带路径分隔符的层级 (int)
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 0, 原始路径: "/foo/bar", 结果: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 1, 原始路径: "/foo/bar", 结果: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
// * stripSlashes = 2, 原始路径: "/foo/bar", 结果: ""
StaticFS(relative string, systemPath string, stripSlashes int)

// StaticWeb same as Static but if index.html e
// StaticWeb 与 Static 相同
// xists and request uri is '/' then display the index.html's contents
// 如果 index.html 存在且 uri 是 '/' 那么显示 index.html 的内容
// accepts three parameters
// 接受3个参数
// first parameter is the request url path (string)
// 第1个参数是url请求的路径 (string)
// second parameter is the system directory (string)
// 第2个参数是系统目录 (string)
// third parameter is the level (int) of stripSlashes
// 第3个参数是带路径分隔符的层级 (int)
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 0, 原始路径: "/foo/bar", 结果: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 1, 原始路径: "/foo/bar", 结果: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
// * stripSlashes = 2, 原始路径: "/foo/bar", 结果: ""
StaticWeb(relative string, systemPath string, stripSlashes int)

// StaticServe serves a directory as web resource
// StaticServe 服务于网站资源目录
// it's the simpliest form of the Static* functions
// 它是 Static* 函数最简单的形式
// Almost same usage as StaticWeb
// 用例与 StaticWeb 几乎相同
// accepts only one required parameter which is the systemPath 
// 如果第2个参数为空的话，只接收系统路径一个必须参数
// ( the same path will be used to register the GET&HEAD routes)
// ( 相同的路径被用作注册 GET&HEAD 路由 )
// if second parameter is empty, otherwise the requestPath is the second parameter
// 否则的话第2个参数是请求路径
// it uses gzip compression (compression on each request, no file cache)
// 它使用 gzip 压缩 (每次请求都会压缩，没有文件缓存)
StaticServe(systemPath string, requestPath ...string)

```

```go

iris.Static("/public", "./static/assets/", 1)
//-> /public/assets/favicon.ico
```

```go
iris.StaticFS("/ftp", "./myfiles/public", 1)
```

```go
iris.StaticWeb("/","./my_static_html_website", 1)
```

```go
StaticServe(systemPath string, requestPath ...string)
```

### 手动静态文件服务 / Manual static file serving

```go
// ServeFile serves a view file, to send a file
// ServeFile 服务于一个视图文件
// to the client you should use the SendFile(serverfilename,clientfilename)
// 使用 SendFile(serverfilename,clientfilename) 向客户端发送文件
// receives two parameters
// 接收2个参数
// filename/path (string)
// 文件路径 (string)
// gzipCompression (bool)
// gzip 压缩 (bool)
//
// You can define your own "Content-Type" header also, after this function call
// 调用该函数后，你可以定义自己的 "Content-Type" 头信息
ServeFile(filename string, gzipCompression bool) error 
```

Serve static individual file

// 服务于独立的静态文件

```go

iris.Get("/txt", func(ctx *iris.Context) {
    ctx.ServeFile("./myfolder/staticfile.txt", false)
}

```

For example if you want manual serve static individual files dynamically you can do something like that:

如下面的示例，如果你想要动态的手动服务于独立文件，可以这样做:

```go
package main

import (
    "strings"
    "github.com/kataras/iris"
    "github.com/kataras/iris/utils"
)

func main() {

    iris.Get("/*file", func(ctx *iris.Context) {
            requestpath := ctx.Param("file")

            path := strings.Replace(requestpath, "/", utils.PathSeperator, -1)

            if !utils.DirectoryExists(path) {
                ctx.NotFound()
                return
            }

            ctx.ServeFile(path, false) // make this true to use gzip compression / 设置为 true 则使用 gzip 压缩
    })
    iris.Listen(":8080")
}



```

The previous example is almost identical with

前面的例子跟下面这个函数几乎完全相同

```go
StaticServe(systemPath string, requestPath ...string)
```

```go
func main() {
  iris.StaticServe("./mywebpage")
  // Serves all files inside this directory to the GET&HEAD route: 0.0.0.0:8080/mywebpage
  // 为 GET & HEAD 路由，提供目录内的所有文件的服务
  // using gzip compression ( no file cache, for file cache with zipped files use the StaticFS)
  // 使用 gzip 压缩 ( 没有文件缓存，压缩的文件缓存使用 StaticFS)
  iris.Listen(":8080")
}

```

```go
func main() {
  iris.StaticServe("./static/mywebpage","/webpage")
  // Serves all files inside filesystem path ./static/mywebpage to the GET&HEAD route: 0.0.0.0:8080/webpage
  // 将文件系统路径 ./static/mywebpage 内的所有文件 向 GET&HEAD 路由: 0.0.0.0:8080/webpage 提供服务
  iris.Listen(":8080")
}

```

## Favicon

Imagine that we have a folder named `static` which has subfolder `favicons` and this folder contains a favicon, for example `iris_favicon_32_32.ico`.

设想我们有一个 `static` 文件夹，内有 `favicons` 子文件夹并且有一个 favicon 文件, 如 `iris_favicon_32_32.ico` 。

```go
// ./main.go
package main

import "github.com/kataras/iris"

func main() {
    iris.Favicon("./static/favicons/iris_favicon_32_32.ico")

    iris.Get("/", func(ctx *iris.Context) {
        ctx.HTML(iris.StatusOK, "You should see the favicon now at the side of your browser.")
    })

    iris.Listen(":8080")
}


```

Practical example [here](https://github.com/iris-contrib/examples/tree/master/favicon)

练习示例 [在这里](https://github.com/iris-contrib/examples/tree/master/favicon)
