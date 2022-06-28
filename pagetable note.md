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
位移操作


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


------------------------ read code ------------------------
kvminit -> kvmmake
      kpgtbl = (pagetable_t) kalloc();
      分配物理内存
      memset(kpgtbl, 0, PGSIZE);
      赋0值
      kvmmap(kpgtbl, UART0, UART0, PGSIZE, PTE_R | PTE_W);
      向kernel page table添加mapping 参数（table,虚拟地址，物理地址，size,权限 ）
      kvmmap(kpgtbl, VIRTIO0, VIRTIO0, PGSIZE, PTE_R | PTE_W);
      kvmmap(kpgtbl, PLIC, PLIC, 0x400000, PTE_R | PTE_W);
      kvmmap(kpgtbl, KERNBASE, KERNBASE, (uint64)etext-KERNBASE, PTE_R | PTE_X);
      kvmmap(kpgtbl, (uint64)etext, (uint64)etext, PHYSTOP-(uint64)etext, PTE_R | PTE_W);
      kvmmap(kpgtbl, TRAMPOLINE, (uint64)trampoline, PGSIZE, PTE_R | PTE_X);
      以上都是direct map kernel直接映射

      proc_mapstacks(kpgtbl);


kvminithart
      w_satp(MAKE_SATP(kernel_pagetable));
      设置satp寄存器，这里实际上是内核告诉MMU来使用刚刚设置好的page table
      在这条指令之前，还不存在可用的page table,执行完这条指令之后，程序计算器pc 加了 4，而之后的下一条指令被执行时，程序计数器会被内存中的page table翻译（how?）

      所以这条指令的执行时刻是一个非常重要的时刻。因为整个地址翻译从这条指令之后开始生效，之后的每一个使用的内存地址都可能对应到与之不同的物理内存地址。因为在这条指令之前，我们使用的都是物理内存地址，这条指令之后page table开始生效，所有的内存地址都变成了另一个含义，也就是虚拟内存地址。

      sfence_vma:flush all TLB entries.


userinit
      设置第一个用户进程
       p = allocproc();
       在process table里找可用进程
            allocpid();
            分配pid
            // Allocate a trapframe page.
            p->pagetable = proc_pagetable(p);
            创建空的用户pagetable
                  pagetable = uvmcreate();


       uvminit(p->pagetable, initcode, sizeof(initcode));
       分配一个用户页然后拷贝指令和数据（因为是第一个进程所以硬编码了）
       // prepare for the very first "return" from kernel to user.
      p->trapframe->epc = 0;      // user program counter
      p->trapframe->sp = PGSIZE;  // user stack pointer


scheduler(void)
      循环
      获取线程lock
      swtch(struct context*, struct context*);
      切换context,Save current registers in old. Load from new.	
      释放锁

