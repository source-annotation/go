# runtime.GOMAXPROCS() 


先了解下 procs 这个变量, 这个变量主要用于作为 src/runtime/proc.go 下的 `func procresize(nprocs int32) *p` 方法的参数。 

调度器初始化时，默认会把 procs 设置为 ncpu，也就是 cpu 的逻辑核心数(双核4线程的 ncpu = 4，详情可自己去查阅 ncpu 值的初始化)。 但如果当前设置了 GOMAXPROCS 这个环境变量, 则尝试用这个环境变量覆盖 procs。

最后，调用 procresize(procs) 确定最大的 p (gmp 的 p)数量。  
```go
func schedinit() {
    // ...
    procs := ncpu
    if n, ok := atoi32(gogetenv("GOMAXPROCS")); ok && n > 0 {
        procs = n
    }
    
    if procresize(procs) != nil {
        throw("unknown runnable goroutine during bootstrap")
    }
    // ... 
}
```

`runtime.GOMAXPROCS()` 的作用就是在运行时修改这个 procs 的值，并重新进行 procresize(procs)。看下源码： 

```go
// GOMAXPROCS sets the maximum number of CPUs that can be executing
// simultaneously and returns the previous setting. If n < 1, it does not
// change the current setting.
// The number of logical CPUs on the local machine can be queried with NumCPU.
// This call will go away when the scheduler improves.
func GOMAXPROCS(n int) int {
	if GOARCH == "wasm" && n > 1 {
		n = 1 // WebAssembly has no threads yet, so only one CPU is possible.
	}

	lock(&sched.lock)
	ret := int(gomaxprocs)
	unlock(&sched.lock)
	if n <= 0 || n == ret {
		return ret
	}

	stopTheWorld("GOMAXPROCS")

	// newprocs will be processed by startTheWorld
	newprocs = int32(n)

	startTheWorld()
	return ret
}
```

如果 arch 是 webassembly 的话，n会被强行覆盖为 1(关于 wasm 的判断，在 go 调度器源码里比比皆是)。 

先 stop the world，把参数赋值给 newprocs，再后面的 start the world 中会把 newprocs 赋值给 procs，最后再 start the world 里再调用一次 `procresize(procs)`。
 
> 注：注释上也说了，This call will go away when the scheduler improves。

我们来看看 procresize 做了什么(src/runtime/proc.go)：

todo

```go
// Change number of processors. The world is stopped, sched is locked.
// gcworkbufs are not being modified by either the GC or
// the write barrier code.
// Returns list of Ps with local work, they need to be scheduled by the caller.
func procresize(nprocs int32) *p {
	
}
```