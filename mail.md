# 发送电子邮件 / Send e-mails

This is a [package](https://github.com/kataras/go-mailer).

这是一个工具[包](https://github.com/iris-contrib/mail).

Sending plain or rich content e-mails is an easy process with Iris.

用 Iris 发送纯文本或富文本内容的邮件处理起来很简单。

**Configuration**

**配置**

```go
// Config keeps the options for the mail sender service
// Config 保存了所有邮件发送器服务的配置选项
type Config struct {
    // Host is the server mail host, IP or address
    // Host 是邮件服务器的 host, IP 或 地址
    Host string
    // Port is the listening port
    // Port 是监听端口
    Port int
    // Username is the auth username@domain.com for the sender
    // Username 是发送者的验证邮箱 username@domain.com
    Username string
    // Password is the auth password for the sender
    // Password 是发送者的验证密码
    Password string
    // FromAlias is the from part, if empty this is the first part before @ from the Username field
    // FromAlias 是发件人From部分，如果为空那么会设置为 Username 字段中 @ 符之前的部分
    FromAlias string
    // UseCommand enable it if you want to send e-mail with the mail command  instead of smtp
    // UseCommand 如果你想使用 mail 命令替代smtp服务的话，可以启用它
    //
    // Host,Port & Password will be ignored
    // Host, Port & Password 将被忽略
    // ONLY FOR UNIX
    // 只在UNIX下
    UseCommand bool
}

```

```go
Send(subject string, body string, to ...string) error
```

**Installation**

```sh go get -u github.com/kataras/go-mailer ```

**Example**
**示例**

File: `./main.go`

```go
package main

import (
    "github.com/kataras/go-mailer"
    "github.com/kataras/iris"
)

func main() {
    // change these to your needs
    // 把这些改成你需要的

    cfg := mail.Config{
        Host:     "smtp.mailgun.org",
        Username: "postmaster@sandbox661c307650f04e909150b37c0f3b2f09.mailgun.org",
        Password: "38304272b8ee5c176d5961dc155b2417",
        Port:     587,
    }
    // change these to your e-mail to check if that works
    // 把这些改成你自己的电子邮件的配置以检查它是否工作

    // create the service
    // 创建服务
    mailService := mail.New(cfg)

    var to = []string{"kataras2006@hotmail.com", "social@ideopod.com"}

    // standalone
    // 独立发送

    //iris.Must(mailService.Send("iris e-mail test subject", "</h1>outside of context before server's listen!</h1>", to...))

    //inside handler
    iris.Get("/send", func(ctx *iris.Context) {
        content := `<h1>Hello From Iris web framework</h1> <br/><br/> <span style="color:blue"> This is the rich message body </span>`

        err := mailService.Send("iris e-mail just t3st subject", content, to...)

        if err != nil {
            ctx.HTML(200, "<b> Problem while sending the e-mail: "+err.Error())
        } else {
            ctx.HTML(200, "<h1> SUCCESS </h1>")
        }
    })

    // send a body by renderingt a template
    // 渲染模板时发送内容
    iris.Get("/send/template", func(ctx *iris.Context) {
        content := iris.TemplateString("body.html", iris.Map{
            "Message": " his is the rich message body sent by a template!!",
            "Footer":  "The footer of this e-mail!",
        }, iris.RenderOptions{"charset" :"UTF-8"}) 
            // iris.RenderOptions is an optional parameter,
            // iris.RenderOptions 都是可选参数,
            // "charset" defaults to UTF-8 but you can change it for a 
            // particular mail receiver
            // "charset" 默认就是 UTF-8 但你可以为一个特别的邮件接收者设定

        err := mailService.Send("iris e-mail just t3st subject", content, to...)

        if err != nil {
            ctx.HTML(200, "<b> Problem while sending the e-mail: "+err.Error())
        } else {
            ctx.HTML(200, "<h1> SUCCESS </h1>")
        }
    })
    iris.Listen(":8080")
}


```

File: `./templates/body.html`

```html
<h1>Hello From Iris web framework</h1>
<br/><br/>
<span style="color:red"> {{.Message}}</span>
<hr/>

<b> {{.Footer}} </b>
```

