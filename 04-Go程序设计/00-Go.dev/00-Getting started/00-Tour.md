[TOC]

### Tour

》Learn Go From Go.dev

#### Install

Linux

~~~bash
$ wget https://go.dev/dl/go1.19.linux-arm64.tar.gz
$ tar -C /usr/local/

$ export PATH=$PATH:/usr/local/go/bin
~~~

#### Package

~~~go
// main package
package main

// import package by path
import (
	"fmt"
	"math/rand"
	"time"
)

// 入口函数
func main() {
	rand.Seed(time.Now().Unix())
	randNum := rand.Intn(100)
	fmt.Println("My favorite number is", randNum)
}
~~~

Go程序是由Package[包]组织的; 程序在main package中启动运行. 此示例中import path:  math/rand, 最后一个Elememnt——rand为实际的包.

注意: rand.Intn(), 如果需要真正随机, 需要rand.Seed()

#### import

~~~go
import "fmt"
import "math/rand"

import (
	"fmt"
    "math/rand"
)
~~~

Go代码组织imports在括号中, 你也可以import多条; 但是最好的方式是分类import[std, our, third] pkg; 

不过我们无须担心, go fmt 会帮助格式化.

#### Exported names

》变量、

