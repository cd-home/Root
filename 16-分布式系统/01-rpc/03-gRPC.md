[TOC]

### gRPC

gRPC 是一个高性能、开源和通用的 RPC 框架

|   特性   |         gRPC         |    RESTful     |
| :------: | :------------------: | :------------: |
|   规范   |        .proto        |    OpenAPI     |
|   协议   |        HTTP/2        |    HTTP 1.1    |
| 有效载荷 |       Protobuf       |      JSON      |
|  流传输  | 客户端、服务端、双向 | 客户端、服务端 |

#### Quick Start

1. Define message formats in a `.proto` file. [编写.proto文件]

~~~protobuf
syntax = "proto3";

package hello;

option go_package = "./api";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
~~~

2. Use the protocol buffer compiler. [编译出pb文件] [只要proto文件变动, 就必须重新编译]

~~~bash
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
$ export PATH="$PATH:$(go env GOPATH)/bin"

# 编译 proto
$ protoc --go_out=. --go-grpc_out=. ./proto/*.proto
~~~

3. Use the Go protocol buffer API to write and read messages. [google.golang.org/protobuf API读写消息]

~~~go
package main

import (
	"fmt"
	"google.golang.org/protobuf/proto"
	"grpc/api"
)

func main() {
	r := &api.HelloRequest{Name: "yao"}
	bytes, err := proto.Marshal(r)
	if err != nil {
		return
	}
	fmt.Println(bytes)
	var req api.HelloRequest
	proto.Unmarshal(bytes, &req)
	fmt.Println(req.Name)
}
~~~

#### Simple Hello

##### server

~~~go
package main

import (
	"context"
	"google.golang.org/grpc"
	"grpc/api"
	"net"
	"os"
	"os/signal"
	"syscall"
)

// Greeter
// protoc --go_out=. --go-grpc_out=. ./proto/*.proto
type Greeter struct {
	api.UnimplementedGreeterServer
}

// SayHello
// implemented service methods
func (g *Greeter) SayHello(ctx context.Context, request *api.HelloRequest) (*api.HelloReply, error) {
	return &api.HelloReply{Message: "He"}, nil
}

func main() {
	listener, _ := net.Listen("tcp", "127.0.0.1:8081")
	// 创建服务
	//var opts []grpc.ServerOption
	//srv := grpc.NewServer(opts...)
	srv := grpc.NewServer()
	// Greeter注册到gRPC服务
	api.RegisterGreeterServer(srv, &Greeter{})

	// 启动服务
	go func() {
		_ = srv.Serve(listener)
	}()
	sigs := make(chan os.Signal, 1)
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
	<-sigs
	srv.GracefulStop()
}
~~~

##### client

~~~go
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"grpc/api"
	"log"
)

func main() {
	conn, _ := grpc.Dial("127.0.0.1:8081",
		grpc.WithTransportCredentials(insecure.NewCredentials()),
	)
	defer conn.Close()

	// 获取gRPC客户端
	// client stub
	client := api.NewGreeterClient(conn)
	// blocking/synchronous mode 阻塞同步模式
	reply, err := client.SayHello(context.Background(), &api.HelloRequest{Name: "Nike"})
	if err != nil {
		log.Println(err)
		return
	}
	fmt.Println(reply.Message)
}
~~~

####  Service API

gRPC 中的 Service API 有如下4种类型：

1. 简单模式 Simple RPC

   客户端发起请求并等待服务端响应

2. 服务端流式 Server-side streaming RPC

   客户端发送请求到服务器, 拿到一个流去读取返回的消息序列. 客户端读取返回的流, 直到里面没有任何消息. 

3. 客户端流式 Client-side streaming RPC

   与服务端数据流模式相反, 这次是客户端源源不断的向服务端发送数据流, 而在发送结束后, 由服务端返回一个响应. 

4. 双向流式 Bidirectional streaming RPC

   双方使用读写流去发送一个消息序列, 两个流独立操作, 双方可以同时发送和同时接收
