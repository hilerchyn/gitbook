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
// this doesn't generates an index page which list all files
// no compression is used also, for these features look at StaticFS func
// accepts three parameters
// first parameter is the request url path (string)
// second parameter is the system directory (string)
// third parameter is the level (int) of stripSlashes
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
Static(relative string, systemPath string, stripSlashes int)

// StaticFS registers a route which serves a system directory
// generates an index page which list all files
// uses compression which file cache, if you use this method it will generate compressed files also
// think this function as small fileserver with http
// accepts three parameters
// first parameter is the request url path (string)
// second parameter is the system directory (string)
// third parameter is the level (int) of stripSlashes
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
StaticFS(relative string, systemPath string, stripSlashes int)

// StaticWeb same as Static but if index.html e
// xists and request uri is '/' then display the index.html's contents
// accepts three parameters
// first parameter is the request url path (string)
// second parameter is the system directory (string)
// third parameter is the level (int) of stripSlashes
// * stripSlashes = 0, original path: "/foo/bar", result: "/foo/bar"
// * stripSlashes = 1, original path: "/foo/bar", result: "/bar"
// * stripSlashes = 2, original path: "/foo/bar", result: ""
StaticWeb(relative string, systemPath string, stripSlashes int)

// StaticServe serves a directory as web resource
// it's the simpliest form of the Static* functions
// Almost same usage as StaticWeb
// accepts only one required parameter which is the systemPath 
// ( the same path will be used to register the GET&HEAD routes)
// if second parameter is empty, otherwise the requestPath is the second parameter
// it uses gzip compression (compression on each request, no file cache)
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

### Manual static file serving

```go
// ServeFile serves a view file, to send a file
// to the client you should use the SendFile(serverfilename,clientfilename)
// receives two parameters
// filename/path (string)
// gzipCompression (bool)
//
// You can define your own "Content-Type" header also, after this function call
ServeFile(filename string, gzipCompression bool) error 
```

Serve static individual file

```go

iris.Get("/txt", func(ctx *iris.Context) {
    ctx.ServeFile("./myfolder/staticfile.txt", false)
}

```

For example if you want manual serve static individual files dynamically you can do something like that:

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

            ctx.ServeFile(path, false) // make this true to use gzip compression
    }
}

iris.Listen(":8080")

```

The previous example is almost identical with

```go
StaticServe(systemPath string, requestPath ...string)
```

```go
func main() {
  iris.StaticServe("./mywebpage")
  // Serves all files inside this directory to the GET&HEAD route: 0.0.0.0:8080/mywebpage
  // using gzip compression ( no file cache, for file cache with zipped files use the StaticFS)
  iris.Listen(":8080")
}

```

```go
func main() {
  iris.StaticServe("./static/mywebpage","/webpage")
  // Serves all files inside filesystem path ./static/mywebpage to the GET&HEAD route: 0.0.0.0:8080/webpage
  iris.Listen(":8080")
}

```

## Favicon

Imagine that we have a folder named `static` which has subfolder `favicons` and this folder contains a favicon, for example `iris_favicon_32_32.ico`.

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

