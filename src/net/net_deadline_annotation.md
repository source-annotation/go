net.Conn.SetDeadline(read deadline/write deadline) 

从 src/net/net.go 的 SetDeadline 方法开始追，会深入到这一步： 

```go
func (fd *netFD) SetDeadline(t time.Time) error {
	return fd.pfd.SetDeadline(t)
}
```

可以看出， SetDeadline 本质是给 fd 设置的。 

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

    // fd 有 incref 和 decref 两个方法？TODO 去仔细看一下！！！ 
    if err := fd.incref(); err != nil {
        return err
    }
    defer fd.decref()

    ...
    runtime_pollSetDeadline(fd.pd.runtimeCtx, d, mode)
    return nil
}
```
