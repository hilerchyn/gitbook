# Websockets

**WebSocket is a protocol providing full-duplex communication channels over a single TCP connection**. The WebSocket protocol was standardized by the IETF as RFC 6455 in 2011, and the WebSocket API in Web IDL is being standardized by the W3C.

**WebSocket 是一个通过一个TCP连接提供全双工交互通道的协议**。 WebSocket 协议 被 IETF 在 2011年时指定成 RFC 6455 标准高， 并且 Web IDL 的 WebSocket API 被 W3C 标准化。

WebSocket is designed to be implemented in web browsers and web servers, but it can be used by any client or server application. The WebSocket Protocol is an independent TCP-based protocol. Its only relationship to HTTP is that its handshake is interpreted by HTTP servers as an Upgrade request. The WebSocket protocol makes more interaction between a browser and a website possible, **facilitating the real-time data transfer from and to the server**. 

WebSocket 分别在 web 浏览器和 web 服务器 中实现，但可以被人和客户端或服务器端应用程序使用。WebSocket协议时一个基于TCP协议的独立实现。跟HTTP唯一相关的事它的握手被HTTP服务器集成为一个升级(Upgrade)后的请求。WebSocket 协议提供更多浏览器和网站间交互的可能性，**便于客户端和服务器端的实时数据传输**。

[Read more about Websockets via wikipedia](https://en.wikipedia.org/wiki/WebSocket)

[通过维基百科了解更多 Websockets 的内容](https://en.wikipedia.org/wiki/WebSocket)

-----

## 配置 / Configuration

```go
type Websocket struct {
	// WriteTimeout time allowed to write a message to the connection.
	// WriteTimeout 允许将消息写入连接的限制时间
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
	// Endpoint is the path which the websocket server will listen for clients/connections
	// Endpoint websocket 服务器的给 客户端或连接 提供访问的监听路径
	// Default value is empty string, if you don't set it the Websocket server is disabled.
	// 默认值为空字符串，如果你不设置的话 Websocket 服务器将被禁用。
	Endpoint string
	// Headers  the response headers before upgrader
	// Headers 在upgrader 之前的应答头信息
	// Default is empty
	// 默认为 空
	Headers map[string]string
	// ReadBufferSize is the buffer size for the underline reader
	// ReadBufferSize 是后台读取器的缓存大小
	ReadBufferSize int
	// WriteBufferSize is the buffer size for the underline writer
	// WriteBufferSize 是后台写入器的缓存大小
	WriteBufferSize int
}

```

```go
iris.Config.Websocket.Endpoint = "/myEndpoint"
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

	// the path which the websocket client should listen/registed to ->
	// websocket 客户端应该 监听或注册到的路径 ->
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

			//send the message to the whole room,
			// 向整个房间发送消息
			//all connections are inside this room will receive this message
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


View a working example by navigating [here](https://github.com/iris-contrib/examples/tree/master/websocket) and if you need more than one websocket server [click here](https://github.com/iris-contrib/examples/tree/master/websocket_unlimited_servers)

到[这里](https://github.com/iris-contrib/examples/tree/master/websocket) 查看一个可用的示例，如果你需要不只一个websocket 服务器的话点击[这里](https://github.com/iris-contrib/examples/tree/master/websocket_unlimited_servers)