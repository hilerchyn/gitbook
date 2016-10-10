# 请求体绑定器 ／ Body binder

Body binder reads values from the body and sets them to a specific object.

请求体绑定器从请求体中读取内容并把相应的值设置到指定的对象。

```go
// ReadJSON reads JSON from request's body
// ReadJSON 从请求体中读取 JSON
ReadJSON(jsonObject interface{}) error

// ReadXML reads XML from request's body
// ReadXML 从请求体中读取 XML
ReadXML(xmlObject interface{}) error

// ReadForm binds the formObject to the requeste's form data
// ReadForm 将 formObject 绑定到请求的表单数据上
ReadForm(formObject interface{}) error
```

How to use:

如何使用:

### JSON

```json
{"public":true,"website":{"Scheme":"http","Opaque":"Opaque-data","User":{"username":"name","password":"password","passwordSet":true},"Host":"bing.com","Path":"/search"},"foundation":"2006-01-02T15:04:05Z","Name":"Hiler","Location":{"Country":"China","City":"Qingdao"},"Products":[{"Name":"product name 1", "Type":"type 1"},{"Name":"product name 2", "Type":"type2"}],"Founders":["Hiler","Chen"],"Employees":55}
```
**注意:JSON中的 Website 即 url.URL 类型字段中的 内嵌 User 字段 为 *url.Userinfo 类型，该字段不能被序列化或反序列化**


```go
package main

import (
	"github.com/kataras/iris"
	"time"
	"net/url"
)

type Company struct {
   Public     bool      `json:"public"`
   Website    url.URL   `json:"website"`
   Foundation time.Time `json:"foundation"`
   Name       string
   Location   struct {
     Country  string
     City     string
   }
   Products   []struct {
     Name string
     Type string
   }
   Founders   []string
   Employees  int64
}

func MyHandler(c *iris.Context) {
  if err := c.ReadJSON(&Company{}); err != nil {
  	panic(err.Error())
  }
}
  
func main() {
  iris.Post("/bind_json", MyHandler)
  iris.Listen(":8080")
}

```

### XML

```xml
<xml>
    <Public>true</Public>
    <Website>
        <Scheme>Scheme</Scheme>
	    <Opaque>Opaque</Opaque>
	    <User></User>
	    <Host>Host</Host>
	    <Path>Path</Path>
	    <RawPath>RawPath</RawPath>
	    <RawQuery>RawQuery</RawQuery>
	    <Fragment>Fragment</Fragment>
    </Website>
    <Foundation>2006-01-02T15:04:05Z</Foundation>
    <Name>Name</Name>
    <Location>
        <Country>Country</Country>
        <City>City</City>
    </Location>
    <Products>
        <Name>Name1</Name>
        <Type>Type1</Type>
    </Products>
    <Products>
        <Name>Name2</Name>
        <Type>Type2</Type>
    </Products>
    <Founders>Founders1</Founders>
    <Founders>Founders2</Founders>
    <Employees>66</Employees>
</xml>
```

```go
package main

import "github.com/kataras/iris"

type Company struct {
   Public     bool      
   Website    url.URL   
   Foundation time.Time
   Name       string
   Location   struct {
     Country  string
     City     string
   }
   Products   []struct {
     Name string
     Type string
   }
   Founders   []string
   Employees  int64
}
  
func MyHandler(c *iris.Context) {  
  if err := c.ReadXML(&Company{}); err != nil {
  	panic(err.Error())
  }
}
  
func main() {
  iris.Post("/bind_xml", MyHandler)
  iris.Listen(":8080")
}

```

### Form

#### 类型 / Types

The supported field types in the destination struct are:

目标结构中支持的字段类型有:

* `string`
* `bool`
* `int`, `int8`, `int16`, `int32`, `int64`
* `uint`, `uint8`, `uint16`, `uint32`, `uint64`
* `float32`, `float64`
* `slice`, `array`
* `struct` and `struct anonymous`
* `map`
* `interface{}`
* `time.Time`
* `url.URL`
*  slices []string
* `custom types` to one of the above types / 上面的任一类型
* a `pointer` to one of the above types / 上面任一类型的指针



----

#### 示例表单 / Example form

```go
 //./main.go

package main

import (
	"fmt"

	"github.com/kataras/iris"
)

type Visitor struct {
	Username string
	Mail     string
	Data     []string `form:"mydata"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("form.html", nil)
	})

	iris.Post("/form_action", func(ctx *iris.Context) {
		visitor := Visitor{}
		err := ctx.ReadForm(&visitor)
		if err != nil {
			fmt.Println("Error when reading form: " + err.Error())
		}
		fmt.Printf("\n Visitor: %v", visitor)
	})

	iris.Listen(":8080")
}

