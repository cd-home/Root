[TOC]

### Language Mechanics 

#### 2. 1 Built-in Types																											(内建类型) 

1. Types provide **integrity** and **readability** by asking 2 questions: 
   
    类型通过两个问题提供**完整性**和**可读性**

    - [ ] What is the amount of **memory** to **allocate**? (e. g.  1, 2, 4, 8 bytes) 
    
        **分配**的**内存**大小
    
    - [ ] What does that memory **represent**? (e. g.  int, uint, bool,. . ) 
    
        内存的外在表现形式(在程序中看到的是什么**表现形式**)
    
2. Types can be specific to a precision such as int32 or int64: 
   
    类型有特定的精度（无符号、有符号, 不同的平台占用不同的大小）
    
    - [ ] uint8 represents an **unsigned integer** with 1 byte of allocation  
    
        uint8一个字节的**无符号整型**
    
    - [ ] int32 represents a **signed intege**r with 4 bytes of allocation  
    
        int32四个字节的**有符号整型**

When I declare a type using a non-precision based type (unit, int) the size of the values constructed for these types are based on the **architecture** being used to build 

当我用一个无精度的类型声明一个类型时, 为这些类型构造的值的大小基于用于构建的**体系结构** 

the program: 

- [ ] 32 bit arch: int represents a signed int at 4 bytes of memory allocation 

    **32位架构, int类型 4字节内存分配**

- [ ] 64 bit arch: int represents a signed int at 8 bytes of memory allocation 

    **64位架构, int类型 8字节内存分配**

**扩展说明 类型尺寸**

|              Type              |    Size of Bytes     |                     |
| :----------------------------: | :------------------: | ------------------- |
|              bool              |          1           |                     |
|        uint8,byte,int8         |          1           |                     |
|          uint16,int16          |          2           |                     |
|      uint32,int32,float32      |          4           |                     |
| Uint64,int64,float64,complex64 |          8           |                     |
|           complex128           |          16          |                     |
|            int,uint            |        1 word        | 32-4Bytes 64-8Bytes |
|            uintptr             |        1 word        |                     |
|             string             |       2 words        |                     |
|            pointer             |        1 word        |                     |
|             slice              |       3 words        |                     |
|              map               |        1 word        |                     |
|            channle             |        1 word        |                     |
|              func              |        1 word        |                     |
|           interface            |        1 word        |                     |
|             struct             | All Fields + Padding | Empty struct = 0    |
|             array              |   Element * length   | Empty array  = 0    |

+++

#### 2. 2 Word Size																													(字大小)

The word size represents the amount of memory allocation required to store integers and pointers for a given architecture. 

字大小 表示 存储给定体系结构的 **整数** 和 **指针** 所需的内存分配量

 For example: 

- [ ] 32 bit arch: word size is 4 bytes of memory allocation  32位架构, 内存分配4字节字大小 
- [ ]  64 bit arch: word size is 8 bytes of memory allocation 64位架构, 内存分配8字节字大小 

This is important because Go has **internal data structures** (slices, interfaces) that store integers and pointers. The size of these data structures will be based on the architecture being used to build the program.  

这很重要,因为Go具有存储整数和指针的**内部数据结构**(切片、接口). 这些数据结构的大小将基于用于构建程序的架构. 

In Go, the amount of memory allocated for a value of type int, a pointer, or a word data, will always be the same 

**在Go中, 为int类型的值、指针或字数据分配的内存量将始终相同**

+++

#### 2. 3 Zero Value Concept																								(零值概念)

Every single value I construct in Go is initialized at least to its zero value state unless I specify the initialization value at construction.  The zero value is the setting of every bit in every byte to zero. 

除非我在构造时指定初始化值, 否则我在Go中构造的每个值都至少初始化为其零值状态, 零值是将每个字节中的每个位设置为零. 

This is done for **data integrity** and it’s not free.  It takes time to push electrons through the machine to reset those bits, but I should always take integrity over performance.   

这样做是为了**数据完整性** ,并且不是无消耗的.  通过机器推动电子重置这些位需要时间, 但我应该始终保持完整性而不是性能. 

~~~go
 Type		 Zero Value
Integer			0
Boolean			false
Floating		0
Complex			0i
String			""
Pointer			nil
~~~

+++

#### 2. 4 Declare and Initialize																							(声明和初始化) 

The keyword var can be used to construct values for all types to their zero value state.  

