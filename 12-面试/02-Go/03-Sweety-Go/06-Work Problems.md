[TOC]

### Go开发易错点

#### 切片容量问题

>   1.  子切片初始化的方法，如果没有指定容量，那就是起始位置start后一个到原始切片、数组的最后
>   2.  如果指定max，那么容量就是max-start

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

#### 初始化切片

>   make初始化对应类型的零值

~~~go
func main() {
	// make会初始化置0
	s := make([]int, 2)
	s = append(s, 1, 2, 3)
	fmt.Println(s)
}
~~~

#### 切片截取

>   注意这种情况下都是公用一个底层的数组，在没有重新进行内存分配的情况下

~~~go
func main() {
    // 记住他们都是同一个切片的View
	s := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
	s3 := s[2:6]
	s4 := s3[3:6]
	fmt.Println(s3) // [3, 4, 5, 6]
	fmt.Println(s4) // [6, 7, 8]
}
~~~

#### append

>   appned追加要注意两种情况，第一种是不定参数，第二种是扩容

~~~go
func main() {
	s1 := []int{1, 2, 3}
	s2 := []int{4, 5, 6}
	s1  = append(s1, s2...)
	fmt.Println(s1)
}
~~~

#### map、slice、chan都是引用传递的

>   1.  函数是值传递
>   2.  map slice chan 本身是引用类型

#### map的value修改

>   map的balue必须是整体修改

~~~go
type User struct {
	Name string
}
func main() {
	Info := make(map[int]User) // map的value本身是不可寻址的
	Info[1] = User{Name: "li"}
	//Info[1].Name = "jack"    //  要修改必须整体修改，不能修改里面的部分
	fmt.Println(Info)
}
// 修改为如下的形式
func main() {
	Info := make(map[int]*User)
	Info[1] = &User{Name: "li"}
	Info[1].Name = "jack"
	fmt.Println(Info)
}
~~~

#### map的key

>   1.  map是无序的，key不能重复
>   2.  key可以使数字、字符串、布尔值，通道、数组
>   3.  不能是切片、函数、nil、接口、map

#### 常量不允许获取地址

>   1.  常量无法寻址
>   2.  常量无法修改

#### type定义类型

~~~go
type MyInt int     // 基于一个类型定义一个新的类型
type AliaInt = int // 给一个类型起别名

func main() {
	var num1 int = 123
	var num2 MyInt = 123
	var num3 AliaInt = 123
	fmt.Println(num1 == num3)
	// fmt.Println(num1 == num2)
}
~~~

#### 初始化

>   1.  make用于 slice、map、chan
>   2.  new用于int、string、bool

#### 断言

>   只能是接口类型断言

~~~go
func Get() int {
	return 1
}
func main() {
	i := Get()
	switch i.(type) {  // 报错，只能是接口类型断言
	case int:
		fmt.Println("int")
	case string:
		fmt.Println("string")
	}
}
~~~

#### interface

>   1.  interface就是interface类型，不是其他的类型
>   2.  内部构造复杂的类型

#### interface{}

>   1.  第一关于同类型赋值问题, 同类型采用赋值
>   2.  第二是空接口接收任意类型，而不是空接口指针接收

~~~go
type S1 struct {}

func f(x interface{}) {}

func g(x *interface{}) {}

func main() {
	s := S1{}
	p := &s
	f(s) //A
	g(s) //B wrong
	f(p) //C
	g(p) //D wrong
}
~~~

#### 空值问题

>   1.  nil ===> map、slice、chan、inteface、function等空值
>   2.  string的空值 "" 注意

#### goto

>   1.  不能跳转到其他函数或者内层代码
>   2.  最好不要使用goto语句
