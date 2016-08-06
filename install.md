# Install

**Compatible with go1.6+ **

**兼容 go1.6+**

```sh
$ go get -u github.com/kataras/iris/iris
```

this will update the dependencies also.

这也将同时更新依赖。

- If you are connected to the Internet through **China**, according to [this](https://github.com/kataras/iris/issues/98) you may having problem install Iris. **Follow the below steps**:

- 如果你是在 **中国** 访问因特网，根据[这个问题](https://github.com/kataras/iris/issues/98)描述的你在安装 Iris 时可能会有遇到问题。 **按照下面的步骤操作**:


1.  https://github.com/northbright/Notes/blob/master/Golang/china/get-golang-packages-on-golang-org-in-china.md 


2. `$ go get github.com/kataras/iris/iris ` **without -u**

 `$ go get github.com/kataras/iris/iris ` **不使用 -u**


- If you have any problems installing Iris, just delete the directory `$GOPATH/src/github.com/kataras/iris` , open your shell and run `go get -u github.com/kataras/iris/iris` .

- 如果你在安装 Iris 过程中还有任何其它问题，只需要删除目录 `$GOPATH/src/github.com/kataras/iris` , 打开 shell 并执行 `go get -u github.com/kataras/iris/iris` .
     