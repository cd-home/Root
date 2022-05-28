[TOC]

# gin

Gin是用 Go 开发的一个微框架，重点是小巧、易用、性能好很多

## 快速入门

### 基本程序

~~~go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)
func main() {
	r := gin.Default() // 默认的引擎
	r.GET("/ping", func(context *gin.Context) {
		context.String(http.StatusOK, "hello world")
	})
	_ = r.Run()
}
~~~

## 请求

### 路由

#### 普通路由

~~~go
r.GET("/get", getting)
r.POST("/post", posting)
r.PUT("/put", putting)
r.DELETE("/delete", deleting)
r.PATCH("/patch", patching)
r.HEAD("/head", head)
r.OPTIONS("/options", options)
r.Any("/some", someHandler)     // 任意的请求
~~~

#### 路由参数

~~~go
r.GET("/user/:name", func(c *gin.Context) {
    name := c.Param("name")
})
~~~

#### 分组路由

~~~go
// 简单的路由组: v1
v1 := r.Group("/v1")
{
    v1.POST("/login", loginEndpoint)
}
// 简单的路由组: v2
v2 := router.Group("/v2")
{
    v2.POST("/login", AdminEndpoint)
}
r.Run(":8080")
~~~

#### 嵌套路由

~~~go
v1 := r.Group("/v1")
{
    v1.POST("/login", loginEndpoint)
    admin := v1.Group("/admin")
    admin.POST("/test", AdminEndpoint)
}
~~~

### 参数

#### 路径参数

~~~go
r.GET("/user/:name", controller.CreateUser)
func CreateUser(c *gin.Context) {
	name := c.Param("name")
	c.JSON(http.StatusOK, gin.H{
		"name": name,
	})
}
~~~

#### 查询参数

~~~go
v2.POST("/param", func(context *gin.Context) {	
    id := context.Query("id")
    name := context.DefaultQuery("name", "baby") // 如果不存在给定默认值
    log.Println(id)
    log.Println(name)
    context.String(http.StatusOK, "param")
})
~~~

#### 表单参数

~~~go
v1.POST("/form", func(context *gin.Context) {
    user := context.PostForm("user") // 常用
    password := context.DefaultPostForm("password", "66666")
    // 其他形式
	context.PostFormMap("key")
	context.PostFormArray("key")
	context.GetPostForm("key")
    context.String(http.StatusOK, "form")
})
~~~

#### body

方式1: 通过json包,解析道map

~~~go
v1.POST("/post", func(context *gin.Context) {
    buf := make([]byte, 1024)
    n, _ := context.Request.Body.Read(buf)
    var m map[string]interface{}
    _ = json.Unmarshal(buf[0:n], &m)
    context.String(http.StatusOK, "OK")
})
~~~

**方式2: 通过绑定方法, 绑定到结构体，但是一般需要知道结构和数据类型**

#### 同名参数

实际上这样的参数是不那么常见的

~~~
GET /ping?ids[a]=1234&ids[b]=hello
POST names[first]=liyao&names[second]=jack
~~~

~~~go
v1.GET("/ping", func(context *gin.Context) {
    ids := context.QueryMap("ids")
    fmt.Printf("ids: %v", ids)
    context.String(http.StatusOK, "hello world")
})
~~~

返回结果, 也就是我们所期望的map结构【这里没有测试POST】`names := context.PostFormMap("names")`

~~~go
ids: map[a:1234 b:hello]
~~~

## 响应

### ASCII

~~~go
v1.GET("/someJSON", func(context *gin.Context) {
    data := map[string]interface{}{
        "lang": "GO语言",
        "tag":  "<br>",
    }
    // 输出 : {"lang":"GO\u8bed\u8a00","tag":"\u003cbr\u003e"}
    context.AsciiJSON(http.StatusOK, data)
})
~~~

### 字符串

~~~go
context.String(http.StatusOK, "hello world")
~~~

### JSON

1.  普通JSON  

~~~go
// type H map[string]interface{}
v1.GET("/json", func(context *gin.Context) {
    context.JSON(200, gin.H{
        "html": "<b>Hello, world!</b>",
    })
})
~~~

2.  安全JSON

~~~go
context.SecureJSON(http.StatusOK, msg)
~~~

3.  结构体

~~~go
v1.GET("/moreJSON", func(context *gin.Context) {
    // 你也可以使用一个结构体
    var msg struct {
        Name    string `json:"user"`
        Message string
        Number  int
    }
    msg.Name = "Lena"
    msg.Message = "hey"
    msg.Number = 123
    // 注意 msg.Name 在 JSON 中变成了 "user"
    // 将输出：{"user": "Lena", "Message": "hey", "Number": 123}
    context.JSON(http.StatusOK, msg)
})
~~~

### 重定向

