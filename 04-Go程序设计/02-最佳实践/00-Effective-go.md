[TOC]

### Effective Go

》From https://go.dev/doc/effective_go

#### Introduction

Go is a new language. Although it borrows ideas from existing languages, it has unusual properties [特性] that make effective Go programs different in character from programs written in its relatives. A straightforward translation of a C++ or Java program into Go is unlikely to produce a satisfactory [令人满意的] result—Java programs are written in Java, not Go. On the other hand, thinking about the problem from a Go perspective [角度] could produce a successful but quite different program. In other words, to write Go well, it's important to understand its properties and idioms [语言习惯]. It's also important to know the established conventions for programming in Go, such as naming, formatting, program construction, and so on, so that programs you write will be easy for other Go programmers to understand.

Go 是一门新语言. 尽管它借鉴了现有语言的思想, 但它具有不同寻常的特性,使得 `effective` Go 程序在性质上不同于用其 `relatives` 编写的程序.  将 C++ 或 Java 程序直接翻译成 Go 不太可能产生令人满意的结果— Java 程序是用 Java 编写的, 而不是 Go. 另一方面, 从 Go 的**角度**思考问题可能会产生一个成功但完全不同的程序. 换句话说, 要写好 Go, 理解它的**特性和语言习惯**很重要. 了解 Go 编程的既定约定也很重要, 例如命名、格式、程序构造等, 以便您编写的程序易于其他 Go 程序员理解. 

