[TOC]

### Chapter3

#### for-loop

> 1. 允许指定迭代次数
> 2. 数组、切片、映射、字符串

形式1

~~~go
for i := 0; i < 10; i++ {
    
}
~~~

注意

> 1. i是一个本地临时变量，循环结束就会"消失"
> 2. break 退出循环
> 3. continue 跳出当次循环，进行下一次循环判断

形式2

~~~go
// while
for condition {
    
}

// forever-loop
for {
    
}

// do-while
for ok := true; ok; ok = someExpression {
    
}
~~~

形式3

> 1. 不需要知道长度
> 2. 返回值中有索引

~~~go
for k, v := range collection {
    
}
~~~

#### Slice

> 1. 切片的底层是数组, 不需要指定长度
> 2. 切片作为函数参数传递是传递引用（虽然函数是值传递，Copy切片的地址）
> 3. 切片的零值是nil

声明

~~~go
s := []int{1, 2, 3, 3}
~~~

make

> 注意：自动初始化为类型的零值

~~~go
s := make([]int, 20)
~~~

appned

> 注意：nil的切片是可以append的

~~~go
s := make([]int, 20)
s = append(s, 1)
log.Println(s)
s = append(s, []int{1, 2, 3}...)
log.Println(s)
~~~

subSlice

> 1. 获取同一个切片或者数组的sub时，共享同样的底层数组
> 2. 返回的切片指向底层数组

~~~go
s[1:3]
~~~

len and cap

> len获取元素个数、cap获取切片容量

~~~go
len(s)
cap(s)
~~~

字节切片

~~~go
bs := make([]byte, 1024)
~~~

copy

~~~go
a := []int{1, 2, 3, 4, 5, 6}
b := []int{7, 8, 9, 10}
copy(a, b)
log.Println(a)
log.Println(b)
~~~

sort

~~~go
sort.Slice(s, func(i, j int) bool {
	return s[i] > s[j]
})
~~~

#### Map

> nil Map 无法操作

~~~go
var m map[string]string
log.Println(m)
m["key"] = "value" // panic: assignment to entry in nil map
~~~

#### const

> 1. 常量值无法改变、数字、字符串、布尔
> 2. 通常是全局变量，不使用也不会报错
> 3. 常量定义无须声明类型，编译器自动判断
> 4. 常量最好定义到一个包中

~~~go
const VERSION = "1.15.5"
~~~

iota

> 1. 每定义一个常量，iota+1
> 2. 遇到定义一个const，iota重置0
> 3. iota是int类型可以参与运算

~~~go
const (
	zaro = iota
    one
    two
    three
)
const (
	B = 1 << (iota * 10)
	KB
	MB
	GB
)

const (
	a = iota
	_        // 跳过声明
	c
	_
	e
)
~~~

#### Pointer

> 1. 指针即是地址
> 2. &获取非指针变量的地址
> 3. \* 获取指针的值，也称为指针的解引用
> 4. 指针可以作为参数也可以作为返回值

#### Time and Date

1. 基本操作

~~~go
t := time.Now()
log.Println(t.Weekday(), t.Year(), t.Month(), t.Day())
log.Println(t.Unix()) // unix时间是1970􏳆1月1日以􏰌来的秒数
// 时间格式化
log.Println(t.Format("20060102150405"))
~~~

2. 指定时区

~~~go
loc , _ := time.LoadLocation("Asia/Shanghai")
localTime := t.In(loc)
log.Println(localTime)
~~~

3. 解析时间

> 解析字符串为时间对象，然后按照需要获取其中的变量，或者再次格式化为需要的字符串

~~~go
// 解析字符串为时间对象, 第一个参数是时间解析的一系列常量
d, _ := time.Parse(time.UnixDate, "Wed Feb 25 11:06:39 PST 2015")
log.Println(d.Hour(), d.Minute())
log.Println(d.Format("2006/01/02"))
~~~

4. 解析日期

~~~go
d, _ = time.Parse("02 January 2006", "12 January 2020")
log.Println(d.Year(), d.Month(), d.Day())
log.Println(d.Format("2006/01/02"))
~~~

5. Tips

> 特殊的时间格式化字符串 2006 01 02 15 04 05

~~~go
const Format = "2006/01/02 15:04:05"
~~~

