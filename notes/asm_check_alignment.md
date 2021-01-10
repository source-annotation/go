# check alignment 

看 `copy()` 的实现时，追到 `memmove()` 这个函数。 而这个函数真正实现是汇编 (src/runtime/memmove_*.s)。

在 memmove_amd64.s 里看到了这样一小段代码：

```go
TEXT runtime·memmove(SB), NOSPLIT, $0-24
...
    
    // Check alignment
    MOVL	SI, AX
    ORL	DI, AX
    TESTL	$7, AX
    JEQ	fwdBy8
...
```

之前略微看过一些内存对齐，但了解不深。 这里刚好结合汇编一起分析一下。 

TODO