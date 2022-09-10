[TOC]

### gRPC

#### Protobuf

》Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.

用于序列化与反序列化数据

- [x] proto描述文件更加的简洁清晰
- [x] 二进制模式体积更加的小, 传输更加快

##### .proto

###### message type

~~~protobuf
// proto3  注释 // /* */
syntax = "proto3";


// 每个字段都有类型
// 唯一的数字id(标识二进制中的字段) 1-15 1bytes 16-2047 2bytes
message Person {
  // required 必须
  required string name = 1;
  int32 id = 2; 
  // 可选的
  optional string email = 3;
  
  // 枚举类型
  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    // 具体使用枚举
    PhoneType type = 2;
  }
  // 重复(切片) [0-N]
  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

// Our address book file is just one of these.
// 嵌套类型
message AddressBook {
  repeated Person people = 1;
}
~~~

###### Oneof

~~~protobuf
// 最多允许一个值
message Profile {
  oneof avatar {
    string image_url = 1;
    bytes image_data = 2;
  }
}
~~~

###### Map

~~~protobuf
message Bar {}

message Baz {
  map<string, Bar> foo = 1;
}
~~~

###### Enumerations

~~~protobuf
message SearchRequest {
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 1;
}
~~~

###### Default Values

没有传递字段的值时, 会有默认值

1. strings => ""
2. bytes => ''
3. bool => false
4. Numeric => 0
5. Enums => enum第一个定义的值
6. 嵌套的其他message

###### Basic Types

| .proto Type | Go Type |
| :---------: | :-----: |
|   double    | float64 |
|    float    | float32 |
|    int32    |  int32  |
|    int64    |  int64  |
|   uint32    | uint32  |
|   uint64    | uint64  |
|   sint32    |  int32  |
|   sint64    |  int64  |
|   fixed32   | uint32  |
|   fixed64   | uint64  |
|  sfixed32   |  int32  |
|  sfixed64   |  int64  |
|    bool     |  bool   |
|   string    | string  |
|    bytes    | []byte  |

#### gRPC

gRPC 是一个高性能、开源和通用的 RPC 框架

|   特性   |         gRPC         |    RESTful     |
| :------: | :------------------: | :------------: |
|   规范   |        .proto        |    OpenAPI     |
|   协议   |        HTTP/2        |    HTTP 1.1    |
| 有效载荷 |       Protobuf       |      JSON      |
|  流传输  | 客户端、服务端、双向 | 客户端、服务端 |

#### Quick Start

##### Step

1. Define message formats in a `.proto` file. [编写.proto文件]

~~~protobuf
syntax = "proto3";

package hello;

// The go_package option defines the import path of the package which
// will contain all the generated code for this file.
option go_package = "./api";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// A message is just an aggregate containing a set of typed fields
message HelloRequest {
  // 1 2 3 是字段的唯一标记
  string name = 1;
}

message HelloReply {
  string message = 1;
}
~~~

2. Use the protocol buffer compiler. [编译出pb文件] [只要proto文件变动, 就必须重新编译]

~~~bash
$ wget protoc-21.5-osx-aarch_64.zip && tar && mv protoc /usr/local/bin
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
# 移动到PATH或者添加PATH
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
