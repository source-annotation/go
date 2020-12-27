# Write barrier

在看 gmp 调度器的源码时，发现里面有很多方法里用到了 `//go:yeswritebarrierrec`  和 `//go:nowritebarrierrec` ，用来表示要或者不要写屏障。 

还是要先去了解什么是内存屏障。  

https://zhuanlan.zhihu.com/p/55767485 