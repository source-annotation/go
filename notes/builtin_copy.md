# copy  

```go
// The copy built-in function copies elements from a source slice into a
// destination slice. (As a special case, it also will copy bytes from a
// string to a slice of bytes.) The source and destination may overlap. Copy
// returns the number of elements copied, which will be the minimum of
// len(src) and len(dst).
func copy(dst, src []Type) int
```

copy 方法会在编译时在 `src/cmd/compile/internal/gc/walk.go` 的 `copyany()` 函数中被替换成掉。 

copyany 最终调用 `memmove` 还是 `copyslice` 方法来实现拷贝，取决于 `runtimecall` 这个参数。 来看看这个 copyany 方法：

```go
// src/cmd/compile/internal/gc/walk.go  

// Lower copy(a, b) to a memmove call or a runtime call.
//
// init {
//   n := len(a)
//   if n > len(b) { n = len(b) }
//   if a.ptr != b.ptr { memmove(a.ptr, b.ptr, n*sizeof(elem(a))) }
// }
// n;
//
// Also works if b is a string.
//
func copyany(n *Node, init *Nodes, runtimecall bool) *Node {
    
    if runtimecall {
    
        fn := syslook("slicecopy")
        
        return mkcall1(fn, n.Type, init, ptrL, lenL, ptrR, lenR, nodintconst(n.Left.Type.Elem().Width))
    }
    
    
    fn := syslook("memmove")

    call := mkcall1(fn, nil, init, nto, nfrm, nwid)
    
    return nlen
}
``` 

我们把大部分代码省略，着重看一下 `slicecopy` 和 `memmove`。   

### runtime.slicecopy 

src/runtime/slice.go 
```go
func slicecopy(toPtr unsafe.Pointer, toLen int, fmPtr unsafe.Pointer, fmLen int, width uintptr) int {
	// TODO 
}
```
### runtime.memmove 

src/runtime/stubs.go
```go
// memmove copies n bytes from "from" to "to".
//
// memmove ensures that any pointer in "from" is written to "to" with
// an indivisible write, so that racy reads cannot observe a
// half-written pointer. This is necessary to prevent the garbage
// collector from observing invalid pointers, and differs from memmove
// in unmanaged languages. However, memmove is only required to do
// this if "from" and "to" may contain pointers, which can only be the
// case if "from", "to", and "n" are all be word-aligned.
//
// Implementations are in memmove_*.s.
//
//go:noescape
func memmove(to, from unsafe.Pointer, n uintptr)
```
# <ins>copy</ins> vs <ins>append</ins>  

```go
func main() {
    a := []byte("hello") 
    b := []byte{}
    
    copy(b, a)
    b = append(b, a...)
}

```
 

# 其它 

copy 的拷贝深度只有一层。
```go
func main() {}
    c := [][]int{{1,1,1}, {2,2,2}}
    d := [][]int{{0,0,0}, {0,0,0}}
    copy(d, c)
    d[0] = []int{999,999,999} // c : [[1 1 1] [2 2 2]],   d : [[999 999 999] [2 2 2]]
    
    e := [][]int{{1,1,1}, {2,2,2}}
    f := [][]int{{0,0,0}, {0,0,0}}
    copy(f, e)
    f[0][0] = 999 // e : [[999 1 1] [2 2 2]],   d : [[999 1 1] [2 2 2]]
}
```