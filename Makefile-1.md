#Uboot Makefile

---------------------------

## 1.1  uboot version 的确定

>U_BOOT_VERSION决定
>version_autogenerate.h


##1.2 环境变量 HOSTARCH 和HOSTOS

###1.2.1 HOSTARCH
>HOSTARCH := $(shell uname -m | \
	sed -e s/i.86/i386/ \
	    -e s/sun4u/sparc64/ \
	    -e s/arm.*/arm/ \
	    -e s/sa110/arm/ \
	    -e s/powerpc/ppc/ \
	    -e s/ppc64/ppc/ \
	    -e s/macppc/ppc/)

HOSTARCH 用来标记编译主机体系架构
>sed -e s/str1/str2 用字符串str2 替换str1


###1.2.2 HOSTOS
主机的操作系统，大写会被转换成小写
>HOSTOS := $(shell uname -s | tr '[:upper:]' '[:lower:]' | \
	    sed -e 's/\(cygwin\).*/cygwin/')

##1.3 静默编译
```Makefile
ifeq (,$(findstring s,$(MAKEFLAGS)))
XECHO = echo
else
XECHO = :
endif
```

如何编译的时候传入`s`,那么编译不会输出任何信息
> make -s

##1.4 两种编译方法
###1.4.1 原地编译
  > .o 和 .c 在同一个文件夹，容易污染源代码

###1.4.2 单独输出文件夹编译（借鉴了Linux kernel）
> 编译时指定输出目录，源代码目录不做任何污染
`make O=`或者`export BUILD_DIR=`

##1.5 MKCONFIG
> Makefile中的一个变量，之后会使用，就是根目录下面的
  mkconfig

`obj = $(OBJTREE)`
+ include/config.mk 是在配置过程（`make x210_sd_config`）中生成的
  ```Makefile
  ARCH   = arm
  CPU    = s5pc11x
  BOARD  = x210
  VENDOR = samsung
  SOC    = s5pc110
  ```
###此mkconfig由一下代码生成
```Makefile
 x210_sd_config :	unconfig
	@$(MKCONFIG) $(@:_config=) arm s5pc11x x210 samsung s5pc110
	@echo "TEXT_BASE = 0xc3e00000" > $(obj)board/samsung/x210/config.mk
```
##1.5 ARCH CROSS_COMPILE
###1.5.1 ARCH
>由上述文件得出
###1.5.2 CROSS_COMPILE
>定义交叉工具链前缀，可以指定在Makefile里面指定。也可以直接make CROSS_COMPILE=XXX

##1.6 config.mk
###1.6.1 autoconf.mk
> 配置过程自动生成，用来指导uboot的整个编译过程。
  这个文件其实就是很多CONFIG_开头的很多宏,这个文件由`include/configs/x210_sd.h`生成


##1.7 链接脚本
> board\samsung\x210 目录下面就有u-boot.lds
###1.7.1 TEXT_BASE
+ board/samsung/x210/ 生成了config.mk 里面的内容就是 TEXT_BASE=0xc3e00000

+ TEXT_BASE是将来整个uboot链接时指定的链接地址，因为uboot中启用了虚拟地址映射，所以`c3e00000`就等于`0x23E0000`(具体要看uboot中的虚拟地址映射关系)

##1.8 目标all
> 第一个目标 `all` ,`make = make all`
