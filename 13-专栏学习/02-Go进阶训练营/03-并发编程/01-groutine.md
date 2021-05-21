### goroutine

#### 对创建的goroutine负责

> 不要创建一个不知道何时退出的goroutine

~~~go
package main

import (
	"log"
	"net/http"
	_ "net/http/pprof"
)

func setup() {
	// 这里面有一些初始化的操作
}

func main() {
	setup()

	// 主服务
	server()

	// for debug
	pprof()

	select {}
}

func server() {
	go func() {
		mux := http.NewServeMux()
		mux.HandleFunc("/ping", func(w http.ResponseWriter, r *http.Request) {
			w.Write([]byte("pong"))
		})

		// 主服务
		if err := http.ListenAndServe(":8080", mux); err != nil {
			log.Panicf("http server err: %+v", err)
			return
		}
	}()
}

func pprof() {
	// 辅助服务，监听了其他端口，这里是 pprof 服务，用于 debug
	go http.ListenAndServe(":8081", nil)
}
~~~

问题

1. 如果server是其他的包导入的，并不明确server是一个异步调用
2. main中的select{}阻塞形式不友好
3. 线上出现故障，导致ppro监听debug服务退出

讲选择权