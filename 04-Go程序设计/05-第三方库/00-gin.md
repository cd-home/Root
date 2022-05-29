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

## 状态保持

### cookie

~~~go
func Cookie(c *gin.Context) {
	cookie, _ := c.Cookie("key") // Get
	c.SetCookie("key", "value", 1000, "/", "127.0.0.1", false, true) // Set
	c.JSON(http.StatusOK, gin.H{
		"msg": "OK",
	})
}
~~~

### session

具体可见`https://github.com/gin-contrib/sessions`

~~~go
package main

import (
	"github.com/gin-contrib/sessions"
	"github.com/gin-gonic/gin"
    "github.com/gin-contrib/sessions/redis"
)
func main() {
	r := gin.Default()
	// store := cookie.NewStore([]byte("secret"))
    store, _ := redis.NewStore(10, "tcp", "localhost:6379", "", []byte("secret"))
	r.Use(sessions.Sessions("mysession", store))
	r.GET("/hello", func(c *gin.Context) {
		session := sessions.Default(c)
		if session.Get("hello") != "world" {
			session.Set("hello", "world")
			session.Save()
		}
		c.JSON(200, gin.H{"hello": session.Get("hello")})
	})
	r.Run(":8000")
}
~~~



## 上传文件

### 单文件

前端

注意：上传文件需要指定`enctype="multipart/form-data"`

~~~html
<form action="/api/v1/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="avatar">
    <input type="submit" name="提交">
</form>
~~~

后端

~~~go
func UploadPic(c *gin.Context) {
	file, _ := c.FormFile("avatar")
	fileName := time.Now().Format("20060102150405_") + header.Filename
    // fileName 充当path就在当前运行的程序目录下，也可以重新定义
	_ = c.SaveUploadedFile(file, fileName)
	c.JSON(http.StatusOK, gin.H{
		"code": 2000,
		"msg":  "OK",
		"url":  fileName,
	})
}
~~~

### 多文件

~~~go
func main() {
	r := gin.Default()
	r.POST("/upload", func(c *gin.Context) {
		// Multipart form
		form, _ := c.MultipartForm()
		files := form.File["upload"]
		for _, file := range files {
			log.Println(file.Filename)
			// 上传文件至指定目录
			c.SaveUploadedFile(file, dst)
		}
		c.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
	})
	r.Run(":8080")
}
~~~



## 中间件

1.  日志
2.  鉴权、权限

### 默认中间件

~~~go
// Default 使用 Logger 和 Recovery 中间件
r := gin.Default()
~~~

### 中间件

#### 定义

1.第一种形式

~~~go
func Auth(context *gin.Context) {
    // code
} 
~~~

2.  第二种形式

~~~go
func Auth() gin.HandlerFunc {
	return func(context *gin.Context) {
        //当前的请求上下文
        context.Set("requestID", "client_request_ID") // 可以设置数据在context中
        /*
        可以在当前的请求的context中获取
        reqID := c.MustGet("requestID").(string) 
        reqID, _ := c.Get("requestID")             // 通常情况下使用Get
        */
        // 请求之前
        context.Next()  // 执行下一个中间件，func (ctx *gin.Context)
        // 还可以中断当前请求 context.Abort()
        // 请求之后, 已经返回响应了
	}
}
~~~

*   context.Next() , 遇到Next就执行下一个中间件函数
*   context.Abort()
*   context.Set()  context.Get()

第二种形式，可以给Auth传递参数，可以定制话一些服务，比如某个标志，决定是否使用中间件

#### 使用

1.  全局中间件

~~~go
r := gin.New()
r.Use(middware.Auth())   // 可以注册多个 r.Use(middware.Auth(), middware.Log())
~~~

2.  分组中间件

~~~go
v1 := r.Group("api/v1/admin", middware.Auth())   // 可以注册多个 
// or
v1 := r.Group("api/v1/admin")
v1.Use(middware.Auth())
~~~

3.  路由中间件

~~~go
v1.POST("/jwt", middware.Auth(), controller.Jwt)  // 可以注册多个
~~~

### 定制验证器

### Goroutine

当在中间件或 handler 中启动新的 Goroutine 时，**不能**使用原始的上下文，必须使用只读副本

