[TOC]

# Uber Go 语言编码规范

## 指导原则

### 指向 interface 的指针

不需要指向接口类型的指针. 应该将接口作为值进行传递，在这样的传递过程中，实质上传递的底层数据仍然可以是指针. 

接口实质上在底层用两个字段表示

1.  一个指向某些特定类型信息的指针
2.  数据指针
    *   如果存储的数据是指针，则直接存储
    *   如果存储的数据是一个值，则存储指向该值的指针

**如果希望接口方法修改基础数据，则必须使用指针传递**

### 接收器与接口

1.  使用值接收器的方法既可以通过值调用，也可以通过指针调用. 

```go
type S struct {
    data string
}
func (s S) Read() string {
    return s.data
}
func (s *S) Write(str string) {
    s.data = str
}
func main() {
    s1 := S{data: "abc"}
    s1.Read()
    s2 := &S{data: "abc"}
    s2.Read()
    s2.Write("def")
}
```

2.  同样，即使该方法具有值接收器，也可以通过指针来满足接口

```go
type F interface {
    f()
}
type S1 struct{}
func (s S1) f() {}

type S2 struct{}
func (s *S2) f() {}

func main() {
    
	s1Val := S1{}
    s1Ptr := &S1{}
    
    s2Val := S2{}
    s2Ptr := &S2{}
    
    var i F
    
    i = s1Val
    i = s1Ptr
    
    i = s2Ptr
//  下面代码无法通过编译. 因为 s2Val 是一个值，而 S2 的 f 方法中没有使用值接收器
//   i = s2Val
}
```

### 零值 Mutex 是有效的

> 零值 `sync.Mutex` 和 `sync.RWMutex` 是有效的. 指向 mutex 的指针基本是不必要的

|          Bad          |       Good        |
| :-------------------: | :---------------: |
| mu := new(sync.Mutex) | var mu sync.Mutex |

1. 为私有类型或需要实现互斥接口的类型嵌入

~~~go
type smap struct {  // 私有类型
    sync.Mutex 		// only for unexported types（仅适用于非导出类型）   
    data map[string]string 
} 
func newSMap() *smap {  
    return &smap{    
        data: make(map[string]string), 
        // 零值有效所以不用初始化 sync.Mutex
    } 
}
func (m *smap) Get(k string) string {  
    m.Lock()  
    defer m.Unlock()   
    return m.data[k] 
}
~~~

2.  对于导出的类型，使用专用字段

~~~go
type SMap struct {  
    mu sync.Mutex    // 对于导出类型，请使用私有锁
    data map[string]string 
} 
func NewSMap() *SMap {  
    return &SMap{    
        data: make(map[string]string),  
    }
} 
func (m *SMap) Get(k string) string { 
    m.mu.Lock()  
    defer m.mu.Unlock()   
    return m.data[k] 
}
~~~

### 在边界处拷贝 Slices 和 Maps

slices 和 maps 包含了指向底层数据的指针，因此在需要复制它们时要特别注意

#### 接收 Slices 和 Maps

> 请记住，当 map 或 slice 作为函数参数传入时，如果存储了对它们的引用，则用户可以对其进行修改

~~~go
// Bad
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)
// Did you mean to modify d1.trips?
trips[0] = ...


// Good
func (d *Driver) SetTrips(trips []Trip) {  
    d.trips = make([]Trip, len(trips))  
    copy(d.trips, trips) 
} 
trips := ... 
d1.SetTrips(trips) 
trips[0] = ...   // 这里我们修改 trips[0]，但不会影响到 d1.trips trips[0] = ... 
~~~

#### 返回 slices 或 maps

同样，请注意用户对暴露内部状态的 map 或 slice 的修改

~~~go
// Bad
type Stats struct {
  sync.Mutex

  counters map[string]int
}

// Snapshot 返回当前状态。
func (s *Stats) Snapshot() map[string]int {
  s.Lock()
  defer s.Unlock()

  return s.counters
}

// snapshot is no longer protected by the lock!
snapshot := stats.Snapshot()