关键字var可用于将所有类型的值构造为其零值状态. 

**Listing 2. 4. 1**

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

一个字符串在Go中是两个字的内部数据结构：

- [ ] The first word represents a pointer to a backing array of bytes 

    第一个字表示指向"backing"字节数组的指针

- [ ] The second word represents the length or the number of bytes in the backing array 

    第二个字表示"backing"数组中的长度或字节数

If the string is set to its zero value state, then the first word is nil and the second word is 0.  

如果字符串设置为零值状态, 则第一个字为nil, 第二个字为0.  

**源码补充**

~~~go
type stringStruct struct {
    str unsafe. Pointer
    len int
}
func gostringnocopy(str *byte) string { 
    ss := stringStruct{str: unsafe. Pointer(str), len: findnull(str)} 
    s := *(*string)(unsafe. Pointer(&ss))
    return s
}
~~~

Using the short variable declaration operator, I can declare, construct, and initialize a value all at the same time.  

使用短变量声明操作符, 我可以同时声明、构造和初始化一个值. 

**Listing 2. 4. 2**

~~~go
aa := 10 			// int [10] 
bb := "hello"   	// string [hello] 
cc := 3. 14159   	// float64 [3. 14159] 
dd := true      	// bool [true] 
fmt.Printf("aa := 10 \t %T [%v]\n", aa, aa)
fmt.Printf("bb := \"hello\" \t %T [%v]\n", bb, bb) 
fmt.Printf("cc := 3. 14159 \t %T [%v]\n", cc, cc) 
fmt.Printf("dd := true \t %T [%v]\n\n", dd, dd)
~~~

+++

#### 2. 5 Conversion vs Casting																							(转换 VS 投影) 

Go doesn't have **casting**, but **conversion**.  Instead of telling the compiler to map a set of bytes to a different representation, the bytes need to be copied to a new memory location for the new representation.  

Go没有**投影**, 只有**转换**.  **需要将字节复制到新表示的新内存位置**, 而不是告诉编译器将一组字节映射到不同的表示.  

**Listing 2. 5. 1**

~~~go
aaa := int32(10) 
fmt.Printf("aaa := int32(10) %T [%v]\n", aaa, aaa)
~~~

#### 2. 6 Struct and Construction Mechanics																(结构定义)

The declaration represents a concrete **user defined type** with a composite of different fields. 

该声明表示一个具体的**用户定义类型**, 包含不同字段的组合. 

**Listing 2. 6. 1**

~~~go
type example struct { 
    flag 	bool 
    counter int16 
    pi 		float32 
}
~~~

Declare a variable of type example and initialize it to its zero value state.  

声明example类型的变量并将其初始化为零值状态. 

**Listing 2. 6. 2**

~~~go
var e example
fmt.Printf("%+v\n", e1)
~~~

Declare a variable of type example not set to its zero value state by using **literal construction syntax**.  

通过使用**字面构造语法**声明类型为example的变量不设置为其零值状态

**Listing 2. 6. 3**

~~~go
e2 := example{ 
    flag: true, 
    counter: 10, 
    pi: 3. 141592,
}
fmt.Println("Flag", e2. flag) 
fmt.Println("Counter", e2. counter) 
fmt.Println("Pi", e2. pi)
~~~

Declare a variable of an unnamed literal type set to its non-zero value state using literal construction syntax.  This is a one-time thing.  

使用字面构造语法将未命名字面类型的变量声明为其非零值状态.  这是一次性的. 

**Listing 2. 6. 4**

~~~go
e3 := struct { 
    flag 	bool 
    counter int16 
    pi 		float32 
}{ 
    flag: true,
    counter: 10,
    pi: 3. 141592, 
}
fmt.Println("Flag", e3. flag) 
fmt.Println("Counter", e3. counter) 
fmt.Println("Pi", e3. pi)
~~~

+++

#### 2. 7 Padding and Alignment																						(填充和对齐)

How much memory is allocated for a value of type example1? 

为example1类型的值分配了多少内存？

**Listing 2. 7. 1**

~~~go
type example1 struct { 
    flag 	bool 
    counter int16 
    pi 		float32 
}
~~~

A bool is 1 byte, int16 is 2 bytes, float32 is 4 bytes.  Add that all together and I get 7 bytes.  

bool是1字节, int16是2字节, float32是4字节.  把这些加起来, 我得到7个字节

