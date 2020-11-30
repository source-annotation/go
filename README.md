# Go 语言源码阅读版 


Go 语言 Git repo : https://go.googlesource.com/go ，Github 上的 repo 只是这个 repo 的镜像。   


[rf]: https://reneefrench.blogspot.com/
[cc3-by]: https://creativecommons.org/licenses/by/3.0/

# Sched   

按需阅读。   

------------- sync --------------

- [x] [sync.Once](src/sync/once_annotation.md)   
- [ ] [sync.Mutex](src/sync/mutex_annotation.md)  
- [ ] sync.RWMutex   
- [ ] sync.Cond  
- [ ] sync.Pool  
- [ ] sync.Map 

------------- runtime --------------
- [ ] runtime.GC()  
- [ ] runtime.GOMAXPROCS()


------------- atomic --------------

- [ ] atomic.CompareAndSwap  
- [ ] atomic.Load/Store    


------------- net --------------

- [ ] [net.Conn.SetDeadline(r/w)](src/net/net_deadline_annotation.md)
 