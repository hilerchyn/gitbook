# 渲染器 / Render

Think the 'Render'  as an action which sends\/responses with a rich content to the client.

将 `渲染` 认作 发送／响应 富文本内容到客户端的一个动作。

The render actions, are separated in two iris-theoretical 'categories'

渲染动作， 被分成了 iris 框架理论上的两个 `分类`

* Response content using Response Engines, by 'Content-Type\/ or Key', you will understand what key is, later.
* 通过 'Content-Type/ 或 Key' 使用 应答引擎发送响应内容，后面你会理解key是什么。

* Templates using Template Engines, by 'filename'.
* 通过 'filename' 使用模版引擎发送模版内容。


### [应答引擎](/response-engines.md) / [Response Engines](/response-engines.md)

Easy and fast way to render any type of data. **JSON, JSONP, XML, Text, Data, Markdown** .or any custom type. 

简单快速渲染任何类型数据的方式。**JSON, JSONP, XML, Text, Data, Markdown** 或者任何自定义类型。

- examples are located [here](https://github.com/iris-contrib/examples/tree/master/response_engines/)
- 示例都放在[这里](https://github.com/iris-contrib/examples/tree/master/response_engines/)

### [模版引擎](/template-engines.md) / [Template Engines](/template-engines.md)

Iris gives you the freedom to render templates through 6+ built'n template engines, you can create your own and 'inject' that to the iris station, you can also use more than one template engines at the same time \(when the file extension is different from the other\).

Iris 给你通过6+种内建模版引擎来渲染模版的自由， 你可以创建自己的引擎并注入到iris网站，你也可以同时使用不只一个模版引擎\(当文件后缀与其它的不同时\)。

- examples are located [here](https://github.com/iris-contrib/examples/tree/master/template_engines/) 
- 示例放在[这里](https://github.com/iris-contrib/examples/tree/master/template_engines/) 

