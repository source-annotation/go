# runtime.Gosched()

需要先了解 Go 的 GMP 和调度器相关知识。 
  
---  

runtime.Gosched() 会强行把让前 g 的执行权让出，把当前 g 放回  schedt.runq(也就是全局 g 队列) 。 

首先让 m.g0 调用 `gosched_m` ，转而调用真正的 Gosched 实现 `func goschedImpl(gp *g)`
```go
// Gosched yields the processor, allowing other goroutines to run. It does not
// suspend the current goroutine, so execution resumes automatically.
func Gosched() {
	checkTimeouts()
	mcall(gosched_m)
}

// Gosched continuation on g0.
func gosched_m(gp *g) {
	if trace.enabled {
		traceGoSched()
	}
	goschedImpl(gp)
}
```

真正的 Gosched 实现如下：
```go
func goschedImpl(gp *g) {
	status := readgstatus(gp)
	if status&^_Gscan != _Grunning {
		dumpgstatus(gp)
		throw("bad g status")
	}
	casgstatus(gp, _Grunning, _Grunnable)
	dropg()
	lock(&sched.lock)
	globrunqput(gp)
	unlock(&sched.lock)

	schedule()
} 
```

casgstatus() - 通过 cas 把这个 g 从 running 变为 runnable 状态。        
dropg() - 移除当前 m 与 g 的关系，例如移除 m.curg = g 的关联。         
globrunqput() - 把当前 g 放回  schedt.runq(也就是全局 g 队列) 。    



# 例子

```go
package main
import (
	"fmt"
	"runtime"
)

func main() {
	go func() {
		for i := 0; i < 5; i++ {
			fmt.Println("go")
		}
	}()

	for i := 0; i < 2; i++ { 
		//runtime.Gosched()
		fmt.Println("hello")
	}
}
```

如果不启用代码里的 `runtime.Gosched`，则 `fmt.Println("go")` 没来得及执行， main.main 就结束了。  
如果启用了代码里的 `runtime.Gosched`，则 main 这个协程会主动让出 g 给其它的 g(也没啥其它了，就上面那个)，此时就会打印出 `go go go go go hello hello`。   
   
