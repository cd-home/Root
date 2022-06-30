### Map

Map的值是不可寻址的

Example

~~~go
type example struct {
	Name string
}

func TestModifyMapValue(t *testing.T) {
	var m = map[string]example{
		"one": {Name: "GodYao"},
	}
	m["one"].Name = "Mike" // compiler error
	t.Log(m)
}
~~~

原因是由于Value是值, m["one"] 拿到的是值的拷贝

解决办法, 采用指针

~~~go
func TestModifyMapValue(t *testing.T) {
	var m = map[string]*example{
		"one": {Name: "GodYao"},
	}
	m["one"].Name = "Mike" 
	t.Log(m)
}
~~~

或者新建对象覆盖

~~~go
func TestModifyMapValue2(t *testing.T) {
	var m = map[string]example{
		"one": {Name: "GodYao"},
	}
	m["one"] = example{Name: "Mike"}
	t.Log(m)
}
~~~

扩展说明

在Go中, 映射的设计保证一个映射值在内存允许的情况下可以加入任意个条目.  另外为了防止一个映射中为其条目开辟的内存段支离破碎, 官方标准编译器使用了哈希表来实现映射.  并且为了保证元素索引的效率, 一个映射值的底层哈希表只为其中的所有条目维护一段连续的内存段.  因此, 一个映射值随着其中的条目数量逐渐增加时, 其维护的连续的内存段需要不断重新开辟来增容, 并把原来内存段上的条目全部复制到新开辟的内存段上.  另外, 即使一个映射值维护的内存段没有增容, 某些哈希表实现也可能在当前内存段中移动其中的条目.  总之, 映射中的元素的地址会因为各种原因而改变.  如果映射元素可以被取地址, 则Go运行时（runtime）必须在元素地址改变的时候修改所有存储了元素地址的指针值.  这极大得增加了Go编译器和运行时的实现难度, 并且严重影响了程序运行效率.  因此, 目前, Go中禁止取映射元素的地址. 