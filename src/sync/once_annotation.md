# sync.Once  

这个同步原语主要用于：确保某件事情只做一次。   



来看看结构体：
```go
type Once struct {
    done uint32
    m    Mutex
}
```

done 指示了 f() 是否已经执行完成。请注意，是"执行完成"，当 f() 执行完成后才可以把 done 置为 1。  

假设你使用 cas 来实现 done 的 swap 会发生什么事情呢？ 
```go
func (o *Once) Do(f func()) {
    // Note: Here is an incorrect implementation of Do: 
    // 官方警告你：下面这种实现 Do 的方式是不对的
    if atomic.CompareAndSwapUint32(&o.done, 0, 1) {
        f()
    }
}
```

"Done" 这个什么意思呀？就是"完成" 的意思!!! 假设使用 cas 来实现 once.done 的 0~1 swap 就会出现这样一种情况：f() 还未执行， once.done 就已经先被标记为 1 了。 
 
举个例子：

```
-> f() 用于初始化连接对象，goroutine A 先执行了 once.Do(f()) 并成功 swap 了 once.done， 但此时 f() 还未执行完(即连接还未初始化完成)。   
-> goroutine B 执行了 once.Do(f()) , 由于此刻 once.done 指示了这件事请已经做完了，所以  goroutine B 大胆地去使用这个连接对象, 但此时这个连接对象还是 nil。    
-> panic
``` 


使用 sync.Once() 有一个要注意的地方： once.Do(f()) 只保证 f() <ins>**执行完成**</ins>，并没有义务保证 <ins>**执行成功**</ins>。还是以初始化单例的连接对象为例，f() 并不能保证连接对象初始化成功。

即便 f() 执行出错了，在外部看来 f() 还是执行完了。你无法把错误丢到外部，除非 f() 里直接 panic(err)。

# 应用场景 

1. lazy 式的单例模式，确保 instance 只初始化一次。 
2. 看了下 go 源码中对 sync.Once 的使用，多用于<ins>初始化资源</ins> 和 <ins>close() 一个 channel</ins>  
3. 待补充 ...   

# 收获 

1. 学了个新术语 niladic(表示没有参数的函数)   
2. sync.Once 适合放在结构体中，比如 
    ```
    type Config struct{ 
        once sync.Once 
    }
    
    config.once.Do(func() {
        config.init(filename) 
    })
    ```
3. 源码注释里特地提到了 `done uint32` 放在结构体首位的好处 
    ```go
    type Once struct {
        // done indicates whether the action has been performed.
        // It is first in the struct because it is used in the hot path.
        // The hot path is inlined at every call site.
        // Placing done first allows more compact instructions on some architectures (amd64/x86),
        // and fewer instructions (to calculate offset) on other architectures.
        done uint32
        ...
    }
    ```
    大概意思是 done 是最被频繁访问的，放在这里 cpu 访问比较快。可以参考 : https://stackoverflow.com/questions/59174176/what-does-hot-path-mean-in-the-context-of-sync-once     
    TODO-annotation : 看了 `hot path is inlined at every call site` 有点疑惑，这个和内联有啥关系？  