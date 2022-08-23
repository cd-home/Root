[TOC]

### Tour

》Learn Go From Go.dev

#### Install Go

Linux

~~~bash
$ wget https://go.dev/dl/go1.19.linux-arm64.tar.gz
$ tar -C /usr/local/

$ export PATH=$PATH:/usr/local/go/bin
~~~

#### Package

Every Go program is made up[组织] of packages. Programs start running in package `main`.

~~~go
// main package
package main

// import package by path
import (
	"fmt"
	"math/rand"
	"time"
)

// "入口函数"
func main() {
	rand.Seed(time.Now().Unix())
	randNum := rand.Intn(100)
	fmt.Println("My favorite number is", randNum)
}
~~~

PS: Go程序是由Package[包]组织的; 程序在main package中启动运行. 

此示例中import path:  math/rand, 最后一个Elememnt——rand为实际的包.

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

PS: Go代码组织imports在括号中, 你也可以分开import多条; 但是最好的方式是分类import[std, our, third] pkg; 不过我们无须担心, go fmt 会帮助格式化.

#### Exported

In Go, a name is exported[导出] if it begins with a capital letter

~~~go
// Exported names
// a name is exported if it begins with a capital letter
// includes packages name space
// eg: variables
// eg: const
// eg: struct
// eg: function

// tips:
// struct fields also satisfies this rule

var ExportVar = "Export Var"

var UnExportVar = "UnExport Var By Getter Func To Export"

const ExportConst = "ExportConst"

// ExportStruct Can Export
type ExportStruct struct {
	Name string
	// Can Not Export When Use This Struct
	age uint8
}

// ExportFunc
// Maybe Some Name Can Not Be Exported, But User 'Getter' Func To Export
func ExportFunc() string {
	return unExportVar
}
~~~

PS: 导出的即是, 名称空间在其他Package中可用

#### Function

Functions 

~~~go
// Function name
// Params   list
// Return   list [support multi return Values]
func Foo(param1 string, param2 string, x, y int) (string, int, error) {
	return param1 + param2, x + y, nil
}
~~~

PS: 通常情况下error习惯放置最后一个返回

Params Type

~~~go
// SameParamsType
// When two or more consecutive named function parameters share a type,
// You can omit[省略] the type from all but the last.
func SameParamsType(x, y int) int {
	return x + y
}
~~~

PS: 如果参数列表类型是相同的, 可以省略前面的, 除了最后一个

Multiple Results

~~~go
// MultiReturnValue
// Return list [support multi return]
// A function can return any number of results.
func MultiReturnValue(x, y int) (int, int) {
	return y, x
}
~~~

PS: 函数支持多个返回值

Named Return Values

~~~go
// NamedReturnValue
// Func return values can be named
// They are treated as variables defined at the top of the function.
func NamedReturnValue(x, y int) (sum int) {
	sum = x + y
	// return can omit sum, if the func is short
	// if this is a longer functions, better to return values
	return
}
~~~

PS: 函数支持多个命名返回值, 其相当于在函数顶级作用域中进行声明;  并且return语句可以省略掉返回的参数[如果函数较短小, 不影响可读性的前提下, 否则你都应该显式的return返回值]

#### Variables

Declare Variables

~~~go
// DeclaresVariable
// Just Declares
var DeclaresVariable string
~~~

PS: Just Declare, You need zero value that It defaults

Declaration And Initializers

~~~go
// SingleGlobalVar
// Package Scope Var
// Declaration And Initializers
var SingleGlobalVar string = "Hello World"

// Type Assets
var a, b , c  = 10, 20, true

// Group
var (
	x string = "Go"
	y string = "Python"
	z string = "JavaScript"
)

func main() {
    // Function Scope
    var name = "Yao"
}
~~~

PS: 声明并且初始化变量, 可不指定类型, 可从初始化值推断出类型. 

变量的声明具有作用域[包、函数、for、if、括号]

Short variable declarations

~~~go
func main() {
    age := 18
}
~~~

PS: 短变量声明初始化形式, 只用于在函数内部

#### Basic Type

~~~go
// Basic Type
// string bool
// int    int8   int16   int32   int64
// uint   uint8  uint16  uint32  uint64
// uintptr

// byte
// alias for uint8

