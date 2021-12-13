[TOC]

### Go Web

#### Server

> 采用处理器函数模式代码更加简洁，但是此模式下使用的是默认的多路复用器

~~~go
func Hello(rw http.ResponseWriter, request *http.Request) {
	rw.Header().Set("Content-Type", "application/json")
    bs := bytes.NewReader([]byte(`{"code": 1, "message": "Success", "data": "Hello"}`))
	_, _ = io.Copy(rw, bs)
}
func main() {
	http.HandleFunc("/", Hello)
	_ = http.ListenAndServe(":8080", nil)
}
~~~

#### Router

#### Query

#### Form

#### Body

#### Files

#### JSON

#### Static FIles

> 通常做静态文件

~~~go
package main

import (
	"net/http"
)

func main() {
	fs := http.FileServer(http.Dir("static/"))
	http.Handle("/static/", http.StripPrefix("/static/", fs))
	http.ListenAndServe(":8080", nil)
}
~~~

#### Database

#### MiddleWare

#### WebSockets

