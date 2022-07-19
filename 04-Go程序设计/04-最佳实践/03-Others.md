[TOC]

## Others

### 反模式

没有对未来的使用者考虑下的编码模式，称为反模式

#### 导出类型

> 1. Go中大写开头才会对包外开放
> 2. 反模式情况下，就算导出了，调用者也必须再次定义一个类型

~~~go
// 反模式
type unexportedType string

func ExportedFunc() unexportedType { 
    return unexportedType("some string")
} 

// 推荐
type ExportedType string
func ExportedFunc() ExportedType { 
    return ExportedType("some string")
}
~~~

#### 空白标识符与range

> 1. 空白标识符用于丢弃变量，不使用
> 2. 当range遍历序列，但是并不需要里面的index、或者value的时候，不必要使用空白标识符

~~~go
// 反模式
for _ = range sequence { 
    run()
} 
x, _ := someMap[key] 
_ = <-ch 

// 推荐
for range something { 
    run()
} 

x := someMap[key] 
<-ch
~~~

#### append

> append可以追加不定参数

~~~go
sliceOne = append(sliceOne, sliceTwo…)
~~~

#### make

> 1. 用于初始化Slice、Map、Chan
> 2. 对于 Chan，缓冲区容量默认为零（不带缓冲）
> 3. 对于 Map，分配的大小默认为较小的起始大小
> 4. 对于Slice，如果省略容量，则容量参数的值默认为与长度相等

~~~go
ch = make(chan int)
sl = make([]int, 1)
~~~

#### return

无返回值的情况下，不需要用return作为结束; 具名返回值, 可以显示用return

~~~go
// 没用的return，不推荐
func alwaysPrintFoofoo() { 
    fmt.Println("foofoo") 
    return
} 

// 推荐
func alwaysPrintFoo() { 
    fmt.Println("foofoo")
}

// 推荐
func printAndReturnFoofoo() (foofoo string) { 
    foofoo := "foofoo" 
    fmt.Println(foofoo) 
    return
}
~~~

#### switch

switch不需要break

~~~go
switch 2 {
case 1: 
    fmt.Print("1") 
    fallthrough
case 2: 
    fmt.Print("2") 
    fallthrough
case 3: fmt.Print("3")
}
~~~

####  nil 切片上的冗余检查

nil切片的长度为0，所以没必要检查切片是否为nil

~~~go
// 不推荐
if x != nil && len(x) != 0 { 
  
}

// 推荐
if len(x) != 0 { 
    
}
~~~

#### select

仅仅有一个case语句，没必要使用select; 如果需要不阻塞，可以添加default

~~~go
// 反模式
select {
    case x := <-ch: fmt.Println(x)
} 

// 推荐
x := <-ch
fmt.Println(x)

// 推荐
select {
    case x := <-ch: 
        fmt.Println(x)
    default: 
        fmt.Println("default")
}
~~~

#### context

context参数应该作为函数第一个参数

~~~go
// 反模式
func badPatternFunc(k favContextKey, ctx context.Context) {    
   
}

// 推荐
func goodPatternFunc(ctx context.Context, k favContextKey) {    
  
}
~~~

### 编程范式

左耳朵耗子-陈浩

#### 切片

切片是结构体, 底层引用了数组

~~~go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
~~~

共享

共享了底层的数组, 也就是数据共享了

~~~go 
foo := make([]int64, 5) // cap = len = 5
bar := foo[1:4]
bar[1] = 99
fmt.Printf("%+v, %d, %d, %p\n", foo, len(foo), cap(foo), foo) 
// [0 0 99 0 0] 5 5 0xc00001a180
fmt.Printf("%+v, %d, %d, %p\n",bar, len(bar), cap(bar), bar) 
// [0 99 0] 3 4     0xc00001a188
~~~

扩容

2倍扩容

~~~go
foo := make([]int, 5) // cap = len = 5
bar := foo[1:4]
foo = append(foo, 100)
// [0 0 0 0 0 100], 6, 10, 0xc00012e000
fmt.Printf("%+v, %d, %d, %p\n", foo, len(foo), cap(foo), foo) 
// [0 0 0], 3, 4, 0xc000118038
fmt.Printf("%+v, %d, %d, %p\n",bar, len(bar), cap(bar), bar) 
~~~

#### 深度比较

需要比较实例中数据是否相等

~~~go
sl1 := []int{1, 2, 3}
sl2 := []int{1, 2, 3}
if reflect.DeepEqual(sl1, sl2) {
    log.Println("sl1 = sl2")
}
m1 := map[string]int{
    "a": 1,
    "b": 2,
}
m2 := map[string]int{
    "a": 1,
    "b": 2,
}
if reflect.DeepEqual(m1, m2) {
    log.Println("m1 = m2")
}
~~~

#### 接口编程

~~~go
package main

import "log"

// 业务类型与控制逻辑通过接口解耦
type PrintNameFace interface {
	PrintStr()
}
// 业务类型
type Country struct {
	Name string
}

type City struct {
	Name string
}

// 控制逻辑
func (c *Country) PrintStr() {
	log.Println(c.Name)
}

func (c *City) PrintStr() {
	log.Println(c.Name)
}

func main() {
	cr := &Country{Name: "China"}
	ct := &City{Name: "BeiJin"}
	
    // 接口完整性检查
    // var _ PrintNameFace = (*Country)(nil)
	// var _ PrintNameFace = (*City)(nil)
    
	var cri PrintNameFace = cr
	var cti PrintNameFace = ct

	cri.PrintStr()
	cti.PrintStr()
}
~~~

#### 时间

> 1. 时间极其复杂, 时区、格式、精度
> 2. 在 Go 语言中, 使用 `time.Time` 和 `time.Duration` 两个类型

#### 性能提示

> 1. 转换请使用strconv包
> 2. 避免将String转换为Byte
> 3. 尽量提前估算好切片的容量, 避免重新申请
> 4. 使用`StringBuffer` 或是`StringBuild` 来拼接字符串
> 5. 使用并发gorouitne, 然后使用waitgroup同步
> 6. 使用sync.Pool重用对象
> 7. 使用 lock-free的操作, 避免使用 mutex, 尽可能使用 `sync/Atomic`包
> 8. 使用 `bufio.NewWrite()` 和 `bufio.NewReader()` 可以带来更高的I/O性能
> 9. 使用 `regexp.Compile()` 编译正则表达式
> 10. 使用 [protobuf](https://github.com/golang/protobuf) 或 [msgp](https://github.com/tinylib/msgp) , 更高性能的协议
> 11. 使用map的时候, 使用整型的key会比字符串的要快

#### 错误处理

> 1. 函数支持多返回值, 将业务语义与控制语义区分开
> 2. 可以用空白标识符忽略错误
> 3. 错误可以自定义扩展
> 4. 推荐采用包装错误[github.com/pkg/errors](https://github.com/pkg/errors)

#### 资源清理

~~~go
defer resp.body.close()

f.close()
~~~

#### Functionnal Options

可选配置选项问题

~~~go
type Queue struct {
	Name     string
	MaxLimit int
	// monitor
	MonitorInterval int
}

type QueueOption func(*Queue)

func WithMaxLimit(max int) QueueOption {
	return func(q *Queue) {
		q.MaxLimit = max
	}
}

func WithMonitorInterval(seconds int) QueueOption {
	return func(q *Queue) {
		q.MonitorInterval = seconds
	}
}
func NewQueue(name string, options ...QueueOption) *Queue {
	queue := &Queue{name, 10, 5}
	for _, o := range options {
		o(queue)
	}
	return queue
}
~~~

