[TOC]

### Language Specification(语言特性)

#### SubSlice

考点：切片的扩容，切片共享底层数组

~~~go
func main() {

	s := []int{1, 2, 3}     //                len = 3 cap = 3
	ss := s[1:]				// ss = 2, 3      len = 2 cap = 2
	ss = append(ss, 4)      // ss = 2, 3, 4   len = 3 cap = 4

	for _, v := range ss {  // ss copy
		v += 10
	}

	for i:= range ss {
		s[i] += 10			// s = 11, 12, 13
	}

	fmt.Println(s)
}
~~~

#### Short Declairation

考点：短声明，重复声明；短变量声明左边必须要有一个没有声明过的

~~~go
// 前提 x 已经声明过，下面表达式是否正确？
x, _ := f()  // 错误
x, _ = f()   // 正确
x, y := f()	 // 正确
x, y = f()   // 错误
~~~

#### Nil Interface

考点：接口赋值；接口类型

实现接口的实例可以赋值给接口类型, 空接口可以接受任意类型接口类型可以和nil类型比较

~~~go
package main

type IF interface {
	F()
}

type S struct {}
func (s S) F()  {}


func initType() S {
	var s S
	return s
}

func InitPointer() *S {
	var s *S
	return s
}

func initefaceType() interface{} {
	var s S
	return s
}

func initeFacePointer() interface{} {
	var s *S
	return s
}

func initfaceType() IF {
	var s S
	return s
}

func initfacePointer() IF {
	var s *S
	return s
}

func main() {
	println(initType() == nil)         // error
	println(InitPointer() == nil)      // true
	println(initefaceType() == nil)    // false
	println(initeFacePointer() == nil) // false
	println(initfaceType() == nil)     // false
	println(initfacePointer() == nil)  // false
}
~~~

#### Map Ok-ldiom

考点：Map的OK模式；Map必须Make初始化才可以使用

~~~go
func main() {
	//var m map[string]int  nil map
	m := make(map[string]int)
	m["a"] = 1
    // ok模式
	if v, ok := m["b"]; ok {
		println(v)
	}
}
~~~

#### Pointers

考点：函数返回指针

~~~go
type S1 struct {
	m string
}

func f() *S1 {
	return &S1{m: "foo"}
}

func main() {
	p := f()
	println(p.m)
}
~~~

#### Interface{} Pointers

考点：空接口与空接口指针

~~~go
type S2 struct {}

func f1(i interface{}) {

}
func g1(i *interface{})  {

}
// cannot use s (type S2) as type *interface {} in argument to g1:
// *interface {} is pointer to interface, not interface
func main() {
	s := S2{}
	p := &s
	f1(s)
	f1(p)
	g1(s) 
	g1(p)
}
~~~

#### Temporary Pointer

考点：临时指针

~~~go
const N  = 3

func main() {
	m := make(map[int]*int)
	for i := 0; i < N; i++ {
        // j := int(i)
        // m[i] = &j
		m[i] = &i
	}
	// m[i] = &i 每一项的value都是临时指针&i，最后i=3
	for _, v := range m {
		println(*v)
	}
}
~~~

#### Break Outer Loop

考点：label强制跳出

~~~go
func main() {
outer:
	for i := 0; i < 3; i++ {
		for j := 0; j < 3; j++ {
			print(i, ",", j, " ")
			break outer
		}
		println()
	}
}
~~~

#### Global Varibles

考点：全局变量声明

~~~go
var g
var g G      // 正确
var g = G{}  // 正确
g := G{}
~~~

#### Defer Stack

考点：Defer延迟调用

~~~go
import (
	"io/ioutil"
	"os"
)

func main() {
	f, err := os.Open("file")
	if err != nil {
		return
	}
    // 必须判断f ？ nil
	defer f.Close()

	b, err := ioutil.ReadAll(f)
	println(string(b))
}
~~~

#### Panic Stack

考点：defer延迟调用函数；panic中断

~~~go
func f4() {
	defer println("f1-begin")
	f1()
	defer println("f1-end")
}

func f2() {
	defer println("f2-begin")
	f3()
	defer println("f2-end")
}

func f3() {
	defer println("f3-begin")
	panic(0)
	defer println("f3-end")
}

