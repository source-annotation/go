# net.Conn.Read()  

```go
func main() {
    ln, _ := net.Listen("tcp", "0.0.0.0:9999")
    for {
        conn, _ := ln.Accept()
        go func(c net.Conn) {
            for {
                buf := make([]byte, 1024)
                n, err := conn.Read(buf)
            }
        }(conn)
    }
}
```

上面的 `conn.Read(buf)` 并不保证填满 `buf` 再返回， 一次循环 `conn.Read(buf)` 执行结束后， buf 里可能有 1~1024 范围内的任意 byte 数。   

可参考 : 

* [TCP server to read an unknown number of bytes but act on it as they come](https://forum.golangbridge.org/t/tcp-server-to-read-an-unknown-number-of-bytes-but-act-on-it-as-they-come/8661)  
* [how many bytes will be read in a conn.Read() call from socket](https://stackoverflow.com/questions/65547801/how-many-bytes-will-be-read-in-a-conn-read-call-from-socket)  

所以当我们使用 tcp 通讯时，在解析字节流时要解决所谓的 "粘包" 和 "半包" 问题(实际上这2个问题不是 tcp 的问题，而是应用层开发者要解决的边界限定问题)。 

解决这个问题我们一般不使用 `net.conn.Read(b []byte)`， 而是 `io.ReadFull()` 或 `bufio` 之类的 API(这里不做讨论)。  

`net.conn.Read()`  的源码如下：

```go
// src/net/net.go 
// Read implements the Conn Read method.
func (c *conn) Read(b []byte) (int, error) {
	if !c.ok() {
		return 0, syscall.EINVAL
	}
	n, err := c.fd.Read(b)
	if err != nil && err != io.EOF {
		err = &OpError{Op: "read", Net: c.fd.net, Source: c.fd.laddr, Addr: c.fd.raddr, Err: err}
	}
	return n, err
}
```
一直深追，我们可以追到`src/internal/poll/fd_unix.go` (以 unix 为例) 下的 `Read()` 方法:  
```go
// Read implements io.Reader.
func (fd *FD) Read(p []byte) (int, error) {
	if err := fd.readLock(); err != nil {
		return 0, err
	}
	defer fd.readUnlock()
	if len(p) == 0 {
		// If the caller wanted a zero byte read, return immediately
		// without trying (but after acquiring the readLock).
		// Otherwise syscall.Read returns 0, nil which looks like
		// io.EOF.
		// TODO(bradfitz): make it wait for readability? (Issue 15735)
		return 0, nil
	}
	if err := fd.pd.prepareRead(fd.isFile); err != nil {
		return 0, err
	}
	if fd.IsStream && len(p) > maxRW {
		p = p[:maxRW]
	}
	for {
		n, err := syscall.Read(fd.Sysfd, p)
		if err != nil {
			n = 0
			if err == syscall.EAGAIN && fd.pd.pollable() {
				if err = fd.pd.waitRead(fd.isFile); err == nil {
					continue
				}
			}

			// On MacOS we can see EINTR here if the user
			// pressed ^Z.  See issue #22838.
			if runtime.GOOS == "darwin" && err == syscall.EINTR {
				continue
			}
		}
		err = fd.eofError(n, err)
		return n, err
	}
}
```
简单阅读下这个方法：
1. 我们知道，在 Linux 中一切皆文件。 所以实际上本质是对 socket fd 的 read，所以看到 `syscall.Read(fd.Sysfd, p)` 这一行并不会觉得奇怪。
2. 假设我们调用 `conn.Read(buf)` make 的 buf 长度为 0，这时避免 syscall.Read 返回一个 io.EOF， 上面这个函数会直接返回，而不会进行 syscall。  
3. 当 fd.IsStream == true (file fd 是 stream 类型；socket fd 取决于 sotype 也就是 socket type，比如 udp socket 的 isStream 就为 false), 那么 buf 长度的最大值会被限定在 1 << 30 。  这是为了避免一次性读取过多内容。

总之，具体一次 read 调用会给 buf 填入多少个 byte，基本上取决于系统调用的 read() : 

```go
// src/syscall/zsyscall_linux_amd64.go 
r0, _, e1 := Syscall(SYS_READ, uintptr(fd), uintptr(_p0), uintptr(len(p)))
```