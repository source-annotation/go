# syscall 

参考 :  

* [Golang 系统调用 syscall](https://www.jianshu.com/p/3b6935e3bb50)  
* [Go 系列文章6: syscall](https://xargin.com/syscall/)


我们说 Go 是 self contained 的，这是因为 go compiler 把 runtime 给静态链接了。 好处是便于分发，也就是你编译后直接把 binary 丢给别人用就行了，用的人不需要再装一堆 dylib 依赖或 vm 之类的东西。坏处是 binary 体积变大了，一个 hello world 也会产生 2MB 起的 binary。

在知乎的 [JavaScript引擎、虚拟机、运行时环境是一回事儿吗？](https://www.zhihu.com/question/39499036/answer/81707112) 这个问题的回答，看到这句话：
> 其它有不少声称自己“不带运行时环境”的native编程语言，其实未必是真的不需要运行时环境，而是因为现代操作系统上通常自带一个C运行时环境，只要只依赖于它就不需要额外依赖别的运行时环境，就忽悠用户说像是“不带运行时环境”一样。

实际上 GO 也是这样， 除非你自己写 assembly 否则大部分高级语言的底层都是要调用 libc(glibc)。 例如你对文件的读写，深究到最底层总是 `read()`,`write()`。

如果有阅读过 GO 源码的话，应该经常可以看到 `Syscall(SYS_xxx)` 之类的函数调用。在 Mac OSX 下则会调用 `/usr/lib/libSystem.B.dylib` 之类的一些系统级动态库。 



## 源码分析   

```go
// src/syscall/syscall_unix.go
func Syscall(trap, a1, a2, a3 uintptr) (r1, r2 uintptr, err Errno)
func Syscall6(trap, a1, a2, a3, a4, a5, a6 uintptr) (r1, r2 uintptr, err Errno)
func RawSyscall(trap, a1, a2, a3 uintptr) (r1, r2 uintptr, err Errno)
func RawSyscall6(trap, a1, a2, a3, a4, a5, a6 uintptr) (r1, r2 uintptr, err Errno)
```

函数的实现在同一个目录下的 `.s`(也就是汇编实现)。  

Syscall 与 Syscall6 的区别在于形参的个数(除trap外)，Syscall6 有 a1~a6 共 6 个。 Syscall 与 RawSyscall 的区别见曹大的 [Go 系列文章6: syscall](https://xargin.com/syscall/) 。

写到这里，发现曹大已经分析的够详细了，所以直接看他的 [Go 系列文章6: syscall](https://xargin.com/syscall/) 。 

以后有想补充的再写到这里。  



