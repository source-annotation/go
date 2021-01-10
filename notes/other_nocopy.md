# noCopy 

先阅读：  

* https://github.com/golang/go/issues/8005  
* https://golang.org/issues/8005#issuecomment-190753527 
 
在 sync.Cond 的源码文件(cond.go)里定义了一个 noCopy 结构体，sync package 的其他原语也用到了这个 noCopy 例如 waitgroup, cond, pool。 

```go
// src/runtime/sync/cond.go

// noCopy may be embedded into structs which must not be copied
// after the first use.
//
// See https://golang.org/issues/8005#issuecomment-190753527
// for details.
type noCopy struct{}

// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}
```

noCopy 的作用就是：当你不希望你的结构体被拷贝时，在结构体里嵌入 noCopy field。 

假如你的结构体嵌入了 noCopy 而你的代码里仍拷贝了这个结构体，并不会导致编译失败。 只有当你用 go vet 检查的时候才会报错。 

