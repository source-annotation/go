# runtime.GOMAXPROCS() 

```go
func schedinit() {
    // ...
    procs := ncpu
    if n, ok := atoi32(gogetenv("GOMAXPROCS")); ok && n > 0 {
        procs = n
    }
    // ... 
}
```

默认的 procs 是 ncpu，也就是 cpu 的逻辑核心数(双核4线程的 ncpu = 4)。 

如果设置了 `GOMAXPROCS` 环境变量，那 procs 就用设置的这个。