net.Conn.SetDeadline(read deadline/write deadline) 

从 src/net/net.go 的 SetDeadline 方法开始追，会深入到这一步： 

```go
func (fd *netFD) SetDeadline(t time.Time) error {
	return fd.pfd.SetDeadline(t)
}
```

可以看出， SetDeadline 本质是给这个 socket 的 fd 设置的。 

但最后都是进入到 src/poll/fd_poll_runtime.go 的 `setDeadlineImpl` 方法。调用 :  
* SetDeadline = `setDeadlineImpl(fd, t, "rw")`  
* SetReadDeadline = `setDeadlineImpl(fd, t, "r")`  
* SetWriteDeadline = `setDeadlineImpl(fd, t, "w")`

```go
func setDeadlineImpl(fd *FD, t time.Time, mode int) error {
    var d int64
    if !t.IsZero() {
        // 如果 t 是 0，那就说明没有 deadline!!!
    }

    // fd 有 incref 和 decref 两个方法
    // incref = increase reference 也就是增加 fd 引用计数 
    // decref = decrease reference 也就是减少 fd 引用计数 
    // 这2个对 fd 引用计数操作的方法，都是 spin + cas (乐观锁)
    if err := fd.incref(); err != nil {
        return err
    }
    defer fd.decref()

    ...
    // emmmm，来到了这里，这个函数又会被编译指令 //go:linkname 给 link 到 src/runtime/netpoll.go 的 poll_runtime_pollSetDeadline 函数  
    runtime_pollSetDeadline(fd.pd.runtimeCtx, d, mode)
    return nil
}
```

```go
//go:linkname poll_runtime_pollSetDeadline internal/poll.runtime_pollSetDeadline
func poll_runtime_pollSetDeadline(pd *pollDesc, d int64, mode int) {
	...
}
```


