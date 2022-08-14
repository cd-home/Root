[TOC]

### Interfaces

Interface types express generalizations or **abstractions** about the behaviors of other types. 

接口类型表达其他类型行为的概括或者抽象.

By generalizing, interfaces let us write functions that are more flexible and adaptable because they are not tied to the details of one particular implementation.

归纳来讲, 接口让我们编写更加灵活、适应性更强的函数. 因为其不会与特定的实现细节绑定. 

Many object-oriented languages have some notion of interfaces, but what makes Go’s interfaces so **distinctive** is that they are **satisfied** **implicitly**.

许多面向对象的语言都有一些接口的概念, 但 Go 的接口如此与众不同的原因在于它们被隐式地实现. 

PS: [不需要像其他语言一样显式的声明类型实现接口]

In other words, there’s no need to declare all the interfaces that a given concrete type satisfies; simply possessing the necessary methods is enough.

换句话说, 不需要声明给定具体类型满足的所有接口; 类型只需拥有必要的方法就足够了. 

This design lets you create new interfaces that are satisfied by existing concrete types without changing the existing types, which is particularly useful for types defined in packages that you don’t control.

这种设计让您可以在不更改现有类型的情况下创建现有具体类型所满足的新接口, 这对于在您无法控制的包中定义的类型特别有用.

TODO:  [创建新的接口, "满足"已有类型,  同时新类型亦可满足新接口]

In this chapter, we’ll start by looking at the basic mechanics[基本机制] of interface types and their values. Along the way, we’ll study several important interfaces from the standard library. Many Go programs make as much use of standard interfaces as they do of their own ones. Finally, we’ll look at type assertions[类型断言] (§7.10) and type switches [类型选择] (§7.13) and see how they enable a different kind of generality[通用性].

在本章中, 我们将从查看接口类型及其值的基本机制开始. 在此过程中, 我们将学习标准库中的几个重要接口.许多 Go 程序使用标准接口的次数与使用它们自己的接口一样多.  最后, 我们将看看类型断言和类型选择, 看看它们如何实现不同类型的通用性.

#### 7.1. Interfaces as Contracts 		接口作为契约

All the types we’ve looked at so far have been concrete types[具体类型]. 

A concrete type specifies the exact representation[表示] of its values and exposes the intrinsic operations of that representation, such as arithmetic for numbers, or indexing, append, and range for slices. 

A concrete type may also provide additional[额外的] behaviors[行为] through its methods. When you have a value of a concrete type, you know exactly what it is and what you can do with it.

There is another kind of type in Go called an interface type. 

An interface is an abstract type. It doesn’t expose[暴露出、公开] the representation or internal structure[内部结构] of its values, or the set of basic operations they support; it reveals[显示、展示] only some of their methods. When you have a value of an interface type, you know nothing about what it is; you know only what it can do, or more precisely, what behaviors are provided by its methods.

PS: [接口类型没有暴露其值内部结构, 你只能知道其方法提供的行为]

Throughout the book, we’ve been using two similar functions for string formatting: 

fmt.Printf, which writes the result to the standard output (a file), and fmt.Sprintf, which returns the result as a string. 

It would be unfortunate if the hard part, formatting the result, had to be duplicated[重复] because of these superficial[表面的] differences in how the result is used.

Thanks to interfaces, it does not. Both of these functions are , in effect, wrappers around a third function, fmt.Fprintf, that is agnostic[不可知的] about what happens to the result it computes.

~~~go
package fmt

// Fprintf formats according to a format specifier and writes to w.
// It returns the number of bytes written and any write error encountered.
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
	p := newPrinter()
	p.doPrintf(format, a)
	n, err = w.Write(p.buf)
	p.free()
	return
}

// Printf formats according to a format specifier and writes to standard output.
// It returns the number of bytes written and any write error encountered.
func Printf(format string, args ...interface{}) (int, error) { 
	return Fprintf(os.Stdout, format, args...)
}

// Sprintf formats according to a format specifier and 
// returns the resulting string.
func Sprintf(format string, args ...interface{}) string {
	var buf bytes.Buffer
	Fprintf(&buf, format, args...)
	return buf.String()
}
~~~

The F prefix of Fprintf stands for[代表] file and indicates[间接提示] that the formatted[格式化] output should be written to the file provided as the first argument. 

In the Printf case, the argument, os.Stdout, is an *os.File. 

In the Sprintf case, however, the argument is not a file, though[尽管] it superficially resembles[从表面上看起来像] one: &buf is a pointer to a memory buffer to which bytes can be written.

The first parameter of Fprintf is not a file either. It’s an io.Writer, which is an interface type with the following declaration[声明]:

》io.go

~~~go
// Writer is the interface that wraps the basic Write method.
//
// Write writes len(p) bytes from p to the underlying data stream.
// It returns the number of bytes written from p (0 <= n <= len(p))
// and any error encountered that caused the write to stop early.
// Write must return a non-nil error if it returns n < len(p).
// Write must not modify the slice data, even temporarily.
//
// Implementations must not retain p.
type Writer interface {
	Write(p []byte) (n int, err error)
}
~~~

The io.Writer interface defines the contract[合约] between Fprintf and its callers[调用者].

io.Writer 接口定义了 Fprintf 和它的调用者之间的合约.  

 On the one hand, the contract requires that the caller[调用者] provide a value of a concrete type like *os.File or *bytes.Buffer that has a method called Write with the appropriate[恰当的] signature[签名] and behavior. 

一方面, 合约要求调用者提供一个具体类型的值, 如 *os.File 或 *bytes.Buffer, 该值有一个带有适当签名和行为名为 Write 的方法. 

On the other hand, the contract guarantees[保证] that Fprintf will do its job given any value that satisfies[满足] the io.Writer interface. Fprintf may not assume[假定] that it is writing to a file or to memory, only that it can call Write.

另一方面, 合约保证 Fprintf 将在给定任何满足 io.Writer 接口的值的情况下完成其工作. Fprintf可能不会假设它正在向文件或内存写入, 仅是可以调用Write. 

Because `fmt.Fprintf` assumes nothing about the representation of the value and relies[依赖] only on the behaviors guaranteed by the io.Writer contract, we can safely pass a value of any concrete type that satisfies io.Writer as the first argument to `fmt.Fprintf`. This freedom[自由] to substitute[替换] one type for another that satisfies the same interface is called substitutability[可替换性], and isa hallmark[特点] of object-oriented[面向对象] programming.

Let’s test this out using a new type. The Write method of the *ByteCounter type below merely counts the bytes written to it before discarding[丢弃] them. (The conversion is required to make the types of len(p) and *c match in the += assignment statement.)

~~~go
type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
    *c = ByteCounter(len(p))
    return len(p), nil
}
~~~

Since *ByteCounter satisfies the io.Writer contract, we can pass it to Fprintf, which does its string formatting oblivious[未察觉] to this change; the ByteCounter correctly accumulates the length of the result.

PS: [接口只关注其方法, 具体的实现是由具体的类型]

