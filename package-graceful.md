# 优雅停止 / Graceful

[This is a package](https://github.com/iris-contrib/graceful).

[这是一个包](https://github.com/iris-contrib/graceful).


Enables graceful shutdown.

可以优雅停止。

```go

package main

import (
	"time"
        "github.com/kataras/iris"
	"github.com/iris-contrib/graceful"
)

func main() {
	api := iris.New()
	api.Get("/", func(c *iris.Context) {
		c.Write("Welcome to the home page!")
	})

	graceful.Run(":3001", time.Duration(10)*time.Second, api)
}


```