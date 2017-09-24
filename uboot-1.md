#Uboot 源码分析 1

##1.1 start.S
> `.word`,定义类似int 类型数组,这是启动代码中的前16字节头部,这个占位的16字节只是保证正式的image 的头部确实有16字节

```Assembly
.word 0x2000
.word 0x0
.word 0x0
.word 0x0
```

##1.2 异常向量表

```Makefile
_start: b	reset
	ldr	pc, _undefined_instruction
	ldr	pc, _software_interrupt
	ldr	pc, _prefetch_abort
	ldr	pc, _data_abort
	ldr	pc, _not_used
	ldr	pc, _irq
	ldr	pc, _fiq
```
> 一般来说每种异常都需要被处理，但是在uboot中，我们没有完全处理各种异常。

复位异常代码
~~~
/*
 * set the cpu to SVC32 mode and IRQ & FIQ disable
 */
@;mrs	r0,cpsr
@;bic	r0,r0,#0x1f
@;orr	r0,r0,#0xd3
@;msr	cpsr,r0
msr	cpsr_c, #0xd3		@ I & F disable, Mode: 0x13 - SVC
~~~
> cpsr 程序状态寄存器，其实CPU在复位就会进入SVC模式，但这里还是软件设置为SVC模式，整个uboot工作时，CPU一直处于SVC模式

##1.3 deadbeef

```
.balignl 16,0xdeadbeef
```
>当前地址对齐排布，如果不对齐，则地址自动向后走，直到对齐，对齐的内存用0xdeadbeaf填充
+ 为什么要对齐访问？ 有时候是效率，有时候是硬件要求。

##1.4 TEXT_BASE
> Makefile中的TEXT_BASE,即链接地址=0xc3e00000

```
_TEXT_BASE:
	.word	TEXT_BASE
```

> 定义指针变量_TEXT_BASE

+ _TEXT_PHY_BASE = 33e0000,uboot 在DDR中的物理地址

##1.5 L2、L1 Cache ,MMU

```
bl	disable_l2cache

bl	set_l2cache_auxctrl_cycle

bl	enable_l2cache
```
0. 刷新cache
1. 关闭MMU

##1.6 启动信息
```
ldr	r0, =PRO_ID_BASE
ldr	r1, [r0,#OMR_OFFSET]
bic	r2, r1, #0xffffffc1
```
> 该寄存器（0xE0000004）记录了 OM引脚的选择，决定启动介质

##1.7 设置栈，并调用lowlevel_init

```
/*
 * Go setup Memory and board specific bits prior to relocation.
 */

ldr	sp, =0xd0036000 /* end of sram dedicated to u-boot */
sub	sp, sp, #12	/* set stack */
mov	fp, #0
```

>为后面函数调用设置栈空间,0xd0036000是自己指定的，也可以按照三星指定的栈地址。主要原因是因为在被调用的函数里面还有第二层调用。
