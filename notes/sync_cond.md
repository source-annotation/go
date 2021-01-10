# sync.Cond 

先阅读： 

* [github 上关于从 go 中移除 Cond 的 issue](https://github.com/golang/go/issues/21165)


* sync.NewCond(l Locker) - 创建一个 cond 原语，必须传入一个实现了 Lock 与 Unlock 方法的 Locker  
* sync.Cond.L.Lock() 
* sync.Cond.L.Unlock() 
* sync.Cond.Wait() - 
* sync.Cond.Signal() - 
* sync.Cond.Broadcast() -

```go
// Cond implements a condition variable, a rendezvous point
// for goroutines waiting for or announcing the occurrence
// of an event.
//
// Each Cond has an associated Locker L (often a *Mutex or *RWMutex),
// which must be held when changing the condition and
// when calling the Wait method.
//
// A Cond must not be copied after first use.
type Cond struct {
	noCopy noCopy

	// L is held while observing or changing the condition
	L Locker

	notify  notifyList
	checker copyChecker
}
``` 




## 禁止拷贝 

noCopy : 嵌入一个 noCopy 结构体，表示"声明 Cond 不可以被拷贝"，这虽然不会让编译失败，但可以让 go vet 检测时暴露出被拷贝的问题。

在调用 Cond 的 Wait, Signal, Broadcast 方法时，都会先调用 `Cond.copyChecker.check()` 来检查 Cond 是否被拷贝了，如果被拷贝了直接 panic。 
```go
// copyChecker holds back pointer to itself to detect object copying.
type copyChecker uintptr

func (c *copyChecker) check() {
	if uintptr(*c) != uintptr(unsafe.Pointer(c)) &&
		!atomic.CompareAndSwapUintptr((*uintptr)(c), 0, uintptr(unsafe.Pointer(c))) &&
		uintptr(*c) != uintptr(unsafe.Pointer(c)) {
		panic("sync.Cond is copied")
	}
}
```  

