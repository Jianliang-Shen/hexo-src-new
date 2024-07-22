---
title: linux内核配置--Boot options
date: 2019-07-24 20:21:19
tags: 
    - Linux
    - 内核
    - Kconfig
categories: 
    - 操作系统
---


内核的配置过程依赖Makefile和arch/arm/Kconfig以及其他文件下的Kconfig文件，通过make menuconfig或者桌面环境下的xconfig/gconfig可以手动配置内核所支持的功能

<!-- more -->


以下为Boot options启动设置的配置。


## -Y- Flattened Device Tree support
支持设备树。  
Include support for flattened device tree machine descriptions.  
* `[Y] Support for the traditional ATAGS boot data passing`  
支持传统的ATAGS启动数据传递。除非仅依赖设备树，否则建议选择。  
ATAGS是传统的linux内核接收参数的方式，另一种是DTB。设备启动时，BOOT向内核传递三个参数，其中R0寄存器内容为一个“0”，R1是机器码（与内核匹配），R3为ATAGS或者DTB传递的地址。  
This is the traditional way of passing data to the kernel at boot time. If you are solely relying on the flattened device tree (or the ARM_ATAG_DTB_COMPAT option) then you may unselect this option to remove ATAGS support from your kernel binary. If unsure, leave this to y.    
  + `[N] Provide old way to pass kernel parameters`  
提供传递内核参数的旧方法。  
This was deprecated in 2001 and announced to live on for 5 years. Some old boot loaders still use this way.
  
## (0x0) Compressed ROM boot loader base address
default "0"  
存放基地址，对zImage压缩镜像启动很重要。  
The physical address at which the ROM-able zImage is to be placed in the target.  Platforms which normally make use of ROM-able zImage formats normally set this to a suitable value in their defconfig file.  
If ZBOOT_ROM is not enabled, this has no effect.
  
## (0x0) Compressed ROM boot loader BSS address
default "0"  
引导的BSS地址。  
The base address of an area of read/write memory in the target for the ROM-able zImage which must be available while the decompressor is running. It must be large enough to hold the entire decompressed kernel plus an additional 128 KiB.Platforms which normally make use of ROM-able zImage formats normally set this to a suitable value in their defconfig file.  
If ZBOOT_ROM is not enabled, this has no effect.
  
## [Y] Use appended device tree blob to zImage (EXPERIMENTAL)
将附加的设备树blob用于zImage（实验）。  
With this option, the boot code will look for a device tree binary (DTB) appended to zImage (e.g. cat zImage <filename>.dtb > zImage_w_dtb).This is meant as a backward compatibility convenience for those systems with a bootloader that can't be upgraded to accommodate the documented boot protocol using a device tree. Beware that there is very little in terms of protection against this option being confused by leftover garbage in memory that might look like a DTB header after a reboot if no actual DTB is appended to Image.  Do not leave this option active in a production kernel if you don't intend to always append a DTB.  
Proper passing of the location into r2 of a bootloader provided DTB is always preferable to this option.  
  * `[Y] Supplement the appended DTB with traditional ATAG information`  
用传统的ATAG信息补充附加的DTB  
Some old bootloaders can't be updated to a DTB capable one, yet they provide ATAGs with memory configuration, the ramdisk address, the kernel cmdline string, etc.Such information is dynamically provided by the bootloader and can't always be stored in a static DTB. To allow a device tree enabled kernel to be used with such bootloaders, this option allows zImage to extract the information from the ATAG list and store it at run time into the appended DTB.
  
## [C] Kernel command line type
内核命令行类型。  
  * `[X] Use bootloader kernel arguments if available`  
优先使用bootloader（uboot）参数，如果没有就使用DTB ARGS。  
因为通过uboot修改环境变量比修改内核容易得多，因此使用bootloader更方便。造成的问题是内核配置的变量可能被覆盖。  
Uses the command-line options passed by the boot loader instead of the device tree bootargs property. If the boot loader doesn't provide any, the device tree bootargs property will be used.  
  * `[N] Extend with bootloader kernel arguments`  
使用DTB ARGS，bootloader提供的追加在其之后。  
The command-line arguments provided by the boot loader will be appended to the the device tree bootargs property.
    
## (N) Default kernel command string
默认的内核命令行字符串。  
On some architectures (EBSA110 and CATS), there is currently no way for the boot loader to pass arguments to the kernel. For these architectures, you should supply some command-line options at build time by entering them here. As a minimum, you should specify the memory size and the root device (e.g., mem=64M root=/dev/nfs). 
  
## [N] Build kdump crash kernel (EXPERIMENTAL)
支持core dump的内核调试工具。Kdump是系统崩溃时，通过kexec工具转储内存（运行在一个主内核不占用的额外的内存区域）。  
Generate crash dump after being started by kexec. This should be normally only set in special crash dump kernels which are loaded in the main kernel with kexec-tools into a specially reserved region and then later executed after a crash by kdump/kexec. The crash dump kernel must be compiled to a memory address not used by the main kernel. For more details see Documentation/kdump/kdump.txt
  
## -Y- Auto calculation of the decompressed kernel image address
自动计算解压缩的内核映像地址。如果选择，开机后内核地址为0xf8000000，即在前128MB的位置（0x8000000=128*1024*1024）。  
ZRELADDR is the physical address where the decompressed kernel image will be placed. If AUTO_ZRELADDR is selected, the address will be determined at run-time by masking the current IP with 0xf8000000. This assumes the zImage being placed in the first 128MB from start of memory.
  
## [Y] UEFI runtime support
此选项提供对UEFI固件提供的运行时服务的支持（例如非易失性变量，实时时钟和平台重置）。还提供UEFI存根以允许内核作为EFI应用程序引导。这仅适用于可能在具有UEFI固件的系统上运行的内核。  
This option provides support for runtime services provided by UEFI firmware (such as non-volatile variables, realtime clock, and platform reset). A UEFI stub is also provided to allow the kernel to be booted as an EFI application. This is only useful for kernels that may run on systems that have UEFI firmware.  
  * `[Y] Enable support for SMBIOS (DMI) tables`  
This enables SMBIOS/DMI feature for systems. This option is only useful on systems that have UEFI firmware. However, even with this option, the resultant kernel should continue to boot on existing non-UEFI platforms.
NOTE: This does *NOT* enable or encourage the use of DMI quirks, i.e., the the practice of identifying the platform via DMI to decide whether certain workarounds for buggy hardware and/or firmware need to be enabled. This would require the DMI subsystem to be enabled much earlier than we do on ARM, which is non-trivial.