func main() {
	f1()
    // f3-begin
	// f2-begin
	// f1-begin
	// panic: 0
}
~~~

#### Recover

考点：panic；recover从panic恢复

~~~go
func bar()  {
	defer func() {
		if r := recover(); r != nil {
			log.Printf("%v", r)
		}
	}()
	panic(1)
	panic(2)
}
func main() {
	bar()
}
~~~

#### Goroutine Closure

考点：闭包

~~~go
const M  = 10
func main() {

	m := make(map[int]int)
	wg := &sync.WaitGroup{}
	mu := &sync.Mutex{}

	wg.Add(M)

	for i := 0; i < M; i++ {
		// 如果不传递i，那么最后所有的g的i都相同
		go func(i int) {
			defer wg.Done()
			mu.Lock()
			m[i] = i
			mu.Unlock()
		}(i)

	}
	wg.Wait()
	println(len(m))
}
~~~

#### Type Shadowing

考点：匿名（内嵌）结构体；方法、接口会被提升

~~~go
type S1 struct{}

func (s1 S1) f() {
	fmt.Println("S1.f()")
}
func (s1 S1) g() {
	fmt.Println("S1.g()")
}
type S2 struct {
	S11
}
func (s2 S2) f() {
	fmt.Println("S2.f()")
}
type I interface {
	f()
}
func printType(i I) {
	fmt.Printf("%T\n", i)
	if s1, ok := i.(S1); ok {
		s1.f()
		s1.g()
	}
	if s2, ok := i.(S2); ok {
		s2.f()
		s2.g()
	}
}
func main() {
	printType(S1{})
	printType(S2{})
}
~~~

#### String to Bytes

考点：字符串与bytes

~~~go
func main() {
	s := "123"
	ps := &s
	
	b := []byte(*ps) // 新的内存空间
	pb := &b

	s += "4"		// 1234
	*ps += "5"      // 12345 ps指向s
	
	b[1] = '0'

	println(*ps) 		 // 12345
	println(string(*pb)) // 103
}
~~~

#### Map Mutex

考点：Map不是并发安全的

~~~go

package main

import (
	"math/rand"
	"sync"
)

const N = 10

func main() {
	m := make(map[int]int)
	wg := &sync.WaitGroup{}
	mu := &sync.Mutex{}
	wg.Add(N)
    
	for i := 0; i < N; i++ {
        
		go func() {
			defer wg.Done()
			mu.Lock()
			m[rand.Int()] = rand.Int()
			mu.Unlock()
		}()
        
	}
    
	wg.Wait()
	println(len(m))
}
~~~

#### DeepEqual

考点：类型相同

~~~go
type S3 struct {
	a, b, c string
}

func main() {
	x := interface{}(&S3{"a", "b", "c"})
	y := interface{}(&S3{"a", "b", "c"})

	fmt.Println(x == y) // false
	
	fmt.Println(reflect.DeepEqual(x, y)) // true
}
~~~

#### Gosched

考点：P执行器个数设置; 让出执行权限

~~~go
const N1 = 26

func main() {
	const GOMAXPROCS = 1
	runtime.GOMAXPROCS(GOMAXPROCS)

	var wg sync.WaitGroup
	wg.Add(2 * N1)
	for i := 0; i < N1; i++ {
		go func(i int) {
			defer wg.Done()
			//runtime.Gosched()
			fmt.Printf("%c", 'a'+i)
		}(i)
		go func(i int) {
			defer wg.Done()
			runtime.Gosched()
			fmt.Printf("%c", 'A' + i)
		}(i)
	}
	wg.Wait()
}
~~~

#### Map Immutability

考点：映射不变形；map的value必须整体修改

~~~go
type S struct {
	name string
}

func main() {
	m := map[string]S{"x": S{"one"}}
	// m["x"].name = "two"
    m["x"] = S{name: "two"}
}
~~~

#### Slice Sorting

考点： 切片排序

~~~go
type S4 struct {
	v int
}
func main() {
	s := []S4{{1}, {3}, {5}, {2}}
	sort.Slice(s, func(i, j int) bool {
		return s[i].v < s[j].v
	})
	fmt.Printf("%#v", s)
}
~~~