// Good
type Stats struct {  
    mu sync.Mutex   
    counters map[string]int 
} 
func (s *Stats) Snapshot() map[string]int {  
    s.mu.Lock()  
    defer s.mu.Unlock() 
    //  复制一份
    result := make(map[string]int, len(s.counters))  
    for k, v := range s.counters {   
        result[k] = v  
    }  
    return result 
} // snapshot 现在是一个拷贝 snapshot := stats.Snapshot()
~~~

### 使用 defer 释放资源

> 使用 defer 释放资源，诸如文件和锁

Defer 的开销非常小，只有在您可以证明函数执行时间处于纳秒级的程度时，才应避免这样做. 使用 defer 提升可读性是值得的，因为使用它们的成本微不足道. 尤其适用于那些不仅仅是简单内存访问的较大的方法，在这些方法中其他计算的资源消耗远超过 `defer`

~~~go
// Bad
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount
// 当有多个 return 分支时，很容易遗忘 unlock


// Good
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// 可读性越佳
~~~

### Channel 的 size 要么是 1，要么是无缓冲的

channel 通常 size 应为 1 或是无缓冲的. 默认情况下，channel 是无缓冲的，其 size 为零. 任何其他尺寸都必须经过严格的审查. 考虑如何确定大小，是什么阻止了 channel 在负载下被填满并阻止写入，以及发生这种情况时发生了什么

~~~go
// 大小：1 
c := make(chan int, 1) 
// 或者 无缓冲 channel，大小为 0 
c := make(chan int) 
~~~

### 枚举从 1 开始

引入枚举的标准方法是声明一个自定义类型和一个使用了 iota 的 const 组. 由于变量的默认值为 0,因此通常应以非零值开头枚举. 

~~~go
// Add=1, Subtract=2, Multiply=3
type Operation int const (  
    Add Operation = iota + 1  
    Subtract  
    Multiply 
) 
~~~

在某些情况下，使用零值是有意义的（枚举从零开始），例如，当零值是理想的默认行为时. 

```go
type LogOutput int
const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)
// LogToStdout=0, LogToFile=1, LogToRemote=2
```

### 错误类型

Go 中有多种声明错误（Error) 的选项：

