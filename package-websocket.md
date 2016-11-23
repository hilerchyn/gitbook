# Websockets

**WebSocket is a protocol providing full-duplex communication channels over a single TCP connection**.   
The WebSocket protocol was standardized by the IETF as RFC 6455 in 2011, and the WebSocket API in Web IDL is being standardized by the W3C.

**Websocket是一种通过一个单独的TCP连接提供全双工交互通道的协议**
WebSocket协议在2011年被IETF标准化为RFC 6455，并且被W3C标准化了Web IDL的WebSocket API。

WebSocket is designed to be implemented in web browsers and web servers, but it can be used by any client or server application. 
The WebSocket protocol is an independent TCP-based protocol. Its only relationship to HTTP is that its handshake is interpreted by HTTP servers as an Upgrade request. 
The WebSocket protocol makes more interaction between a browser and a website possible, **facilitating real-time data transfer from and to the server**.

WebSocket 的设计需要在浏览器和网站服务器实现，它可以在任何客户端和服务器端应用程序中使用。
WebSocket 协议是基于TCP协议的独立协议。它跟HTTP的唯一关系就是它的握手协议由HTTP服务器作为一个升级(Upgrade)请求集成。
Websocket 协议使得浏览器和站点之间可能很多交互， **来自或发往服务器的灵活的实时数据传输**。

