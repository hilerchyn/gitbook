# Gzip

Gzip compression is easy.

Gzip 压缩很简单。


For **auto-gzip** to all response and template engines, 
just set the `iris.Config.Gzip = true`, which you can also change for specific render options:

所有应答和模版引擎都为 **自动gzip压缩**， 是需要设置 `iris.Config.Gzip = true` ， 你也可以给变指定渲染器的选项:

```go
//...
context.Render("mytemplate.html", bindingStruct{}, iris.RenderOptions{"gzip": false})
context.Render("my-custom-response", iris.Map{"anything":"everything"} , iris.RenderOptions{"gzip": false}) 
```

```go
// WriteGzip writes response with gzipped body to w.
// WriteGzip 将 gzip 压缩的 body 内容 写入到 w.
//
// The method gzips response body and sets 'Content-Encoding: gzip'
// 这个方法压缩响应体内容 并在写入到 w. 之前设置 'Content-Encoding: gzip' 头信息
// header before writing response to w.
//
// WriteGzip doesn't flush response to w for performance reasons.
// 因为性能原因 WriteGzip 不会将应答刷新到 w
WriteGzip(w *bufio.Writer) error 


// WriteGzipLevel writes response with gzipped body to w.
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
// The method gzips response body and sets 'Content-Encoding: gzip'
// 这个方法压缩响应体内容 并在写入到 w. 之前设置 'Content-Encoding: gzip' 头信息
// header before writing response to w.
//
// WriteGzipLevel doesn't flush response to w for performance reasons.
// 因为性能原因 WriteGzip 不会 直接将响应写入 
WriteGzipLevel(w *bufio.Writer, level int) error
```

How to use

如何使用

```go
iris.Get("/something", func(ctx *iris.Context){
    ctx.Response.WriteGzip(...) 
})

```

## 其它 / Other

See [Static files](static-files.md) and learn how you can serve big files, assets or webpages with gzip compression.

查看 [静态文件](static-files.md) 相关内容了解如何为大文件、辅助资源 或 使用gzip 压缩的网页提供服务。