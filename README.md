# Go 语言源码阅读版 


Go 语言 Git repo : https://go.googlesource.com/go ，Github 上的 repo 只是这个 repo 的镜像。   


go version : 1.13.9 

[rf]: https://reneefrench.blogspot.com/
[cc3-by]: https://creativecommons.org/licenses/by/3.0/

# Sched   

[Go 源码阅读指北](notes/gosrc_structure.md)

------------- datatypes --------------  

- [ ] map  
- [ ] slice 
- [ ] [channel](notes/datatype_channel.md)  
- [ ] interface  
- [ ] string
- [ ] func 
- [ ] [arbitrary type](notes/datatype_arbitrary.md) 

------------- builtin --------------   

- [ ] make  
- [ ] new  
- [ ] [len](notes/builtin_len.md)   
- [ ] cap  
- [ ] append   
- [ ] panic   
- [ ] recover  
- [ ] [copy](notes/builtin_copy.md)  
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

- [x] [sync.Once](notes/sync_once.md)   
- [ ] [sync.Mutex](notes/sync_mutex.md)  
- [ ] sync.RWMutex   
- [ ] [sync.Cond](notes/sync_cond.md)  
- [ ] sync.Pool  
- [ ] sync.Map 

------------- runtime --------------   
- [ ] [runtime.GC()]()  
- [ ] [runtime.GOMAXPROCS()](notes/runtime_gomaxprocs.md)  
- [ ] runtime.lfstack 
- [x] [runtime.Gosched()](notes/runtime_gosched.md)  
- [ ] [runtime.osinit()](notes/runtime_osinit.md)  
- [ ] [死锁检查 : runtime.checkdead()](notes/runtime_checkdead.md)


------------- atomic --------------

- [ ] atomic.CompareAndSwap  
- [ ] atomic.Load/Store     
- [ ] atomic.AddInt32 


------------- io --------------  

- [ ] io.Reader  
- [ ] io.Reader.Read()   
- [ ] io.Writer 
- [ ] io.Closer  

------------- bufio --------------  

//TODO gopush 解析 tcp 字节流，就是用 bufio

------------- net --------------


- [ ] [netpoller](notes/net_netpoller.md) 
- [ ] [net.Conn.SetDeadline(Read/Write)](notes/net_deadline_annotation.md)
- [x] [net.Conn.Read()](notes/net_conn_read.md)   
- [x] [nagle 算法：net.TCPConn.SetNodelay()](notes/net_tcp_nagle.md)  
 
 ------------- reflect --------------  
 
 - [ ] reflect.DeepEqual()
   
 ------------- pragmas --------------  

- [ ] [write barrier(写屏障)](notes/other_barrier.md)  
- [ ] [nosplit]()  
- [ ] [noinline]()

------------- 一些汇编 --------------  

- [ ] [Check alignment](notes/asm_check_alignment.md)


------------- 待分类 --------------  

- [x] [syscall](notes/syscall.md)  
- [ ] select  
- [ ] defer   
- [ ] [go scheduler(调度器)](notes/other_scheduler.md)
- [ ] [gmp](notes/other_gmp.md) 
- [ ] [GC]()  
- [ ] [noCopy](notes/other_nocopy.md)
  
