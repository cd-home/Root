[TOC]

## 问题3: 逃逸分析？

### 变量的逃逸现象？

1.  C/C++ 函数返回局部地址给外界会报错
2.  Go函数返回局部地址给上层函数，变量发生逃逸现象，并没有分配到栈上

### 编译器逃逸分析？

~~~go
package main
func Bar(num int) *int {
	var barValue1 = 1
	var barValue2 = 2
	var barValue3 = 3
	var barValue4 = 4
	// 防止内联优化
	for i := 0; i < 5; i++ {
		println(&num, &barValue1, &barValue2, &barValue3, &barValue4)
	}
	// 返回局部变量地址，runtime.newObject() 逃逸
	return &barValue3
}
func main() {
	mainValue := Bar(666)
	println(*mainValue, mainValue)
}
// go tool compile -m 07-escape.go
// go tool compile -S 07-escape.go
~~~

### new分配在栈还是堆？

1.  Golang中new开辟的不一定是在栈上的
2.  Golang函数局部变量定义的变量，不论是普通定义还是new，分配在堆还是栈我们无法决定

~~~go
package main
func Bar(num int) *int {
	var barValue1 = new(int)
	var barValue2 = new(int)
	var barValue3 = new(int)
	var barValue4 = new(int)
	// 防止内联优化
	for i := 0; i < 5; i++ {
		println(&num, barValue1, barValue2, barValue3, barValue4)
	}
	// 返回局部变量地址，runtime.newObject() 逃逸
	return barValue3
}
func main() {
	mainValue := Bar(666)
	println(*mainValue, mainValue)
}
~~~

### new or make？

1.  new为简单类型分配空间，返回类型的地址，初始值为0。在栈还是堆，有逃逸现象。new不常用。
    *   int、float
    *   string
    *   bool
2.  make为复合类型分配空间，返回复合类型引用，也就是类型本身。常用，无可替代。
    *   chan
    *   slice
    *   map