However, the actual answer is 8 bytes.  Why, because there is a **padding** byte sitting between the flag and counter fields for the reason of **alignment**.  

然而, 实际答案是8字节.  为什么, 因为由于**对齐**, flag和counter字段之间有一个**填充**字节.  

<img src="./images/struct_aligment.svg" alt="struct_aligment" style="zoom:150%;" />

The idea of alignment is to allow the hardware to read memory more efficiently by placing memory on specific alignment boundaries.  The compiler takes care of the alignment boundary mechanics so I don’t have to. 

对齐的思想是通过将内存放置在特定的对齐边界上, **使硬件能够更高效地读取内存**.  编译器负责对齐边界机制, 因此我不必这样做.  

Depending on the size of a particular field, Go determines the alignment I need

根据特定字段的大小, Go确定我需要的对齐方式

**Listing 2. 7. 2**

~~~go
type example2 struct { 
	flag 	bool 
    counter int16 
    flag2 	bool 
    pi 		float32 
}
~~~

In this example, I’ve added a new field called flag2 between the counter and pi fields.  This causes more padding inside the struct

在本例中, 我在counter和pi字段之间添加了一个名为flag2的新字段.  这会在结构内部产生更多的填充字节

**Listing 2. 7. 3**

~~~go
type example3 struct { 
    flag 	bool 	  // 0xc000100020 <- Starting Address 
         	byte 	  // 0xc000100021 <- 1 byte padding 
    counter int16     // 0xc000100022 <- 2 byte alignment 
    flag2 	bool      // 0xc000100024 <- 1 byte alignment 
          	byte      // 0xc000100025 <- 1 byte padding 
          	byte      // 0xc000100026 <- 1 byte padding 
          	byte      // 0xc000100027 <- 1 byte padding 
    pi 		float32   // 0xc000100028 <- 4 byte alignment 
}
~~~

This is how the alignment and padding play out if I pretend a value of type example2 starts at address 0xc000100020.  The flag field represents the starting address and is only 1 byte in size.  Since the counter field requires 2 bytes of allocation, it must be placed in memory on a 2-byte alignment, meaning it needs to fall on an address that is a multiple of 2.  This means the counter field must start at address 0xc000100022.  This creates a 1-byte gap between the flag and counter fields

如果我假设example2类型的值从地址0xc00000100020开始, 那么对齐和填充就是这样进行的.  flag字段表示起始地址, 大小仅为1字节.  由于counter字段需要2字节的分配, 因此它必须以2字节对齐方式放置在内存中, 这意味着它需要位于2的倍数地址上.  这意味着计数器字段必须从地址0xc000100022开始.  这将在标志字段和计数器字段之间创建1字节的间隙

<img src="./images/struct_alignment2.svg" alt="struct_alignment2" style="zoom:150%;" />

The flag2 field is a bool and can fall at the next address 0xc000100024.  The final field is pi and requires 4 bytes of allocation so it needs to fall on a 4-byte alignment.  The next address for a 4 byte value is at 0xc000100028.  That means 3 more padding bytes are needed to maintain a proper alignment.  This results in a value of type example2 requiring 12 bytes of total memory allocation.  

flag2字段为布尔值, 可位于下一个地址0xc000100024.  最后一个字段是pi, 需要4个字节的分配, 因此需要采用4字节的对齐方式.  4字节值的下一个地址为0xc000100028.  这意味着还需要3个填充字节来保持正确的对齐.  这导致example2类型的值需要12字节的总内存分配.  

The largest field in a struct represents the alignment boundary for the entire struct.  In this case, the largest field is 4 bytes so the starting address for this struct value must be a multiple of 4.  I can see the address 0xc000100020 is a multiple of 4. 

结构中最大的字段表示整个结构的对齐边界.  在这种情况下, 最大字段是4字节, 因此此结构值的起始地址必须是4的倍数.  我可以看到地址0xc000100020是4的倍数.  

If I need to minimize the amount of padding bytes, I must lay out the fields from highest allocation to smallest allocation.  This will push any necessary padding bytes down to the bottom of the struct and reduce the total number of padding bytes necessary. 

如果我需要最小化填充字节的数量, 我必须从最高分配到最小分配排列字段.  这将把任何必要的填充字节推到结构的底部, 并减少必要的填充字节总数. 

**Listing 2. 7. 4**