~~~go
func main() {
	r := gin.Default()
	r.GET("/long_async", func(c *gin.Context) {
		// 创建在 goroutine 中使用的副本
		cCp := c.Copy()
		go func() {
			// 用 time.Sleep() 模拟一个长任务。
			time.Sleep(5 * time.Second)
			// 请注意您使用的是复制的上下文 "cCp"，这一点很重要
			log.Println("Done! in path " + cCp.Request.URL.Path)
		}()
	})
}
~~~

## 测试用例

~~~go
package main
import (
	"net/http"
	"net/http/httptest"
	"testing"
	"github.com/stretchr/testify/assert"
)
func TestPingRoute(t *testing.T) {
	router := setupRouter()
	w := httptest.NewRecorder()
	req, _ := http.NewRequest("GET", "/ping", nil)
	router.ServeHTTP(w, req)
	assert.Equal(t, 200, w.Code)
	assert.Equal(t, "pong", w.Body.String())
}
~~~



## 其他

### 定制HTTP服务启动

~~~go
func main() {
	router := gin.Default()
	s := &http.Server{
		Addr:           ":8080",
		Handler:        router,
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	s.ListenAndServe()
}
~~~

### JSONP

~~~go
func main() {
	r := gin.Default()
	r.GET("/JSONP?callback=x", func(c *gin.Context) {
		data := map[string]interface{}{
			"foo": "bar",
		}
		// callback 是 x
		// 将输出：x({\"foo\":\"bar\"})
		c.JSONP(http.StatusOK, data)
	})
	r.Run(":8080")
}
~~~

### 日志

#### 记录日志

~~~go
func main() {
    // 禁用控制台颜色，将日志写入文件时不需要控制台颜色。
    gin.DisableConsoleColor()
    // 记录到文件。
    f, _ := os.Create("gin.log")
    gin.DefaultWriter = io.MultiWriter(f)
    // 如果需要同时将日志写入文件和控制台，请使用以下代码。
    // gin.DefaultWriter = io.MultiWriter(f, os.Stdout)
    router := gin.Default()
    router.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })
    router.Run(":8080")
}
~~~

#### 路由日志

~~~go
func main() {
	r := gin.Default()
	gin.DebugPrintRouteFunc = func(
        httpMethod, absolutePath, handlerName string, nuHandlers int) {
		log.Printf("endpoint %v %v %v %v\n", httpMethod, 
                   absolutePath, handlerName, nuHandlers)
	}
}

~~~

### 静态文件

~~~go
func main() {
	router := gin.Default()
    // 静态文件一般用这个 /static/1.jpg  去./static下面找
	router.Static("/static", "./static") 
    // 暴露文件，可以提供下载
    router.StaticFS("/more_static", http.Dir("."))
	router.StaticFS("/more_static", http.Dir("my_file_system"))
    // 单个文件
	router.StaticFile("/favicon.ico", "./resources/favicon.ico")
	router.Run(":8080")
}
~~~

### 多服务

~~~go
import (
	"log"
	"net/http"
	"time"

	"github.com/gin-gonic/gin"
	"golang.org/x/sync/errgroup"
)
var (
	g errgroup.Group
)
func router01() http.Handler {
	e := gin.New()
	e.Use(gin.Recovery())
	e.GET("/", func(c *gin.Context) {
		c.JSON(
			http.StatusOK,
			gin.H{
				"code":  http.StatusOK,
				"error": "Welcome server 01",
			},
		)
	})

	return e
}

func router02() http.Handler {
	e := gin.New()
	e.Use(gin.Recovery())
	e.GET("/", func(c *gin.Context) {
		c.JSON(
			http.StatusOK,
			gin.H{
				"code":  http.StatusOK,
				"error": "Welcome server 02",
			},
		)
	})

	return e
}
func main() {
	server01 := &http.Server{
		Addr:         ":8080",
		Handler:      router01(),
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}
	server02 := &http.Server{
		Addr:         ":8081",
		Handler:      router02(),
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}
	g.Go(func() error {
		return server01.ListenAndServe()
	})
	g.Go(func() error {
		return server02.ListenAndServe()
	})
	if err := g.Wait(); err != nil {
		log.Fatal(err)
	}
}
~~~

