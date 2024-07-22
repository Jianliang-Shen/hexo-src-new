---
title: Linux内核Kconfig解读
date: 2019-07-28 20:55:44
tags:
    - Linux
    - Kconfig
    - Kernel
categories: 
    - OS
---

Linux内核Kconfig文件和语法。

<!-- more -->


- [内核的编译过程](#内核的编译过程)
- [配置过程--以驱动为例](#配置过程--以驱动为例)
  - [新建Kconfig](#新建kconfig)
  - [新建Makefile](#新建makefile)
  - [配置上层Makefile和Kconfig](#配置上层makefile和kconfig)
- [Kconfig详细语法](#kconfig详细语法)
  - [关键词](#关键词)
  - [结构语法](#结构语法)
    - [if...endif](#ifendif)
    - [choice...endchoice](#choiceendchoice)
    - [menu](#menu)
    - [menuconfig](#menuconfig)
  
## 内核的编译过程
arch=arm，tisdk_am57xx-evm-rt_defconfig确定了`.config`中的配置，主目录的`Makefile`按照`.config`文件中的选择遍历所有文件中的`Kconfig`（首先是arch/arm/Kconfig），按照`Kconfig`文件中的依赖关系编译文件，再回到顶层目录生成目标文件。 
 ```
Kconfig ---> （每个源码目录下）提供选项
.config ---> （源码顶层目录下）保存选择结果
Makefile---> （每个源码目录下）根据.config中的内容来告知编译系统如何编译
```

## 配置过程--以驱动为例
### 新建Kconfig
在/drivers/test下建立`Kconfig`：
```
config TEST
    bool "Test driver"
    help
    This driver is for test kernel making.
```
这里定义了TEST选项，也就是.config中的`CONFIG_TEST`，Test driver是显示在menuconfig中的名称。
### 新建Makefile
建立编译规则，例如CONFIG_TEST=y，目录下源码编译进内核；CONFIG_TEST=m，目录下源码编译为模块。
```C
obj-$(CONFIG_TEST) += test.o
```
### 配置上层Makefile和Kconfig
修改Kconfig，增加test子项，在drivers/Kconfig中添加：
```
source "drivers/test/Kconfig"
```
修改Makefile，添加编译规则：
```C
obj-y               += test/
```
## Kconfig详细语法
### 关键词
以下面为例：
```
config SMP
	bool "Symmetric Multi-Processing"
	depends on CPU_V6K || CPU_V7
	depends on GENERIC_CLOCKEVENTS
	depends on HAVE_SMP
	depends on MMU || ARM_MPU
	select IRQ_WORK
	help
	  This enables support for systems with more than one CPU. If you have
	  a system with only one CPU, say N. If you have a system with more
	  than one CPU, say Y.
      ......
```
| 关键词         | 功能                                                                     |
| -------------- | ------------------------------------------------------------------------ |
| config         | 紧跟句柄，SMP即为CONFIG_SMP                                              |
| bool           | 表示选择，0或1 [Y]                                                       |
| "name"         | 在menuconfig中显示的字符串，没有则不显示                                 |
| depends on     | 表示依赖某一项，支持 `||` 、`&&`、`!`运算符                              |
| select         | 表示选中后，也会选择select指定的项（被选择的无法取消，符号为-Y-或者{Y}） |
| help           | 打印帮助，menuconfig可以用shift+？查看                                   |
| tristate       | 模块选项，`<M>`或者`<Y>`或者`<N>`（三态）                                      |
| int/hex/string | 选项值为整型数或十六进制数或字符串                                       |
| range          | 整型数的范围                                                             |
| default        | 默认值，数值或者布尔y/n或者字符串或者choice中的选项                      |
| prompt         | 出现在choice中的菜单文字                                                 |
| source         | 引入子目录的Kconfig                                                      |  

补充：
* 无depends on，default 为y：默认为y，一般用于必须要设置的选项，此时不要设置prompt；  

* 有depends on，default 为y：所依赖的条目己设置，则默认为y；所依赖的条目未设置，则为n；  

* 有depends on，default 为n：所依赖的条目己设置，则默认为n；所依赖的条目未设置，则为n；  

* 无depends on，default 为n：在为设置prompt的情况下，此选项想要被设置，需要由其他选项来select它。  
  
### 结构语法
#### if...endif
```
if ARCH_S5PC100 --->如果ARCH_S5PC100选项选中了，则在endif范围内的选项才会被选
config CPU_S5PC100
    bool "选项名"
    select S5P_EXT_INT
    select SAMSUNG_DMADEV
    help
      Enable S5PC100 CPU support

endif
```
#### choice...endchoice
```
choice      --->表示选择列表
    prompt "Default I/O scheduler"         //主目录名字
    default DEFAULT_CFQ                    //默认CFQ
    help
      Select the I/O scheduler which will be used by default for all
      block devices.

    config DEFAULT_DEADLINE
        bool "Deadline" if IOSCHED_DEADLINE=y 

    config DEFAULT_CFQ
        bool "CFQ" if IOSCHED_CFQ=y

    config DEFAULT_NOOP
        bool "No-op"

endchoice
```
#### menu
```
menu "Boot options"  ----> menu表示该选项是`不可选`的菜单，其后是在选择列表的菜单名

config USE_OF
    bool "Flattened Device Tree support"
    select IRQ_DOMAIN
    select OF
    select OF_EARLY_FLATTREE
    help
      Include support for flattened device tree machine descriptions.
....

endmenu     ----> menu菜单结束
```
#### menuconfig
```
menuconfig TEST ---> menuconfig表示TEST是一个`可选`菜单，其选中后是CONFIG_TEST
    bool "菜单名"
if TEST
...
endif # TEST
```
* menu 和 choice 的区别：  
menu可以多选；choice为单选。
* menuconfig 和 menu 的区别：  
menuconfig本身是可以选中，而menu只能进入子菜单选择，一般menuconfig的句柄是子项的依赖项，而menu是子项的汇总。
