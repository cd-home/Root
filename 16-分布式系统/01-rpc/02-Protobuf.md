[TOC]

### Protobuf

语言无关、平台无关的数据交换格式. 相比较JSON, 二进制压缩率更加的高体积小, 传输更加快.

#### protoc

用于编译proto api 文件

~~~bash
$ https://github.com/protocolbuffers/protobuf/releases/
~~~

~~~bash
# 编译
$ wget protoc-21.5-osx-aarch_64.zip && tar && mv protoc /usr/local/bin
# Go-plugin
$ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
$ export PATH=$PATH:$(go env GOPATH)/bin

$ protoc --version
$ protoc --go_out=. ./proto/*.proto
~~~

#### proto

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
  string name = 1;
}

message HelloReply {
  string message = 1;
}
~~~

##### basic types

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

##### message

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

##### oneof

~~~protobuf
// 最多允许一个值
message Profile {
  oneof avatar {
    string image_url = 1;
    bytes image_data = 2;
  }
}
~~~

##### map

~~~protobuf
message Bar {}

message Baz {
  map<string, Bar> foo = 1;
}
~~~

##### enumerations

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

##### default values

没有传递字段的值时, 会有默认值

1. strings => ""
2. bytes => ''
3. bool => false
4. Numeric => 0
5. Enums => enum第一个定义的值
6. 嵌套的其他message