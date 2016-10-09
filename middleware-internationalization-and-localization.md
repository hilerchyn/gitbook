# 本地化与国际化 / Internationalization and Localization

[This is a middleware](https://github.com/iris-contrib/middleware/tree/master/i18n)

[这是个中间件](https://github.com/iris-contrib/middleware/tree/master/i18n)

## 教程 / Tutorial

Create folder named 'locales':

创建名为 `locales` 的文件夹：
```
// Files: 

./locales/locale_en-US.ini 
./locales/locale_el-US.ini 
```
Contents on locale_en-US:

locale_en-US 的内容:

``` 
hi = hello, %s
``` 
Contents on locale_el-GR:

locale_el-GR 的内容:

``` 
hi = Γειά, %s
``` 

```go

package main

import (
	"fmt"
	"github.com/kataras/iris"
	"github.com/iris-contrib/middleware/i18n"
)

func main() {

	iris.Use(i18n.New(i18n.Config{Default: "en-US",
		Languages: map[string]string{
			"en-US": "./locales/locale_en-US.ini",
			"el-GR": "./locales/locale_el-GR.ini",
			"zh-CN": "./locales/locale_zh-CN.ini"}}))	
	
	iris.Get("/", func(ctx *iris.Context) {
		hi := ctx.GetFmt("translate")("hi", "maki") // hi is the key, 'maki' is the %s, the second parameter is optional / // hi 是key，'maki' 是 %s (格式化字符串), 第二个参数是可选的
		language := ctx.Get("language") // language is the language key, example 'en-US' / 'language' 是语言的 kye, 如 'en-US'

		ctx.Write("From the language %s translated output: %s", language, hi)
	})

	iris.Listen(":8080")

}

```