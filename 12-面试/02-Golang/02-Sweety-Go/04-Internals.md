[TOC]

### Internals(原理)

#### String Internals

考点： convert b from []byte to string without causing memory copy

~~~go
func main() {
	var b = []byte("123")
    // 字节转换为字符串不会导致内存复制
	s := *(*string)(unsafe.Pointer(&b))
	b[1] = '4'
	fmt.Printf("%+v", s)
}
~~~

#### Slice Internals

考点：通过子切片获取原始切片

~~~go
const MM = 10
const NN = 5

func printOriginalSlice(subslice *[]int) {
	data := (*[MM]int)(unsafe.Pointer(((*reflect.SliceHeader)(unsafe.Pointer(subslice))).Data))
	fmt.Printf("original\t%p:%+v\n", data, *data)
}
func main() {
	slice := make([]int, MM)
	for i, _ := range slice {
		slice[i] = i
	}
	subslice := slice[0:NN]
	fmt.Printf("slice\t%p:%+v\n", &slice, slice)
	fmt.Printf("subslice\t%p:%+v\n", &subslice, subslice)
	printOriginalSlice(&subslice)
}
~~~

#### Defer overhead

考点：defer的开销 

~~~
runtime.deferproc and runtime.deferreturn cause context copy and retrieval on stack memory
~~~

#### Map malloc Threshold

考点：Map的malloc阈值

~~~
The default limit is 128.
It can be modified by changing the value of maxKeySize and maxValueSize in runtime.hashmap
~~~

#### runtime.newobject()

考点：runtime.newobject() 是做什么的？make与new是否总是调用runtime.newobject()

~~~
1. runtime.newobject() 用于分配堆内存
2. make和new当被内联或者优化时，不会调用runtime.newobject()
~~~

#### Go Bootstrapping

考点：简要描述go可执行文件的过程

~~~
1. 运行位于runtime下特定平台的程序集
2. runtime.args()：解析终端参数            
3. runtime.osinit()：初始化CPU内核          
4. runtime.schedinit()：
	初始化goroutine调度程序 堆栈内存 终端参数 环境变量 调试参数 gc GOMAXPROCS
5. runtime.mstart(): 启动gc监视器 导入依赖运行init 最后运行主程序
~~~

#### Unbuffered and Buffered Channels

考点：有缓冲与无缓冲通道

~~~
For unbuffered channel, the sender will block on the channel until the receiver receives the data from the channel, whilst the receiver will also block on the channel until sender sends data into the channel. 
Compared with unbuffered counterpart, the sender of buffered channel will block when there is no empty slot of the channel, while the receiver will block on the channel when it is empty.
~~~

#### Destructor

考点：析构函数

~~~
There is no destructor in go. But runtime.SetFinalizer() can set a callback function for a pointer.
在go中没有析构函数。但是运行时.SetFinalizer（）可以为指针设置回调函数。          
~~~

#### Garbage Collection

考点：垃圾回收

~~~
1. MarkWorker goroutine recursively scan all the objects and colors them into white(inaccessible), gray(pending), black(accessible). But finally they will only be black and white objcts.

2. In compile time, the compiler has already injected a snippet called write barrier to monitor all the modifications from any goroutines to heap memory.

3. When "Stop the world" is performed, scheduler hibernates all threads and preempt all goroutines.

4. Garbage collector will recycle all the inaccessible objects so heap or central can reuse.

5. If the whole span of memory are unused, it can be freed to OS.

6. Perform "Start the world" to wake cpu cores and threads, and resume the execution of goroutines.

1. MarkWorker goroutine递归地扫描所有的对象，并将它们涂成白色（不可访问）、
   灰色（  待定）、黑色（可访问）。但最终他们只会是黑白相间的目标。
2. 在编译时，编译器已经注入了一个名为write barrier的代码片段来监视从任何goroutine
   到堆内存的所有修改。
3. 当执行“Stop the world”时，调度器休眠所有线程并抢占所有goroutine。
4. 垃圾回收器将回收所有不可访问的对象，以便heap或central可以重用。
5. 如果整个内存段未使用，则可以将其释放给操作系统。
6. 执行“启动世界”来唤醒cpu核心和线程，并恢复goroutines的执行。
~~~

#### Goroutine Sleep

考点：goroutine sleep

~~~
1. C.sleep() 调用系统sleep 导致线程空闲
2. time.Sleep 针对goroutine优化，
~~~

#### Memory Allocation

考点：内存分配 go运行时如何分配内存的策略

~~~
For small objects(<=32KB), go runtime starts with cache firstly, then central, and finally heap.
For big objects(>32KB), directly from heap.

1. 对于小对象（<=32KB），go运行时首先从cache开始，然后是central，最后是heap
2. 对于大对象（>32KB），直接从heap
~~~

#### Stack vs Heap

考点：运行时何时从堆分配内存，何时从堆栈分配内存？

~~~
1. For small objects whose life cycle is only within the stack frame, 	
   stack memory is allocated.
2. For small objects that will be passed across stack frames, heap  
   memory.
3. For big objects(>32KB), heap memory.
4. For small objects that could escape to heap but actually inlined, 
   stack memory.
   
1. 对于生命周期仅在堆栈帧内的小对象，将分配栈内存
2. 对于将通过堆栈帧传递的小对象，分配堆内存
3. 对于大对象（>32KB），分配堆内存
4. 对于可以转义到堆但实际上内联的小对象，分配栈内存
~~~

#### Goroutine Pause or Halt

考点：暂停或者停止goroutine的函数

~~~
1. runtime.Gosched: give up CPU core and join the queue, thus will be executed automatically. 放弃当前CPU核心，加入队列，等待下次调度执行

2. runtime.gopark: blocked until the callback function unlockf in argument list return false.

3. runtime.notesleep: hibernate the thread. 休眠线程

4. runtime.Goexit: stop the execution of goroutine immediately and call defer, but it will not cause panic. 立即停止并且调用defer，但是不会panic
~~~
