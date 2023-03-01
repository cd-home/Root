[TOC]

### Go开发易错点

#### 左括号问题

左括号必须严格放置在函数参数列表括号后面

~~~go
func main() {  
    fmt.Println("Hello World")
}
~~~

#### 切片容量问题

子切片初始化的方法，如果没有指定容量，那就是起始位置start后一个到原始切片、数组的最后

如果指定max，那么容量就是max-start

~~~go
func main() {
	s := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := s[2:3:4]
	s2 := s[2:3]
	fmt.Println(len(s1))  // 2 3  === 1
	fmt.Println(cap(s1))  // 2 -- 4  == 2

	fmt.Println(len(s2))
	fmt.Println(cap(s2))
}
~~~

#### 多行的array、map、slice、struct逗号

~~~go
func main() {
    a := []int{1, 2, 3}
    m := map[string]int{
        "one": 1,
        "two": 2,
    }
    m := map[string]int{
        "one": 1,
        "two": 2}
    s := User{
        Name: "li_yao",
        Age: 25,
    }
    s := User{
        Name: "li_yao",
        Age: 25}
    // } 放到最后单行最后不用加逗号
}
~~~

#### 未使用的变量

不允许出现未使用的局部变量

~~~go
var gV int  // 全局变量未使用可以不使用
func main() {
    var one int
    two := 2 // 未使用报错
    // fmt.Println(one) 
    fmt.Println(two)
}
~~~

#### 未使用import

未使用的import会报错, 可以使用 _ 或者 删除import的包

_ 会执行包里的init方法

~~~go
import (
	_ "fmt"   
    "log"
)
~~~

#### 简短声明

~~~go
// 简短声明位置
func main() {
    // 简短变量声明只能在函数中使用
    first := 1
}
// 简短声明设置字段值
type User struct {
    Name string
}
~~~

#### 简短声明重复变量

~~~go
func main() {
    first := 0
    // := 左边必须要有一个新的变量
    first, two := 1, 2 
}
~~~

#### 简短声明字段

~~~go
func main() {
    var u User
    var err error
    // 这种必须是用= 不能用 :=
    // u.Name, err := xxFunc()
    u.Name, err = xxFunc()
}
~~~

#### nil声明

显式类型的变量无法使用nil来初始化

~~~go
func main() {
    // var x = nil //错误, 必须要显式制定类型
    var x interface{} = nil
}
~~~

#### nil的slice和map

map必须初始化才能使用

slice必须初始化才能使用, 但是可以append

~~~go
func main() {
    var m map[string]int
    m["one"] = 1      //报错
    var s []int  
    s = append(s, 1)  // 正确, nil的slice可以append 
}
~~~

#### map容量

创建map可以指定容量; map的容量是动态增长的, 无法使用cap()

~~~go
func main() {
    m := make(map[string]string, 10)
}
~~~

#### string类型零值

string的零值是""; string不允许修改

~~~go
func main() {
    var s string
    if s == "" {
        fmt.Println(" string的零值是"" ")
    }
    // 注意并且 string类型是常量, 不允许修改
}
~~~

#### 数组作为函数参数

函数参数是值传递; 数组是值类型; 如想修改原函数, 可以传递数组指针, 或者使用切片

~~~go
func main() {
    arr := []int{1, 2, 3}
    func(arr []int) {
        arr[0] = 999
        fmt.Println(arr)
    }(arr)
    fmt.Println(arr)
}
~~~

#### range遍历集合返回两个值

返回索引和值;返回的值是复制

~~~go
func main() {
    arr := []int{1, 2, 3}
    for i, v := range arr {
        fmt.Println(i, v)
    }
}
~~~

#### map-OK模式取值

~~~go
func main() {
    m := map[string]int{
        "one": 1,
        "two": 2,
    }
    // map取值不会报错, 采用ok模式来判断
    if v, ok := m["one"]; ok {
        fmt.Println(v)
    }
}
~~~

#### range迭代string

~~~go
func main() {
    s := "liyao"
    subs := s[0]               // 返回的是byte值, uint8类型
    fmt.Printf("%T", subs)
    for i, v := range s {
        fmt.Printf("%T\n", v)  // v 是一个rune, uint32类型
        break
    }
}
~~~

#### 字符串长度

 ~~~go
func main() {
    s := "abc"
    fmt.Println(len(s)) // len返回的是字符串的byte数量
}
 ~~~

#### map是无序的

~~~go
func main() {
    m := map[string]int{
        "one": 1,
        "two": 2,
    }
    for k, v := range m {
        fmt.Println(k, v)
    }
}
~~~

#### switch-case

~~~go
func main() {
    isSpace := func(char byte) bool {
        switch char {
        	case " ":
            fallthrough
            case "\t":
            return true
        }
        return false
    }
}
~~~

#### ++ --

~~~go
func main() {
    for i:= 0; i < 10; i++ {
        // ++ -- 
        // 只有后置, 并且属于运算符而非表达式
    }
}
~~~

#### 操作符优先级

~~~go
Precedence   Operator
  5 			 * / % << >> & &^ 
  4 			 + - | ^
  3 			 == != < <= > >= 
  2            &&
  1            ||
~~~

#### 导出字段

导出字段才能被encode

~~~go
type User struct {
    Name string
    age int
}
func main() {
    u := User{
        Name: "li_yao",
        age: 25
    }
    encoded, _ := json.Marshal(u)
}
~~~

#### 向关闭的channle发送数据panic