```

```html

<!-- ./templates/form.html -->
<!DOCTYPE html>
<head>
<meta charset="utf-8">
</head>
<body>
<form action="/form_action" method="post">
<input type="text" name="Username" />
<br/>
<input type="text" name="Mail" /><br/>
<select multiple="multiple" name="mydata">
<option value='one'>One</option>
<option value='two'>Two</option>
<option value='three'>Three</option>
<option value='four'>Four</option>
</select>
<hr/>
<input type="submit" value="Send data" />

</form>
</body>
</html>

```




##### HTML 表单 / In form html

- Use symbol `.` to access a field/key of a structure or map. (i.e., `struct.key`)
- 使用 `.` 符号访问结构体或映射的 field/key。 （ 如, `struct.key`)
- Use `[int_here]` to access an index of a slice/array. (i.e, `struct.array[0]`)
- 使用 `[整数]` 访问切片或数组索引位置的内容。（如，`struct.array[0]`)

```html
<form action="/bind_form" method="POST">
  <input type="text" name="Name" value="Sony"/>
  <input type="text" name="Location.Country" value="Japan"/>
  <input type="text" name="Location.City" value="Tokyo"/>
  <input type="text" name="Products[0].Name" value="Playstation 4"/>
  <input type="text" name="Products[0].Type" value="Video games"/>
  <input type="text" name="Products[1].Name" value="TV Bravia 32"/>
  <input type="text" name="Products[1].Type" value="TVs"/>
  <input type="text" name="Founders[0]" value="Masaru Ibuka"/>
  <input type="text" name="Founders[0]" value="Akio Morita"/>
  <input type="text" name="Employees" value="90000"/>
  <input type="text" name="public" value="true"/>
  <input type="url" name="website" value="http://www.sony.net"/>
  <input type="date" name="foundation" value="1946-05-07"/>
  <input type="text" name="Interface.ID" value="12"/>
  <input type="text" name="Interface.Name" value="Go Programming Language"/>
  <input type="submit"/>
</form>
```

##### 后端 / Backend

You can use the tag `form` if the name of an input or form starts lowercase.

如果表单input框的名字的第一个字母是小写，你可以使用 `form` 标签来绑定。

```go
package main

type InterfaceStruct struct {
    ID   int
    Name string
}

type Company struct {
  Public     bool      `form:"public"`
  Website    url.URL   `form:"website"`
  Foundation time.Time `form:"foundation"`
  Name       string
  Location   struct {
    Country  string
    City     string
  }
  Products   []struct {
    Name string
    Type string
  }
  Founders   []string
  Employees  int64
  
  Interface interface{}
}

func MyHandler(c *iris.Context) {
  m := Company{
      Interface: &InterfaceStruct{},
  }
  
  if err := c.ReadForm(&m); err != nil {
  		panic(err.Error())
  }
}
  
func main() {

  iris.Get("/", func(c *iris.Context) {
    c.Render("form.html", nil)
  })
  
  iris.Post("/bind_form", MyHandler)
  
  iris.Listen(":8080")
}
```

### Custom Decoder per Object

### 给每个对象自定义解析器

`BodyDecoder` gives the ability to set a custom decoder **per passed object** when `context.ReadJSON` and `context.ReadXML` 

`BodyDecoder` 给了我们在使用 `context.ReadJSON` 和 `context.ReadXML` 时给 **每个传递的对象** 设置自定义解析器的能力。

```go
// BodyDecoder is an interface which any struct can implement in order to customize the decode action
// from ReadJSON and ReadXML
// BodyDecoder 是一个实现了来自 ReadJSON 和 ReadXML 的解析动作接口的任意结构体
//
// Trivial example of this could be:
// 简单的例子可以这样：
// type User struct { Username string }
//
// func (u *User) Decode(data []byte) error {
//	  return json.Unmarshal(data, u)
// }
//
// the 'context.ReadJSON/ReadXML(&User{})' will call the User's
// Decode option to decode the request body
// 'context.ReadJSON/ReadXML(&User{})' 将调用用户的解析选项解析请求体
//
// Note: This is totally optionally, the default decoders
// 注意：这完全是可选项，默认解析器有
// for ReadJSON is the encoding/json and for ReadXML is the encoding/xml
// ReadJSON 的 encoding/json 和 ReadXML 的 encoding/xml
type BodyDecoder interface {
	Decode(data []byte) error
}

```

> for a usage example go to https://github.com/kataras/iris/blob/master/context_test.go#L262
>
> 在 https://github.com/kataras/iris/blob/master/context_test.go#L262 文件中可以查看更多使用示例
