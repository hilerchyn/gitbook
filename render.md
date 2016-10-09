# 渲染器 / Render

Think of 'Render' as an action which sends/responds with rich content to the client.

将 `渲染` 认作 发送／响应 富文本内容到客户端的一个动作。
The render actions are separated into two categories:

渲染动作被分成了两类:

* **Responses** send content using `Serialize Engines` which use the `Content-Type` as a `Key (explained later)`. (i.e. JSON, XML etc.)

* **Responses** 通过使用'Content-Type'作为 'Key (后面再解释)' (如 JSON/XML 等)的 'Serialize Engines' 发送内容。

* **Templates** send content using `Template Engines` which use file name extensions. (i.e. Markdown, Jade etc.)

* **Templates** 通过使用文件扩展名的 'Template Engines' (如 Markdown, Jade  等) 发送内容。

### [Serialize Engines](/serialize-engines.md)

Easy and fast way to render any type of data. **JSON, JSONP, XML, Text, Data, Markdown** or any custom type. 

简单快速渲染任何类型数据的方式。**JSON, JSONP, XML, Text, Data, Markdown** 或者任何自定义类型。

- examples are located [here](https://github.com/iris-contrib/examples/tree/master/serialize_engines/).
- 示例都放在[这里](https://github.com/iris-contrib/examples/tree/master/serialize_engines/).

### [模版引擎](/template-engines.md) / [Template Engines](/template-engines.md)

Iris gives you the freedom to render templates through 6+ built-in template engines, 
you can create your own and 'inject' it to the iris station. 
You can also use more than one template engines at the same time (when the file extensions are different from each other). 

Iris 给你通过6+种内建模版引擎来渲染模版的自由，
你可以创建自己的引擎并注入到iris网站，
你也可以同时使用不只一个模版引擎\(当文件后缀与其它的不同时\)。


- examples are located [here](https://github.com/iris-contrib/examples/tree/master/template_engines/).
- 示例放在[这里](https://github.com/iris-contrib/examples/tree/master/template_engines/) 

