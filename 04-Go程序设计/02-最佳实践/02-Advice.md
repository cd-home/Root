[TOC]

### Go Advice

#### From Rob Pike

》TODO

Go Proverbs Simple, Poetic, Pithy

[Don't communicate by sharing memory, share memory by communicating.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=2m48s)

通过通信共享内存

[Concurrency is not parallelism.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=3m42s)

并发不是并行

[Channels orchestrate; mutexes serialize.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=4m20s)

通道编排: 互斥体序列化

[The bigger the interface, the weaker the abstraction.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=5m17s)

接口越大, 抽象越弱

[Make the zero value useful.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=6m25s)

让零值有用

[interface{} says nothing.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=7m36s)

**interface{} 就是 interface{}**

[Gofmt's style is no one's favorite, yet gofmt is everyone's favorite.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=8m43s)

代码强烈推荐 `gofmt`

[A little copying is better than a little dependency.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=9m28s)

[Syscall must always be guarded with build tags.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=11m10s)

[Cgo must always be guarded with build tags.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=11m53s)

[Cgo is not Go.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=12m37s)

Cgo不是Go

[With the unsafe package there are no guarantees.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=13m49s)

[Clear is better than clever.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=14m35s)

清晰胜于聪明

[Reflection is never clear.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=15m22s)

反射永远不清晰

[Errors are values.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=16m13s)

[Don't just check errors, handle them gracefully.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=17m25s)

不要只检查错误, 还要优雅的处理

[Design the architecture, name the components, document the details.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=18m09s)

设计架构、命名组件、记录细节

[Documentation is for users.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=19m07s)

文档供用户使用

[Don't panic.](https://github.com/golang/go/wiki/CodeReviewComments#dont-panic)

不要panic

#### 禅道

1.  **package实现单一目的**
2.  **显示处理错误**
3.  **尽早返回, 避免深嵌套**
4.  让调用者选择并发
5.  **启动g, 需要知道何时停止**
6.  避免package级别状态编写测试以锁定 package API 的行为
7.  编写benchmark测试
8.  节约内存
9.  可维护性

#### 代码

1.  go fmt
2.  多个if可以使用switch代替
3.  chan struct{} 传递信号
4.  30 * time.Second 比 time.Duration(30) * time.Second 好
5.  用 var foo time.Dutation 比 var fooMillis int64 好
6.  总是把for-select换成一个函数
7.  **分组定义const和var**
8.  _ 表示不需要的参数
9.  不要panic
10.  每个阻塞或者IO函数都是可取消或者可超时的
11.  for-range来迭代集合, 复制的集合去迭代
12.  **多行字符串使用反引号**
13.  函数的同类型参数类型放最后
14.  slice的零值是nil
15.  **ok模式读取map的值**
16.  强制显式按照成员名字初始化结构体

~~~go
type Point struct {
	X, Y float64
	_    struct{} // to prevent unkeyed literals
}
// p := Point{X:13.5, Y:213.3}
~~~

17.  防止结构体比较

~~~go
 type Point struct {
    _ [0]func() // unexported, zero-width non-comparable field
    X, Y float64
  }
~~~

18.  移动defer到顶部

#### 并发

1.  以线程安全创建一些东西最好选择是 sync.Once
2.  永远不要使用select{}
3.  当你需要一个自定义类型的 atomic 值时, 可以使用 [atomic.Value](https://godoc.org/sync/atomic#Value)

#### 性能

1.  **不要省略defer**
2.  关闭http body 【defer r.Body.Close()】
3.  请勿在你的热路径中过度使用 `fmt.Sprintf`. 由于维护接口的缓冲池和动态调度, 它是很昂贵的。
    -   如果你正在使用 `fmt.Sprintf("%s%s", var1, var2)`, 考虑使用简单的字符串连接。
    -   如果你正在使用 `fmt.Sprintf("%x", var)`, 考虑使用 `hex.EncodeToString` or `strconv.FormatInt(var, 16)`
4.  `regexp.MustCompile` 比 `regexp.Compile` 更好
5.  **不要在循环中使用 defer, 否则会导致内存泄露**
6.  不要忘记停止 ticker, 除非你需要泄露 channel
7.  **`sync.Map` 不是万能的, 没有很强的理由就不要使用它**
8.  在 `sync.Pool` 中分配内存存储非指针数据
9.  清空map【循环删除、make一个新的赋值】

#### 测试

1.  测试名称 `package_test` 比 `package` 要好
2.  在编译期检查接口的实现

~~~go
type Point struct {
	X, Y float64
	_    struct{} // to prevent unkeyed literals
}
type Modifier interface {
	Foo()
}
func (p *Point) Foo()  {

}
var _ Modifier = (*Point)(nil)  // 骚操作
~~~

3.  匿名结构体

~~~go
 type hits struct {
     sync.Mutex
     n int
 }
func (h *hits) Modify() {
    h.Lock()  // 匿名结构体提升了方法和属成员
    n++
    h.Unlock()
}
~~~

#### 编程指南

1.  生产环境避免panic
2.  避免可变全局变量
3.  类型转换strconv代替fmt
4.  避免字符串转字节
5.  初始化Map指定容量
6.  类型断言采用 ok模式, 否则容易panic
7.  枚举从1开始, 也有需求从0开始
8.  channel容量为1或者无缓冲
9.  defer清理资源
10.  注意Slice和Map引用类型作为参数共享
11.  零值得Mutex是有效的
12.  结构体匿名成员不公开, 如要公开需要声明
13.  不要使用指针指向接口
14.  接口调用方法要注意接收者
15.  错误处理
     *   简单错误信息errors.New()
     *   定制Error()
     *   fmt.Errorf()

16.  测试数据：结构体多组数据(表测试)

#### Others

##### 导出类型

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

##### 空白标识符与range

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

##### append

> append可以追加不定参数

~~~go
sliceOne = append(sliceOne, sliceTwo…)
~~~

##### make

> 1. 用于初始化Slice、Map、Chan
> 2. 对于 Chan，缓冲区容量默认为零（不带缓冲）
> 3. 对于 Map，分配的大小默认为较小的起始大小
> 4. 对于Slice，如果省略容量，则容量参数的值默认为与长度相等

~~~go
ch = make(chan int)
sl = make([]int, 1)
~~~

##### return

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

##### switch

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

#####  nil 切片上的冗余检查

nil切片的长度为0，所以没必要检查切片是否为nil

~~~go
// 不推荐
if x != nil && len(x) != 0 { 
  
}

// 推荐
if len(x) != 0 { 
    
}
~~~

##### select

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

##### context

context参数应该作为函数第一个参数

~~~go
// 反模式
func badPatternFunc(k favContextKey, ctx context.Context) {    
   
}

// 推荐
func goodPatternFunc(ctx context.Context, k favContextKey) {    
  
}
~~~

#### 性能提示

1. 转换请使用strconv包
2. 避免将String转换为Byte
3. 尽量提前估算好切片的容量, 避免重新申请
4. 使用`StringBuffer` 或是`StringBuild` 来拼接字符串
5. 使用并发gorouitne, 然后使用waitgroup同步
6. 使用sync.Pool重用对象
7. 使用 lock-free的操作, 避免使用 mutex, 尽可能使用 `sync/Atomic`包
8. 使用 `bufio.NewWrite()` 和 `bufio.NewReader()` 可以带来更高的I/O性能
9. 使用 `regexp.Compile()` 编译正则表达式
10. 使用 [protobuf](https://github.com/golang/protobuf) 或 [msgp](https://github.com/tinylib/msgp) , 更高性能的协议
11. 使用map的时候, 使用整型的key会比字符串的要快

#### 安全指南

1. 索引越界
2. 进行指针操作, 必须判断该指针是否为nil, 防止程序panic
3. make分配内存, 必须判断size是否在合理范围
4. 已经关闭的channel不能再关闭, 亦不能写入
5. 确保每个协程都能正常退出, 在控制内
