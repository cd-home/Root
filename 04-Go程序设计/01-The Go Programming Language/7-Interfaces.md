[TOC]

### Interfaces

Interface types express generalizations or abstractions about the behaviors of other types. By generalizing, interfaces let us write functions that are more flexible[灵活] and adaptable[适应性] because they are not tied[绑定] to the details of one particular[特定的] implementation[实现].

接口类型表达其他类型行为的概括或者抽象. 归纳来讲, 接口让我们编写更加灵活、适应性更强的函数. 因为其不会与特定的实现细节绑定. 

Many object-oriented[面向对象] languages have some notion[概念] of interfaces, but what makes Go’s interfaces so distinctive[与众不同] is that they are **satisfied** **implicitly**[满足隐式]. In other words, there’s no need to declare all the interfaces that a given **concrete type**[具体类型] satisfies; simply possessing the necessary methods is enough. This design lets you create new interfaces that are satisfied by existing concrete types without changing the existing types, which is particularly useful for types defined in packages that you don’t control.

许多面向对象的语言都有一些接口的概念, 但 Go 的接口如此与众不同的原因在于它们被隐式地实现.  换句话说, 不需要声明给定具体类型满足的所有接口; 类型只需拥有必要的方法就足够了. 这种设计让您可以在不更改现有类型的情况下创建现有具体类型所满足的新接口, 这对于在您无法控制的包中定义的类型特别有用.

PS: [不需要像其他语言一样显式的声明类型实现接口]

TODO:  [创建新的接口, "满足"已有类型,  同时新类型亦可满足新接口]

In this chapter, we’ll start by looking at the basic mechanics[基本机制] of interface types and their values. Along the way, we’ll study several important interfaces from the standard library. Many Go programs make as much use of standard interfaces as they do of their own ones. Finally, we’ll look at **type assertions**[类型断言] (§7.10) and **type switches** [类型选择] (§7.13) and see how they enable a different kind of generality[通用性].

在本章中, 我们将从查看接口类型及其值的基本机制开始. 在此过程中, 我们将学习标准库中的几个重要接口.许多 Go 程序使用标准接口的次数与使用它们自己的接口一样多.  最后, 我们将看看类型断言和类型选择, 看看它们如何实现不同类型的通用性.

#### 7.1. Interfaces as Contracts 		接口作为契约

All the types we’ve looked at so far have been concrete types. A concrete type specifies[明确规定] the exact representation[表示] of its values and exposes[暴露出] the intrinsic[内部] operations of that representation, such as arithmetic[算术] for numbers, or indexing[索引], append[追加], and range[遍历] for slices. A concrete type may also provide additional[额外的] behaviors[行为] through its methods. When you have a value of a concrete type, you know exactly what it is and what you can do with it.

PS: [具体类型的属性与行为是明确的]

There is another kind of type in Go called an interface type[接口类型]. An interface is an abstract type[抽象类型]. It doesn’t expose the representation or internal structure[内部结构] of its values, or the set of basic operations they support; it reveals[显示、展示] only some of their methods. When you have a value of an interface type, you know nothing about what it is; you know only what it can do, or more precisely, what behaviors are provided by its methods.

PS: [接口类型没有暴露其值内部结构和支持的操作, 你只能知道其方法提供的行为是什么]

Throughout the book, we’ve been using two similar[相似的] functions for string formatting: `fmt.Printf`, which writes the result to the standard output (a file), and `fmt.Sprintf`, which returns the result as a string. It would be unfortunate if the hard part, formatting the result, had to be duplicated[重复] because of these superficial[表面的] differences in how the result is used. Thanks to interfaces, it does not. Both of these functions are , in effect, wrappers[包裹] around a third function, `fmt.Fprintf`, that is agnostic[不可知的] about what happens to the result it computes[计算].

PS：[fmt.Fprintf并不知道运行的结果, 这一切都是由传入的具体类型实现的方法决定]

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

The F prefix of Fprintf stands for[代表] file and indicates[间接提示] that the formatted[格式化] output should be written to the file provided as the first argument. In the Printf case, the argument, os.Stdout, is an *os.File. In the Sprintf case, however, the argument is not a file, though[尽管] it superficially resembles[从表面上看起来像] one: &buf is a pointer[指针] to a memory buffer[缓冲区] to which bytes can be written.

PS: [os.Stdout, bytes.Buffer 就是具体的类型, 并且实现了Write方法]

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
// 写入操作不得修改切片数据, 即使是临时修改
// Implementations must not retain p.
type Writer interface {
	Write(p []byte) (n int, err error)
}
~~~

The io.Writer interface defines the **contract**[合约] between Fprintf and its **callers**[调用者].  On the one hand, the contract requires that the caller[调用者] provide a value of a concrete type like *os.File or *bytes.Buffer that has a method called Write with the appropriate[恰当的] signature[签名] and behavior.  On the other hand, the contract guarantees[保证] that Fprintf will do its job given any value that satisfies[满足] the io.Writer interface. Fprintf may not assume[假定] that it is writing to a file or to memory, only that it can call Write.