1. [`errors.New`](https://golang.org/pkg/errors/#New) 对于简单静态字符串的错误
2. [`fmt.Errorf`](https://golang.org/pkg/fmt/#Errorf) 用于格式化的错误字符串
3. 实现 `Error()` 方法的自定义类型
4. 用 [`"pkg/errors".Wrap`](https://godoc.org/github.com/pkg/errors#Wrap) 的 Wrapped errors

返回错误时，请考虑以下因素以确定最佳选择

1. 这是一个不需要额外信息的简单错误吗？如果是这样，[`errors.New`](https://golang.org/pkg/errors/#New) 足够了. 
2. 客户需要检测并处理此错误吗？如果是这样，则应使用自定义类型并实现该 `Error()` 方法. 
3. 您是否正在传播下游函数返回的错误？如果是这样，请查看本文后面有关错误包装 [section on error wrapping](#错误包装) 部分的内容. 
4. 否则 [`fmt.Errorf`](https://golang.org/pkg/fmt/#Errorf) 就可以了. 

如果客户端需要检测错误，并且您已使用创建了一个简单的错误 [`errors.New`](https://golang.org/pkg/errors/#New)，请使用一个错误变量. 

~~~go
// Bad
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar
func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}

---------------------------------------------------------------------------------------------


// Good
//package foo 
var ErrCouldNotOpen = errors.New("could not open") 

func Open() error {  
    return ErrCouldNotOpen 
} 
//package bar 
if err := foo.Open(); err != nil {  
    if err == foo.ErrCouldNotOpen {   
        // handle  
    } else {   
        panic("unknown error")  
    } 
}
~~~

如果您有可能需要客户端检测的错误，并且想向其中添加更多信息，则应使用自定义类型. 

~~~go
// Bad
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  if err := open(); err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}

// Good
type errNotFound struct {  
    file string 
} 
func (e errNotFound) Error() string {  
    return fmt.Sprintf("file %q not found", e.file) 
} 
func open(file string) error {  
    return errNotFound{file: file} 
} 
func use() {  
    if err := open(); err != nil {    
        if _, ok := err.(errNotFound); ok {
            // handle    
        } else {      
            panic("unknown error")    
        }  
    } 
}
~~~

直接导出自定义错误类型要小心,因为它们已成为程序包公共 API 的一部分. 最好公开匹配器功能以检查错误. 

```go
// package foo
type errNotFound struct {
  file string
}
func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}
func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}
func Open(file string) error {
  return errNotFound{file: file}
}
// package bar
if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

### 错误包装

一个（函数 / 方法）调用失败时，有三种主要的错误传播方式：

1. 如果没有要添加的其他上下文，并且您想要维护原始错误类型，则返回原始错误。

2. 添加上下文，使用 "pkg/errors".Wrap 以便错误消息提供更多上下文，"pkg/errors".Cause 可用于提取原始错误。 Use fmt.Errorf if the callers do not need to detect or handle that specific error case.

3. 如果调用者不需要检测或处理的特定错误情况，使用 fmt.Errorf。

    建议在可能的地方添加上下文，以使您获得诸如 “调用服务 foo：连接被拒绝” 之类的更有用的错误，而不是诸如 “连接被拒绝” 之类的模糊错误

### 处理类型断言失败

[type assertion](https://golang.org/ref/spec#Type_assertions) 的单个返回值形式针对不正确的类型将产生 panic. 因此，请始终使用“comma ok”的惯用法. 

| Bad                        | Good                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `t := i.(string) 复制代码` | `t, ok := i.(string) if !ok {  // 优雅地处理错误 } 复制代码` |

### 不要 panic

1.  在生产环境中运行的代码必须避免出现 panic.
2.  如果发生错误，该函数必须返回错误，并允许调用方决定如何处理它. 

~~~go
func foo(bar string) error {  
    if len(bar) == 0 {    
        return errors.New("bar must not be empty") 
    }  // ...  return nil 
} 
func main() {  
    if len(os.Args) != 2 {    
        fmt.Println("USAGE: foo ")    
        os.Exit(1)  
    }  
    if err := foo(os.Args[1]); err != nil {
        panic(err) 
    } 
} 
~~~

panic/recover 不是错误处理策略. 仅当发生不可恢复的事情（例如：nil 引用）时，程序才必须 panic. 程序初始化是一个例外：程序启动时应使程序中止的不良情况可能会引起 panic. 

```go
var _statusTemplate = template.Must(template.New("name").Parse("_statusHTML"))
```

即使在测试代码中，也优先使用`t.Fatal`或者`t.FailNow`而不是 panic 来确保失败被标记. 

~~~go
func TestFoo(t *testing.T) { 
    f, err := ioutil.TempFile("", "test"); if err != nil { 
        t.Fatal("failed to set up test") 
    }
}
~~~

## 性能

性能方面的特定准则只适用于高频场景. 

### 优先使用 strconv 而不是 fmt

将原语转换为字符串或从字符串转换时，`strconv`速度比`fmt`快. 

~~~go
for i := 0; i < b.N; i++ {  
    s := strconv.Itoa(rand.Int()) 
}
~~~

### 避免字符串到字节的转换

不要反复从固定字符串创建字节 slice. 相反，请执行一次转换并捕获结果. 

~~~go
data := []byte("Hello world") 
for i := 0; i < b.N; i++ {  
    w.Write(data) 
}
~~~

### 尽量初始化时指定 Map 容量

1.  在尽可能的情况下，在使用 `make()` 初始化的时候提供容量信息

```go
make(map[T1]T2, hint)
```

为 `make()` 提供容量信息（hint）尝试在初始化时调整 map 大小， 这减少了在将元素添加到 map 时增长和分配的开销.  注意，map 不能保证分配 hint 个容量. 因此，即使提供了容量，添加元素仍然可以进行分配. 

~~~go
// m是有大小提示创建的；在运行时可能会有更少的分配
files, _ := ioutil.ReadDir("./files") 
m := make(map[string]os.FileInfo, len(files)) 
for _, f := range files {    
    m[f.Name()] = f 
}
~~~

## 规范

### 一致性

最重要的是，**保持一致**.

一致性的代码更容易维护、是更合理的、需要更少的学习成本、并且随着新的约定出现或者出现错误后更容易迁移、更新、修复 bug

### 相似的声明放在一组

Go 语言支持将相似的声明放在一个组内. 

~~~go
import (
    "a"  
    "b"
) 
~~~

这同样适用于常量、变量和类型声明：

~~~go
const (  
    a = 1  
    b = 2 
) 
var ( 
    a = 1  
    b = 2 
) 
type (  
    Area float64  
    Volume float64 
)
~~~

仅将相关的声明放在一组. 不要将不相关的声明放在一组. 

~~~go
type Operation int 
const (  
    Add Operation = iota + 1  
    Subtract  
    Multiply
)
const ENV_VAR = "MY_ENV"
~~~

分组使用的位置没有限制，例如：你可以在函数内部使用它们：

~~~go
func f() string {  
    var (    
        red   = color.New(0xff0000)    
        green = color.New(0x00ff00)    
        blue  = color.New(0x0000ff)  
    )   
    ... 
}
~~~

### import 分组

导入应该分为两组：

1. 标准库
2. 其他库

默认情况下，这是 goimports 应用的分组. 

~~~go
import (  
    "fmt"  
    "os"   
    "go.uber.org/atomic"  
    "golang.org/x/sync/errgroup" 
)
~~~

### 包名

当命名包时，请按下面规则选择一个名称：

1. 全部小写. 没有大写或下划线. 
2. 大多数使用命名导入的情况下，不需要重命名. 
3. 简短而简洁. 请记住，在每个使用的地方都完整标识了该名称. 
4. 不用复数. 例如`net/url`，而不是`net/urls`. 
5. 不要用“common”，“util”，“shared”或“lib”. 这些是不好的，信息量不足的名称. 

### 函数名

我们遵循 Go 社区关于使用 [MixedCaps 作为函数名](https://golang.org/doc/effective_go.html#mixed-caps) 的约定. 有一个例外，为了对相关的测试用例进行分组，函数名可能包含下划线，如：`TestMyFunction_WhatIsBeingTested`.

### 导入别名

1.  如果程序包名称与导入路径的最后一个元素不匹配，则必须使用导入别名

```go
import (
    "net/http"
    client "example.com/client-go"
    trace "example.com/trace/v2"
)
```

2.  在所有其他情况下，除非导入之间有直接冲突，否则应避免导入别名. 

~~~go
import (  
    "fmt"  
    "os" 
    "runtime/trace"  
    nettrace "golang.net/x/trace" 
)
~~~

### 函数分组与顺序

1. 函数应按粗略的调用顺序排序

2. 同一文件中的函数应按接收者分组

    因此，导出的函数应先出现在文件中，放在`struct`, `const`, `var`定义的后面

    在定义类型之后，但在接收者的其余方法之前，可能会出现一个 `newXYZ()`/`NewXYZ()`

    由于函数是按接收者分组的，因此普通工具函数应在文件末尾出现. 

~~~go
const PI = 3.14
// 类型
type something struct{ 
    weights int
} 
// 工厂
func NewSomething() *something {    
    return &something{} 
} 
// 方法
func (s *something) Cost() {  
    return calcCost(s.weights) 
} 
func (s *something) Stop() {...} 
// 工具函数
func calcCost(n []int) int {...} 
~~~

### 减少嵌套

代码应通过尽可能先处理错误情况/特殊情况并尽早返回或继续循环来减少嵌套.

~~~go
for _, v := range data {  
    if v.F1 != 1 {    
        log.Printf("Invalid v: %v", v)    
        continue  
    }   
    v = process(v)  
    if err := v.Call(); err != nil {    
        return err 
    } 
    v.Send() 
}
~~~

### 不必要的 else

如果在 if 的两个分支中都设置了变量，则可以将其替换为单个 if. 

~~~go
a := 10 if b {  a = 100 }
~~~

### 顶层变量声明

1.  在顶层，使用标准`var`关键字. 请勿指定类型，除非它与表达式的类型不同. 

~~~go
var _s = F() 
// 由于 F 已经明确了返回一个字符串类型，因此我们没有必要显式指定_s 的类型 
// 还是那种类型 func F() string { return "A" }
~~~

2.  如果表达式的类型与所需的类型不完全匹配，请指定类型. 

```go
type myError struct{}

func (myError) Error() string { return "error" }
func F() myError { return myError{} }
var _e error = F()
// F 返回一个 myError 类型的实例，但是我们要 error 类型
```

### 对于未导出的顶层常量和变量，使用_作为前缀

在未导出的顶级`vars`和`consts`， 前面加上前缀_，以使它们在使用时明确表示它们是全局符号. 

例外：未导出的错误值，应以`err`开头. 

基本依据：顶级变量和常量具有包范围作用域. 使用通用名称可能很容易在其他文件中意外使用错误的值. 

~~~go
const (  
    _defaultPort = 8080  
    _defaultUser = "user"
)
~~~

### 结构体中的嵌入

嵌入式类型应位于结构体内的字段列表的顶部，并且必须有一个空行将嵌入式字段与常规字段分隔开. 

~~~go
type Client struct { 
    http.Client   
    
    version int 
}
~~~

### 使用字段名初始化结构体

初始化结构体时，几乎始终应该指定字段名称. 现在由 [`go vet`](https://golang.org/cmd/vet/) 强制执行. 

~~~go
k := User{    
    FirstName: "John",    
    LastName: "Doe",    
    Admin: true, 
}
~~~

例外：如果有 3 个或更少的字段，则可以在测试表中省略字段名称. 

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

### 本地变量声明

如果将变量明确设置为某个值，则应使用短变量声明形式 (`:=`). 

|       Bad        |     Good     |
| :--------------: | :----------: |
| `var s = "foo" ` | `s := "foo"` |

但是，在某些情况下，`var` 使用关键字时默认值会更清晰. 例如，声明空切片. 

~~~go
func f(list []int) {  
    var filtered []int  
    for _, v := range list {    
        if v > 10 {      
            filtered = append(filtered, v)   
        }  
    } 
}
~~~

### nil 是一个有效的 slice

`nil` 是一个有效的长度为 0 的 slice，这意味着，

1. 不应明确返回长度为零的切片. 应该返回`nil` 来代替. 

~~~go
if x == "" {  return nil }
~~~

2. 要检查切片是否为空，请始终使用`len(s) == 0`. 而非 `nil`. 

~~~go
func isEmpty(s []string) bool {  return len(s) == 0 }
~~~

3. 零值切片（用`var`声明的切片）可立即使用，无需调用`make()`创建. 

~~~go
var nums []int 
if add1 { 
    nums = append(nums, 1) 
} 
if add2 {  
    nums = append(nums, 2) 
}
~~~

### 小变量作用域

1.  如果有可能，尽量缩小变量作用范围. 除非它与 [减少嵌套](#减少嵌套)的规则冲突. 

~~~go
if err := ioutil.WriteFile(name, data, 0644); err != nil {
    return err 
}
~~~

2.  如果需要在 if 之外使用函数调用的结果，则不应尝试缩小范围. 

~~~go
data, err := ioutil.ReadFile(name) 
if err != nil {   
    return err 
} 
if err := cfg.Decode(data); err != nil {  
    return err
} 
fmt.Println(cfg) 
return nil
~~~

### 避免参数语义不明确

1.  函数调用中的`意义不明确的参数`可能会损害可读性. 当参数名称的含义不明显时，请为参数添加 C 样式注释 (`/* ... */`)


~~~go
func printInfo(name string, isLocal, done bool) 
printInfo("foo", true /* isLocal */, true /* done */)
~~~

对于上面的示例代码，还有一种更好的处理方式是将上面的 `bool` 类型换成自定义类型. 将来，该参数可以支持不仅仅局限于两个状态（true/false）. 

```go
type Region int
const (
  UnknownRegion Region = iota
  Local
)
type Status int
const (
  StatusReady = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)
func printInfo(name string, region Region, status Status)
```

### 使用原始字符串字面值，避免转义

 " ` " 来表示原生字符串，在需要转义的场景下，我们应该尽量使用这种方案来替换. 

可以跨越多行并包含引号. 使用这些字符串可以避免更难阅读的手工转义的字符串. 

~~~go
wantError := `unknown error:"test"`
~~~

### 初始化 Struct 引用

在初始化结构引用时，请使用`&T{}`代替`new(T)`，以使其与结构体初始化一致. 

~~~go
sval := T{Name: "foo"} sptr := &T{Name: "bar"}
~~~

### 初始化 Maps

对于空 map 请使用 `make(..)` 初始化， 并且 map 是通过编程方式填充的.  这使得 map 初始化在表现上不同于声明，并且它还可以方便地在 make 后添加大小提示. 

~~~go
var (  
    // m1 读写安全;  
    // m2 在写入时会 panic 
    m1 = make(map[T1]T2)  
    m2 map[T1]T2 
)
~~~

在尽可能的情况下，请在初始化时提供 map 容量大小，详细请看 [尽量初始化时指定 Map 容量](#尽量初始化时指定-Map-容量). 

另外，如果 map 包含固定的元素列表，则使用 map literals(map 初始化列表) 初始化映射. 

~~~go
m := map[T1]T2{ 
    k1: v1,  
    k2: v2, 
    k3: v3, 
}
~~~

基本准则是

在初始化时使用 map 初始化列表 来添加一组固定的元素. 否则使用 `make` (如果可以，请尽量指定 map 容量). 

### 字符串 string format

如果你为`Printf`-style 函数声明格式字符串，请将格式化字符串放在外面，并将其设置为`const`常量. 

这有助于`go vet`对格式字符串执行静态分析. 

~~~go
const msg = "unexpected values %v, %v\n" 
fmt.Printf(msg, 1, 2) 
~~~

## 编程模式

### 表驱动测试

当测试逻辑是重复的时候，通过  [subtests](https://blog.golang.org/subtests) 使用 table 驱动的方式编写 case 代码看上去会更简洁. 

~~~go
func TestSplitHostPort(t *testing.T) { 
	tests := []struct{
		give     string
		wantHost string
		wantPort string
	}{
		{
			give:     "192.0.2.0:8000",
			wantHost: "192.0.2.0",
			wantPort: "8000",
		},
		{
			give:      "192.0.2.0:http",
			wantHost: "192.0.2.0",
			wantPort: "http",
		},
	}
	for _, tt := range tests {  
        t.Run(tt.give, func(t *testing.T) {   
         host, port, err := net.SplitHostPort(tt.give)    
            require.NoError(t, err)    
            assert.Equal(t, tt.wantHost, host)   
            assert.Equal(t, tt.wantPort, port)  
        }) 
    }
}
~~~

很明显，使用 test table 的方式在代码逻辑扩展的时候，比如新增 test case，都会显得更加的清晰. 

我们遵循这样的约定：将结构体切片称为`tests`.  每个测试用例称为`tt`. 此外，我们鼓励使用`give`和`want`前缀说明每个测试用例的输入和输出值. 

```go
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}
for _, tt := range tests {
  // ...
}
```

### 功能选项

功能选项是一种模式，您可以在其中声明一个不透明 Option 类型，该类型在某些内部结构中记录信息. 您接受这些选项的可变编号，并根据内部结构上的选项记录的全部信息采取行动. 

将此模式用于您需要扩展的构造函数和其他公共 API 中的可选参数，尤其是在这些功能上已经具有三个或更多参数的情况下. 

~~~go
type options struct {  
    timeout time.Duration  
    caching bool 
} // Option overrides behavior of Connect. 
type Option interface {  
    apply(*options) 
} 
type optionFunc func(*options) 

func (f optionFunc) apply(o *options) { 
    f(o) 
} 
func WithTimeout(t time.Duration) Option {  
    return optionFunc(func(o *options) {   
        o.timeout = t 
    })
}
func WithCaching(cache bool) Option {  
    return optionFunc(func(o *options) {    
        o.caching = cache 
    }) 
} // Connect creates a connection.
func Connect(addr string,  opts ...Option) (*Connection, error) {  
    options := options{    
        timeout: defaultTimeout,    
        caching: defaultCaching,  
    }   
    for _, o := range opts {    
        o.apply(&options)  
    }   // ... 
} // Options must be provided only if needed.
db.Connect(addr) 
db.Connect(addr, db.WithTimeout(newTimeout)) 
db.Connect(addr, db.WithCaching(false)) 
db.Connect(addr, db.WithCaching(false), db.WithTimeout(newTimeout)) 
~~~