1.  HTTP 重定向很容易

~~~go
r.GET("/test", func(c *gin.Context) {
	c.Redirect(http.StatusMovedPermanently, "http://www.google.com/")
})
~~~

2.  路由重定向

~~~go
r.GET("/test", func(c *gin.Context) {
    c.Request.URL.Path = "/test2"
    r.HandleContext(c)
})
~~~

### HTML模版

具体可见模版渲染

~~~go
v1.LoadHTMLGlob("templates/*")
v1.GET("/index", func(context *gin.Context) {
    context.HTML(http.StatusOK, "index.tmpl", gin.H{
        "title": "Main website",
    })
})
~~~

### protobuf

~~~go
v1.GET("/someProtoBuf", func(c *gin.Context) {
    reps := []int64{int64(1), int64(2)}
    label := "test"
    // protobuf 的具体定义写在 testdata/protoexample 文件中。
    data := &protoexample.Test{
        Label: &label,
        Reps:  reps,
    }
    // 请注意，数据在响应中变为二进制数据
    // 将输出被 protoexample.Test protobuf 序列化了的数据
    c.ProtoBuf(http.StatusOK, data)
})
~~~

## 数据绑定到模型

>   Gin提供了两类绑定方法

### Must bind

-   **Methods** - `Bind`, `BindJSON`, `BindXML`, `BindQuery`, `BindYAML`
-   **Behavior** - 这些方法属于 `MustBindWith` 的具体调用. 如果发生绑定错误，则请求终止，并触发 `c.AbortWithError(400, err).SetType(ErrorTypeBind)`.响应状态码被设置为 400 并且 `Content-Type` 被设置为 `text/plain; charset=utf-8`. 如果您希望更好地控制绑定，考虑使用 `ShouldBind` 等效方法

### Should bind

-   **Methods** - `ShouldBind`, `ShouldBindJSON`, `ShouldBindXML`, `ShouldBindQuery`, `ShouldBindYAML`
-   **Behavior** - 属于 `ShouldBindWith` 的具体调用, 如果发生绑定错误，Gin 会返回错误并由开发者处理错误和请求

使用 `ShouldBind`方法时，Gin 会尝试根据 Content-Type 推断如何绑定.你也可以指定必须绑定的字段. 如果一个字段的 tag 加上了 `binding:"required"`，但绑定时是空值, Gin 会报错

==说明==

-   `c.ShouldBindBodyWith` 会在绑定之前将 body 存储到上下文中. 这会对性能造成轻微影响，如果调用一次就能完成绑定的话，那就不要用这个方法.
-   只有某些格式需要此功能，如 `JSON`, `XML`, `MsgPack`, `ProtoBuf`. 对于其他格式, 如 `Query`, `Form`, `FormPost`, `FormMultipart` 可以多次调用 `c.ShouldBind()` 而不会造成任任何性能损失

#### 绑定查询参数

~~~go
type Person struct {
	Name    string `form:"name"`
	Address string `form:"address"`
}
v1.GET("/bindQuery", func(context *gin.Context) {
    var person Person
    if context.ShouldBindQuery(&person) == nil {
        log.Println("====== Only Bind By Query String and Ignore Post======")
        log.Println(person.Name)
        log.Println(person.Address)
    }
    context.String(http.StatusOK, "bind Query")
})
~~~

#### 绑定表单

~~~go
type LoginForm struct {
	User     string `form:"user" binding:"required"`
	Password string `form:"password" binding:"required"`
}
// 如果是GET请求，只使用Form绑定引擎（`query`）。
// 如果是POST请求,首先检查content-type是否为JSON或XML然后再使用Form（form-data）
v1.POST("/bindForm", func(context *gin.Context) {
    var form LoginForm
    if context.ShouldBind(&form) == nil {
        log.Println(form.User)
        log.Println(form.Password)
    }
    context.String(http.StatusOK, "bind Form")
})
~~~

#### 绑定uri

~~~go
type Person struct {
	ID string `uri:"id" binding:"required,uuid"`
	Name string `uri:"name" binding:"required"`
}
r.GET("/:name/:id", func(context *gin.Context) {
    var person Person
    if err := context.ShouldBindUri(&person); err != nil {
        context.JSON(400, gin.H{"msg": err})
        return
    }
    c.JSON(200, gin.H{"name": person.Name, "uuid": person.ID})
})
~~~

#### 绑定JSON

~~~go
type Person struct {
	Name    string `form:"name"`
	Address string `form:"address"`
}
v1.POST("/bindjson", func(context *gin.Context) {
    var p Person
    if err := context.ShouldBindJSON(&p); err == nil {
        log.Println(p.Name)
        log.Println(p.Address)
    }
    context.JSON(http.StatusOK, gin.H{"msg": "OK"})
})
~~~
