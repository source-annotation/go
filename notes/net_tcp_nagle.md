# net.TCPConn.SetNoDelay 

预备知识： 

* [TCP连接中启用和禁用TCP_NODELAY有什么影响？](https://www.zhihu.com/question/42308970/answer/246334766)  
* [Nagle's algorithm](https://en.wikipedia.org/wiki/Nagle%27s_algorithm)

我们知道，早期网络条件比较差，所以有了 tcp nagle 算法来解决网络中存在大量 small packet 造成的拥塞。 nagle 会缓存一部分数据，然后一次发送。虽然缓解拥塞问题但是造成了延迟问题(Delay)。 

但是现在网络条件好了，所以很多 tcp 封装都默认不开启 nagle，也就是设定 `TCP_NODELAY`。

看一下 Golang net package里这部分内容： 

```go
// SetNoDelay controls whether the operating system should delay
// packet transmission in hopes of sending fewer packets (Nagle's
// algorithm).  The default is true (no delay), meaning that data is
// sent as soon as possible after a Write.
func (c *TCPConn) SetNoDelay(noDelay bool) error {
	if !c.ok() {
		return syscall.EINVAL
	}
	if err := setNoDelay(c.fd, noDelay); err != nil {
		return &OpError{Op: "set", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return nil
}
```

Golang net 包中 tcp 连接对象默认就是不开启 nagle 的，但允许我们调用 `net.TCPConn.SetNoDelay` 来设定是否针对该 tcp conn 开启 nagle。 

而真正的 setNoDelay 也是委托更底层的 dylib 去实现的 :  
```go
func setNoDelay(fd *netFD, noDelay bool) error {
	err := fd.pfd.SetsockoptInt(syscall.IPPROTO_TCP, syscall.TCP_NODELAY, boolint(noDelay))
	runtime.KeepAlive(fd)
	return wrapSyscallError("setsockopt", err)
}

...  

// 一直往下追，最后你会追到这个方法
func setsockopt(s int, level int, name int, val unsafe.Pointer, vallen uintptr) (err error) {
	_, _, e1 := syscall6(funcPC(libc_setsockopt_trampoline), uintptr(s), uintptr(level), uintptr(name), uintptr(val), uintptr(vallen), 0)
	if e1 != 0 {
		err = errnoErr(e1)
	}
	return
}

func libc_setsockopt_trampoline()

//go:linkname libc_setsockopt libc_setsockopt
//go:cgo_import_dynamic libc_setsockopt setsockopt "/usr/lib/libSystem.B.dylib"
```

以 MacOS 为例，go 会调用 /usr/lib/libSystem.B.dylib 去给这个 TCP Conn 对应的 socket 设置 NoDelay 或 Delay。

而在 go net 中，新连接总是 NoDelay 的，源码如下：
```go
// src/net/tcpsock.go 
func newTCPConn(fd *netFD) *TCPConn {
	c := &TCPConn{conn{fd}}
	setNoDelay(c.fd, true)
	return c
}
```
