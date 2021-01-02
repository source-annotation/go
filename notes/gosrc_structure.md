### Go 源码的组织结构  

### func path cheatsheet  

|功能| func | path |   
|---|---|---|  
|检查死锁|runtime.checkdead() | src/runtime/proc.go |    
|

### 找不到函数实现问题
我们常用编辑器的 "转到定义" 来查看一个函数的实现。 但在 Go 源码中，经常会发现一个函数经常找不到其实现。 

可能原因： 

* 被指定了 `//go:linkname ` 这个 pragma，编译器把这个函数 link 到另一个函数了. 
* 直接调用了汇编代码(比如 atomic.CAS 的实现其实是在 sync 包的 asm.s 下并 JMP 到 src/runtime/internal/atomic/asm_{arch}.s)    
* 编译器把函数名给改了(比如 make 函数在编译阶段，函数名被改成 makeslice/makechan/... 了，真正的函数实现其实在 makeslice/makechan/... 里)