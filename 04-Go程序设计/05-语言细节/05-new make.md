[TOC]

### 内存分配

#### new

针对基础类型 数值、字符串、布尔、结构体,  开辟内存, 返回指针

源码

~~~go
// The new built-in function allocates memory. 
// The first argument is a type, not a value,     	第一个参数是类型, 不是一个值
// and the value returned is a pointer to a newly   返回值是一个指向该类型分配零值的指针
// allocated zero value of that type.
func new(Type) *Type
~~~

Example

~~~go
 p := new(int)
~~~

#### make

针对 切片, 映射, 通道

源码

~~~go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
func make(t Type, size ...IntegerType) Type
~~~

Example

~~~go
//	Slice: The size specifies the length. The capacity of the slice is
//	equal to its length. A second integer argument may be provided to
//	specify a different capacity; it must be no smaller than the
//	length. For example, make([]int, 0, 10) allocates an underlying array
//	of size 10 and returns a slice of length 0 and capacity 10 that is
//	backed by this underlying array.
slices := make([]int, len)			// len = cap
slices := make([]int, len, cap)		// diff len and cap

//	Map: An empty map is allocated with enough space to hold the
//	specified number of elements. The size may be omitted, in which case
//	a small starting size is allocated.
maps := make(map[string]string, cap)

//	Channel: The channel's buffer is initialized with the specified
//	buffer capacity. If zero, or the size is omitted, the channel is
//	unbuffered.
channle := make(chan bool, cap)
~~~

总结

- [ ] make只能为 slice map chan 分配内存, 并返回的是类型Type本身, 而且内存会出初始化
- [ ] new只能为基础类型、数组、结构体分配内存, 内存初始化为零值, 返回指针, 即是*Type

更加深入的原理, 就是slice map chan (引用类型) 需要初始化内部的数据接口, 而new所针对的是简单类型, 只需要初始化内存即可. 