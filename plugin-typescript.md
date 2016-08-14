# Typescript

[This is a plugin](https://github.com/iris-contrib/plugin/tree/master/typescript).

[这是个插件](https://github.com/iris-contrib/plugin/tree/master/typescript).

This is an Iris and typescript bridge plugin.

是 Iris 与 typescript 的桥接插件。

### What?

1. Search for typescript files (.ts)
2.    Search for typescript projects (.tsconfig)
3.    If 1 || 2 continue else stop
4.    Check if typescript is installed, if not then auto-install it (always inside npm global modules, -g)
5.    If typescript project then build the project using tsc -p $dir
6.    If typescript files and no project then build each typescript using tsc $filename
7.    Watch typescript files if any changes happens, then re-build (5|6)

 >Note: Ignore all typescript files & projects whose path has '/node_modules/'
 
### 什么?

1. 搜索 typescript 文件 (.ts)
2.    搜索 typescript 项目文件 (.tsconfig)
3.    如果1或2文件存在的话继续，否则停止执行
4.    检查 typescript 是否已经安装，如果没有那么将自动安装(总是npm内部全局模块，-g)
5.    如果是 typescript 项目那么使用 tsc -p $dir 构建这个项目
6.    如果是 typescript 文件 且没有项目文件，那么使用 tsc $filename 构建每一个typescript 文件
7.    观察 typescript 文件是否发生任何改变，那么通过(步骤5或6）重新构建

 >Note: Ignore all typescript files & projects whose path has '/node_modules/'
 
 >注意: 忽略路径下有 '/node_modules/' 目录的 typescript 文件和项目
 




### 选项 / Options

 - **Bin**: string, the typescript installation path/bin/tsc or tsc.cmd, if empty then it will search to the global npm modules
 - **Bin**: string, typescript 安装的路径 path/bin/tsc 或 tsc.cmd ， 如果为空那么会搜索nmp全局模块
 - **Dir**: string, Dir set the root, where to search for typescript files/project. Default "./" 
 - **Dir**: string, Dir 设置搜索typescript文件或项目的根路径，默认为 "./"
 - **Ignore**: string, comma separated ignore typescript files/project from these directories. Default "" (node_modules are always ignored) 
 - **Ignore**: string, 使用逗号将这些目录中忽略的 文件或项目 分隔开。默认为 "" (node_modules 会一直被忽略)
 - **Tsconfig**: config.Tsconfig{}, here you can set all compilerOptions if no tsconfig.json exists inside the 'Dir' 
 - **Tsconfig**: config.Tsconfig{}, 如果 'Dir' 目录内没有 tsconfig.json，你可以在此处设置所有的编译器选项
 - **Editor**: config.Typescript { Editor: config.Editor{}, if setted then alm-tools browser-based typescript IDE will be available. Defailt is nil
 - **Editor**: config.Typescript{Editor:config.Editor{}}, 如果设置了那么基于浏览器的 alm-tools typescript IDE 将被弃用，默认为 nil
 
 >All these are optional
 
 >所有这些都是可选的


### 如何使用 / How to use

```go
package main

import (
    "github.com/kataras/iris"
    "github.com/iris-contrib/plugin/typescript"
)

func main(){
    ts := typescript.Config {
        Dir: "./scripts/src",
        Tsconfig: typescript.Tsconfig{Module: "commonjs", Target: "es5"}, 
    }
    // or typescript.DefaultConfig()
    // 或 typescript.DefaultConfig()

    iris.Plugins.Add(typescript.New(ts)) //or with the default options just: typescript.New()
    // 或只使用默认选项: typescript.New()

    iris.Get("/", func (ctx *iris.Context){})

    iris.Listen(":8080")
}
```

Enable [web browser editor](plugin-editor.md)

启用[浏览器编辑器](plugin-editor.md)

```go
ts := typescript.Typescript {
    //...
    Editor: typescript.Editor{Username:"admin", Password: "admin!123"}
    //...
}

```