This document gives tips for writing clear, idiomatic Go code. It augments the [language specification](https://go.dev/ref/spec), the [Tour of Go](https://go.dev/tour/), and [How to Write Go Code](https://go.dev/doc/code.html), all of which you should read first.

本文档提供了编写清晰、惯用的 Go 代码的技巧. 它扩充了语言规范、Go 之旅和如何编写 Go 代码, 所有这些都是您应该首先阅读的. 

Note added January, 2022: This document was written for Go's release in 2009, and has not been updated significantly since. Although it is a good guide to understand how to use the language itself, thanks to the **stability of the language**, it says little about the libraries and nothing about significant changes to the Go ecosystem since it was written, such as the build system, testing, modules, and polymorphism. There are no plans to update it, as so much has happened and a large and growing set of documents, blogs, and books do a fine job of describing modern Go usage. Effective Go continues to be useful, but the reader should understand it is far from a complete guide. See [issue 28782](https://github.com/golang/go/issues/28782) for context.

2022 年 1 月添加的注释: 本文档是为 2009 年的 Go 版本编写的, 此后没有进行重大更新. 尽管它是理解如何使用语言本身的一个很好的指南,  多亏**语言的稳定性**, 它很少提及标准库, 也没有提及自编写以来对 Go 生态系统的重大变化, 例如构建系统、测试 、模块和多态性.  目前没有更新它的计划, 因为已经发生了很多事情, 而且越来越多的文档、博客和书籍很好地描述了现代 Go 的使用.  Effective Go 仍然有用, 但读者应该明白它远非完整的指南. 有关上下文, 请参见问题 28782. 

#### Examples

The Go package sources (https://go.dev/src/) are intended to serve not only as the core library but also as examples of how to use the language. Moreover, many of the packages contain working, self-contained executable examples you can run directly from the http://go.dev/ web site, such as this one(https://pkg.go.dev/strings) (if necessary, click on the word "Example" to open it up). If you have a question about how to approach a problem or how something might be implemented, the documentation, code and examples in the library can provide answers, ideas and background.

Go 源码包 (https://go.dev/src) 不仅用作核心库, 而且用作如何使用该语言的示例.  此外, 许多包都包含有效的、独立的可执行示例, 您可以直接从 http://go.dev/ 网站运行, 例如这个(https://pkg.go.dev/strings)(如有必要,单击“示例”一词打开它) 如果您对如何解决问题或如何实现有疑问, 库中的文档、代码和示例可以提供答案、想法和背景. 

#### Formatting 																	格式化

Formatting issues [问题] are the most contentious [争议] but the least consequential [影响]. People can adapt to [适应] different formatting styles but it's better if they don't have to, and less time is devoted to the topic if everyone adheres to the same style. The problem is how to approach this Utopia without a long prescriptive style guide.

格式问题是最有争议但影响最小的问题. 人们可以适应不同的格式风格, 但如果不需要的话会更好, 如果每个人都坚持相同的风格, 那么花在这个主题上的时间就会更少. 问题是, 如果没有一个冗长的规定性风格指南, 如何接近这个乌托邦. 

With Go we take an unusual approach and let the machine take care of most formatting issues. The `gofmt` program (also available as `go fmt`, which operates at the package level rather than source file level) reads a Go program and emits the source in a standard style of indentation [缩进] and vertical alignment [垂直对齐], retaining [保留] and if necessary reformatting comments. If you want to know how to handle some new layout situation, run `gofmt`; if the answer doesn't seem right, rearrange your program (or file a bug about `gofmt`), don't work around it.

使用Go, 我们采取了一种不同寻常的方法, 让机器处理大多数格式问题. gofmt程序(也称为go fmt), 在包级别而不是源文件级别运行) 读取go程序, 并以**缩进和垂直对齐**的标准样式发出源代码, 保留注释, 并在必要时重新格式化注释. 如果您想知道如何处理一些新的布局情况, 请运行gofmt；如果答案似乎不正确, 请重新安排您的程序(或提交有关gofmt的错误), 不要绕过它.

- [ ] 以包为对象进行格式化
- [ ] 缩进, 使用Tab缩进
- [ ] if switch for无需圆括号
- [ ] 行长 (Go没有行长的限制)
- [ ] 空格
- [ ] 注释
- [ ] 字段对齐

~~~bash
go fmt
~~~

**建议: 使用 go fmt, 使得代码规范统一.** 

#### Commentary 																注释

Go provides C-style /* */ block comments and C++-style // line comments. Line comments are the norm; block comments appear mostly as package comments, but are useful within an expression or to disable large swaths of code.

Go提供C风格的/**/块注释和C++风格的//行注释. 行注释是标准; 块注释主要以包注释的形式出现, 但在表达式中很有用, 或者可以禁用大量代码. 

- [ ] 单行更为常用, 尽量是完整的说明, 并且简洁明了
- [ ] 每个包都应包含一段包注释, 即放置在包子句前的一个块注释

~~~go
// 单行注释
/*
	多行注释
*/
~~~

**建议: 对包、结构体、函数、方法、枚举类型、核心逻辑做注释. 并且注释需要完整简洁.** 

#### Name 																				命名

Names are as important in Go as in any other language. They even have semantic [语义] effect: the visibility [可见性] of a name outside a package is determined [决定, 取决于] by whether its first character is upper case. It's therefore worth spending a little time talking about naming conventions in Go programs.

名称在 Go 中与在任何其他语言中一样重要. 它们甚至具有语义效果: 名称在包外的可见性取决于它的第一个字符是否为大写.  因此, 值得花一点时间讨论 Go 程序中的命名约定.  

- [x] 名称首字母大写, 包外可见

##### Package names 							        

When a package is imported, the package name becomes an accessor [访问器] for the contents

当一个包被导入时, 包名成为内容的访问器.

After import "bytes" the importing package can talk about `bytes.Buffer`

It's helpful if everyone using the package can use the same name to refer to its contents, which implies [意味着] that the package name should be good: short, concise [简洁], evocative [印象深刻]. By convention [按照惯例], packages are given lower case, single-word names; there should be no need for underscores or mixedCaps. Err on the side of brevity, since everyone using your package will be typing that name. And don't worry about collisions a priori. The package name is only the default name for imports; it need not be unique across all source code, and in the rare case of a collision the importing package can choose a different name to use locally. In any case, confusion is rare because the file name in the import determines just which package is being used.

如果每个使用包的人都可以使用相同的名称来引用其内容, 这会很有帮助, 这意味着包的名称应该是好的: 简短、简洁、令人印象深刻.  按照惯例, 包被赋予小写的单个名称; 不需要下划线或混合大写字母. 简洁起见, 因为每个使用您的包的人都会输入该名称. 并且不要担心冲突. 包名只是导入的默认名称; 它不需要在所有源代码中都是唯一的, 并且在极少数发生冲突的情况下, 导入包可以选择不同的名称以在本地使用. 无论如何, 混淆很少见, 因为导入中的文件名决定了正在使用哪个包. 

- [x] 简短、简介、小写、见名知意

Another convention is that the package name is the base name of its source directory; the package in `src/encoding/base64` is imported as `"encoding/base64"` but has name `base64`, not `encoding_base64` and not `encodingBase64`.

另一个约定是包名是其源目录的基名; src/encoding/base64 中的包被导入为"encoding/base64", 但名称为 base64, 不是 encoding_base64, 也不是 encodingBase64.

- [x] 包名是其源代码文件目录名

The importer of a package will use the name to refer to its contents, so exported names in the package can use that fact to avoid repetition. (Don't use the import .  notation [表示法], which can simplify tests that must run outside the package they are testing, but should otherwise be avoided.) For instance, the buffered reader type in the ` bufio` package is called Reader, not BufReader, because users see it as bufio.Reader, which is a clear, concise name. Moreover, because imported entities are always **addressed** with their package name, bufio.Reader does not conflict with io.Reader. Similarly, the function to make new instances of ring.Ring—which is the definition of a constructor in Go—would normally be called NewRing, but since Ring is the only type exported by the package, and since the package is called ring, it's called just New, which clients of the package see as ring.New. Use the package structure to help you choose good names.

包的导入器将使用包名称来引用其内容, 因此包中的导出名称可以使用该事实来避免重复. (不要使用 import . 表示法, 它可以简化必须在他们正在测试的包之外运行的测试, 但应该避免使用). 例如, bufio 包中的缓冲读取器类型称为 Reader, 而不是 BufReader, 因为用户将其视为 bufio.Reader, 这是一个简洁明了的名称. 此外, 由于导入的实体总是以其包名来**寻址**, 因此 bufio.Reader 不会与 io.Reader 冲突.

- [x] **不要使用 import . 倒入全部名称**
- [x] **通过包来寻地址后, 不同包名称空间中可出现相同** 

类似地, 创建 ring.Ring 的新实例的函数——它是 Go 中构造函数的定义——通常称为 NewRing, 但由于 Ring 是**包导出的唯一类型**, 并且由于包被称为 ring, 所以它是只称为 New, 包的客户将其视为ring.New.使用包结构来帮助您选择好名称. 「**如果包中只有一个类型, 构造函数可使用New更为简洁明了**」

- [x] 包类唯一导出类型的工厂函数使用New会更加好

Another short example is `once.Do`; `once.Do(setup)` reads well and would not be improved by writing `once.DoOrWaitUntilDone(setup)`. Long names don't automatically make things more readable. A helpful doc comment can often be more valuable than an extra long name.

另一个简短的例子是 once.Do; once.Do(setup) 可读性很好,并且不会通过编写once.DoOrWaitUntilDone(setup) 来改进. 长名称不会自动使内容更具可读性. 有用的文档注释通常比超长的名称更有价值. 

- [x] **简短名字配合文档注释常常会更好**

##### Getters 		

Go doesn't provide automatic [自动] support for getters and setters. There's nothing wrong with providing getters and setters yourself, and it's often appropriate to do so, but it's neither idiomatic nor necessary to put `Get` into the getter's name. If you have a field called `owner` (lower case, unexported), the getter method should be called `Owner` (upper case, exported), not `GetOwner`. The use of upper-case names for export provides the hook to discriminate [区分] the field from the method. A setter function, if needed, will likely be called `SetOwner`. Both names read well in practice:

- [x] 获取 owner 采用 Owner 作为 Getter 更好
- [x] 设置 owner 采用 SetOwner  作为 Setter 更好

##### Interface names

By convention, one-method interfaces are named by the method name plus an -er suffix or similar modification to construct an agent noun: `Reader`, `Writer`, `Formatter`, `CloseNotifier` etc.

There are a number of such names and it's productive to honor them and the function names they capture. `Read`, `Write`, `Close`, `Flush`, `String` and so on have canonical signatures and meanings. To avoid confusion, don't give your method one of those names unless it has the same signature and meaning. Conversely [相反的], if your type implements a method with the same meaning as a method on a well-known type, give it the same name and signature; call your string-converter method `String` not `ToString`.

- [x] 只包含一个方法的接口应当以该方法的名称加上-er 后缀来命名
- [x] 接口与方法尽量不要使用标准库等常用的名称. [如果确实需要的话, 命名方式可参考标准库的方式]

##### MixedCaps   

Finally, the convention in Go is to use `MixedCaps` or `mixedCaps` rather than underscores to write multiword names.

Go中约定使用大驼峰或者小驼峰

#### Semicolons 																	分号

Like C, Go's formal grammar uses semicolons to terminate statements, but unlike in C, those semicolons do not appear in the source. Instead the lexer uses a simple rule to insert semicolons automatically as it scans, so the input text is mostly free of them.

与 C 一样, Go 的形式语法使用分号来终止语句, 但与 C 不同的是, 这些分号不会出现在源代码中. 相反, 词法分析器使用一个简单的规则在扫描时自动插入分号, 因此输入文本大部分没有分号. 

The rule is this. If the last token before a newline is an identifier (which includes words like `int` and `float64`), a basic literal such as a number or string constant, or one of the tokens

规则是这样的.  如果换行符之前的最后一个标记是标识符(包括诸如 `int` 和 `float64` 之类的词), 则为基本文字, 例如数字或字符串常量, 或其中一个标记

~~~go
break continue fallthrough return ++ -- ) }
~~~

the lexer always inserts a semicolon after the token. This could be summarized as, "if the newline comes after a token that could end a statement, insert a semicolon".

词法分析器总是在标记后插入一个分号.  这可以概括为"如果换行符出现在可以结束语句的标记之后, 则插入分号".

- [x] 正式语法使用分号来结束语句, 词法分析器会使用一条简单的规则来自动插入分号, **源码中不用分号**

A semicolon can also be omitted immediately before a closing brace, so a statement such as

~~~go
go func() { for { dst <- <-src } }()
~~~

needs no semicolons. Idiomatic Go programs have semicolons only in places such as `for` loop clauses, to separate the initializer, condition, and continuation elements. They are also necessary to separate multiple statements on a line, should you write code that way.

- [x] 分号显式的出现使用于for循环隔断初始化值、条件、后置语句. 

One consequence [后果] of the semicolon insertion rules is that you cannot put the opening brace of a control structure (`if`, `for`, `switch`, or `select`) on the next line. If you do, a semicolon will be inserted before the brace, which could cause unwanted effects. 

分号插入规则的一个后果是您不能将控制结构(if、for、switch 或 select)的左大括号放在下一行.  如果这样做, 将在大括号前插入分号, 这可能会导致不良影响

- [x] 不能将一个控制结构（`if`、`for`、`switch` 或 `select`）的左大括号放在下一行

#### Control structures 														控制结构

The control structures of Go are related to those of C but differ in important ways. There is no `do` or `while` loop, only a slightly generalized `for`; `switch` is more flexible; `if` and `switch` accept an optional initialization statement like that of `for`; `break` and `continue` statements take an optional label to identify what to break or continue; and there are new control structures including a type switch and a multiway communications multiplexer, `select`. The syntax is also slightly different: there are no parentheses and the bodies must always be brace-delimited.

Go 的控制结构与 C 的控制结构相关, 但在重要方面有所不同. 没有 do 或 while 循环, 只有一个稍微概括的 for; switch更灵活;  if 和 switch 接受类似于 for 的可选初始化语句;  break 和 continue 语句采用可选标签来标识要中断或继续的内容; 并且有新的控制结构; 包括类型选择和多路通信多路复用器, select. 语法也略有不同: 没有括号,  逻辑主体必须始终用大括号分隔. 

##### if 

In Go a simple `if` looks like this:

~~~go
if x > 0 {
    
}

if condition {
    // code
}
~~~

Mandatory braces encourage writing simple `if` statements on multiple lines. It's good style to do so anyway, especially when the body contains a control statement such as a `return` or `break`.

强制大括号鼓励在多行上编写简单的 if 语句. 无论如何, 这样做是一种很好的风格, 尤其是当主体包含控制语句(如返回或中断)时.

Since `if` and `switch` accept an initialization statement, it's common to see one used to set up a local variable.

由于 if 和 switch 接受初始化语句, 因此很常见用于设置局部变量.

~~~go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
~~~

In the Go libraries, you'll find that when an `if` statement doesn't flow into the next statement—that is, the body ends in `break`, `continue`, `goto`, or `return`—the unnecessary `else` is omitted.

在 Go 库中, 您会发现当 if 语句没有进入下一条语句时(即主体以 break、continue、goto 或 return 结尾)时, 省略了不必要的 else.

~~~go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
~~~

This is an example of a common situation where code must guard against a sequence of error conditions. The code reads well if the successful flow of control runs down the page, eliminating error cases as they arise. Since error cases tend to end in `return` statements, the resulting code needs no `else` statements.

这是代码必须防范一系列错误条件的常见情况的示例. 如果成功的控制流顺着页面向下运行, 则代码可读性很好, 从而消除了出现的错误情况. 由于错误情况往往以 return 语句结束, 因此生成的代码不需要 else 语句. 

~~~go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
~~~

##### Redeclaration and reassignment

An aside: The last example in the previous section demonstrates a detail of how the `:=` short declaration form works. The declaration that calls `os.Open` reads,

顺便说一句: 上一节中的最后一个示例详细说明了 := 短声明形式的工作原理.  调用 os.Open 的声明如下:

~~~go
f, err := os.Open(name)
~~~

This statement declares two variables, `f` and `err`. A few lines later, the call to `f.Stat` reads,

~~~go
d, err := f.Stat()
~~~

which looks as if it declares `d` and `err`. Notice, though, that `err` appears in both statements. This duplication is legal: `err` is declared by the first statement, but only *re-assigned* in the second. This means that the call to `f.Stat` uses the existing `err` variable declared above, and just gives it a new value.

看起来好像它声明了 d 和 err.  但是请注意, 错误出现在两个语句中. 这种重复是合法的: err 由第一个语句声明, 但仅在第二个语句中重新分配.  这意味着对 f.Stat 的调用使用上面声明的现有 err 变量, 并为其赋予一个新值. 

In a `:=` declaration a variable `v` may appear even if it has already been declared, provided:

 在 := 声明中, 即使已经声明了变量 v ,它也可能出现, 前提是:

this declaration is in the same scope as the existing declaration of `v` (if `v` is already declared in an outer scope, the declaration will create a new variable §), the corresponding value in the initialization is assignable to v, and there is at least one other variable that is created by the declaration.

该声明与现有声明的范围相同(如果在外部范围中已声明v,则该声明将创建一个新的变量), 初始化中的相应值可分配给 v, 并且声明创建了至少一个其他变量.  [例如, 上述的 d, err := ]

This unusual property is pure pragmatism, making it easy to use a single `err` value, for example, in a long `if-else` chain. You'll see it used often.

这种不寻常的特性是纯粹的实用主义, 例如, 在一个长if-else链中很容易使用一个错误值. 你会看到它经常被使用. 

§ It's worth noting here that in Go the scope of function parameters and return values is the same as the function body, even though they appear lexically outside the braces that enclose the body.

这里值得注意的是, 在Go中, 函数参数和返回值的作用域与函数体相同, 即使它们在词汇上出现在包围函数体的大括号之外. 

##### For

The Go `for` loop is similar to—but not the same as—C's. It unifies `for` and `while` and there is no `do-while`. There are three forms, only one of which has semicolons.

Go for循环与C类似, 但不相同. 它统一了for和while, 没有do while. 有三种形式, 其中只有一种带有分号. 

~~~go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
~~~

Short declarations make it easy to declare the index variable right in the loop.

简短的声明可以很容易地在循环中声明索引变量. 

~~~go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
~~~

If you're looping over an array, slice, string, or map, or reading from a channel, a `range` clause can manage the loop. [range 用于数组、切片、映射、通道、字符串]

~~~go
for key, value := range oldMap {
    newMap[key] = value
}
~~~

If you only need the first item in the range (the key or index), drop the second:

~~~go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
~~~

If you only need the second item in the range (the value), use the *blank identifier* [空白标识符], an underscore[下划线], to discard[丢弃] the first: 

~~~go
sum := 0
for _, value := range array {
    sum += value
}
~~~

For strings, the `range` does more work for you, breaking out individual Unicode code points by parsing the UTF-8. Erroneous encodings consume one byte and produce the replacement rune U+FFFD. (The name (with associated builtin type) `rune` is Go terminology for a single Unicode code point. See [the language specification](https://go.dev/ref/spec#Rune_literals) for details.) The loop

对于字符串, range更适合你, 通过解析UTF-8来分解单个Unicode代码点. 错误编码消耗一个字节并产生替换符文U+FFFD. (内建类型)rune是单个Unicode代码点的Go术语. 有关详细信息, 请参见语言规范. 如下: 

~~~go
for pos, char := range "中国\x80话" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
~~~

Finally, Go has no comma operator and `++` and `--` are statements not expressions. Thus if you want to run multiple variables in a `for` you should use parallel assignment (although that precludes `++` and `--`).

最后, Go没有逗号运算符, ++和--是语句而不是表达式(单独一行出现). 因此, 如果您想在for中运行多个变量, 应该使用并行赋值（尽管这排除了++和--）. 

~~~go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
~~~

##### Switch

Go's `switch` is more general than C's. The expressions need not be constants or even integers, the cases are evaluated top to bottom until a match is found, and if the `switch` has no expression it switches on `true`. It's therefore possible—and idiomatic—to write an `if`-`else`-`if`-`else` chain as a `switch`.

Go的switch比C的更通用。表达式不需要是常量或甚至整数，从上到下对大小写进行求值，直到找到匹配为止，如果switch没有表达式，则选择为true。因此，编写if-else-if-else链作为switch是可能的，也是惯用的。

~~~go
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
~~~

There is no automatic fall through, but cases can be presented in comma-separated lists.

~~~go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
~~~

Although they are not nearly as common in Go as some other C-like languages, `break` statements can be used to terminate a `switch` early. Sometimes, though, it's necessary to break out of a surrounding loop, not the switch, and in Go that can be accomplished by putting a label on the loop and "breaking" to that label. This example shows both uses.

~~~go
Loop:
    for n := 0; n < len(src); n += size {
        switch {
        case src[n] < sizeOne:
            if validateOnly {
                break
            }
            size = 1
            update(src[n])

        case src[n] < sizeTwo:
            if n+1 >= len(src) {
                err = errShortInput
                break Loop
            }
            if validateOnly {
                break
            }
            size = 2
            update(src[n] + src[n+1]<<shift)
        }
    }
~~~

Of course, the `continue` statement also accepts an optional label but it applies only to loops.

To close this section, here's a comparison routine for byte slices that uses two `switch` statements:

~~~go
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
~~~

##### Type switch

A switch can also be used to discover the dynamic type of an interface variable. Such a *type switch* uses the syntax of a type assertion with the keyword `type` inside the parentheses. If the switch declares a variable in the expression, the variable will have the corresponding type in each clause. It's also idiomatic to reuse the name in such cases, in effect declaring a new variable with the same name but a different type in each case.

~~~go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
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

> 1.  Go的 `defer` 语句用于预设一个函数调用, 该函数会在执行 `defer` 的函数返回之前立即执行
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
2.  `init` 函数还常被用在程序真正开始执行前, 检验或校正程序的状态
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

不要通过共享内存来通信, 而应该通过通信来共享内存

**goroutine**

它是与其它Go程并发运行在同一地址空间的函数, 它是轻量级的,  所有消耗几乎就只有栈空间的分配, 而且栈最开始是非常小的, 所以它们很廉价,  仅在需要时才会随着堆空间的分配(和释放)而变化

Go程在多线程操作系统上可实现多路复用, 因此若一个线程阻塞,  那么其它的线程就会运行

**通道**

1.   `make` 来分配内存

~~~go
ci := make(chan int)            // 整数类型的无缓冲信道
cj := make(chan int, 0)         // 整数类型的无缓冲信道
cs := make(chan *os.File, 100)  // 指向文件指针的带缓冲信道
~~~

2.  无缓冲信道在通信时会同步交换数据, 它能确保（两个Go程的）计算处于确定状态

3.  若信道是带缓冲的, 则发送者仅在值被复制到缓冲区前阻塞, 若缓冲区已满, 发送者会一直等待直到某个接收者取出一个值为止


#### 错误

error

按照约定, 错误的类型通常为 `error`, 这是一个内建的简单接口. 

```go
type error interface {
	Error() string
}
```

panic

不可恢复的错误

~~~go
panic("something wrong")
~~~

recover

当 `panic` 被调用后(包括不明确的运行时错误, 例如切片检索越界或类型断言失败),  程序将立刻终止当前函数的执行, 并开始回溯Go程的栈, 运行任何被推迟的函数.  若回溯到达Go程栈的顶端, 程序就会终止. 不过我们可以用内建的 `recover` 函数来重新或来取回Go程的控制权限并使其恢复正常执行. 

~~~go
defer func() {
    if err := recover(); err != nil {
        log.Println(err)
    }    
}()
~~~