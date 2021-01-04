# Goroutine scheduler  

Go 里所有的代码都在 goroutine中执行(包括 main.main) ，Goroutine 的调度属于 go runtime 的工作。  

推荐学习资料： 

* https://speakerdeck.com/retervision/go-runtime-scheduler   

## 调度器的初始化  


```go
// src/runtime/runtime2.go
type schedt struct {
    lock mutex
    
    midle        muintptr // idle m's waiting for work(空闲的m列表) 
    
    pidle      puintptr // idle p's(空闲的p列表)
    
    
    // Global runnable queue.
    runq     gQueue  // 全局 g 队列 
    
    deferpool [5]*_defer
}
``` 

src/runtime/proc.go


```go
// Goroutine scheduler
// The scheduler's job is to distribute ready-to-run goroutines over worker threads.
//
// The main concepts are:
// G - goroutine.
// M - worker thread, or machine.
// P - processor, a resource that is required to execute Go code.
//     M must have an associated P to execute Go code, however it can be
//     blocked or in a syscall w/o an associated P.
```
调度器初始化，函数注释已经写清楚了 bootstrap 顺序： 

```go
// The bootstrap sequence is:
//
//	call osinit    // 调用 osinit() - 一些硬件资源，操作系统信息初始化 
//	call schedinit // 调用 schedinit() - 初始化 go 的调度器
//	make & queue new G  // 第一个 g 
//	call runtime·mstart // mstart
//
// The new G calls runtime·main.  // 初始化 main.main 这个 goroutine，也就是主 goroutine  
func schedinit() {
	
}
```

main goroutine :  

```go
// The main goroutine.
// 几个重点操作 
//   1. sysmon(一个用于监控的 M，它不需要P)  
func main() {
	
    // new一个用于监控的m，这个m不需要 P 所以第二个参数传 nil 
    newm(sysmon, nil)
}
```

sysmon(用于监控的 M) : 

```go
// Always runs without a P, so write barriers are not allowed.
//
//go:nowritebarrierrec
func sysmon() {}
```

## 一次调度的过程 

```go
// src/runtime/proc.go  

// One round of scheduler: find a runnable goroutine and execute it.
// Never returns.
func schedule() {
	
}
```


## g 之间的 channel 通信 

问个问题。 如果 GOMAXPROCS = 4，但是有5个goroutine 之间在用 channel 通信(也就是说至少有一个 g 肯定不是某个 m 的 curg(current running g))，那 GMP 针对这种情况是怎么调度的？

看看 sudog :   
```go
type sudog struct {}
```

## m 的阻塞  

被 park 的 g 处于 waiting 状态，放弃 cpu。  

runtime.Gosched() 会让当前所在 g 放弃 cpu(强行让出cpu)，与 park 不同的是 Gosched() 会把当前所在 g 放入全局队列使之处于 runnable 状态。  

parking/noparking


## Syscall 

系统调用时，可能会阻塞。 所以 Go 中有2种系统调用函数：`Syscall()` 和 `RawSyscall()`。

`Syscall()` 在真正地去调用系统函数之前会先调用 `runtime.entersyscall`，好让这次系统调用可以被 go 的调度器调度。若此时 `Syscall()` 阻塞了，调度器会判断要不要把这个 M 上的 P 挪给其他的 M。

`RawSyscall()` 只是为了在执行那些一定不会阻塞的系统调用时，能节省两次对 runtime 的函数调用消耗。

这部分看曹大的 [Go 系列文章6: syscall](https://xargin.com/syscall/) 就够了。 