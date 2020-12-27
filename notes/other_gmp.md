# GMP  

src/runtime/runtime2.go  


# G  

go gmp 源码里，大写 G 指的是全局队列里的 goroutine，小写 g 指的是 p 本地队列里的 goroutine。  

```go
// src/runtime/runtime2.go
type g struct {
    // ...
    
    // 每个 g 都要有自己的栈，这个 sched 维护了栈地址、程序计数器等基本调度信息。 
    // sched 可以理解为这个 g 的上下文(保存的现场)   
    // 让出 cpu 时需要保存 sched，下次获得 cpu 时要把 sched 载入到 cpu 寄存器里   
    sched  gobuf 
}
```
创建一个 g :  
```go
// src/runtime/proc.go  
func newproc(siz int32, fn *funcval) {}
```

从本地 p 队列取一个 g :  
```go
// src/runtime/proc.go

// Get g from local runnable queue.
func runqget(_p_ *p) (gp *g, inheritTime bool) {}
```

从全局 p 队列取一个 g : 
```go
// src/runtime/proc.go

// Try get a batch of G's from the global runnable queue.
// Sched must be locked.
func globrunqget(_p_ *p, max int32) *g {}
```

# M 
```go
type m struct {
    // ...	
}
```

创建一个 m (需要绑定一个 P，当然也有 M 不需要 P的例子比如 sysmon) :     

```go
// src/runtime/proc.go  
// 本质上是调用 syscall 的 clone() 创建一个 KSE(内核线程)  
func newm(fn func(), _p_ *p) {}
```

```go
// 停止 p，放入空闲 m 列表中(schedt.midle)。 
// src/runtime/proc.go  
func stopm() {}
```

```go
// src/runtime/proc.go  

// 把 p 和 m 分离
// Disassociate p and the current m.
func releasep() *p {)
```

如果 m 的本地 p 队列没有 g 了，就会想法子去找一个 g :  
```go
// Finds a runnable goroutine to execute.
// Tries to steal from other P's, get g from local or global queue, poll network.
func findrunnable() (gp *g, inheritTime bool) {
    // local runq 
    // 从本地p队列拿 
    if gp, inheritTime := runqget(_p_); gp != nil {
        return gp, inheritTime
    }
    
    // ... 
    
    // global runq 
    // 从全局 p 队列拿 
    if sched.runqsize != 0 {
        // ... 
    }
    
    // Steal work from other P's.
    // 从其他 p 的本地队列里偷 
    
stop:
    // ... 
    // 哪里都找不到可用的 g，直接让 m sleep （即把 m 放入睡眠列表里)
    stopm()
    
    // ... 
    
    // 把这个 m 的 p 释放掉，并放到空闲 p 列表里  
    if releasep() != _p_ {
        throw("findrunnable: wrong p")
    }
    pidleput(_p_)
}
```

```go
// src/rumti,e/proc.go

// Schedules gp to run on the current M.
// m 要开始执行 goruntine 了
func execute(gp *g, inheritTime bool) {}
```

# P  
```go
type p struct {
    // ...
}
```

```go 
// m 把自己的 p 拱手让给其他的 m 的操作，叫 hand off p。  
// Hands off P from syscall or locked M.
// Always runs without a P, so write barriers are not allowed.
//go:nowritebarrierrec 
func handoffp(_p_ *p) {}
```

```go
// 在很多情况下(比如 sysmon)，会去检查当前是不是有闲置的 p。  
// 如果有闲置的 p 说明一定可以找到(创建一个新的or从空闲m队列) 一个 m 来使用这个闲置的 p。 因为 m 和 p 是一一对应的嘛  
// Tries to add one more P to execute G's.
// Called when a G is made runnable (newproc, ready).
func wakep() {
    if atomic.Load(&sched.npidle) == 0 {
        return
    }
    // be conservative about spinning threads
    if atomic.Load(&sched.nmspinning) != 0 || !atomic.Cas(&sched.nmspinning, 0, 1) {
        return
    }
    startm(nil, true)
}
```




