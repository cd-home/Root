[TOC]

### sync.Mutex

》互斥锁, 保护代码边界

~~~go
// A Mutex is a mutual exclusion lock.
// The zero value for a Mutex is an unlocked mutex.
//
// A Mutex must not be copied after first use.
type Mutex struct {
	state int32
	sema  uint32
}
~~~

竞态检测

~~~go

~~~

~~~bash
$ go run -race mock_race.go
~~~

