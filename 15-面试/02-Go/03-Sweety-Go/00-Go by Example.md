[TOC]

### Go By Example

>   通过例子、快速学习go语言

#### Hello World

~~~go
package main
import "fmt"
func main() {
    fmt.Println("Hello world!")
}
~~~

~~~bash
go run hello.go
go build hello.go & ./hello
~~~

#### Value

>   数字类型、字符串类型、布尔值

~~~go
func main() {
    fmt.Println("Hello" + "World")
    fmt.Println(true && false)
    fmt.Println(!true)
}
~~~

#### Var

>   声明变量

~~~go
func main() {
    var num int = 20
    var name string = "God Yao"
   	var high, weight = 160, 55
}
~~~

#### Const

>   声明常量

~~~go
const PI = 3.14
func main() {
    const M = 12
}
~~~

#### For

>   for循环， 初始化、条件终止、增量 的圆括号不是必须的

~~~go
func main() {
    var num = 0
    
    for num <= 5 {
        fmt.Println(num)
        num++
    }
    
    for i := 0; i < 10; i++ {
        if i == 5 {
            break
        }
        fmt.Println(i)
    }
    
    for {
        fmt.Println("forever loop")
    }
}
~~~

#### IF

>   条件的圆括号不是必须的

~~~go
func main() {
    num := 10
    if num % 2 == 0 {
        fmt.Println("num偶数")
    } else {
        fmt.Println("num奇数")
    }
}
~~~

#### Switch

>   1.  case可以使表达式
>   2.  case可以更多个值
>   3.  全部不符合走default

~~~go
func main() {
    typefunc := func(i interface{}) {
        switch t := i.(type) {
        case bool:
            fmt.Println("bool")
        case string:    
            fmt.Println("string")
        case int8, int16, int32, int64:
            fmt.Println("int")
        default:
            fmt.Println("default")    
        }
    } 
}
~~~

