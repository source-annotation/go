# GMP  

src/runtime/runtime2.go  

# Quick look

如果你已经了解 GMP，想要快速查阅一些机制在源码中的位置，可以看这个标题下的内容。  

### 寻找可执行的 g   

如果当前 m 的 g 执行完了，如何去找下一个 g 来执行。 一轮调度(`schedule()`) 的目的就是找一个 g 来执行。  

src/runtime/proc.go 
```go
func findrunnable() (gp *g, inheritTime bool) {
    // local runq(从自己的本地队列，也就是 p.runq 里找可执行 g，最后返回一个 g)  
    runqget()
	
    // global runq(从全局队列也就是 schedt.runq 里找可一堆执行 g，然后放入自己的 p.runq，最后返回一个 g)
    globrunqget()
    
    // net poller 
    // todo  
    
    // Steal work from other P's.
    // 从其它的 m.p.runq 里偷 g(一次偷 p.runq 里的一半 g)，然后放入自己的 p.runq，最后返回一个 g 
    runqsteal()
}
```
### work sharing 与 work stealing  

**work sharing(协作)** : m block/syscall 时， 主动把自己的 p 让出给其他 m( 见: src/runtime/proc.go 的 `handoffp` 函数)。   
**work stealing(窃取)** : m 执行完了最后一个 g，发现无论 local queue(p.runq) 还是 global queue(schedt.runq) 都没有 g 可执行了，会尝试从其它 m.p.runq 里偷一半到自己的 p.runq (见 ： src/runtime/proc.go 的 `runqsteal` 函数) 。  
  

### m 的状态流转  

* spinning  
* newm 
* exitm  

### g 的状态流转  


### p 的状态流转 

* newproc   
* wakep

### 什么情况下 m 没有对应的 p


有 3 种情况 m 没有 p :  

* m block/syscall ，此时 m 的 p 会被 hand off 给其他空闲 m ，自己等待阻塞 or Syscall 结束后才会自己去另找一个 P。  
* 在 midle 里的 m 处于空闲状态没事可干，下场只有2个：等待被召回 or 空闲太久了被销毁。     
* 还有一些特殊的 m  是没有 p的，比如 sysmon, templateThread。

### m.g0 

m.g0 干哪些事情？ 

### mcall

src/runtime/stubs.go (好好看看 TODO)

```go
// mcall switches from the g to the g0 stack and invokes fn(g),
// where g is the goroutine that made the call.
// mcall saves g's current PC/SP in g->sched so that it can be restored later.
// It is up to fn to arrange for that later execution, typically by recording
// g in a data structure, causing something to call ready(g) later.
// mcall returns to the original goroutine g later, when g has been rescheduled.
// fn must not return at all; typically it ends by calling schedule, to let the m
// run other goroutines.
//
// mcall can only be called from g stacks (not g0, not gsignal).
//
// This must NOT be go:noescape: if fn is a stack-allocated closure,
// fn puts g on a run queue, and g executes before fn returns, the
// closure will be invalidated while it is still executing.
func mcall(fn func(*g))
```


### sysmon  





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




