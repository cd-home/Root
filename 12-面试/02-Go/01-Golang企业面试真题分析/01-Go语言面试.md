[TOC]

### Golang企业面试真题分析

#### 第一部分 数据定义【结构体、常量、内存四区】

##### 函数返回值

1.  函数的返回值可以有多个
2.  函数的返回值有名那么所有的返回值都必须命名
3.  一般来说error作为最后一个返回值

~~~go
func Foo(a int, s string) (num int, err error) {
    // code
}
~~~

##### 结构体比较

1.  同类型完全相同的结构体可以比较
    *   成员顺序相同
    *   成员也必须可以比较，复合类型不能比较
2.  空结构体

~~~go
func main() {
	m := new(struct{})
	n := new(struct{})
	fmt.Println(m == n)     // true 发生逃逸
	fmt.Printf("%p\n", &m)  // 这两行导致逃逸， 如果没有则 false，如下
	fmt.Printf("%p\n", &n)

	// 没有逃逸
	k := new(struct{})
	j := new(struct{})
	fmt.Println( k == j )  // false
	
    // 地址本身就是 0 0
	x := struct {}{}
	y := struct {}{}
	fmt.Println(x == y)    // true
}
~~~

##### string与nil

1.  nil不能赋值给string类型
2.  string的空值使用""
3.  map，chan，slice，inteface，function可以用nil赋值

##### 常量

1.  常量定义必须初始化
2.  **常量位于全局区/数据区**
3.  **常量的地址不可访问**
4.  尽量减少使用全局变量

##### 数据类型

1.  数据类型本质，固定内存大小的别名
2.  数据类型作用，编译器预算对象或变量分配内存空间的大小

##### 内存四区

1.  栈
    *   空间较小，读写性能高，数据存放时间短
    *   存放函数参数、函数调用流程方法地址、局部变量，编译器自动分配和释放
    *   局部变量可能发生逃逸现象
2.  堆
    *   空间较大，数据存活时间长
    *   Golang自己决定对象是否分配（逃逸）到堆上，采用GC回收
3.  全局区
    *   静态全局变量
    *   常量区
4.  代码区，存放代码逻辑

#### 第二部分 数组与切片 【初始化、拼接】

##### 切片初始化和追加

1.  初始化

~~~go
func main() {
	sli := make([]int, 10)  // 会初始化为 [0 0 0 0 0 0 0 0 0 0]
	sli = append(sli, 1, 2, 3)
	fmt.Println(sli)        // [0 0 0 0 0 0 0 0 0 0 1 2 3]

	var sli2 []int
	sli2 = append(sli2, 1, 2, 3)
	fmt.Println(sli2)       // [1, 2, 3]
}
~~~

2.  追加

~~~go
sli3 := []int{4, 5 ,6}
sli2 = append(sli2, sli3...)  // 必须展开
fmt.Println(sli2)
~~~

3.  通过数组创建切片要注意多个切片共享底层数组情况

#### 第三部分 Map 【赋值、遍历】

1.  map初始化

~~~go
type User struct {
    Name string
}
m := make(map[string]User) 
// map key的value要修改必须整体修改，不能修改value的局部属性
m["first"] = User{Name: "liyao"}
// 推荐如下使用形式
m := make(map[string]*User)
~~~

2.  遍历

~~~go
// 获取某个value
if value, exist := m["key"]; exist {
    
}
type User struct {
	ID int
	Name string
	Age int
}
func main() {
	m := make(map[int]User)
	users := []User{
		{ID: 1, Name: "jack", Age: 20},
		{ID: 2 , Name: "mike", Age: 25},
	}
    // range 是 复制 users中的成员
	for _, user := range users {
		m[user.ID] = user
	}
	for k, v := range m {
		fmt.Println(k, "==>",v)
	}
}
~~~

#### 第四部分 interface 【iface、eface内部构造】

1.  接口赋值
    *   如果接收者是值接收者, 那么类型的值或者指针都能实现对应的接口, 也就是能赋值给对应的接口变量
    *   如果接收者是指针接收者, 那么只有指向类型的指针才能实现对应的接口

~~~go
type Notifier interface {
    Notify()
}
type User struct {
    Name string
    Email string
}
// 实现接口
func (u *User) Notify() {}
func SendNotification(n Notifier) {
    n.Notify()
}
func main() {
    u := User{"liyao", "qq.com"}
    // 由于实现的方法接口是指针接收者，所以只能是对象的指针赋值给接口变量
    SendNotification(&u)
}
~~~

2.  内部构造
    *   不管是空还是非空接口，接口类型都是复合类型不可能是nil，data可以为nil
    *   将一个对象【不论是否是nil】赋值给接口类型【如果可以】，**接口变量保存对象的类型和对象的指针**
        *   变量具有类型和值
        *   **所以看似将一个值为nil的对象赋值给接口，实际上接口保存了对象的类型和值**

~~~go
var user User = nil
var nf Notifier = user
~~~

~~~go
// 空接口是interface{}
type eface struct{ 
   _type *_type            // 对象类型
   data unsafe.Pointer     // 实际对象指针 
} 
~~~

~~~go
// 非空接口
type iface struct{ 
   tab *itab                // 类型信息 
   data unsafe.Pointer      // 实际对象指针 
} 
type itab struct{ 
   inter *interfacetype     // 接口类型 
   _type *_type             // 实际对象类型 
   fun   [1]uintptr         // 实际对象方法地址  数组
}
~~~

3.  空接口interface{}, 任何类型都实现了空接口，所以空接口可以可以接受任何的类型的对象
4.  *inteface{}作为形式参数，只能接收\*interface{}类型的实际参数, 他是一个指向接口的指针

#### 第五部分 channle 【通道特性】

1.  给nil的channle发送数据会阻塞
2.  从nil的channle接收数据会阻塞
3.  给关闭的channle发送会造成panic
4.  从一个关闭的channle接受数据，如果缓冲区为空，就会返回零值
5.  无缓冲channle同步的，有缓冲channle是异步的

#### 第六部分 WaitGroup 【同步竞速】

~~~go
// 注意 Add 和 Done 的位置
func main() {
	var wg sync.WaitGroup
	for i := 0; i < 10 ; i++ {
		wg.Add(1)  // 不要放在goroutine中，也就是不要放在同一个g中
		go func(num int) {
			defer wg.Done()
			fmt.Println(num)
		}(i)
	}
	wg.Wait()
}
~~~
