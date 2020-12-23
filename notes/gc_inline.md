# inline(内联)  

请先阅读： 

* [cmd/compile: enable mid-stack inlining #19348](https://github.com/golang/go/issues/19348)  
* [\[Slide\] Mid-stack inlining in the Go compiler](https://docs.google.com/presentation/d/1Wcblp3jpfeKwA0Y4FOmj63PW52M_qmNqlQkNaLj0P5o/edit#slide=id.p)
* [Proposal: Mid-stack inlining in the Go compiler](https://go.googlesource.com/proposal/+/master/design/19348-midstack-inlining.md)    
* [Mid-stack inlining in Go](https://dave.cheney.net/2020/05/02/mid-stack-inlining-in-go)   

inline 是现代编译器的常规优化手段。 

优点是：

* 减少函数调用开销  
* 编译器对内联后的 ast node 还能继续优化。 比如：
    * 编译器发现内联后还能顺手消除死代码。  
    * [talk: Mid-stack inlining in the Go compiler (external)](https://docs.google.com/presentation/d/1Wcblp3jpfeKwA0Y4FOmj63PW52M_qmNqlQkNaLj0P5o/edit#slide=id.g1b2157b5d1_0_6) 里举了一个例子：把循环中的不变变量计算提到外部。 

这2个优点很诱人，可以把所有函数内联吗？  不能，内联会增加编译时间和 binary 体积。 试想如果你用 Go 去编译 wasm，你能接受一个输出 hello world 的 wasm 几十 MB 吗？ 


## 内联处于编译的哪个阶段 ? 

仅 go compiler 而言。

## 可内联与不可内联

执行  `go tool compile -m=2 xxx.go` 命令来编译，就会输出哪些函数可内联，哪些不可内联，以及对应的原因。非常直观。   


能否内联的判断在 src/cmd/compile/internal/gc/inl.go 中的 `func caninl(fn *Node)` 方法。


`caninl` 这个函数根据一个 func 的 ast node 来判断这个 func 能否被内联。它是非黑即白的。即，排除掉无法被内联的，剩下的都是可以内联的。 所以我们只要研究哪些情况无法内联即可。

#### #1 AST Node超过80无法被内联  

根据抽象语法树节点的数量来判断。  

```go
const (
	inlineMaxBudget = 80  // 可内联 node 的最大预算是 80 
)

func caninl(fn *Node) {
	// ... 
	
 	visitor := hairyVisitor{
 		budget:        inlineMaxBudget,
 		extraCallCost: cc,
 		usedLocals:    make(map[*Node]bool),
 	}
 	if visitor.visitList(fn.Nbody) {
 		reason = visitor.reason
 		return
 	}
 	if visitor.budget < 0 {
 		reason = fmt.Sprintf("function too complex: cost %d exceeds budget %d", inlineMaxBudget-visitor.budget, inlineMaxBudget)
 		return
 	}

 	// ... 
}
```   

#### #2 写代码的人主动要求不内联 

go compiler 提供了一个 `//go:noinline` 的编译指令。 这个指令告诉编译器不要内联这个方法。

比如: 
```go
//go:noinline
func max(a,b int) int {
	if a > b {
		return a
	}
	return b
}
```

TODO : 在 go 源码搜索 `//go:noinline` 会发现许多 small function 要求不内联，为什么？

#### #3 编译的时候禁用内联 

这个没啥好讲的，-l -N 大家应该都很熟了。  

## 为了内联而外联 

举个 sync.Mutex.Lock() 的例子：
```go
// Lock locks m.
// If the lock is already in use, the calling goroutine
// blocks until the mutex is available.
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		if race.Enabled {
			race.Acquire(unsafe.Pointer(m))
		}
		return
	}
	// Slow path (outlined so that the fast path can be inlined)
	m.lockSlow()
}
```

看到 `m.lockSlow()` 上面的注释了没有：

> // Slow path (outlined so that the fast path can be inlined)

把没抢到锁的 goroutine 的下次竞争逻辑给封装成 lockSlow 方法(即 outline)。 这样做是为了让 fast path 可以被内联。

可以看一下 mid-stack inlining : 

* [cmd/compile: enable mid-stack inlining #19348](https://github.com/golang/go/issues/19348)  
* [\[Slide\] Mid-stack inlining in the Go compiler](https://docs.google.com/presentation/d/1Wcblp3jpfeKwA0Y4FOmj63PW52M_qmNqlQkNaLj0P5o/edit#slide=id.p)
* [Proposal: Mid-stack inlining in the Go compiler](https://go.googlesource.com/proposal/+/master/design/19348-midstack-inlining.md)    
* [Mid-stack inlining in Go](https://dave.cheney.net/2020/05/02/mid-stack-inlining-in-go)   

mid-stack inlining 一般和 leaf inlining 相提并论。 一个函数里不会再调用任何函数了，那它就是 leaf(叶子节点)，否则就是 mid-stack。  


//TODO  

## 内联后 stack trace 看起来还正常吗？

假如函数 `foo` 被内联了，但同时这个函数里触发了 panic，那么打印出来的 stack trace 还能看到 foo 的踪迹吗？ 