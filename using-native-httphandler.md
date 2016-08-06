# 使用原生 http.Handler / Using native http.Handler

> Not recommended and I will not help you if any issue comes up, it is just there for your first conversional steps.
> Note also that using native http handler you cannot access url params.

> 不推荐使用，如果出现任何问题我不会帮你，它只是为你第一次转变而存在的。
> 需要注意，使用原生http处理器你不能访问url参数。

```go

type nativehandler struct {}

func (_ nativehandler) ServeHTTP(res http.ResponseWriter, req *http.Request) {

}

func main() {
    iris.Handle("", "/path", iris.ToHandler(nativehandler{}))
    //"" means ANY(GET,POST,PUT,DELETE and so on)
    //"" 意味着(GET,POST,PUT,DELETE 等)中的任一一个方式
}


```