~~~go
type example4 struct { 
    pi float32 		// 0xc000100020 <- Starting Address 
    counter int16   // 0xc000100024 <- 2 byte alignment 
    flag bool       // 0xc000100026 <- 1 byte alignment 
    flag2 bool      // 0xc000100027 <- 1 byte alignment 
}
~~~

After the reordering of the fields, the struct value only requires 8 bytes of allocation and not 12 bytes.  Since all the fields allow the struct value to fall on a 4-byte alignment, no extra padding bytes are necessary.  

字段重新排序后, 结构值只需要8个字节的分配, 而不需要12个字节.  由于所有字段都允许结构值采用4字节对齐方式, 因此不需要额外的填充字节.  

**扩展内存对齐**

为什么需要内存对齐

- [ ] 平台原因: 不是所有的硬件平台都能访问任意地址上的任意数据的; 某些硬件平台只能在某些地址处取某些特定类型的数据, 否则抛出硬件异常; 某些硬件平台不支持未对齐的内存访问. 
- [ ] 性能原因：数据结构(尤其是栈)应该尽可能地在自然边界上对齐. 原因在于, 为了访问未对齐的内存, 处理器需要作两次内存访问; 而对齐的内存访问仅需要一次访问. 操作系统并非一个字长读取, 而是2,4,8多字长. 当CPU从存储器读数据到寄存器, 或者从寄存器写数据到存储器, IO的数据长度通常是字长

CPU访问内存是以平台字大小(64, 8Bytes | 32, 4Bytes)

内存对齐

 为保证程序顺利高效的运行, 编译器会把各种类型的数据安排到合适的地址, 并占用合适的长度

- [ ] 每种类型的对齐值就是它的对齐边界
- [ ] **内存对齐要求数据存储地址以及占用的字节数都要是它的对齐边界的倍数**

对齐边界

- [ ] 编译器(平台)对应最大对齐边界: 字长
- [ ] 数据类型对齐边界 = n * min(数据类型, 平台最大对齐边界) [n >= 1]

以上文例子来看(偏移是相对于结构体起始地址)

**Listing 2. 7. 5**

~~~go
type example2 struct { 
    flag 	bool 	  // 0xc000100020 <- Starting Address 
         	byte 	  // 0xc000100021 <- 1 byte padding 
    counter int16     // 0xc000100022 <- 2 byte alignment 
    flag2 	bool      // 0xc000100024 <- 1 byte alignment 
          	byte      // 0xc000100025 <- 1 byte padding 
          	byte      // 0xc000100026 <- 1 byte padding 
          	byte      // 0xc000100027 <- 1 byte padding 
    pi 		float32   // 0xc000100028 <- 4 byte alignment 
}
~~~

字段的对齐系数(对齐边界)(类型的尺寸)

~~~go
unsafe.Alignof(example2{}.flag)
~~~

flag: 		 min(8, 1) => 1, 	 偏移为0 	 [flag]

counter:  min(8, 2) => 2, 	偏移为2	  [flag x counter counter]

flag2:      min(8, 1) => 1,        偏移为1       [flag x counter counter flag]

pi: min(8, 4) => 偏移为4, 但是已经存在5位, 所以大于5并且是4的倍数的就是8, 那么偏移就是8, 结果即是

[flag x counter counter flag2 x x x pi pi pi pi] => 12  按照理论来说是12Bytes

实际上呢？经过**Go编译器优化**, 重排字段顺序只需要8Bytes

~~~go
unsafe.Sizeof(example2{}) ==> 8
~~~

注意如下问题, 空结构体不占内存空间, 那是否该S结构分配32Bytes?

**Listing 2. 7. 6**

~~~go
type S struct {
	A uint32
	B uint64
	C uint64
	D uint64
	E struct{}
}
~~~

答案是40Bytes. 

实际上结构体尾部size为0的字段会被分配内存空间进行填充, **原因是如果不给它分配内存, 该字段指针将指向一个非法的内存空间**

**回到最初. 结构体分配内存取决于各个字段的类型尺寸和由字段的排列顺序引起的填充, 但是我们并不需要主动的进行内存对齐, 这一切由编译器为我们完成.** 

#### 2.8 Assigning Values 																										(赋值)

If I have two different named types that are identical in structure, I can't assign a value of one to the other. 

如果我有两个结构完全相同的不同命名的类型, 我不能无法将一个值赋给另一个.

For example, if the types example1 and example2 are declared using the same exact declaration and we initialize two variables

例如, 如果example1和example2类型(见上文)是使用相同的字段声明的, 并且我们用两个变量初始化