// rune
// alias for uint32
// unicode

// float32
// float64

// complex64
// complex128

func main() {
	fmt.Println("Hello World!")
	// 不同平台 不同位
	var a int = 10
	fmt.Println(unsafe.Sizeof(a))

	var b int16 = 50
	fmt.Println(unsafe.Sizeof(b))
}
~~~

PS: 类型 = 外在表现 + 内在值 例如 uint8 = "uint8" + "无符号8位的整型",  在程序中的外在表现以及实际存储的大小. 注意 int、uint的大小依据不同的平台. 在实际使用过程中, 根据需要选择类型.

#### Zero values

Variables declared without an explicit[明确的] initial value[初始化值] are given their *zero value*.

~~~go
func main() {
	var name string
	fmt.Println(name)

	var age uint8
	fmt.Println(age)

	var flag bool
	fmt.Println(flag)
}
~~~

PS: 0 for numeric type, false for boolean, "" for empty string

#### Type convert

The expression `T(v)` converts the value `v` to the type `T`.

~~~go
func main() {
	var age uint8
	fmt.Println(age)

	// The expression T(v) converts the value v to the type T.
	age = 18
	var f float32 = float32(age)
	fmt.Println(f)
}
~~~

PS: must explicit conversion[显式转换]

#### Type inference

When declaring a variable without specifying an explicit type (either by using the `:=` syntax or `var =` expression syntax), the variable's type is inferred from the value on the right hand side.

When the right hand side of the declaration is typed, the new variable is of that same type:

~~~go
func main() {
    var j = 10
	fmt.Printf("%T\n", j)

	k := j
	fmt.Printf("%T\n", k)

	l := 3.14
	fmt.Printf("%T\n", l)
}
~~~

PS: 声明并且初始化一个值时, 如果未指定具体的类型, 那么其类型将由右值推断而出.

#### Constants

~~~go
const PI = 3.14
const Version = "v1.0.0"
const Switch = false
const CHAR = 'a'
~~~

PS: constants must be numeric, string, boolean, character [数字、字符串、布尔、字符]

Numeric Constants

Numeric constants are high-precision *values*.

An untyped constant takes the type needed by its context. 

~~~go
const num = 2
func DoubleNumInt(x int) int {
	return x * x
}

func DoubleNumFloat(x float32) float32 {
	return x * x
}
func main() {
    fmt.Println(DoubleNumInt(num))
	fmt.Println(DoubleNumFloat(num))
}
~~~

PS: 未指定类型的数值常量通过具体的上下文获取类型

#### For

》Just One Loop Construct => for

The basic `for` loop has three components separated by semicolons[分号]:

1. the init statement: executed before the first iteration
2. the condition expression: evaluated before every iteration
3. the post statement[后置语句]: executed at the end of every iteration

~~~go
// ForLoopBaseControl
// Basic For-Loop Iter
func ForLoopBaseControl() {
	sum := 0
	for i := 0; i < 10; i++ {
		fmt.Println(i)
		sum += i
	}
	fmt.Println(sum)
}

// ForOtherStyleControl
// Init And Post Can Be Out
func ForOtherStyleControl() {
	i := 0
	for ;i < 10; {
		i++
		fmt.Println(i)
	}
}

// ForWhileControl
// For-"While"
func ForWhileControl() {
	i := 10
	for i > 0 {
		fmt.Println(i)
		i--
	}
}

// ForEverLoopControl
// ForEver Loop
func ForEverLoopControl() {
	for {
		fmt.Println()
		time.Sleep(time.Second)
	}
}
~~~

PS: 初始化语句通常是使用短变量方式初始化, 并且其作用域属于for循环;

#### If

the expression need not be surrounded by parentheses `( )` but the braces `{ }` are required

~~~go
// IFControlSqrt
// x = v*v
func IFControlSqrt(x float64) string {
	if x < 0 {
		return IFControlSqrt(-x) + "i"
	}
	return fmt.Sprint(math.Sqrt(x))
}

// IFControlPow
// x^n
func IFControlPow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}

// IFControlPow2
// x^n
func IFControlPow2(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%f > %f", v, lim)
	}
	return lim
}
~~~

PS: if的表达式不需要圆括号; 支持简短的变量声明;
