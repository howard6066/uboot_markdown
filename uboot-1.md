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

##1.8 lowlevel_init

###1.8.1 检查复位状态
> 冷启动DDR需要初始化，热启动或者低功耗 不需要

###1.8.2 IO 状态恢复
> 无需关系

###1.8.3 关闭看门狗
> 参考裸机代码
###1.8.4 SRAM SROM 相关GPIO设置
>
###1.8.5 开发板供电锁存

```Assembly
/* PS_HOLD pin(GPH0_0) set to high */
ldr	r0, =(ELFIN_CLOCK_POWER_BASE + PS_HOLD_CONTROL_OFFSET)
ldr	r1, [r0]
orr	r1, r1, #0x300
orr	r1, r1, #0x1
str	r1, [r0]
```
###1.8.6 判断当前代码执行位置

```Assembly
/* when we already run in ram, we don't need to relocate U-Boot.
 * and actually, memory controller must be configured before U-Boot
 * is running in ram.
 */
ldr	r0, =0xff000fff
bic	r1, pc, r0		/* r0 <- current base addr of code */
ldr	r2, _TEXT_BASE		/* r1 <- original base addr in ram */
bic	r2, r2, r0		/* r0 <- current base addr of code */
cmp     r1, r2                  /* compare r0, r1                  */
beq     1f			/* r0 == r1 then skip sdram init   */
```


__判断位置技术__:
> bic	r1, pc, r0，意思是：将PC的值中的某些位清0，剩下的一些特殊位赋值给r1(ro中那些为1的位清0) ,r1 = pc & ~(r0).清除低地址位，然后比较两者地址是否一致。相等时在DDR中。


> 判断当前位置是在SRAM中还是DDR中,原因如下：
+ BL1(Uboot 的前一部分)在SRAM中有一份，在DDR中也有一份。因此如果是冷启动，当前代码在SRAM中运行的BL1.如果是热启动，这时候在DDR中。
+ 判断当前运行代码位置是有用的，比如判断当前地址，确定是否要执行时钟初始化和DDR初始化。
> beq 1f ，如果相等，就跳到 1 处执行。

###1.8.7 system_clock_init
>bl system_clock_init

###1.8.8 mem_ctrl_asm_init
> bl mem_ctrl_asm_init 初始化内存，在`uboot/cpu/s5pc110/cpu_init.s`

+ 在裸机中DMC0的 256MB 内存地址范围为0x20000000 - 0x2FFFFFFF,uboot中为0x30000000 - 0x3FFFFFFF.DMC0上允许的地址是20000000 - 3FFFFFFF,我们实际上只接了256MB。

+ 在uboot中。DMC0 为0x30000000-0x3FFFFFFF.DMC1 为0x40000000 - 4FFFFFFF.


###1.8.9 uart_asm_init

> 串口初始化,发送 "O"

###1.8.10 tzpc_init

结束  发送 "K"
所以在第一阶段完成后，串口发送"OK"