**Listing 2. 8. 1**

~~~go
var ex1 example1
var ex2 example2
~~~

I can’t assign these two variables to each other since they are of different named  types. The fact that they are identical in structure is irrelevant. 

我无法将这两个变量彼此赋值, 因为它们的命名类型不同. 它们在结构上完全相同这一事实与此无关

**Listing 2. 8. 2**

~~~go
ex1 = ex2 // Not Allowed compiler error
~~~

To perform any assignment, I would have to use conversion syntax and since they are identical in structure, the compiler will allow this. 

要执行任何赋值, 我必须使用转换语法, 因为它们在结构上是相同的, 编译器将允许这样做

**Listing 2. 8. 3**

~~~go
ex1 = example1(ex2) // Allowed, NO compiler error
~~~

However, if one of the variable’s (like ex2) was declared as an unnamed type with the same structure, no conversion syntax would be required. 

但是, 如果其中一个变量(如ex2)被声明为具有相同结构的未命名类型, 则不需要转换语法. 

~~~go
var ex2 struct { 
    flag bool 
    counter int16 
    pi float32 
}
ex1 = ex2 // Allowed, NO need for conversion syntax
~~~

The compiler will allow this assignment without the need for conversion. 

编译器将允许此赋值, 而无需进行转换. 

#### 2.9 Pointers 																													    (指针)

**Pointers** serve the purpose of **sharing**. Pointers allow me to share values across program boundaries. There are several types of **program boundaries**. The most common one is between function calls. There is also a boundary between Goroutines which I have notes for later. 

**指针**用于**共享**. **指针允许我跨程序边界共享值**. 有几种类型的**程序边界**. 最常见的是函数调用之间. Goroutines之间还有一个边界, 我稍后会有注释. 

When a Go program starts up, the Go runtime creates a Goroutine. **Every Goroutine is a separate path of execution that manages the instructions that need to be executed by the machine**. I can also think of Goroutines as lightweight application level threads because they are. Every Go program has at least 1 Goroutine called the main Goroutine.

当一个Go程序启动时, Go运行时会创建一个Goroutine. **每个Goroutine都是一个单独的执行路径, 用于管理需要由机器执行的指令**. 我还可以将Goroutine视为轻量级应用程序级线程, 因为它们就是. 每个Go程序至少有一个Goroutine, 称为main Goroutine. 

**Every Goroutine is given a block of memory, called stack memory.** The memory for the stack starts out at 2K bytes. It’s very small. Stacks can grow over time. Every time a function is called, a block of stack space is taken to help the Goroutine execute the instructions associated with that function. Each individual block of memory is called a **frame.** 

**每个Goroutine都有一个内存块, 称为堆栈内存**. 堆栈的内存从2K字节开始. 它非常小. 堆栈可以随时间增长. **每次调用函数时, 都会占用一块堆栈空间来帮助Goroutine执行与该函数相关的指令. 每个单独的内存块称为一个帧.** 

The size of a frame for a given function is calculated at **compile time**. No value can be constructed on the stack unless the compiler knows the size of that value at compile time. If the compiler doesn’t know the size of a value at compile time, the value has to be constructed on the heap. 

给定函数的帧大小是在**编译时**计算的.  除非编译器在编译时知道该值的大小, 否则不能在堆栈上构造任何值.  如果编译器在编译时不知道值的大小, 则必须在堆上构造该值. 

Stacks are self cleaning and zero value helps with the **initialization** of the stack. Every time I make a function call, and a frame of memory is blocked out, the memory for that frame is initialized, which is how the stack is self cleaning. On a function return, the memory for the frame is left alone since it’s unknown if that memory will be needed again. It would be inefficient to initialize memory on returns. 

堆栈是自清理的, 零值有助于堆栈的**初始化**.  每次我进行函数调用, 并且一帧内存被阻塞时, 该帧的内存都会被初始化, 这就是堆栈自我清理的方式.  在函数返回时, 帧的内存被单独留下, 因为不知道是否会再次需要该内存.  在返回时初始化内存效率低下. 

#### 2.10 Pass By Value																											(值传递)

All data is moved around the program by value. This means as data is being passed across program boundaries, each function or goroutine is given it’s own copy of the data. There are two types of data I work with, the value itself (int, string, user) or the value's address. Addresses are data that need to be copied and stored across program boundaries. 

