# 3 Start_armboot 解析

-------------------------------------------------------



## 3.1 Start_armboot 简介

```C
  typdef int (init_fnc_t) (void);
```
## 3.2 uboot 内存排布

```C
#define DECLEAR_CLOBAL_PTR register voliatile gd_t * gd asm("r8")
```
> register 修饰表示尽量放到寄存器里面
> arm("r8") 表示gd 放到寄存器r8里面

### 3.2.1 为什么要分配内存
> gd和Bd 需要内存，当前的内存没有被人管理

1. uboot区 CFG_UBOOT_BASE-xx
2. 堆区    CFG_MALLOC_LEN
3. 栈区    CFG_STACK_SIZE
4. gd      sizeof(gd_t)
5. bd      szieof(bd_t)
6. 内存间隔

### 3.2.3 init_sequence
>init_sequence 都是board级别的各种硬件初始化

### 3.2.4 board_init
1. DECLARE_GLOBAL_DATA_PTR在这里声明是为了后面使用gd方便,可以看出把gd的声明定义成一个宏的原因就是我们要到处去使用gd,因此就要到处声明，定义成宏比较方便
2. 网卡初始化。
3. 这个函数中主要是网卡的GPIO和端口的配置，而不是驱动，因为网卡驱动都是现成的。关键是这里的基本初始化

### 3.2.5 bi_arch_number
> bi_arch_number 是board_info的一个元素含义是开发板的机器吗，所谓机器码就是uboot给这个开发板定义的唯一编号
+ 机器码的主要作用就是在uboot和内核之间进行比对和适配
+ uboot中配置这个机器码，会作为uboot给内核的传参的一部分，内核启动过程中会对比这个接收到的机器码，和自己本身的机器码对比，如果相等就启动。

### 3.2.6 gd->bd->bi_boot_params
  1. kernel启动时的传参的内存地址。uboot给内核的传参的时候是这样传的: uboot事先将主板好的传参（bootargs)放到内存的一个地址处（bi_boot_params）,然后uboot就启动了内核(uboot在内核启动时真正是通过寄存器r0 r1 r2)来传递
