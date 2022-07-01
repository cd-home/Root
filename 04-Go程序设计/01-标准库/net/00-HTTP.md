[TOC]

### HTTP

#### Server

> HTTP Server

##### 核心流程图

![HTTP](images/HTTP.svg)

##### 处理器模式

~~~go
package main

import (
	"bytes"
	"io"
	"net/http"
)

// HelloHandler 处理器
type HelloHandler struct{}

// 实现Handler接口
func (h *HelloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	io.Copy(w, bytes.NewReader([]byte("Hello!")))
}

// WorldHandler 处理器
type WorldHandler struct{}

// 实现Handler接口
func (h *WorldHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	io.Copy(w, bytes.NewReader([]byte("World!")))
}

func main() {
	svr := &http.Server{
		Addr:    "127.0.0.1:8080",
		Handler: nil, // 如果是nil 程序就会默认设置一个多路复用的处理器
	}
	// pattern-handler 注册到 多路复用处理器上
	http.Handle("/hello", &HelloHandler{})
	http.Handle("/world", &WorldHandler{})
	svr.ListenAndServe()
}
~~~

##### 处理器函数模式

> 实际上内部还是调用处理器模式

~~~go
package main

import (
	"bytes"
	"io"
	"net/http"
)

func main() {
	http.HandleFunc("/hello", Hello)
	http.ListenAndServe(":8080", nil)
}

func Hello(rw http.ResponseWriter, request *http.Request) {
	io.Copy(rw, bytes.NewReader([]byte("Hello")))
}
~~~

#### Client