所有数据都按值在程序中移动.  这意味着当数据跨越程序边界传递时, 每个函数或 goroutine 都会被赋予它自己的数据副本. 我使用两种类型的数据, 值本身或值的地址. 地址是需要被复制和存储的数据当跨程序边界. 

**Listing 2. 10. 1**

~~~go
func incrementByV(inc int) {
	inc++
	log.Printf("incrementByV: inc: %v, addr: %p", inc, &inc)
}

func incrementByP(inc *int) {
	*inc++
	log.Printf("incrementByP: inc: %v, addr: %p", *inc, inc)
}

func TestMoveByValue(t *testing.T) {
	counter := 10
	log.Printf("TestMoveByValue1: counter: %v, addr: %p", counter, &counter)
    
	incrementByV(counter)
	log.Printf("TestMoveByValue1: counter: %v, addr: %p", counter, &counter)

	incrementByP(&counter)
	log.Printf("TestMoveByValue2: counter: %v, addr: %p", counter, &counter)
}
~~~

**扩展指针、堆栈、堆、逃逸分析、值/指针语意的设计机制** TODO

#### 2.11 Escape Analysis 																										(逃逸分析)

I don’t like the term "escape analysis" for the algorithm the compiler uses to determine if a value should be constructed on the stack or heap because it makes it sound like all values are constructed on the stack and then escape (or move) to the heap when necessary. This is NOT the case. The construction of any value only happens once, and the escape analysis algorithm decides where that will be (stack or heap). Only construction on the heap is called an allocation in Go.

我不喜欢编译器用来确定是否应该在堆栈或堆上构造值的算法的术语"逃逸分析", 因为它听起来好像所有值都在堆栈上构造然后必要时逃逸(或移动)到堆中. 事实并非如此. 任何值的构造只发生一次,逃逸分析算法决定将在哪里(堆栈或堆). 在 Go 中, 只有堆上的构造才称为分配. 

Understanding escape analysis is about understanding value ownership. The idea is, when a value is constructed within the scope of a function, then that function owns the value. From there ask the question, does the value being constructed still have to exist when the owning function returns? If the answer is no, the value can be constructed on the stack. If the answer is yes, the value must be constructed on the heap.

理解逃避分析就是理解值的所有权. 其思想是, 当一个值在一个函数的范围内构造时, 该函数就拥有该值. 由此提出一个问题, 当所属函数返回时, 正在构造的值是否仍必须存在? 如果答案为否, 则可以在堆栈上构造该值.如果答案是"是", 则必须在堆上构造该值. 

Note: Escape analysis does have flaws and there are other reasons why a value might be constructed on the heap, but for now this general rule is a good starting point.

注意: Escape分析确实存在缺陷,而且在堆上构建值还有其他原因, 但目前这条一般规则是一个很好的起点.

**Listing 2. 11. 1**

~~~go
type user struct {
	name  string
	email string
}

func stayOnstack() user {
	u := user{
		name:  "GodYao",
		email: "example.com",
	}
	return u
}
~~~

~~~bash
$ go build[test] -gcflags='-m -l'
~~~

The stayOnStack function is using value semantics to return a user value back to the caller. In other words, the caller gets their own copy of the user value being constructed.

stayOnStack函数使用值语义将`user`值返回给调用者. 换句话说, 调用者获取自己的`user`值的副本. 

When stayOnStack returns, the user value it constructs no longer needs to exist, since the caller is getting their own copy. Therefore, the construction of the user value inside of stayOnStack can happen on the stack. No allocation. 

当stayOnStack返回时, 它构造的`user`值不再需要存在, 因为调用方正在获取自己的副本. 因此stayOnStack中的`user`值的构造可以在堆栈上进行. 没有分配. 

**Listing 2. 11. 2**

~~~go
func excapeToHeap() *user {
	u := user{
		name:  "GodYao",
		email: "example.com",
	}
	return &u
}
~~~

The escapeToHeap function is using **pointer semantics** to return a user value back to the caller. In other words, the caller gets **shared access (an address)** to the user value being constructed.

escapeToHeap函数使用**指针语义**将用户值返回给调用者. 换句话说, 调用者获得对正在构造的用户值的**共享访问权(地址).** 

When escapeToHeap returns, the user value it constructs does still need to exist, since the caller is getting shared access to the value. Therefore, the construction of the user value inside of escapeToHeap can’t happen on the stack, it must happen on the heap. Yes allocation. 

