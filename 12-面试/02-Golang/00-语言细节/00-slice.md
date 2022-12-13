[TOC]

### slice

1.  动态数组、依靠底层数组实现
2.  自动扩容、轻量

#### 底层结构

~~~go
type slice struct {
    array unsafe.Pointer  // 指向底层数组
    len   int             // 切片长度
    cap   int             // 底层数组容量
}
~~~

#### make

~~~go
s := make([]int, len, cap)
~~~

#### 扩容

~~~go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
    newcap = cap
} else {
    if old.cap < 1024 {
        newcap = doublecap
    } else {
        // Check 0 < newcap to detect overflow
        // and prevent an infinite loop.
        for 0 < newcap && newcap < cap {
            newcap += newcap / 4
        }
        // Set newcap to the requested cap when
        // the newcap calculation overflowed.
        if newcap <= 0 {
            newcap = cap
        }
    }
}
~~~

根据append的元素进行预估扩容: cap

如果预估的容量cap大于两倍的原始容量, 那么新的容量就是预估容量

~~~go
if cap > doublecap {
	newcap = cap
}
~~~

否则

~~~go
if old.cap < 1024 {
        newcap = doublecap
} else {
    // Check 0 < newcap to detect overflow
    // and prevent an infinite loop.
    for 0 < newcap && newcap < cap {
        newcap += newcap / 4
    }
    // Set newcap to the requested cap when
    // the newcap calculation overflowed.
    if newcap <= 0 {
        newcap = cap
    }
}
~~~
