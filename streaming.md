# Streaming


Do  progressive rendering via multiple flushes, streaming.

通过多次刷新、流 来逐步呈现。


```go
// StreamWriter registers the given stream writer for populating the response body.
// StreamWriter 为填充应答体注册流写入器。
//
// This function may be used in the following cases:
// 这个函数也许会在下面几种情况下被用到：
//
//     * if response body is too big (more than 10MB).
//     * 如果响应体太大 (大于19MB)
//     * if response body is streamed from slow external sources.
//     * 如果响应体来自外部缓慢的流
//     * if response body must be streamed to the client in chunks.
//     * 如果响应体必须分成小块的发送到客户端
//     (aka `http server push`).
//     (亦称作 ｀HTTP 服务器推送｀)
StreamWriter(cb func(writer *bufio.Writer))
``` 


## 用例 / Usage example

```go
package main

import(
	"github.com/kataras/iris"
	"bufio"
	"time"
	"fmt"
)

func main() {
	iris.Any("/stream",func (ctx *iris.Context){
		ctx.StreamWriter(stream)
	})

	iris.Listen(":8080")
}

func stream(w *bufio.Writer) {
	for i := 0; i < 10; i++ {
		fmt.Fprintf(w, "this is a message number %d", i)

		// Do not forget flushing streamed data to the client.
		// 不要忘记将流数据刷新到客户端。
		if err := w.Flush(); err != nil {
			return
		}
		time.Sleep(time.Second)
	}
}

```

To achieve the oposite make use of the ` StreamReader`:

完成相反的作用可使用 `StreamREader`:

```go
// StreamReader sets the response body stream and optionally body size.
// StreamReader 设置应答体流和可选应答体尺寸。
//
// If bodySize is >= 0, then the bodyStream must provide the exact bodySize bytes 
// before returning io.EOF.
// 如果读区内容尺寸 >= 0, 那么在返回 io.EOF 之前 bodyStream 必须给定确切的 字节 大小
//
// If bodySize < 0, then bodyStream is read until io.EOF.
// 如果读区内容大小 < 0, 那么 bodyStream 会一直读取至 io.EOF 。
//
// bodyStream.Close() is called after finishing reading all body data
// if it implements io.Closer.
// 如果读取器实现了 io.Closer 那么在读取完所有响应体数据后调用 bodyStream.Close()
//
// See also StreamReader.
// 那么看一下 StreamReader
StreamReader(bodyStream io.Reader, bodySize int)
```