io.Writer 接口定义了 Fprintf 和它的调用者之间的合约.  一方面, 合约要求调用者提供一个具体类型的值, 如 *os.File 或 *bytes.Buffer, 该值有一个带有恰当签名和行为被叫做 Write 的方法. 另一方面, 合约保证 Fprintf 将在给定任何满足 io.Writer 接口的值的情况下完成其工作. Fprintf 可能不会假设它正在向文件或内存写入, 仅是可以调用Write. 

PS: [只要传入的具体类型的值满足接口, 那么接口就会正常'工作',  细节由具体类型的方法实现]

Because `fmt.Fprintf` assumes nothing about the representation of the value and relies[依赖] only on the behaviors guaranteed by the io.Writer contract, we can safely pass a value of any concrete type that satisfies io.Writer as the first argument to `fmt.Fprintf`. This freedom[自由] to substitute[替换] one type for another that satisfies the same interface is called substitutability[可替换性], and isa hallmark[特点] of object-oriented[面向对象] programming.

PS: [对于传入的具体类型的值不会做限定, 只要其满足接口, 就可以安全的传递]

Let’s test this out using a new type. The Write method of the *ByteCounter type below merely[仅仅] counts the bytes written to it before discarding[丢弃] them. (The conversion is required to make the types of len(p) and *c match in the += assignment statement.)

~~~go
type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
    // convert int to ByteCounter
    *c += ByteCounter(len(p))
    return len(p), nil
}
~~~

Since *ByteCounter satisfies[满足] the io.Writer contract, we can pass it to Fprintf, which does its string formatting oblivious[未察觉] to this change; the ByteCounter correctly accumulates the length of the result.

~~~go
func main() {
    var c ByteCounter
    c.Write([]byte("hello"))
    fmt.Println(c)  			// "5", = len("hello")
    c = 0 						// reset the counter
    var name = "Dolly"
    fmt.Fprintf(&c, "hello, %s", name)
    fmt.Println(c)  			// "12", = len("hello, Dolly")
}
~~~

Besides io.Writer, there is another interface of great importance to the fmt package. Fprintf and Fprintln provide a way for types to control how their values are print ed. In Section 2.5, we defined a String method for the Celsius type so that temperatures would print as "100°C", and in Section 6.5 we equipped *IntSet with a String method so that sets would be rendered using traditional set notation[符号] like "{1 2 3}". Declaring a String method makes a type satisfy on e of the most widely[广泛的] used interfaces of all, fmt.Stringer:

~~~go
package fmt
// The String method is used to print values passed
// as an operand to any format that accepts a string
// or to an unformatted printer such as Print.
type Stringer interface {
	String() string
}
~~~

We’ll explain how the fmt package discovers[发现] which values satisfy this interface in Section 7.10.

**Exercise 7.1:** 

Using the ideas from ByteCounter, implement counters for words and for lines. You will find bufio.ScanWords useful.

~~~go
type WordCount int

func (wc *WordCount) Write(p []byte) (n int, err error) {
	scanner := bufio.NewScanner(bytes.NewReader(p))
    // bufio.ScanLines
	scanner.Split(bufio.ScanWords)
	for scanner.Scan() {
		*wc++
	}
	return len(p), nil
}

func TestExercise71(t *testing.T) {
	var wc WordCount
	wc.Write([]byte("Hello"))
	fmt.Println(wc)

	wc = 0
	name := "Dolly"
	fmt.Fprintf(&wc, "Hello %s", name)
	fmt.Println(wc)
}
~~~

PS: [bufio 以及 bytes 库的使用]

Exercis e 7.2:

Write a function CountingWriter with the signature below that, given an io.Writer, returns a new Writer that wraps the original, and a pointer to an int64 variable that at any moment contains the number of bytes written to the new Writer.

~~~go
func CountingWriter(w io.Writer) (io.Writer, *int64)
~~~

Answer

~~~go
type WrapperWriter struct {
	writer  io.Writer
	counter int64
}

func (w *WrapperWriter) Write(p []byte) (n int, err error) {
	n, err = w.writer.Write(p)
	w.counter += int64(n)
	return n, err
}

// CountingWriter
func CountingWriter(w io.Writer) (io.Writer, *int64) {
	newWriter := &WrapperWriter{writer: w}
	return newWriter, &newWriter.counter
}

func TestExercise72(t *testing.T) {
	writer, counter := CountingWriter(os.Stdout)
	fmt.Fprintf(writer, "Hello\n")
	fmt.Println(*counter)
	fmt.Fprintf(writer, "World\n")
	fmt.Println(*counter)
}
~~~

**Exercis e 7.3:**  TODO

Write a String method for the *tree type in gopl.io/ch4/treesort (§4.4) that reveals the sequence of values in the tree.

~~~go
~~~

#### 7.2 Interface Types 						接口类型

