# copy  

```go
// The copy built-in function copies elements from a source slice into a
// destination slice. (As a special case, it also will copy bytes from a
// string to a slice of bytes.) The source and destination may overlap. Copy
// returns the number of elements copied, which will be the minimum of
// len(src) and len(dst).
func copy(dst, src []Type) int
```

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