~~~go
func main() {
	ch := make(chan int) 
    // 向 nil 的channle中发送或者接收数据会死锁 
    for i := 0; i < 3; i++ {
		go func(idx int) { 
        	ch <- idx
		}(i) 
    }
	fmt.Println(<-ch)  
    close(ch)  // 关闭后, 但是还向chan里面写数据, 就会panic
    time.Sleep(2 * time.Second) 
}
~~~

#### string和byte slice之间转换

参与转换的是拷贝的原始值; 如下两个方面的优化

~~~go
map[string]中查找key采用了[]byte, 避免了m[string(key)]内存分配
for range 迭代string转换为[]byte迭代 
~~~

#### 关闭HTTP响应体

~~~go
func main() {
    resp, err := http.Get("https://www.baidu.com")
    if resp != nil {
        defer resp.Body.Close()
    }
    if err != nil {
        return
    }
}
~~~

#### 关闭HTTP连接

标准库的默认服务端主动要求关闭才断开链接

如果需要向同一服务器发起大量请求, 使用长连接

~~~go
func main() {
    req, err := http.NewRequest("GET", "https://www.baidu.com", nil)
    if err != nil {
        return
    }
    req.Close = true // req.Header.Add("Connection", "close")
    resp, err := http.DefaultClient.Do(req)
}
// 或者创建自定义client取消全局
func main() {
    tr := http.Transport{DisableKeepAlive: true}
    client := http.Client{Transport: tr}
    resp, err := client.Get("https://www.baidu.com")
}
~~~

#### JSON解码为interface{}

解码数字默认为float64

~~~go
func main() {
    var data = []byte(`{"status": 200}`)
    var result = map[string]interface{}
    if err := json.Unmashal(data, &result); err != nil {
        log.Fatalln(err)
        return
    }
    var status = (result["status"]).(float64)
    // var status = uint64((result["status"]).(float64))
}
// 指定字段类型
func main() {
    var data = []byte(`{"status": 200}`)
    var result = map[string]interface{}
    var decoder = json.NewDecoder(bytes.NewReader(data))
    decoder.UserNumber()
    if err := decoder.Decode(&result); err != nil {
        return
    }
    var status, _ = result["status"].(json.Number).Int64()
}
~~~

#### == 

同struct 当成员可以比较时才能比较

同数组 当成员可以比较时才能比较

slice 无法比较

map无法比较

~~~go
reflect.DeepEqual()  // nil和空slice不相等
~~~

#### panic恢复

~~~go
func main() {
    defer func() {
        if err := recover(); err != nil {
            
        }    
    }()
    panic("panic")
}
~~~

#### range迭代是复制值

~~~go
func main() {
    data := []int{1, 2, 3}
    for i, v := range data {
        data[i] = v * 10     // 需要索引修改
    }
    info := []*struct{ num int}{{1}, {2}, {3}}
    for _, item := range info {
        item.num *= 10       // 指针可以直接修改
    }
}
~~~

#### slice隐藏数据

通过旧slice创建新slice公用底层数组; 注意旧slice的数据

~~~go
func Get() (res []byte) {
    raw := make([]byte, 10000)
    res = make([]byte, 5)
    copy(res, raw[:5])   // 采用copy的方式, 最好不要用共享的形式
    return
}
~~~

#### type不会继承

~~~go
type myMutex sync.Mutex
func main() {
    var mtx myMutex
    mtx.Lock()  // error
    mtx.UnLock()
}
// 采用内嵌
type myLocker struct {
    sync.Mutex
}
~~~

#### 跳出for-switch和for-select

~~~go
func main() {
loop:
    for {
        switch {
            case true:
            break loop
        }
    }
    fmt.Println("out")
}
~~~

#### for语句迭代变量

~~~go
func main() {
    data := []string{"one", "two", "three"}
    for _, v := range data {
        go func() {
            fmt.Println(v)  // 会全部打印three
        }()
    }
    time.Sleep(3 * time.Second)
}
// 解决办法
func main() {
    data := []string{"one", "two", "three"}
    for _, v := range data {
        go func(v string) {
            fmt.Println(v) 
        }(v)
    }
    time.Sleep(3 * time.Second)
}
~~~

#### defer函数的参数

defer函数的参数会在声明时就会执行

~~~go
func main() {
    var i = 1
    defer fmt.Println("result: ", func() int { return i * 2})
    i++
    // result: 2
}
~~~

#### defer执行时机

调用defer的函数执行结束时执行

#### 断言

避免断言数据与接口变量名一样, 引起错误

~~~go
func main() {
    var data interface{} = "greeter" 
    if res, ok := data.(int); ok {
        
    } else {
        
    }
}
~~~

#### goroutine资源泄漏

避免开启大量的g, 而不知道他们什么时候停止

开启的g需要知道他的生命周期, 避免资源泄漏

#### 更新map的值

map的值是一个结构体必须整体更新; map的元素是不可寻址的

~~~go
type User struct {
    Name string
}
func main() {
    user := User{Name: "liyao"}
    m := make(map[string]User)
    m["1"] = user
    // m["1"].Name = "jack" // error
    m["1"] = User{Name: "jack"} // right
    // or
    newUser := m["1"]
    newUser.Name = "jack"
    m["1"] = newUser
}
// or 使用指针
func main() {
    m := make(map[string]*User)
    user := &User{Name: "liyao"}
    m["1"] = user
    m["1"].Name = "Tom"
}
~~~

#### 堆栈分配

new make创建的变量, 内存分配由编译器决定; 逃逸分析

#### 并发

Go默认执行使用的CPU核心数为系统CPU最大核心; P的数目=CPU核心数目

~~~go
runtime.NumCPU()      // 返回CPU核心数
runtime.GOMAXPROCS(n) // 设置P的数目, 默认P的数目是CPU核心数目
n = 1
n < 1
n > 1
~~~
