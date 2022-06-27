software    --->
                
                      --->
                CLINT
                      --->
Timer       --->


                                RISC-V CORE


AON         --->
ADC         --->
SPI         ---> PLIC --->
I2C         --->
GPIO        --->



一个典型的中断流程
中断流程只要是CPU都大同小异的。对于RISC-V来讲，中断流程是这样的。

外设发出中断信号。
PLIC或者CLINT响应中断，RISC-V核心保存此时的CSR（control and status registers，包括了PC啊，中断原因啊一堆信息）。
跳转到中断处理程序（直接换PC值取指令即可）。
关闭其他中断响应使能（RISC-V不支持嵌套，所以一个中断要屏蔽其他中断）。
软件保存通用的寄存器。
然后处理中断（过程中会清掉外设的中断）。
软件恢复通用的寄存器。
然后回复CSR。
然后跳转PC跳回原来位置退出异常。
一个完整流程就结束了。其他细节直接阅读RISC-V core的说明书即可。


----------------------------------------------------------------------------
PTE2PA



PA2PTE



PTEFLAGS


----------------------------------------------------------------------------

https://zhuanlan.zhihu.com/p/68501351


debug:
exec
      loadseg -> walkaddr(Can only be used to look up user pages)


user space:
proc_pagetable
      uvmcreate