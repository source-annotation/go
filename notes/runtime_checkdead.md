runtime.checkdead() 

我们经常会看到 `all goroutines are asleep - deadlock!` 的错误输出，表示检测到死锁，这个错误是 fatalthrow 抛出的所以无法 recover。

死锁的检测是 go 在 runtime 中检查的，检查死锁的方法是 runtime.checkdead (src/runtime/proc.go)。  

 