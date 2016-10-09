# Gzip

Gzip compression is easy.

Gzip 压缩很简单。


Activate **auto-gzip** for all responses and template engines, 
just set `iris.Config.Gzip = true` or  or  . You can also enable gzipping for specific `Render()` calls:

激活所有应答和模版引擎都为 **自动gzip压缩**， 
只需要设置 `iris.Config.Gzip = true` 或 `iris.New(iris.OptionGzip(true))` 或 `iris.Set(OptionGzip(true))` ， 你也可以给指定渲染器`Render()`调用启用压缩:

```go
//...
context.Render("mytemplate.html", bindingStruct{}, iris.RenderOptions{"gzip": false})
context.Render("my-custom-response", iris.Map{"anything":"everything"} , iris.RenderOptions{"gzip": false}) 
```

```go
// WriteGzip writes the gzipped body of the response to w.
// WriteGzip 将gzip压缩的body应答内容写入到 w.
//
// Gzips the response body and sets the 'Content-Encoding: gzip'
// header before writing response to w.
// Gzip压缩应答体内容并在写入应答内容到w之前设置 'Content-Encoding: gzip' 头信息。
//
// WriteGzip doesn't flush the response to w for performance reasons.
// WriteGzip 因为性能原因不会刷新应答内容到 w
WriteGzip(w *bufio.Writer) error 


// WriteGzip writes the gzipped body of the response to w.
// WriteGzipLevel 将压缩的响应体内容写入 w.
//
// Level is the desired compression level:
// Level 是渴望被压缩的级别:
//
//     * CompressNoCompression
//     * CompressBestSpeed
//     * CompressBestCompression
//     * CompressDefaultCompression
//
// Gzips the response body and sets the 'Content-Encoding: gzip'
// header before writing response to w.
// 在写入应答w之前，使用gzip压缩应答体并设置 'Content-Encoding: gzip' 头信息 
//
// WriteGzipLevel doesn't flush the response to w for performance reasons.
// WriteGzipLevel 因为性能原因不回刷新应答内容到w。
WriteGzipLevel(w *bufio.Writer, level int) error
```

How to use:

如何使用: 

```go
iris.Get("/something", func(ctx *iris.Context){
    ctx.Response.WriteGzip(...) 
})

```

## 其它 / Other

See [Static files](static-files.md) and learn how you can serve big files, assets or webpages with gzip compression.

查看 [静态文件](static-files.md) 相关内容了解如何为大文件、辅助资源 或 网页 使用gzip压缩提供服务。