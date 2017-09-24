#Uboot Makefile-2
----------------------------------------

##2 mkconfig.mk 文件详解
###2.1 参数详解
x210_sd_config中_config被替换为空

```Makefile
x210_sd_config: unconfig
@$(MKCONFIG) $(@:_config=) arm s5pc11x x210 samsung s5pc110
```
+ $1 : x210_sd
+ $2 : arm
+ $3 : s5pc11x
+ $4 : x210
+ $5 : samsung
+ $6 : s5pc110

###2.2 创建符号链接
> 从33行开始到118行，都是在创建符号链接，这些链接文件的存在就是整个配置的核心，根本目的是让uboot就有可移植性。

+ asm --> asm-arm
+ asm_arm/arch --> asm_arm/arch-s5pc11x
+ include/regs.h --> include/s5pc110.h
+ include/asm-arm/proc  --> include/asm-arm/proc-armv

###2.3 config.mk
> include/config.mk

其内容如下，主`Makefile`会包含此文件
```Makefile
ARCH   = arm
CPU    = s5pc11x
BOARD  = x210
VENDOR = samsung
SOC    = s5pc110
```
###2.4 创建config.h
> include/config.h,其包含了configs/x210_sd.h

###2.5 链接脚本
> board\samsung\x210\u-boot.lds

指定程序的链接地址有两种方法:
1. 在Makefile中的ld的flags用 -Ttext指定
2. 在链接脚本中的SECTIOINS 开头用.=addr 指定

>这两种技巧是可以配合使用的，就是说两者可以一起使用，Makefile中的链接地址会覆盖链接脚本里的，最终来源TEXT_BASE
+ 在代码段`.text`中必须注意文件的排列顺序，这些文件属性uboot前16KB中的代码

+ 自定义段，`.u_boo_cmd`，自定义段很重要（uboot命令相关）
