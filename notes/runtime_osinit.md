# runtime.osinit()

不同 os/arch 下的的 osinit() 也不同。 举个 src/runtime/os_linux.go 的例子：  
```go
func osinit() {
	ncpu = getproccount()
	physHugePageSize = getHugePageSize()
	osArchInit()
}
```