[Read more about Websockets on Wikipedia](https://en.wikipedia.org/wiki/WebSocket).

[通过维基百科了解更多 Websockets 的内容](https://en.wikipedia.org/wiki/WebSocket)。

-----

## 配置 / Configuration

```go
type WebsocketConfiguration struct {
	// WriteTimeout time allowed to write a message to the connection
	// WriteTimeout 允许将消息写入连接的时间
	// Default value is 15 * time.Second
	// 默认值为 15 * time.Second
	WriteTimeout time.Duration
	// PongTimeout allowed to read the next pong message from the connection
	// PongTimeout 允许从连接中读取下一个 pong 消息的超时时间
	// Default value is 60 * time.Second
	// 默认值 60 * time.Second
	PongTimeout time.Duration
	// PingPeriod send ping messages to the connection with this period. Must be less than PongTimeout
	// PingPeriod 发送 ping 消息到连接的时间周期，必须小于 PongTimeout
	// Default value is (PongTimeout * 9) / 10
	// 默认值为 (PongTime*9)/10
	PingPeriod time.Duration
	// MaxMessageSize max message size allowed from connection
	// MaxMessageSize 从连接中读取消息的最大尺寸
	// Default value is 1024
	// 默认值为 1024
	MaxMessageSize int64
	// BinaryMessages set it to true in order to denotes binary data messages instead of utf-8 text
	// BinaryMessages 如果设置为 true，那么意味着用二进制数据消息代替utf-8文本消息
	// see https://github.com/kataras/iris/issues/387#issuecomment-243006022 for more
	// 从 https://github.com/kataras/iris/issues/387#issuecomment-243006022 了解更多
	// defaults to false
	// 默认为 false
	BinaryMessages bool
	// Endpoint is the path which the websocket server will listen for clients/connections
	// Endpoint websocket服务器监听客户端/连接的路径
	// Default value is empty string, if you don't set it the Websocket server is disabled.
	// 默认值为空字符串，如果你不设置的话，Websocket服务器将被禁用。
	Endpoint string
	// ReadBufferSize is the buffer size for the underline reader
	// ReadBufferSize 是后台读取器的缓存大小
	ReadBufferSize int
	// WriteBufferSize is the buffer size for the underline writer
	// WriteBufferSize 是后台写入器的缓存大小
	WriteBufferSize int
	// Headers  if true then the client's headers are copy to the websocket connection
	// Headers 如果为 true 那么客户端的头信息将被拷贝到websocket连接
	//
	// Default is true
	// 默认为 true
	Headers bool
	// Error specifies the function for generating HTTP error responses.
	// Error 指定生成HTTP错误应答的函数
	//
	// The default behavior is to store the reason in the context (ctx.Set(reason)) and fire any custom error (ctx.EmitError(status))
	// 默认行为是用来在上下文(cts.Set(reason))中存储原因和发送任何自定义错误(ctx.EmitError(status))
	Error func(ctx *Context, status int, reason string)
	// CheckOrigin returns true if the request Origin header is acceptable. If
	// CheckOrigin 如果Origin头可接受那么返回true.
	// CheckOrigin is nil, the host in the Origin header must not be set or
	// must match the host of the request.
	// 如果 CheckOrigin 为 nil， 那么Origin头中的主机host要么不设置要么必需与请求的host信息匹配。
	//
	// The default behavior is to allow all origins
	// 默认行为允许所有的源
	// you can change this behavior by setting the iris.Config.Websocket.CheckOrigin = iris.WebsocketCheckSameOrigin
	// 你可以通过设置 iris.Config.Websocket.CheckOrigin = iris.WebsocketCheckSameOrigin 来改变这个行为
	CheckOrigin func(ctx *Context) bool
}

```

```go
iris.Config.Websocket.Endpoint = "/myEndpoint"
// or
iris.Set(iris.OptionWebsocketEndpoint("/myEndpoint"))
// or 
iris.New(iris.Configuration{Websocket: iris.WebsocketConfiguration{Endpoint: "/myEndpoint"}})
```

## 概述 / Outline
```go
 iris.Websocket.OnConnection(func(c iris.WebsocketConnection){})
```

WebsocketConnection's methods

WebsocketConnection 的方法

```go

// Receive from the client
// 从客户端接收
On("anyCustomEvent", func(message string) {})
On("anyCustomEvent", func(message int){})
On("anyCustomEvent", func(message bool){})
On("anyCustomEvent", func(message anyCustomType){})
On("anyCustomEvent", func(){})

// Receive a native websocket message from the client
// 从客户端接收一个原生 websocket 消息
// compatible without need of import the iris-ws.js to the .html
// 不将 iris-ws.js 导入 .html 也可以兼容
OnMessage(func(message []byte){})

// Send to the client
// 向客户端发送消息
Emit("anyCustomEvent", string)
Emit("anyCustomEvent", int)
Emit("anyCustomEvent", bool)
Emit("anyCustomEvent", anyCustomType)

// Send via native websocket way, compatible without need of import the iris-ws.js to the .html
// 通过原生 websocket 方式发送数据，不导入 iris-ws.js 到 .html 也可以兼容
EmitMessage([]byte("anyMessage"))

// Send to specific client(s)
// 向指定的客户端发送数据
To("otherConnectionId").Emit/EmitMessage...
To("anyCustomRoom").Emit/EmitMessage...

// Send to all opened connections/clients
// 向所有开启的连接和客户端发送数据
To(iris.All).Emit/EmitMessage...

// Send to all opened connections/clients EXCEPT this client(c)
// 向除了这个客户端以外的所有开启的连接或客户端发送数据
To(iris.NotMe).Emit/EmitMessage...

// Rooms, group of connections/clients
// 连接或客户端的房间或组
Join("anyCustomRoom")
Leave("anyCustomRoom")


// Fired when the connection is closed
// 当连接关闭时触发此函数
OnDisconnect(func(){})

// Force-disconnect the client from the server-side
// 从服务器端强制断开客户端的连接
Disconnect() error
```

## 如何使用 / How to use

**Server-side**

**服务器端**

```go
// ./main.go
package main

import (
	"fmt"

	"github.com/kataras/iris"
)

type clientPage struct {
	Title string
	Host  string
}

func main() {
	iris.Static("/js", "./static/js", 1)

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("client.html", clientPage{"Client Page", ctx.HostString()})
	})

	// the path at which the websocket client should register itself to
	// 客户端应该将自己注册到的路径
	iris.Config.Websocket.Endpoint = "/my_endpoint"
	// for Allow origin you can make use of the middleware
	// 你可以使用中间件来 Allow 源
	//iris.Config.Websocket.Headers["Access-Control-Allow-Origin"] = "*"

	var myChatRoom = "room1"
	iris.Websocket.OnConnection(func(c iris.WebsocketConnection) {

		c.Join(myChatRoom)

		c.On("chat", func(message string) {
			// to all except this connection ->
			// 给除了此连接的其它连接发送消息 ->
			//c.To(iris.Broadcast).Emit("chat", "Message from: "+c.ID()+"-> "+message)

			// to the client ->
			// 发送给客户端 ->
			//c.Emit("chat", "Message from myself: "+message)

			// send the message to the whole room,
			// 向整个房间发送消息
			// all connections which are inside this room will receive this message
			// 所有房间内的连接都会接收到这个消息
			c.To(myChatRoom).Emit("chat", "From: "+c.ID()+": "+message)
		})

		c.OnDisconnect(func() {
			fmt.Printf("\nConnection with ID: %s has been disconnected!", c.ID())
		})
	})

	iris.Listen(":8080")
}

```

**Client-side**

**客户端**

```js
// js/chat.js
var messageTxt;
var messages;

$(function () {

	messageTxt = $("#messageTxt");
	messages = $("#messages");


	ws = new Ws("ws://" + HOST + "/my_endpoint");
	ws.OnConnect(function () {
		console.log("Websocket connection enstablished");
	});

	ws.OnDisconnect(function () {
		appendMessage($("<div><center><h3>Disconnected</h3></center></div>"));
	});

	ws.On("chat", function (message) {
		appendMessage($("<div>" + message + "</div>"));
	})

	$("#sendBtn").click(function () {
		//ws.EmitMessage(messageTxt.val());
		ws.Emit("chat", messageTxt.val().toString());
		messageTxt.val("");
	})

})


function appendMessage(messageDiv) {
    var theDiv = messages[0]
    var doScroll = theDiv.scrollTop == theDiv.scrollHeight - theDiv.clientHeight;
    messageDiv.appendTo(messages)
    if (doScroll) {
        theDiv.scrollTop = theDiv.scrollHeight - theDiv.clientHeight;
    }
}

```


```html

<html>

<head>
	<title>My iris-ws</title>
</head>

<body>
	<div id="messages" style="border-width:1px;border-style:solid;height:400px;width:375px;">

	</div>
	<input type="text" id="messageTxt" />
	<button type="button" id="sendBtn">Send</button>
	<script type="text/javascript">
		var HOST = {{.Host}}
	</script>
	<script src="js/vendor/jquery-2.2.3.min.js" type="text/javascript"></script>
	<!-- /iris-ws.js is served automatically by the server -->
	<script src="/iris-ws.js" type="text/javascript"></script>
	<!-- -->
	<script src="js/chat.js" type="text/javascript"></script>
</body>

</html>


```


View a working example by navigating [here](https://github.com/iris-contrib/examples/tree/master/websocket) and if you need more than one websocket server [click here](https://github.com/iris-contrib/examples/tree/master/websocket_unlimited_servers).

到[这里](https://github.com/iris-contrib/examples/tree/master/websocket) 查看一个可用的示例，如果你需要不只一个websocket 服务器的话点击[这里](https://github.com/iris-contrib/examples/tree/master/websocket_unlimited_servers)。
