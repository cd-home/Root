[TOC]

### Language Mechanics 

#### 2.1 Built-in Types(内建类型) 

1. Types provide integrity and readability by asking 2 questions: 
    
    类型通过两个问题提供**完整性**和**可读性**

    - [ ] What is the amount of **memory** to **allocate**? (e.g. 1, 2, 4, 8 bytes) 
    
        **分配**的**内存**大小
    
    - [ ] What does that memory represent? (e.g. int, uint, bool,..) 
    
        内存的外在表现形式(在程序中看到的是什么表现形式)
    
2. Types can be specific to a precision such as int32 or int64: 
    
    类型有特定的精度（类型修饰、不同的平台不同的大小）
    
    - [ ] uint8 represents an unsigned integer with 1 byte of allocation  
    
        uint8一个字节的无符号整型
    
    - [ ] int32 represents a signed integer with 4 bytes of allocation  
    
        int32四个字节的有符号整型

When I declare a type using a non-precision based type (unit, int) the size of the values constructed for these types are based on the **architecture** being used to build 

当我用一个无精读的类型声明一个类型时, 为这些类型构造的值的大小基于用于构建的**体系结构** 

the program: 

- [ ] 32 bit arch: int represents a signed int at 4 bytes of memory allocation , 32位架构, int类型 4字节内存分配
- [ ] 64 bit arch: int represents a signed int at 8 bytes of memory allocation , 64位架构, int类型 8字节内存分配

#### 2.2 Word Size(字大小)

The word size represents the amount of memory allocation required to store integers and pointers for a given architecture.

字大小表示存储给定体系结构的整数和指针所需的内存分配量

 For example: 

- [ ] 32 bit arch: word size is 4 bytes of memory allocation  32位架构, 字大小 4字节内存分配
- [ ]  64 bit arch: word size is 8 bytes of memory allocation 64位架构, 字大小 8字节内存分配

This is important because Go has **internal data structures** (slices, interfaces) that store integers and pointers.The size of these data structures will be based on the architecture being used to build the program. 

这很重要,因为Go具有存储整数和指针的**内部数据结构**(切片、接口).这些数据结构的大小将基于用于构建程序的架构.

In Go, the amount of memory allocated for a value of type int, a pointer, or a word data, will always be the same 

在Go中，为int类型的值、指针或字数据分配的内存量将始终相同

#### 2.3 Zero Value Concept(零值概念)

Every single value I construct in Go is initialized at least to its zero value state unless I specify the initialization value at construction. The zero value is the setting of every bit in every byte to zero.

除非我在构造时指定初始化值, 否则我在Go中构造的每个值都至少初始化为其零值状态, 零值是将每个字节中的每个位设置为零.

This is done for **data integrity** and it’s not free. It takes time to push electrons through the machine to reset those bits, but I should always take integrity over performance.  

这样做是为了**数据完整性** , 而不是免费的. 推动电子通过机器重置这些位需要时间, 但我应该始终保持完整性而不是性能.

~~~go
 Type		Zero Value
Integer			0
Boolean			false
Floating		0
Complex			0i
String			""
Pointer			nil
~~~

#### 2.4 Declare and Initialize(声明和初始化) 

The keyword var can be used to construct values for all types to their zero value state. 

关键字var可用于将所有类型的值构造为其零值状态.

~~~go
var a int
var b string
var c float64
var d bool
fmt.Printf("var a int \t %T [%v]\n", a, a) 
fmt.Printf("var b string \t %T [%v]\n", b, b) 
fmt.Printf("var c float64 \t %T [%v]\n", c, c) 
fmt.Printf("var d bool \t %T [%v]\n\n", d, d)
~~~

Strings use the UTF8 **character set**, but are really just a collection of bytes. 

字符串使用UTF8**字符集**, 但实际上只是字节的集合.

A string is a two-word internal data structure in Go: 

字符串是Go中的两个字的内部数据结构：

- [ ] The first word represents a pointer to a backing array of bytes 

    第一个字表示指向"backing"字节数组的指针

- [ ] The second word represents the length or the number of bytes in the backing array 

    第二个字表示"backing"数组中的长度或字节数

If the string is set to its zero value state, then the first word is nil and the second word is 0. 

如果字符串设置为零值状态，则第一个字为nil, 第二个字为0. 

源码补充

~~~go
type stringStruct struct {
    str unsafe.Pointer
    len int
}
func gostringnocopy(str *byte) string { 
    ss := stringStruct{str: unsafe.Pointer(str), len: findnull(str)} 
    s := *(*string)(unsafe.Pointer(&ss))
    return s
}
~~~

Using the short variable declaration operator, I can declare, construct, and initialize a value all at the same time. 

使用短变量声明操作符, 我可以同时声明、构造和初始化一个值.

~~~go
aa := 10 // int [10] 
bb := "hello" // string [hello] 
cc := 3.14159 // float64 [3.14159] 
dd := true // bool [true] 
fmt.Printf("aa := 10 \t %T [%v]\n", aa, aa)
fmt.Printf("bb := \"hello\" \t %T [%v]\n", bb, bb) 
fmt.Printf("cc := 3.14159 \t %T [%v]\n", cc, cc) 
fmt.Printf("dd := true \t %T [%v]\n\n", dd, dd)
~~~

#### 2.5 Conversion vs Casting(转换 VS 投影) 

Go doesn't have **casting**, but **conversion**. Instead of telling the compiler to map a set of bytes to a different representation, the bytes need to be copied to a new memory location for the new representation. 

Go没有**投影**，只有**转换**。需要将字节复制到新表示的新内存位置，而不是告诉编译器将一组字节映射到不同的表示。

~~~go
aaa := int32(10) 
fmt.Printf("aaa := int32(10) %T [%v]\n", aaa, aaa)
~~~

#### 2.6 Struct and Construction Mechanics(结构定义)

The declaration represents a concrete **user defined type** with a composite of different fields.

该声明表示一个具体的**用户定义类型**, 包含不同字段的组合.

~~~go
type example struct { 
    flag bool 
    counter 
    int16 
    pi float32 
}
~~~

Declare a variable of type example and initialize it to its zero value state. 

声明example类型的变量并将其初始化为零值状态.

~~~go
var e example
fmt.Printf("%+v\n", e1)
~~~

Declare a variable of type example not set to its zero value state by using **literal construction syntax**. 

通过使用**字面构造语法**声明类型为example的变量不设置为其零值状态

~~~go
e2 := example{ 
    flag: true, 
    counter: 10, 
    pi: 3.141592,
}
fmt.Println("Flag", e2.flag) 
fmt.Println("Counter", e2.counter) 
fmt.Println("Pi", e2.pi)
~~~

Declare a variable of an unnamed literal type set to its non-zero value state using literal construction syntax. This is a one-time thing. 

使用字面构造语法将未命名文字类型的变量声明为其非零值状态. 这是一次性的.

~~~go
e3 := struct { 
    flag bool 
    counter int16 
    pi float32 
}{ 
    flag: true,
    counter: 10,
    pi: 3.141592, 
}
fmt.Println("Flag", e3.flag) 
fmt.Println("Counter", e3.counter) 
fmt.Println("Pi", e3.pi)
~~~

#### 2.7 Padding and Alignment(填充和对齐)



