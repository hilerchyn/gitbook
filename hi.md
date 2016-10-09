# 嘿 Hi

```go
package main

import "github.com/kataras/iris"

func main() {
    iris.Get("/hi", func(ctx *iris.Context) {
        ctx.Write("Hi %s", "iris")
    })
    iris.Listen(":8080")
}

```

The same:

同样可写成

```go
package main

import "github.com/kataras/iris"

func main() {
    api := iris.New()
    api.Get("/hi", hi)
    api.Listen(":8080")
}

func hi(ctx *iris.Context){
   ctx.Write("Hi %s", "iris")
}

```

Rich Hi with **html\/template**:

使用 **html\/template** 丰富 Hi 示例

```html
<!-- ./templates/hi.html -->
<html><head> <title> Hi Iris</title> </head>
  <body>
    <h1> Hi {{.Name}} </h1>
  </body>
</html>


```

```go
// ./main.go
import "github.com/kataras/iris"

func main() {
    iris.Get("/hi", hi)
    iris.Listen(":8080")
}

func hi(ctx *iris.Context){
   ctx.Render("hi.html", struct { Name string }{ Name: "iris" })
}

```

Rich Hi with **Django-syntax**:

使用 **Django-syntax** 丰富 Hi 示例

```html
<!-- ./mytemplates/hi.html -->
<html><head> <title> Hi Iris </title> </head>
  <body>
    <h1> Hi {{ Name }}
  </body>
</html>


```

```go
// ./main.go
import (
    "github.com/kataras/iris"
    "github.com/kataras/go-template/django"
)

func main() {
    iris.UseTemplate(django.New()).Directory("./mytemplates", ".html")
    iris.Get("/hi", hi)
    iris.Listen(":8080")
}

func hi(ctx *iris.Context){
   ctx.Render("hi.html", map[string]interface{}{"Name": "iris"}, iris.RenderOptions{"gzip":true})
}

```

More about render and template engines [here](render.md).

* 关于渲染和模版引擎的更多内容在 [这里](render.md)

