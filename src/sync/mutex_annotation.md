# sync.Mutex 

sync.Mutex 和 sync.RWMutex 里的错误都是以 `throw()` 的方式抛出的，在 `$GOSRC/runtime/panic.go` 里可以 `throw()` 的方法体: 

```go
func throw(s string) {
    ...
    fatalthrow()
    ...
}

// fatalthrow implements an unrecoverable runtime throw. It freezes the
// system, prints stack traces starting from its caller, and terminates the
// process.
//
//go:nosplit
func fatalthrow() {
	...
}
```

看 `fatalthrow()` 方法的注释就知道 `throw()` 出来的错误无法被 recover。也就是说： sync.Mutex 和 sync.RWMutex 的错误(race,死锁等..)无法被 recover，只能找到问题原因赶紧改。


# 收获 

用锁原则：

* 谁申请，谁释放  
* lock 和 unlock 最好在同一个 function 里   
