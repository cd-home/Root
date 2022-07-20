[TOC]

### Effective Go

》From https://go.dev/doc/effective_go

#### Introduction

Go is a new language. Although it borrows ideas from existing languages, it has unusual properties that make effective Go programs different in character from programs written in its relatives. A straightforward translation of a C++ or Java program into Go is unlikely to produce a satisfactory result—Java programs are written in Java, not Go. On the other hand, thinking about the problem from a Go **perspective** could produce a successful but quite different program. In other words, to write Go well, it's important to understand its **properties and idioms**. It's also important to know the established conventions for programming in Go, such as naming, formatting, program construction, and so on, so that programs you write will be easy for other Go programmers to understand.

Go 是一门新语言. 尽管它借鉴了现有语言的思想, 但它具有不同寻常的特性,使得 `effective` Go 程序在性质上不同于用其 `relatives` 编写的程序.  将 C++ 或 Java 程序直接翻译成 Go 不太可能产生令人满意的结果— Java 程序是用 Java 编写的, 而不是 Go. 另一方面, 从 Go 的**角度**思考问题可能会产生一个成功但完全不同的程序. 换句话说, 要写好 Go, 理解它的**特性和语言习惯**很重要. 了解 Go 编程的既定约定也很重要, 例如命名、格式、程序构造等, 以便您编写的程序易于其他 Go 程序员理解. 