当escapeToHeap返回时,它构造的`user`值仍然需要存在, 因为调用者正在获得对该值的共享访问权. 因此，escapeToHeap中`user`值的构造不能发生在堆栈上, 而必须发生在堆上. 当然是分配. 

Think about what would happen if the value was constructed on the stack when using pointer semantics on the return.

思考如果在返回时使用指针语义时在堆栈上构造值会发生什么. 

The caller would get a copy of a stack address from the frame below. Integrity would be lost. Once control goes back to the calling function, the memory on the stack where the user value exists is reusable again. The moment the calling function makes another function call, a new frame is sliced and the memory will be overridden, destroying the shared value. 

调用者将从下面的帧中获得堆栈地址的副本.  完整性就会丧失.  一旦控制权返回到调用函数, `user`值所在的堆栈上的内存就可以再次重用. 在调用函数进行另一个函数调用的那一刻, 一个新的帧被切片并且内存将被覆盖,从而破坏共享值.

This is why I think about the stack being self cleaning. Zero value initialization helps every stack frame that I need to be cleaned without the use of GC. The stack is self cleaning since a frame is taken and initialized for the execution of each function call. The stack is cleaned during function calls and not on returns because the compiler doesn't know if that memory on the stack will ever be needed again. 

这就是为什么我认为堆栈是自清理的. 零值初始化有助于在不使用 GC 的情况下清理我需要清理的每个堆栈帧. 堆栈是自清理的, 因为每个函数调用的执行都会获取并初始化一个帧. **堆栈在函数调用期间被清理**，而不是在返回时被清理, 因为编译器不知道是否会再次需要堆栈上的内存. 

Escape analysis decides if a value is constructed on the stack (the default) or the heap (the escape). With the stayOnStack function, I’m passing a copy of the value back to the caller, so it’s safe to keep the value on the stack. With the escaoeToHeap function, I’m passing a copy of the values address back to the caller (sharing up the stack) so it’s not safe to keep the value on the stack. 

逃逸分析决定一个值是在堆栈(默认)还是堆(逃逸)上构造的. 使用 stayOnStack 函数, 我将值的副本传回给调用者, 因此将值保留在堆栈上是安全的. 使用 escaoeToHeap 函数, 我将值地址的副本传回给调用者(共享堆栈), 因此将值保留在堆栈上是不安全的. 

#### 2.12 Stack Growth (栈) 																									(堆栈增长)

The size of each frame for every function is calculated at compile time. This means, if the compiler doesn’t know the size of a value at compile time, the value must be constructed on the heap. An example of this is using the built-in function make to construct a slice whose size is based on a variable. 

每个函数的每个帧的大小是在编译时计算的. 这意味着, 如果编译器在编译时不知道值的大小. 则必须在堆上构造该值. 这方面的一个例子是使用内置函数 make 来构造一个大小基于变量的切片.

~~~go
slices := make([]int, size) // Backing array allocates on the heap.
~~~

Go uses a **contiguous stack** vs using a segmented stack mechanic. In general, these are mechanics that dictate how stacks grow and shrink.

Go使用**连续堆栈**, 而不是使用分段堆栈机制. 一般来说, 这些都是决定堆栈如何增长和收缩的机制.

Every function call comes with a little preamble that asks, "Is there enough stack space for this new frame?". If yes, then no problem and the frame is taken and initialized. If not, then a new larger stack must be constructed and the memory on the existing stack must be copied over to the new one, with changes to pointers that reference memory on the stack. The benefits of contiguous memory and linear traversals with modern hardware is the tradeoff for the cost of the copy. 

每个函数调用都有一个小前导, 询问"是否有足够的堆栈空间来容纳这个新帧?". 如果是, 则没有问题, 帧被获取并初始化. 如果否, 则必须构造一个新的更大堆栈, 并且必须将现有堆栈上的内存复制到新堆栈上, 同时更改引用堆栈上内存的指针. 使用现代硬件进行连续内存和线性遍历的好处是对拷贝成本的权衡.

Because of the use of contiguous stacks, no Goroutine can have a pointer to some other Goroutine’s stack. There would be too much overhead for the runtime to keep track of every pointer to every stack and readjust those pointers to the new location. 

由于使用了**连续堆栈**, 没有 Goroutine 可以拥有指向其他 Goroutine 堆栈的指针.  运行时跟踪每个堆栈的每个指针并将这些指针重新调整到新位置会产生太多开销. 



