# Go 语言源码阅读版 


Go 语言 Git repo : https://go.googlesource.com/go ，Github 上的 repo 只是这个 repo 的镜像。   


[rf]: https://reneefrench.blogspot.com/
[cc3-by]: https://creativecommons.org/licenses/by/3.0/

# Sched   

按需阅读。   
------------- builtin --------------   

- [ ] make  
- [ ] new  
- [ ] len   
- [ ] cap  
- [ ] append   
- [ ] panic   
- [ ] defer   
- [ ] recover  
- [ ] copy  
- [ ] delete  
- [ ] print   

------------- gc(go compiler) --------------    

- [ ] [ast(抽象语法树)](notes/gc_ast.md)
- [ ] [inline(内联)](notes/gc_inline.md) 
- [ ] [typecheck(类型检查)](notes/gc_inline.md)  
- [ ] [ssa(静态单赋值)](notes/gc_ssa.md)  
- [ ] [死代码消除](notes/gc_deadcode.md) 
- [ ] [dwarf 相关](notes/gc_dwarf.md)  
- [ ] [linker:静态链接/动态链接](notes/gc_linker.md)  

------------- sync --------------

- [x] [sync.Once](notes/once_annotation.md)   
- [ ] [sync.Mutex](notes/mutex_annotation.md)  
- [ ] sync.RWMutex   
- [ ] sync.Cond  
- [ ] sync.Pool  
- [ ] sync.Map 

------------- runtime --------------
- [ ] runtime.GC()  
- [ ] runtime.GOMAXPROCS()  
- [ ] runtime.lfstack 


------------- atomic --------------

- [ ] atomic.CompareAndSwap  
- [ ] atomic.Load/Store     
- [ ] atomic.AddInt32 


------------- io --------------  

- [ ] io.Reader/io.Wrier/io.Closer 

------------- net --------------


- [ ] [netpoller](notes/net_netpoller.md) 
- [ ] [net.Conn.SetDeadline(r/w)](notes/net_deadline_annotation.md)
 