This document gives tips for writing clear, idiomatic Go code. It augments the [language specification](https://go.dev/ref/spec), the [Tour of Go](https://go.dev/tour/), and [How to Write Go Code](https://go.dev/doc/code.html), all of which you should read first.

本文档提供了编写清晰、惯用的 Go 代码的技巧. 它扩充了语言规范、Go 之旅和如何编写 Go 代码, 所有这些都是您应该首先阅读的. 

Note added January, 2022: This document was written for Go's release in 2009, and has not been updated significantly since. Although it is a good guide to understand how to use the language itself, thanks to the **stability of the language**, it says little about the libraries and nothing about significant changes to the Go ecosystem since it was written, such as the build system, testing, modules, and polymorphism. There are no plans to update it, as so much has happened and a large and growing set of documents, blogs, and books do a fine job of describing modern Go usage. Effective Go continues to be useful, but the reader should understand it is far from a complete guide. See [issue 28782](https://github.com/golang/go/issues/28782) for context.

2022 年 1 月添加的注释: 本文档是为 2009 年的 Go 版本编写的, 此后没有进行重大更新. 尽管它是理解如何使用语言本身的一个很好的指南,  多亏**语言的稳定性**, 它很少提及标准库, 也没有提及自编写以来对 Go 生态系统的重大变化, 例如构建系统、测试 、模块和多态性.  目前没有更新它的计划, 因为已经发生了很多事情，而且越来越多的文档、博客和书籍很好地描述了现代 Go 的使用.  Effective Go 仍然有用, 但读者应该明白它远非完整的指南. 有关上下文, 请参见问题 28782. 

#### Examples

The Go package sources (https://go.dev/src/) are intended to serve not only as the core library but also as examples of how to use the language. Moreover, many of the packages contain working, self-contained executable examples you can run directly from the http://go.dev/ web site, such as this one(https://pkg.go.dev/strings) (if necessary, click on the word "Example" to open it up). If you have a question about how to approach a problem or how something might be implemented, the documentation, code and examples in the library can provide answers, ideas and background.

Go 源码包 (https://go.dev/src) 不仅用作核心库, 而且用作如何使用该语言的示例.  此外, 许多包都包含有效的、独立的可执行示例, 您可以直接从 http://go.dev/ 网站运行, 例如这个(https://pkg.go.dev/strings)(如有必要,单击“示例”一词打开它) 如果您对如何解决问题或如何实现有疑问, 库中的文档、代码和示例可以提供答案、想法和背景. 

#### Formatting 	格式化

Formatting issues are the most contentious but the least consequential. People can adapt to different formatting styles but it's better if they don't have to, and less time is devoted to the topic if everyone adheres to the same style. The problem is how to approach this Utopia without a long prescriptive style guide.

格式问题是最有争议但影响最小的问题. 人们可以适应不同的格式风格, 但如果不需要的话会更好, 如果每个人都坚持相同的风格, 那么花在这个主题上的时间就会更少. 问题是, 如果没有一个冗长的规定性风格指南, 如何接近这个乌托邦. 

With Go we take an unusual approach and let the machine take care of most formatting issues. The `gofmt` program (also available as `go fmt`, which operates at the package level rather than source file level) reads a Go program and emits the source in a standard style of **indentation and vertical alignment**, retaining and if necessary reformatting comments. If you want to know how to handle some new layout situation, run `gofmt`; if the answer doesn't seem right, rearrange your program (or file a bug about `gofmt`), don't work around it.

使用Go, 我们采取了一种不同寻常的方法, 让机器处理大多数格式问题. gofmt程序(也称为go fmt), 在包级别而不是源文件级别运行) 读取go程序, 并以**缩进和垂直对齐**的标准样式发出源代码, 保留注释，并在必要时重新格式化注释. 如果您想知道如何处理一些新的布局情况, 请运行gofmt；如果答案似乎不正确, 请重新安排您的程序(或提交有关gofmt的错误), 不要绕过它.

「以包为对象进行格式化」

「缩进, 使用Tab缩进」

「if switch for无需圆括号(Go需要更少的圆括号)」

「行长 (Go没有行长的限制)」

「空格」

「注释」

「字段对齐」

~~~bash
go fmt
~~~

**建议: 使用 go fmt, 使得代码规范统一.** 

#### Commentary 注释

Go provides C-style /* */ block comments and C++-style // line comments. Line comments are the norm; block comments appear mostly as package comments, but are useful within an expression or to disable large swaths of code.

Go提供C风格的/**/块注释和C++风格的//行注释. 行注释是标准; 块注释主要以包注释的形式出现, 但在表达式中很有用, 或者可以禁用大量代码. 

「单行更为常用, 尽量是完整的说明, 并且简洁明了」

「每个包都应包含一段包注释, 即放置在包子句前的一个块注释」

~~~go
// 单行注释
/*
	多行注释
*/
~~~

**建议: 对包、结构体、函数、方法、枚举类型、核心逻辑做注释. 并且注释需要完整简洁.** 

#### Name 				命名

Names are as important in Go as in any other language. They even have semantic effect: the visibility of a name outside a package is determined by whether its first character is upper case. It's therefore worth spending a little time talking about naming conventions in Go programs.

名称在 Go 中与在任何其他语言中一样重要. 它们甚至具有语义效果: 名称在包外的可见性取决于它的第一个字符是否为大写.  因此, 值得花一点时间讨论 Go 程序中的命名约定. 

「名称首字母大写, 包外可见」

##### Package names 	包名称

When a package is imported, the package name becomes an accessor for the contents

当一个包被导入时, 包名成为内容的访问器.

After import "bytes" the importing package can talk about `bytes.Buffer`

It's helpful if everyone using the package can use the same name to refer to its contents, which implies that the package name should be good: short, concise, evocative. By convention, packages are given lower case, single-word names; there should be no need for underscores or mixedCaps. Err on the side of brevity, since everyone using your package will be typing that name. And don't worry about collisions a priori. The package name is only the default name for imports; it need not be unique across all source code, and in the rare case of a collision the importing package can choose a different name to use locally. In any case, confusion is rare because the file name in the import determines just which package is being used.

如果每个使用包的人都可以使用相同的名称来引用其内容, 这会很有帮助, 这意味着包的名称应该是好的: 简短、简洁、令人印象深刻.  按照惯例, 包被赋予小写的单个名称; 不需要下划线或混合大写字母. 简洁起见, 因为每个使用您的包的人都会输入该名称. 并且不要担心冲突. 包名只是导入的默认名称; 它不需要在所有源代码中都是唯一的, 并且在极少数发生冲突的情况下, 导入包可以选择不同的名称以在本地使用. 无论如何, 混淆很少见, 因为导入中的文件名决定了正在使用哪个包. 

「小写的单个单词来命名, 且不应使用下划线或驼峰记法」

Another convention is that the package name is the base name of its source directory; the package in `src/encoding/base64` is imported as `"encoding/base64"` but has name `base64`, not `encoding_base64` and not `encodingBase64`.

另一个约定是包名是其源目录的基名; src/encoding/base64 中的包被导入为"encoding/base64", 但名称为 base64, 不是 encoding_base64, 也不是 encodingBase64.

The importer of a package will use the name to refer to its contents, so exported names in the package can use that fact to avoid repetition. (Don't use the import . notation, which can simplify tests that must run outside the package they are testing, but should otherwise be avoided.) For instance, the buffered reader type in the ` bufio` package is called Reader, not BufReader, because users see it as bufio.Reader, which is a clear, concise name. Moreover, because imported entities are always **addressed** with their package name, bufio.Reader does not conflict with io.Reader. Similarly, the function to make new instances of ring.Ring—which is the definition of a constructor in Go—would normally be called NewRing, but since Ring is the only type exported by the package, and since the package is called ring, it's called just New, which clients of the package see as ring.New. Use the package structure to help you choose good names.

包的导入器将使用包名称来引用其内容, 因此包中的导出名称可以使用该事实来避免重复. 

(不要使用 import . 表示法, 它可以简化必须在他们正在测试的包之外运行的测试, 但应该避免使用)

 例如, bufio 包中的缓冲读取器类型称为 Reader, 而不是 BufReader, 因为用户将其视为 bufio.Reader, 这是一个简洁明了的名称. 此外, 由于导入的实体总是以其包名来**寻址**, 因此 bufio.Reader 不会与 io.Reader 冲突.

**「不要使用 import . 倒入全部名称」**

**「通过包来寻地址后, 不同包名称空间中可出现相同」** 

类似地, 创建 ring.Ring 的新实例的函数——它是 Go 中构造函数的定义——通常称为 NewRing, 但由于 Ring 是**包导出的唯一类型**, 并且由于包被称为 ring, 所以它是只称为 New, 包的客户将其视为ring.New.使用包结构来帮助您选择好名称. 「**如果包中只有一个类型, 构造函数可使用New更为简洁明了**」

Another short example is `once.Do`; `once.Do(setup)` reads well and would not be improved by writing `once.DoOrWaitUntilDone(setup)`. Long names don't automatically make things more readable. A helpful doc comment can often be more valuable than an extra long name.

另一个简短的例子是 once.Do; once.Do(setup) 可读性很好,并且不会通过编写once.DoOrWaitUntilDone(setup) 来改进. 长名称不会自动使内容更具可读性. 有用的文档注释通常比超长的名称更有价值. 

「**简短名字配合文档注释常常会更好**」

##### Interface names 	接口名字

1. 只包含一个方法的接口应当以该方法的名称加上-er后缀来命名
2. 接口与方法尽量不要使用标准库等常用的名称

**驼峰命名**

1. 勿使用关键字来命名
2. Go中约定使用驼峰记法

#### 分号

1.  正式语法使用分号来结束语句, 词法分析器会使用一条简单的规则来自动插入分号, **源码中不用分号**

2.  不能将一个控制结构（`if`、`for`、`switch` 或 `select`）的左大括号放在下一行

#### 控制

~~~go
if condition {
    // code
}

if err := file.Chmod(0664); err != nil {
	// code
}

// 注意 for中的局部变量
for init; cond; ins {
    // code
}

for condition {
 	// code   
}

for {
   // code 
}

// 遍历数组、切片、字符串或者映射, 或从信道中读取消息
for index, value := range collection {
    // code
}

for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}

func unhex(c byte) byte {
	switch {
	case '0' <= c && c <= '9':
		return c - '0'
	case 'a' <= c && c <= 'f':
		return c - 'a' + 10
	case 'A' <= c && c <= 'F':
		return c - 'A' + 10
	}
	return 0
}

func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}

switch t := t.(type) {
case bool:
	fmt.Printf("boolean %t\n", t)            
case int:
	fmt.Printf("integer %d\n", t)             
case *bool:
	fmt.Printf("pointer to boolean %t\n", *t) 
case *int:
	fmt.Printf("pointer to integer %d\n", *t) 
default:
	fmt.Printf("unexpected type %T", t)       
}
~~~

#### 函数

**多值返回**

1.  普通返回

~~~go
func Foo(s string) (string error) {
    // code
}
~~~

2.  命名返回

~~~go
func Foo(s string) (str string err error) {
    // code
}
~~~

3. 变参数

~~~go
func Bar(nums ...int) int {
    // code
}
~~~

**defer**

> 1.  Go的 `defer` 语句用于预设一个函数调用，该函数会在执行 `defer` 的函数返回之前立即执行
> 2.  被推迟的函数按照后进先出（LIFO）的顺序执行
> 3.  常被用作资源关闭或者配合Recover()

#### 数据

**new**

> 1.  它不会**初始化**内存, 只会将内存**置零**
> 2.  `new(T)` 会分配类型为 `T` 已置零的内存空间,  并返回它的地址. 【它返回一个指针,  该指针指向新分配的, 类型为 `T` 的零值】
> 3.  new用于值类型的内存分配【数字、字符串、布尔、数组、结构体】

**make**

> 1.  它只用于创建**切片、映射和信道**, 并返回类型为 `T`（而非 `*T`）的一个**已初始化** （而非**置零**）的值
>
> 2.  出现这种用差异的原因在于, 这三种类型本质上为引用数据类型, 它们在使用前必须初始化
> 3.  对于切片、映射和信道, `make` 用于初始化其内部的数据结构并准备好将要使用的值

#### 数组

> 1.  数组是值. 将一个数组赋予另一个数组会复制其所有元素
>
> 2.  特别地, 若将某个数组传入某个函数, 它将接收到该数组的一份**副本**而非指针
>
> 3.  数组的大小是其类型的一部分. 类型 `[10]int` 和 `[20]int` 是不同的
> 4.  数组为值的属性很有用, 但代价高昂；若你想要C那样的行为和效率, 你可以传递一个指向该数组的指针

#### 切片

> 1.  切片保存了对底层数组的引用, 若你将某个切片赋予另一个切片, 它们会引用同一个数组
> 2.  若某个函数将一个切片作为参数传入, 则它对该切片元素的修改对调用者而言同样可见,  这可以理解为传递了底层数组的指针
> 3.  只要切片不超出底层数组的限制, 它的长度就是可变的, 只需将它赋予其自身的切片即可
> 4.  切片的**容量**可通过内建函数 `cap` 获得, 它将给出该切片可取得的最大长度

~~~go
s = append(s, nums...)
~~~

#### 映射

1.  其键可以是任何相等性操作符支持的类型
2.  值可以是任意的类型

~~~go
value, ok := m["key"]
delete(m['key'])
~~~

#### 常量

1.  Go中的常量就是不变量. 它们在编译时创建, 即便它们可能是函数中定义的局部变量
2.  常量只能是数字、字符（符文）、字符串或布尔值

~~~go
type ByteSize float64

const (
    _           = iota // iota 从0开始 通过赋予空白标识符来忽略第一个值
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
~~~

#### init

1.  每个源文件都可以通过定义自己的无参数 `init` 函数来设置一些必要的状态
2.  `init` 函数还常被用在程序真正开始执行前，检验或校正程序的状态
3.  只有该包中的所有变量声明都通过它们的初始化器求值后 `init` 才会被调用

#### 方法

指针vs值作为接受者

> 以指针或值为接收者的区别在于：值方法可通过指针和值调用,  而指针方法只能通过指针来调用

#### 接口

接口

> Go中的接口为指定对象的行为提供了一种方法：如果某样东西可以完成**这个**,  那么它就可以用在**这里**

接口断言

~~~go
type Stringer interface {
	String() string
}
var value interface{} // 调用者提供的值
switch str := value.(type) {
case string:
	return str
case Stringer:
	return str.String()
}

if _, ok := val.(json.Marshaler); ok {
	fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
~~~

接口检查

~~~go
var _ json.Marshaler = (*RawMessage)(nil)
~~~

#### 空白标识

1.  空白标识符可被赋予或声明为任何类型的任何值, 而其值会被无害地丢弃
2.  未使用的导入和变量可以使用空白标识符

~~~go
var _, ok = m["key"] {
    // code
}
if _, err := os.Stat(path); os.IsNotExist(err) {
    // code
}
~~~

#### Import

> 1. 不允许未使用的导入
> 2. 可以重命名导入的包
> 3. 辅助作用导入

~~~go
import _ "net/http/pprof"
~~~

#### 并发

> 不要通过共享内存来通信，而应该通过通信来共享内存

**goroutine**

1.  它是与其它Go程并发运行在同一地址空间的函数，它是轻量级的,  所有消耗几乎就只有栈空间的分配，而且栈最开始是非常小的, 所以它们很廉价,  仅在需要时才会随着堆空间的分配（和释放）而变化

3.  Go程在多线程操作系统上可实现多路复用, 因此若一个线程阻塞,  那么其它的线程就会运行

**通道**

1.   `make` 来分配内存

~~~go
ci := make(chan int)            // 整数类型的无缓冲信道
cj := make(chan int, 0)         // 整数类型的无缓冲信道
cs := make(chan *os.File, 100)  // 指向文件指针的带缓冲信道
~~~

2.  无缓冲信道在通信时会同步交换数据, 它能确保（两个Go程的）计算处于确定状态

3.  若信道是带缓冲的, 则发送者仅在值被复制到缓冲区前阻塞，若缓冲区已满, 发送者会一直等待直到某个接收者取出一个值为止


#### 错误

error

> 按照约定, 错误的类型通常为 `error`, 这是一个内建的简单接口. 

```go
type error interface {
	Error() string
}
```

panic

> 不可恢复的错误

~~~go
panic("something wrong")
~~~

recover

当 `panic` 被调用后（包括不明确的运行时错误, 例如切片检索越界或类型断言失败）,  程序将立刻终止当前函数的执行, 并开始回溯Go程的栈, 运行任何被推迟的函数.  若回溯到达Go程栈的顶端, 程序就会终止. 不过我们可以用内建的 `recover` 函数来重新或来取回Go程的控制权限并使其恢复正常执行

~~~go
defer func() {
    if err := recover(); err != nil {
        log.Println(err)
    }    
}()
~~~