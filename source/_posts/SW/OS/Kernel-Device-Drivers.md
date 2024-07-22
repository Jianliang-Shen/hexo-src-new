---
title: linux内核配置--Device drivers
date: 2019-07-23 13:20:57
tags: 
    - Linux
    - Kconfig
    - 内核
    - 驱动
categories: 
    - 操作系统
---
内核的配置过程依赖Makefile和arch/arm/Kconfig以及其他文件下的Kconfig文件，通过make menuconfig或者桌面环境下的xconfig/gconfig可以手动配置内核所支持的功能。

<!-- more -->  


以下为Device drivers启动设置的配置。

<details>
<summary>Generic Driver Options ---></summary>  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/base/Kconfig  
* `[Y] Support for uevent helper`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;支持uevent（例如热插拔）事件。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;早年的内核(切换到基于netlink机制之前),在发生uevent事件(通常是热插拔)时,需要调用用户空间程序(通常是"/sbin/hotplug"),以帮助完成uevent事件的处理.此选项就是用于开启此功能.由于目前的发行版都已不再需要此帮助程序,所以请选"N".此外,如果你使用了systemd或udev则必须选"N".  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The uevent helper program is forked by the kernel for every uevent. Before the switch to the netlink-based uevent source, this was used to hook hotplug scripts into kernel device events. It usually pointed to a shell script at /sbin/hotplug. This should not be used today, because usual systems create many events at bootup or device discovery in a very short time frame. One forked process per event can create so many processes that it creates a high system load, or on smaller systems it is known to create out-of-memory situations during bootup.  
  + `( ) path to uevent helper`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To disable user space helper program execution at by default specify an empty string here. This setting can still be altered via /proc/sys/kernel/hotplug or via /sys/kernel/uevent_helper later at runtime.  
* `[Y] Maintain a devtmpfs filesystem to mount at /dev`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在系统/dev文件夹下挂载devtmpf文件系统。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;devtmpfs是一种基于CONFIG_TMPFS的文件系统(与proc和sys有几分相似).在系统启动过程中,随着各个设备的初始化完成,内核将会自动在devtmpfs中创建相应的设备节点(使用默认的文件名和权限)并赋予正确的主次设备号.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This creates a tmpfs/ramfs filesystem instance early at bootup. In this filesystem, the kernel driver core maintains device nodes with their default names and permissions for all registered devices with an assigned major/minor number. Userspace can modify the filesystem content as needed, add  symlinks, and apply needed permissions.   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;It provides a fully functional /dev directory, where usually udev runs on top, managing permissions and adding meaningful symlinks.   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;In very limited environments, it may provide a sufficient functional /dev without any further help. It also allows simple rescue systems, and reliably handles dynamic major/minor numbers.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Notice: if CONFIG_TMPFS isn't enabled, the simpler ramfs file system will be used instead.  
  + `[Y] Automount devtmpfs at /dev, after the kernel mounted the rootfs`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在内核挂载根文件系统的同时，立即自动将devtmpfs挂载到"/dev"目录。因为此时init进程都还尚未启动，所以这就确保在进入用户空间之前，所有设备文件就都已经准备完毕.开启此选项相当于设置内核引导参数"devtmpfs.mount=1"，关闭此选项相当于设置内核引导参数"devtmpfs.mount=0"。开启此项后,你就可以放心的使用"init=/bin/sh"直接进入救援模式，而不必担心"/dev"目录空无一物。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意:此选项并不影响基于initramfs的启动，此种情况下，devtmpfs必须被手动挂载.所以,如果你的系统使用initrd或者有专门的启动脚本用于挂载"/dev"目录(大多数发行版都有这样的脚本)，或者你看了前面的解释，还是不确定，那就选"N".对于实在想要使用"init=/bin/sh"直接进入救援模式的人来说，还是使用"init=/bin/sh devtmpfs.mount=1"吧!  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This will instruct the kernel to automatically mount the devtmpfs filesystem at /dev, directly after the kernel has mounted the root filesystem. The behavior can be overridden with the commandline parameter: devtmpfs.mount=0|1. This option does not affect initramfs based booting, here the devtmpfs filesystem always needs to be mounted manually after the rootfs is mounted.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;With this option enabled, it allows to bring up a system in rescue mode with init=/bin/sh, even when the /dev directory on the rootfs is completely empty.  
* `[Y] Select only drivers that don't need compile-time external firmware`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;只显示那些编译时不需要额外固件支持的驱动程序，除非你有某些怪异硬件，否则请选"Y"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Select this option if you don't have magic firmware for drivers that need it. If unsure, say Y.  
* `[Y] Prevent firmware from being built`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不编译固件(firmware)。固件一般是随硬件的驱动程序提供的，仅在更新固件的时候才需要重新编译。建议选"Y"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say yes to avoid building firmware. Firmware is usually shipped with the driver and only when updating the firmware should a rebuild be made. If unsure, say Y here.  
* `{Y} Userspace firmware loading support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用户空间固件加载支持。如果内核自带的模块需要它，它将会被自动选中。但某些内核树之外的模块也可能需要它，这时候就需要你根据实际情况手动开启了.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option is provided for the case where none of the in-tree modules require userspace firmware loading support, but a module built out-of-tree does.  
  + `[Y] Include in-kernel firmware blobs in kernel binary`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内核源码树中包含了许多驱动程序需要的二进制固件(blob),推荐的方法是通过"make firmware_install"将"firmware"目录中所需的固件复制到系统的"/lib/firmware/"目录中,然后由用户空间帮助程序在需要的时候进行加载.开启此项后,将会把所需的"blob"直接编译进内核,这样就可以无需用户空间程序的帮助,而直接使用这些固件了(例如:当根文件系统依赖于此类固件,而你又不想使用initrd的时候).每个需要此类二进制固件的驱动程序,都会有一个"Include firmware for xxx device"的选项,如果此处选"Y",那么这些选项都将被隐藏.建议选"N".  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Various drivers in the kernel source tree may require firmware, which is generally available in your distribution's linux-firmware package.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The linux-firmware package should install firmware into /lib/firmware/ on your system, so they can be loaded by userspace helpers on request.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enabling this option will build each required firmware blob specified by EXTRA_FIRMWARE into the kernel directly, where request_firmware() will find them without having to call out to userspace. This may be useful if your root file system requires a device that uses such firmware and you do not wish to use an initrd.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This single option controls the inclusion of firmware for every driver that uses request_firmware(), which avoids a proliferation of 'Include firmware for xxx device' options.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say 'N' and let firmware be loaded from userspace.    
  + `( ) External firmware blobs to build into the kernel binary`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;指定要额外编译进内核的二进制固件(blob).此选项的值是一个空格分隔的固件文件名字符串,这些文件必须位于CONFIG_EXTRA_FIRMWARE_DIR目录中(其默认值是内核源码树下的"firmware"目录).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option allows firmware to be built into the kernel for the case where the user either cannot or doesn't want to provide it from userspace at runtime (for example, when the firmware in question is required for accessing the boot device, and the user doesn't want to use an initrd).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option is a string and takes the (space-separated) names of the firmware files -- the same names that appear in MODULE_FIRMWARE() and request_firmware() in the source. These files should exist under the directory specified by the EXTRA_FIRMWARE_DIR option, which is by default the firmware subdirectory of the kernel source tree.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For example, you might set CONFIG_EXTRA_FIRMWARE="usb8388.bin", copy the usb8388.bin file into the firmware directory, and build the kernel. Then any request_firmware("usb8388.bin") will be satisfied internally without needing to call out to userspace.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;WARNING: If you include additional firmware files into your binary kernel image that are not available under the terms of the GPL, then it may be a violation of the GPL to distribute the resulting image since it combines both GPL and non-GPL work. You should consult a lawyer of your own before distributing such an image.  
* `[N] Fallback user-helper invocation for firmware loading`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在内核自己直接加载固件失败后,作为补救措施,调用用户空间帮助程序(通常是udev)再次尝试加载.通常这个动作是不必要的,因此应该选"N",如果你使用了udev或systemd,则必须选"N".仅在某些特殊的固件位于非标准位置时,才需要选"Y".  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option enables / disables the invocation of user-helper (e.g. udev) for loading firmware files as a fallback after the direct file loading in kernel fails.  The user-mode helper is no longer required unless you have a special firmware file that resides in a non-standard path. Moreover, the udev support has been deprecated upstream.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you are unsure about this, say N here.  
* `[Y] Allow device coredump`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为驱动程序开启core dump机制,仅供调试.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;核心转储（core dump），是操作系统在进程收到某些信号而终止运行时，将此时进程地址空间的内容以及有关进程状态的其他信息写出的一个磁盘文件，这种信息往往用于调试，可以用gdb来调试。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option controls if the device coredump mechanism is available or not; if disabled, the mechanism will be omitted even if drivers that can use it are enabled.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say 'N' for more sensitive systems or systems that don't want to ever access the information to not have the code, nor keep any data.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say Y.  
* `[N] Driver Core verbose debug messages`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;让驱动程序核心在系统日志中产生冗长的调试信息,仅供调试  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if you want the Driver core to produce a bunch of debug messages to the system log. Select this if you are having a problem with the driver core and want to see more of what is going on.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you are unsure about this, say N here.  
* `[N] Managed device resources verbose debug messages`
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为内核添加一个"devres.log"引导参数.当被设为非零值时,将会打印出设备资源管理驱动(devres)的调试信息.仅供调试使用.推荐设置为N。   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option enables kernel parameter devres.log. If set to non-zero, devres debug messages are printed. Select this if you are having a problem with devres or want to debug resource management for a managed device. devres.log can be switched on and off from sysfs node.   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you are unsure about this, Say N here.  
* `[N] Test driver remove calls during probe (UNSTABLE)`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if you want the Driver core to test driver remove functions by calling probe, remove, probe. This tests the remove path without having to unbind the driver or unload the driver module.  This option is expected to find errors and may render your system unusable. You should say N here unless you are explicitly looking to test this functionality.  
* `<N> Build kernel module to test asynchronous driver probing`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enabling this option produces a kernel module that allows testing asynchronous driver probing by the device core. The module name will be test_async_driver_probe.ko  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure say N.  
* `[N] Enable verbose DMA_FENCE_TRACE messages`   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enable the DMA_FENCE_TRACE printks. This will add extra spam to the console log, but will make it easier to diagnose lockup related problems for dma-buffers shared across multiple devices.  
* `[Y] DMA Contiguous Memory Allocator`    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This enables the Contiguous Memory Allocator which allows drivers to allocate big physically-contiguous blocks of memory for use with hardware components that do not support I/O map nor scatter-gather.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;You can disable CMA by specifying "cma=0" on the kernel's command line.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For more information see <include/linux/dma-contiguous.h>. If unsure, say "n".  
  + `***Default contiguous memory area size:***`  
  + `(24) Size in Mega Bytes`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;default 0 if X86  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;default 16  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Defines the size (in MiB) of the default memory area for Contiguous Memory Allocator.  If the size of 0 is selected, CMA is disabled by default, but it can be enabled by passing cma=size[MG] to the kernel.  
  + `(C) Selected region size`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(X) Use mega bytes value only  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(N) Use percentage value only  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(N) Use lower value (minimum)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(N) Use higher value (maximum)  
  + `(8) Maximum PAGE_SIZE order of alignment for contiguous buffers`  
  
</details>  
  
<details>
<summary>Bus Devices ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/bus/Kconfig  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;总线设备.此类设备仅出现在ARM平台。下列是勾选的：  
* `<Y> OMAP INTERCONNECT DRIVER`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;支持TI的omap开放式多媒体应用平台  
* `<Y> OMAP OCP2SCP DRIVER`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ocp/scp协议  
* `<Y> Simple Power-Managed Bus Driver`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;电源管理总线  
  
</details>  
  

<details>
<summary>< > Connector - unified userspace <-> kernelspace linker</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;连接器是一种新的用户态与内核态的通信方式，它使用起来非常方便。本质上，连接器是一种netlink，它的netlink协议号为 NETLINK_CONNECTOR，与一般的 netlink 相比，它提供了更容易的使用接口，使用起来更方便。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The Connector driver makes it easy to connect various agents using a netlink based network. One must register a callback and an identifier. When the driver receives a special netlink message with the appropriate identifier, the appropriate callback will be called.  
  
</details>  
  
<details>
<summary><Y> Memory Technology Device (MTD) support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;详细参考/drivers/mtd/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MTD子系统是一个闪存转换层。其主要目的是提供一个介于闪存硬件驱动程序与高级应用程序之间的抽象层，以简化闪存设备的驱动。注意：MTD常用于嵌入式系统，而我们常见的U盘/MMC卡/SD卡/CF卡等移动存储设备以及固态硬盘(SSD)，虽然也叫"flash"，但它们并不是使用MTD技术的存储器。仅在你需要使用主设备号为31的MTD块设备(/dev/romX、/dev/rromX、/dev/flashX、/dev/rflashX)，或者主设备号为90的MTD字符设备(/dev/mtdX、/dev/mtdrX)时选"Y"，否则选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Memory Technology Devices are flash, RAM and similar chips, often used for solid state file systems on embedded devices. This option will provide the generic support for MTD drivers to register themselves with the kernel and for potential users of MTD devices to enumerate the devices which are present and obtain a handle on them. It will also allow you to select individual drivers for particular hardware and users of MTD devices. If unsure, say N.  
  
</details>  
  

<details>
<summary>-Y- Device Tree and Open Firmware support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/mtd/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Device Tree基础架构与Open Firmware支持，主要用于嵌入式环境。不确定的选"N"。内核中若有其它选项依赖于它，则会自动选中此项。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option enables the device tree infrastructure. It is automatically selected by platforms that need it or can be enabled manually for unit tests, overlays or compile-coverage.  
  
</details>  
  

<details>
<summary>< > Parallel port support -----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/parport/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;25针并口(LPT接口)支持，古董级的打印机或扫描仪可能使用这种接口。目前已被淘汰。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you want to use devices connected to your machine's parallel port (the connector at the computer with 25 holes), e.g. printer, ZIP drive, PLIP link (Parallel Line Internet Protocol is mainly used to create a mini network by connecting the parallel ports of two local machines) etc., then you need to say Y here; please read <file:Documentation/parport.txt> and <file:drivers/parport/BUGS-parport>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For extensive information about drivers for many devices attaching to the parallel port see <http://www.torque.net/linux-pp.html> on the WWW. It is possible to share a single parallel port among several devices and it is safe to compile all the corresponding drivers into the kernel. To compile parallel port support as a module, choose M here: the module will be called parport. If you have more than one parallel port and want to specify which port and IRQ to be used by this driver at module load time, take a look at <file:Documentation/parport.txt>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say Y.  
  
</details>  
  
<details>
<summary>[Y] Block devices -----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;块设备，必选。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/block/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to get to see options for various different block device drivers. This option alone does not add any kernel code.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say N, all options in this submenu will be skipped and disabled; only do this if you know what you are doing.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列是勾选的：  
* `<Y> Loopback device support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在进行某些测试的时候，往往需要新建一些磁盘分区或者设备（ISO9660镜像文件或者CD-ROM）等，此时对硬盘进行重新划分往往不太方便。在这种情况下，可以通过loop伪设备来实现循环挂载，从而达到目的。在使用之前，循环设备必须与现存文件系统上的文件相关联。这种关联将提供给用户一个应用程序接口，接口将允许文件视为块特殊文件（参见设备文件系统）使用。因此，如果文件中包含一个完整的文件系统，那么这个文件就能如同磁盘设备一般被挂载。这种设备文件经常被用于光盘或是磁盘镜像。通过循环挂载来挂载包含文件系统的文件，便使处在这个文件系统中的文件得以被访问。这些文件将出现在挂载点目录。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Linux中，设备名按照相应设备驱动程序的符号表项进行命名。这些设备被叫做“loop”设备，设备节点通常命名为/dev/loop0、/dev/loop1之类。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;默认情况下Linux支持的loop设备是8个。如果需要超过8个的loop设备，那么可能会遇到类似的错误“no such device”或“could not find any free loop device”，这是因为超过了可用loop设置设备的最大限制。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Saying Y here will allow you to use a regular file as a block device; you can then create a file system on that block device and mount it just as you would mount other block devices such as hard drive partitions, CD-ROM drives or floppy drives. The loop devices are block special device files with major number 7 and typically called /dev/loop0, /dev/loop1 etc.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This is useful if you want to check an ISO 9660 file system before burning the CD, or if you want to use floppy images without first writing them to floppy. Furthermore, some Linux distributions avoid the need for a dedicated Linux partition by keeping their complete root file system inside a DOS FAT file using this loop device driver.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To use the loop device, you need the losetup utility, found in the util-linux package, see <https://www.kernel.org/pub/linux/utils/util-linux/>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The loop device driver can also be used to "hide" a file system in a disk partition, floppy, or regular file, either using encryption (scrambling the data) or steganography (hiding the data in the low bits of, say, a sound file). This is also safe if the file resides on a remote file server.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;There are several ways of encrypting disks. Some of these require kernel patches. The vanilla kernel offers the cryptoloop option and a Device Mapper target (which is superior, as it supports all file systems). If you want to use the cryptoloop, say Y to both LOOP and CRYPTOLOOP, and make sure you have a recent (version 2.12 or later) version of util-linux. Additionally, be aware that the cryptoloop is not safe for storing journaled filesystems.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Note that this loop device has nothing to do with the loopback device used for network connections from the machine to itself. To compile this driver as a module, choose M here: the module will be called loop.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Most users will answer N here.  
  + `(8) Number of loop devices to pre-create at init time`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;设置loop分区启动时默认分配的个数。  
* `<Y> RAM block device support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在内存中开辟一个和普通存储同样支持读写的块设备，一般用于启动时存放一个最小文件系统。详细参考<file:Documentation/blockdev/ramdisk.txt>，一般选择N。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Saying Y here will allow you to use a portion of your RAM memory as a block device, so that you can make file systems on it, read and write to it and do all the other things that you can do with normal block devices (such as hard drives). It is usually used to load and store a copy of a minimal root file system off of a floppy into RAM during the initial install of Linux.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Note that the kernel command line option "ramdisk=XX" is now obsolete. For details, read <file:Documentation/blockdev/ramdisk.txt>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called brd. An alias "rd" has been defined for historical reasons.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Most normal users won't need the RAM disk functionality, and can thus say N here.  
  + `(16) Default number of RAM disks`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;设置默认RAM disks的个数。  
  + `(65536) Default RAM disk size (kbytes)`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;设置默认大小。  
* `<Y> Virtio block driver`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Virtio虚拟块设备驱动，仅可用在基于lguest或QEMU的半虚拟化客户机中(一般是KVM或XEN)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This is the virtual block driver for virtio. It can be used with QEMU based VMMs (like KVM or Xen). Say Y or M.  
  
</details>  
  
<details>
<summary>< > NVM Express block device</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NVM Express是专门针对PCI-E接口高性能固态硬盘的标准规范。有了这一标准，操作系统厂商只需要编写一种驱动，就可以支持不同厂商的不同PCI-E SSD设备，以解决过去PCI-E SSD产品形态与规格五花八门，缺乏通用性和互用性的问题。如果你有一块较新的PCIE固态硬盘，那么很大可能就是NVMe接口。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The NVM Express driver is for solid state drives directly connected to the PCI or PCI Express bus.  If you know you don't have one of these, it is safe to answer N. To compile this driver as a module, choose M here: the module will be called nvme.  
  
</details>
  
<details>
<summary>< > NVM Express over Fabrics FC host driver</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This provides support for the NVMe over Fabrics protocol using the FC transport.This allows you to use remote block devices exported using the NVMe protocol set.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To configure a NVMe over Fabrics controller use the nvme-cli tool from https://github.com/linux-nvme/nvme-cli. If unsure, say N.
  
</details>
  


<details>
<summary><N> NVMe Target support </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This enabled target side support for the NVMe protocol, that is it allows the Linux kernel to implement NVMe subsystems and controllers and export Linux block devices as NVMe namespaces. You need to select at least one of the transports below to make this functionality useful.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To configure the NVMe target you probably want to use the nvmetcli tool from http://git.infradead.org/users/hch/nvmetcli.git.  
  
</details>  
  

<details>
<summary><N> Misc devices ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其他设备。以下是勾选的：  
* `<Y> Analog Devices Digital Potentiometers`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say yes here, you get support for the Analog Devices AD5258, AD5259, AD5251, AD5252, AD5253, AD5254, AD5255 AD5160, AD5161, AD5162, AD5165, AD5200, AD5201, AD5203, AD5204, AD5206, AD5207, AD5231, AD5232, AD5233, AD5235, AD5260, AD5262, AD5263, AD5290, AD5291, AD5292, AD5293, AD7376, AD8400, AD8402, AD8403, ADN2850, AD5241, AD5242, AD5243, AD5245, AD5246, AD5247, AD5248, AD5280, AD5282, ADN2860, AD5273, AD5171, AD5170, AD5172, AD5173, AD5270, AD5271, AD5272, AD5274 digital potentiometer chips.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;See Documentation/misc-devices/ad525x_dpot.txt for the userspace interface.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This driver can also be built as a module. If so, the module will be called ad525x_dpot.  
  + `<Y> support I2C bus connection`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if you have a digital potentiometers hooked to an I2C bus. To compile this driver as a module, choose M here: the module will be called ad525x_dpot-i2c.  
* `<Y> Integrated Circuits ICS932S401`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;依赖I2C驱动。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IDT ICS932S401系列时钟频率控制芯片支持(可能会出现在某些主板上)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say yes here you get support for the Integrated Circuits ICS932S401 clock control chips. This driver can also be built as a module. If so, the module will be called ics932s401.  
* `<Y> Medfield Avago APDS9802 ALS Sensor module`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;依赖I2C驱动。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say yes here you get support for the ALS APDS9802 ambient light sensor.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This driver can also be built as a module. If so, the module will be called apds9802als.  
* `<Y> Intersil ISL29003 ambient light sensor`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;依赖I2C和SYSFS驱动。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say yes here you get support for the Intersil ISL29003ambient light sensor. This driver can also be built as a module. If so, the module will be called isl29003.  
* `[Y] Generic on-chip SRAM driver`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;许多SoC系统都有芯片内嵌的SRAM。开启此项后，就可以声明将此段内存范围交给通用内存分配器(genalloc)管理。不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This driver allows you to declare a memory region to be managed by the genalloc API. It is supposed to be used for small on-chip SRAM areas found on many SoCs.  
* `<M> PCI Endpoint Test driver`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PCI相关。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enable this configuration option to enable the host side test driver for PCI Endpoint.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;EEPROM support --->  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一些eeprom设备的支持。  
  + `<M> I2C EEPROMs / RAMs / ROMs from most vendors`  
  + `-Y- EEPROM 93CX6 support`  
  
</details>  
  
<details>
<summary> ATA/ATAPI/MFM/RLL support (DEPRECATED) ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;已被废弃的IDE硬盘和ATAPI光驱等接口的驱动(已被CONFIG_ATA取代)。  
  
</details>
  

<details>
<summary>SCSI device support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果有SCSI（小型计算机系统接口）设备，例如硬盘、软驱、光驱、打印机以及扫描仪等，就需要勾选。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/scsi/support。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you want to use a SCSI hard disk, SCSI tape drive, SCSI CD-ROM or any other SCSI device under Linux, say Y and make sure that you know the name of your SCSI host adapter (the card inside your computer that "speaks" the SCSI protocol, also called SCSI controller), because you will be asked for it.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;You also need to say Y here if you have a device which speaks the SCSI protocol.  Examples of this include the parallel port version of the IOMEGA ZIP drive, USB storage devices, Fibre Channel, and FireWire storage.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here and read <file:Documentation/scsi/scsi.txt>. The module will be called scsi_mod.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;However, do not compile this as a module if your root file system (the one containing the directory /) is located on a SCSI device.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选的子项：  
* `{M} SCSI device support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SCSI协议支持。有任何SCSI/SAS/SATA/USB/Fibre Channel/FireWire设备之一就必须选上，选"Y"。注意USB设备驱动也需要此项支持！  
* `[Y] legacy /proc/scsi/ support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;过时的/proc/scsi/接口。某些老旧的刻录程序可能需要它，建议选"N"。  
* `<M> SCSI disk support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用SCSI/SAS/SATA/PATA/USB/Fibre Channel存储设备的必选，选"Y"。  
* `<M> SCSI CDROM support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过SCSI/FireWire/USB/SATA/IDE接口连接的DVD/CD驱动器(基本上涵盖了所有常见的接口)。  
* `[Y] SCSI low-level drivers --->`    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;底层SCSI驱动程序。  
  
</details>
  
<details>
<summary><M> Serial ATA and Parallel ATA drivers (libata)--->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SATA与PATA(IDE)设备。桌面级PC以及低端服务器的硬盘基本都是此种接口。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/ata/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you want to use an ATA hard disk, ATA tape drive, ATA CD-ROM or any other ATA device under Linux, say Y and make sure that you know the name of your ATA host adapter (the card inside your computer that "speaks" the ATA protocol, also called ATA controller), because you will be asked for it.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NOTE: ATA enables basic SCSI support; *however*, 'SCSI disk support', 'SCSI tape support', or 'SCSI CDROM support' may also be needed, depending on your hardware configuration.  
  
</details>
  

<details>
<summary>[N] Multiple devices driver support (RAID and LVM) ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;多设备支持(RAID和LVM)。RAID和LVM的功能是使用多个物理设备组建成一个单独的逻辑设备。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/md/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Support multiple physical spindles through a single logical device. Required for RAID and logical volume management.  
  
</details>
  

<details>
<summary><N> Generic Target Core Mod (TCM) and ConfigFS Infrastructure ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通用TCM存储引擎与ConfigFS虚拟文件系统(/sys/kernel/config)支持，看不懂就说明你不需要。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/target/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y or M here to enable the TCM Storage Engine and ConfigFS enabled control path for target_core_mod.  This includes built-in TCM RAMDISK subsystem logic for virtual LUN 0 access.  
  
</details>
  

<details>
<summary>[N] Fusion MPT device support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fusion MPT(Message Passing Technology) 是 LSI Logic 公司为了更容易实现SCSI和光纤通道而提出的技术，支持Ultra320 SCSI/光纤通道/SAS。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/message/fusion/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to get to see options for Fusion Message Passing Technology (MPT) drivers.This option alone does not add any kernel code.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say N, all options in this submenu will be skipped and disabled.  
  
</details>
  

<details>
<summary>IEEE 1394 (FireWire) support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;火线(IEEE 1394)是苹果公司开发的串行接口，类似于USB，但PC上并不常见，算得上是个没有未来的技术了。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/firewire/Kconfig。  
  
</details>
  

<details>
<summary>[Y] Network device support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;网络设备。除非你不想连接任何网络，否则必选"Y"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/net/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;You can say N here if you don't intend to connect your Linux box to any other computer at all.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;You'll have to say Y if your computer contains a network card that you want to use under Linux. If you are going to run SLIP or PPP over telephone line or null modem cable you need say Y here. Connecting two machines with parallel ports using PLIP needs this, as well as AX.25/KISS for sending Internet traffic over amateur radio links.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;See also "The Linux Network Administrator's Guide" by Olaf Kirch and Terry Dawson. Available at <http://www.tldp.org/guides.html>. If unsure, say Y.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选的：  
* `[Y] Network core driver support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果你不想使用任何高级网络功能(拨号网络/EQL/VLAN/bridging/bonding/team/光纤通道/虚拟网络等)，仅仅是一般性质的联网(普通低端服务器，通过路由器或者局域网上网的常规个人电脑或办公电脑)，可以选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;You can say N here if you do not intend to use any of the networking core drivers (i.e. VLAN, bridging, bonding, etc.)  
  + `<M> Dummy net driver support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Dummy网络接口本质上是一个可以配置IP地址的bit-bucket(位桶，所有发送到此设备的流量都将被湮灭)，以使应用程序看上去正在和一个常规的网络接口进行通信。使用SLIP(小猫拨号，目前应该已经绝迹了)或PPP(常用于PPPoE ADSL)的用户需要它。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This is essentially a bit-bucket device (i.e. traffic you send to this device is consigned into oblivion) with a configurable IP address. It is most commonly used in order to make your currently inactive SLIP address seem like a real address for local programs.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you use SLIP or PPP, you might want to say Y here. It won't enlarge your kernel. What a deal. Read about it in the Network Administrator's Guide, available from <http://www.tldp.org/docs.html#guide>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called dummy.  
  + `<M> MAC-VLAN support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MAC-VLAN是通过MAC地址来划分VLAN的方式,在Linux则用来给网卡添加多个MAC地址。你可以使用"ip link add link <real dev> [ address MAC ] [ NAME ] type macvlan"命令创建一个虚拟的"macvlan"设备(系统会自动打开网卡的混杂模式)，然后就可以在同一个物理网卡上虚拟出多个以太网口。Docker依赖于它.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This allows one to create virtual interfaces that map packets to or from specific MAC addresses to a particular interface.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Macvlan devices can be added using the "ip" command from the iproute2 package starting with the iproute2-2.6.23 release:"ip link add link <real dev> [ address MAC ] [ NAME ] type macvlan"   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called macvlan.  
  + `<M> Virtual eXtensible Local Area Network (VXLAN)`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;“vxlan”虚拟接口可以在第三层网络上创建第二层网络(跨多个物理IP子网的虚拟二层子网)，是一种在UDP中封装MAC的简单机制，主要用于虚拟化环境下的隧道虚拟网络(tunnel virtual network)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This allows one to create vxlan virtual interfaces that provide Layer 2 Networks over Layer 3 Networks. VXLAN is often used to tunnel virtual network infrastructure in virtualized environments.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For more information see: http://tools.ietf.org/html/draft-mahalingam-dutt-dcops-vxlan-02  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called vxlan.  
  + `<M> Virtual ethernet pair device`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该驱动提供了一个本地以太网隧道(设备会被成对的创建)，Docker依赖于它。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This device is a local ethernet tunnel. Devices are created in pairs. When one end receives the packet it appears on its pair and vice versa.  
  + `<Y> Virtio network driver	`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;virtio虚拟网卡驱动，仅可用在基于lguest或QEMU的半虚拟化客户机中(一般是KVM或XEN)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This is the virtual network driver for virtio.  It can be used with QEMU based VMMs (like KVM or Xen). Say Y or M.  
* `[Y] Ethernet driver support --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最常见的以太网卡驱动，针对各厂商。参考：drivers/net/ethernet/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This section contains all the Ethernet device drivers  
* `-Y- MDIO bus device drivers ----`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MDIO devices and driver infrastructure code.  
* `-Y- PHY Device support and infrastructure --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数据链路层芯片简称为MAC控制器，物理层芯片简称之为PHY，通常的网卡把MAC和PHY的功能做到了一颗芯片中，但也有一些仅含PHY的“软网卡”。此选项就是对这些“软网卡”的支持。请根据实际情况选择其下的子项，参考：drivers/net/phy/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ethernet controllers are usually attached to PHY devices. This option provides infrastructure for managing PHY devices.  
* `<M> USB Network Adapters --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;USB网络适配器，参考：drivers/net/usb/Kconfig。  
* `[Y] Wireless LAN --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;无线网卡，参考：drivers/net/wireless/Kconfig。  
  
</details>
  

<details>
<summary>[N] Open-Channel SSD target support ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Open-channel SSD 是一种遵守NVMe规范且不使用FTL技术的固态硬盘。目前此种SSD由于过于前卫还非常罕见，但是非常有前途。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/lightnvm/Kconfig。  
  
</details>
  

<details>
<summary>Input device support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;输入设备。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/input/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `-Y- Generic input layer (needed for keyboard, mouse, ...)`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通用输入层。只要你有任何输入设备(键盘、鼠标、手写板、触摸板、游戏杆、方向盘,游戏键盘...)，就必须选"Y"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if you have any input device (mouse, keyboard, tablet, joystick, steering wheel ...) connected to your system and want it to be available to applications. This includes standard PS/2 keyboard and mouse.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say N here if you have a headless (no monitor, no keyboard) system.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;More information is available: <file:Documentation/input/input.txt>  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say Y.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called input.  
* `<Y> Export input device LEDs in sysfs`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将输入设备上的LED指示灯当作标准的LED类设备导出到sysfs中。不确定的选"Y"。  
* `{M} Polled input device skeleton`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用轮询机制的输入设备支持，此项主要是为源码树之外的驱动准备的，内核自带的驱动若有需要会自动选中，不确定的选"N"。  
* `*{M} Matrix keymap support library`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用"matrix keymap"的输入设备支持，此项主要是为源码树之外的驱动准备的，内核自带的驱动若有需要会自动选中，不确定的选"N"。  
* `<Y> Joystick interface`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;游戏杆(joystick)和游戏键盘(gamepad)支持(/dev/input/jsX)。  
* `<Y> Event interface`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将所有的输入设备事件都通过"/dev/input/eventX"以一种通用的方式进行处理.Xorg需要使用此接口，不确定的选"Y"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if you want your input device events be accessible under char device 13:64+ - /dev/input/eventX in a generic way. To compile this driver as a module, choose M here: the module will be called evdev.  
* `[Y] Keyboards --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;键盘。  
* `[Y] Mice --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;鼠标。  
* `[Y] Touchscreens --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;触摸屏。  
* `[Y] Miscellaneous devices --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其他杂项输入，支持GPIO、SPI、I2C，参考：drivers/input/misc/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here, and a list of miscellaneous input drivers will be displayed. Everything that didn't fit into the other categories is here. This option doesn't affect the kernel.  
  
</details>
  

<details>
<summary>Character devices ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符设备。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `[Y] Enable TTY`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字符终端和串口都需要TTY的支持，选"Y"。参考/drivers/tty/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Allows you to remove TTY support which can save space, and blocks features that require TTY from inclusion in the kernel. TTY is required for any text terminals or serial port communication. Most users should leave this enabled.  
  + `[Y] Virtual terminal`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;虚拟终端可以在一个物理终端设备上虚拟出多个"显示器+键盘"的组合(可以使用"Alt+Fn"组合键在多个虚拟终端间切换)，除非是嵌入式系统，否则必选"Y"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say Y here, you will get support for terminal devices with display and keyboard devices. These are called "virtual" because you can run several virtual terminals (also called virtual consoles) on one physical terminal. This is rather useful, for example one virtual terminal can collect system messages and warnings, another one can be used for a text-mode user session, and a third could run an X session, all in parallel. Switching between virtual terminals is done with certain key combinations, usually Alt-<function key>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The setterm command ("man setterm") can be used to change the properties (such as colors or beeping) of a virtual terminal. The man page console_codes(4) ("man console_codes") contains the special character sequences that can be used to change those properties directly. The fonts used on virtual terminals can be changed with the setfont ("man setfont") command and the key bindings are defined with the loadkeys ("man loadkeys") command.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;You need at least one virtual terminal device in order to make use of your keyboard and monitor. Therefore, only people configuring an embedded system would want to say N here in order to save some memory; the only way to log into such a system is then via a serial or network connection.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say Y, or else you won't be able to do much with your new shiny Linux system :-)  
    + `[Y] Enable character translations in console`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在虚拟控制台(console)上支持字体映射和Unicode转换。  
    + `[Y] Support for console on virtual terminal`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内核默认将第一个虚拟终端(/dev/tty0)用作系统控制台(可以通过"console=tty3"这样的参数去修改)，将诸如模块错误/内核错误/启动信息之类的警告信息发送到这里，而且以单用户模式登录时也需要使用这个控制台。若选"N"则会导致黑屏，除非是嵌入式系统，否则必选"Y"。  
    + `-Y- Support for binding and unbinding console drivers`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;虚拟终端是通过控制台驱动程序与物理终端交互的，但在某些系统上可以使用多个控制台驱动程序(如framebuffer控制台驱动程序)，该选项使得你可以选择其中之一。  
  + `[Y] Unix98 PTY support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;伪终端(PTY)是指一个"软件终端"，它是由slave(等价于一个物理终端)和master(被一个诸如xterm之类的进程用来读写slave设备)两部分组成的软设备。图形界面用户与需要支持ssh/telnet远程登录者必选。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A pseudo terminal (PTY) is a software device consisting of two halves: a master and a slave. The slave device behaves identical to a physical terminal; the master device is used by a process to read data from and write data to the slave, thereby emulating a terminal. Typical programs for the master side are telnet servers and xterms.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux has traditionally used the BSD-like names /dev/ptyxx for masters and /dev/ttyxx for slaves of pseudo terminals. This scheme has a number of problems. The GNU C library glibc 2.1 and later, however, supports the Unix98 naming standard: in order to acquire a pseudo terminal, a process opens /dev/ptmx; the number of the pseudo terminal is then made available to the process and the pseudo terminal slave can be accessed as /dev/pts/<number>. What was traditionally /dev/ttyp2 will then be /dev/pts/2, for example.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;All modern Linux systems use the Unix98 ptys.  Say Y unless you're on an embedded system and want to conserve memory.  
  + `[Y] Legacy (BSD) PTY support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用过时的BSD风格的/dev/ptyxx作为master，/dev/ttyxx作为slave，这个方案有一些安全问题，建议选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A pseudo terminal (PTY) is a software device consisting of two halves: a master and a slave. The slave device behaves identical to a physical terminal; the master device is used by a process to read data from and write data to the slave, thereby emulating a terminal. Typical programs for the master side are telnet servers and xterms.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux has traditionally used the BSD-like names /dev/ptyxx for masters and /dev/ttyxx for slaves of pseudo terminals. This scheme has a number of problems, including security.  This option enables these legacy devices; on most systems, it is safe to say N.  
    + `(256) Maximum number of legacy PTY in use`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The maximum number of legacy PTYs that can be used at any one time. The default is 256, and should be more than enough. Embedded systems may want to reduce this to save memory. When not in use, each legacy PTY occupies 12 bytes on 32-bit architectures and 24 bytes on 64-bit architectures.  
* `[Y] /dev/mem virtual device support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"/dev/mem"虚拟设备是整个处理器地址空间的全映射(包括所有物理内存/设备IO空间/总线映射空间)，可以用来直接存取物理内存，常用于访问物理IO设备，例如dmidecode工具可以从中提取系统识别信息(序列号、制造商、型号、等等)，或者Xorg可以用来访问显卡的物理内存或者实现用户空间驱动，同时拥有root权限的攻击者也可以使用它完成很多标准rootkit的行为。如果你需要使用用户空间的驱动或不确定，那么选"Y"。如果你觉得安全特别重要，可以选"N"。  
* `*Serial drivers --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;串口(COM)驱动,串口在台式机主板上正在逐渐消亡,而在笔记本和服务器上早就已经绝迹了.大多数人应该将所有子项都选"N"。但在嵌入式系统仍然使用，参考/drivers/tty/serial/Kconfig。下列为勾选项：  
  + `<Y> 8250/16550 and compatible serial support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是标准串口(COM)驱动，只要你想使用串口，就必选此项。  
  + `<Y> Support for OMAP internal UART (8250 based driver)`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TI OMAP平台芯片可选。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you have a machine based on an Texas Instruments OMAP CPU you can enable its onboard serial ports by enabling this option. This driver uses ttyS instead of ttyO.  
    + `[Y] Replace ttyO with ttyS`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将ttyO代替为ttyS。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option replaces the "console=ttyO" argument with the matching ttyS argument if the user did not specified it on the command line. This ensures that the user can see the kernel output during boot which he wouldn't see otherwise. The getty has still to be configured for ttyS instead of ttyO regardless of this option. This option is intended for people who "automatically" enable this driver without knowing that this driver requires a different console= argument. If you read this, please keep this option disabled and instead update your kernel command line. If you prepare a kernel for a distribution or other kind of larger user base then you probably want to keep this option enabled. Otherwise people might complain about a not booting kernel because the serial console remains silent in case they forgot to update the command line.  
  + `<Y> Devicetree based probing for 8250 ports`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;没有相关资料默认选择。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option is used for all 8250 compatible serial ports that are probed through devicetree, including Open Firmware based PowerPC systems and embedded systems on architectures using the flattened device tree format.  
  + `<Y> Broadcom BCM63xx/BCM33xx UART support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;没有相关资料默认选择。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This enables the driver for the onchip UART core found on the following chipsets:      
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BCM33xx (cable modem)   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BCM63xx/BCM63xxx (DSL)   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BCM68xx (PON)   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BCM7xxx (STB) - DOCSIS console  
* `<Y> Virtio console`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Virtio虚拟控制台设备驱动，此外，该驱动还可以作为普通的串口设备(/dev/vportNpX)，用于客户机和宿主机之间的通信。仅可用在基于lguest或QEMU的半虚拟化客户机中(一般是KVM或XEN)。参考：drivers/char/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Also serves as a general-purpose serial device for data transfer between the guest and host.  Character devices at /dev/vportNpn will be created when corresponding ports are found, where N is the device number and n is the port number within that device.  If specified by the host, a sysfs attribute called 'name' will be populated with a name for the port which can be used by udev scripts to create a symlink to the device.  
* `<Y> Hardware Random Number Generator Core support --->`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;硬件随机数发生器设备(/dev/hw_random)支持。此设备并不会直接向内核的随机数发生器填充(这是"rngd"守护进程的职责)。详情参见"Documentation/hw_random.txt"文档。参考：drivers/char/hw_random/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hardware Random Number Generator Core infrastructure.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called rng-core.  This provides a device that's usually called /dev/hwrng, and which exposes one of possibly several hardware random number generators.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;These hardware random number generators do feed into the kernel's random number generator entropy pool.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say Y.  
* `[Y] /dev/port character device`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/char/Kconfig  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if you want to support the /dev/port device. The /dev/port device is similar to /dev/mem, but for I/O ports.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;I2C support --->  
  
</details>
  

<details>
<summary>-Y- I2C support</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;I2C与SMBus支持，I2C(读着"I-squared-C")是用于单片机(又称"微控制器")的低速串行总线协议,它为微控制器(Microcontroller)与各种不同的低速设备通信提供了一种廉价的总线(因为只需要使用两个引脚,称为"2线")，因此广泛的应用于嵌入式环境.SMBus(System Management Bus)差不多相当于是I2C的子集，最初的目的是为了管理智能电池，现在常用于硬件监控(电压/风扇转速/温度/电池等)以及内存模块的配置(使用I2C EEPROM)，因此所有PC主板都依赖于SMBus协议。系统硬件监控工具lm_sensors和i2c-tools依赖于此模块，硬件传感器和"Video For Linux"也需要该模块的支持。详情参见"Documentation/i2c/summary"文档及整个"i2c"文件夹。不确定的选"Y"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/i2c/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;I2C (pronounce: I-squared-C) is a slow serial bus protocol used in many micro controller applications and developed by Philips.  SMBus, or System Management Bus is a subset of the I2C protocol.  More information is contained in the directory <file:Documentation/i2c/>, especially in the file called "summary" there.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Both I2C and SMBus are supported here. You will need this for hardware sensors support, and also for Video For Linux support.If you want I2C support, you should say Y here and also to the specific driver for your bus adapter(s) below.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This I2C support can also be built as a module.  If so, the module will be called i2c-core.  
  
</details>
  

<details>
<summary>[Y] SPI support  ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;串行外设接口(Serial Peripheral Interface)是一种标准的四线同步双向串行总线，SPI类似于I2C，但比I2C的"2线"稍微复杂一些，SPI需要4个引脚("4线")，不但传输速率比I2C更高，还能实现全双工通信。大多数SPI设备不支持动态设备检测，有些甚至是只读或者只写的。SPI常用于微控制器(Microcontroller)与外围设备(RTC、传感器、EEPROM、FLASH、解/编码器、模数转换器、数字信号处理器)之间的通信，MMC和SD卡也可以通过SPI协议访问，而MMC接口的DataFlash卡则必须通过SPI才能访问。仅用于嵌入式环境，PC平台上没有这样的设备。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/spi/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The "Serial Peripheral Interface" is a low level synchronous protocol.  Chips that support SPI can have data transfer rates up to several tens of Mbit/sec.  Chips are addressed with a controller and a chipselect.  Most SPI slaves don't support dynamic device discovery; some are even write-only or read-only.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SPI is widely used by microcontrollers to talk with sensors, eeprom and flash memory, codecs and various other controller chips, analog to digital (and d-to-a) converters, and more. MMC and SD cards can be accessed using SPI protocol; and for DataFlash cards used in MMC sockets, SPI must always be used.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SPI is one of a family of similar protocols using a four wire interface (select, clock, data in, data out) including Microwire (half duplex), SSP, SSI, and PSP.  This driver framework should work with most such devices and controllers.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;重要的子项：  
* `<Y> User mode SPI device driver support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This supports user mode SPI protocol drivers. Note that this application programming interface is EXPERIMENTAL and hence SUBJECT TO CHANGE WITHOUT NOTICE while it stabilizes.  
  
</details>
  

<details>
<summary><Y> SPMI support ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;系统电源管理接口(SPMI, System Power Management Interface)是一种连接PMIC(Power Management Integrated Circuits)的双线串行接口，仅用于嵌入式环境。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/spmi/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SPMI (System Power Management Interface) is a two-wire serial interface between baseband and application processors and Power Management Integrated Circuits (PMIC).  
  
</details>
  

<details>
<summary><N> HSI support ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;高速同步串行接口(High speed synchronous Serial Interface)是移动产业处理器接口(MIPI)联盟的高速同步接口工作组发布的一项技术规范。MIPI(Mobile Industry Processor Interface)是2003年由ARM、Nokia、ST、TI等公司成立的一个联盟，目的是把手机内部的接口(如摄像头、显示屏接口、射频/基带接口等)标准化，从而减少手机设计的复杂程度和增加设计灵活性。MIPI联盟下面有不同的工作组，分别定义了一系列的手机内部接口标准，比如摄像头接口CSI，显示接口DSI，射频接口DigRF，麦克风/扬声器接口SLIMbus等。统一接口标准的好处是手机厂商根据需要可以从市面上灵活选择不同的芯片和模组，更改设计和功能时更加快捷方便。目前，MIPI联盟的董事成员包括英特尔、摩托罗拉、诺基亚、三星、意法半导体、德州仪器。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/hsi/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The "High speed synchronous Serial Interface" is synchronous serial interface used mainly to connect application engines and cellular modems.  
  
</details>
  

<details>
<summary>-Y- PPS support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;秒脉冲(Pulse Per Second)驱动用来控制电流脉冲速率，可用于计时。PPS的精度可以到纳秒级，而且没有累积误差。这通常是GPS天线的一项功能，用于获取GPS卫星的授时。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/pps/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PPS (Pulse Per Second) is a special pulse provided by some GPS antennae. Userland can use it to get a high-precision time reference.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Some antennae's PPS signals are connected with the CD (Carrier Detect) pin of the serial line they use to communicate with the host. In this case use the SERIAL_LINE client support. Some antennae's PPS signals are connected with some special host inputs so you have to enable the corresponding client support.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called pps_core.ko.  
  
</details>
  

<details>
<summary>PTP clock support  ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;精密时间协议(Precision Time Protocol)是IEEE 1588定义的一种基于以太网的高精度时间同步协议。PTP采用硬件与软件结合设计，可以提供比纯软件方式的NTP(网络时间协议)高的多的精度(微秒级)。与GPS授时相比，在提供和GPS相同的精度情况下，PTP不需要为每个设备安装GPS那样昂贵的组件，只需要一个高精度的本地时钟和提供高精度时钟戳的部件，成本较低。一般的PC和服务器上没有PTP硬件。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/ptp/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The IEEE 1588 standard defines a method to precisely synchronize distributed clocks over Ethernet networks. The standard defines a Precision Time Protocol (PTP), which can be used to achieve synchronization within a few dozen microseconds. In addition, with the help of special hardware time stamping units, it can be possible to achieve synchronization to within a few hundred nanoseconds.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This driver adds support for PTP clocks as character devices. If you want to use a PTP clock, then you should also enable at least one clock driver as well.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called ptp.  
  
</details>
  
<details>
<summary>-Y- Pin controllers ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Pin控制器。其下的各选项请根据实际硬件状况选择(皆为低功耗或嵌入式平台)。一般由其他项选中。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/pinctrl/Kconfig。  
  
</details>
  

<details>
<summary>-Y- GPIO Support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每个芯片都会有至少一个引脚(PIN)，像CPU或者芯片组这种复杂的芯片，其引脚会有成百上千个，这些PIN就是芯片与外部沟通的渠道，每个PIN都会有它特定的功能。GPIO(General Purpose I/O)就是芯片上的一种通用功能的引脚，其功能可由使用者通过编程的方式自定义(所谓"可编程引脚")，比如使用两条PIN就可以组成I2C，使用4条PIN就可以组成SPI。嵌入式系统经常需要控制结构简单但数量众多的外部设备(比如LED的亮与灭)，使用传统的串口或者并口就太"大炮打蚊子"，而GPIO则非常适合用于控制此类数量众多的简单设备。GPIO在嵌入式设备中使用广泛，但PC平台的芯片组南桥大多也集成有GPIO引脚(但只有BIOS才知道如何使用他们)，以支持某些特殊的定制硬件。详情参见"Documentation/gpio/gpio.txt"文档，不确定的选"N"。一般由其他项选中。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/gpio/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This enables GPIO support through the generic GPIO library. You only need to enable this, if you also want to enable one or more of the GPIO drivers below.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N.  
  
</details>
  

<details>
<summary><M> Dallas's 1-wire support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Dallas公司发明的单总线是比I2C更简单的总线，仅使用一个引脚(1-wire)，使用Master-Slave结构，用于连接慢速的单引脚设备，比如iButton和热传感器。主要用于嵌入式系统。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/w1/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Dallas' 1-wire bus is useful to connect slow 1-pin devices such as iButtons and thermal sensors.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you want W1 support, you should say Y here.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This W1 support can also be built as a module.  If so, the module will be called wire.  
  
</details>
  

<details>
<summary>[Y] Adaptive Voltage Scaling class support ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;自适应电压调节(Adaptive Voltage Scaling)技术能够动态的对设备工作电压进行精细的调整，拥有比DVFS更佳的电力利用效率，是一种降低功耗与优化性能并举的电源与性能管理技术。AVS在OMAP设备上也被称为"SmartReflex"，目前仅用于嵌入式领域。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/power/avs/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AVS is a power management technique which finely controls the operating voltage of a device in order to optimize (i.e. reduce) its power consumption. At a given operating point the voltage is adapted depending on static factors (chip manufacturing process) and dynamic factors (temperature depending performance). AVS is also called SmartReflex on OMAP devices.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to enable Adaptive Voltage Scaling class support.  
  
</details>
  


<details>
<summary>[Y] Board level reset or power off ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;允许通过操作板载的主电源，关闭或重启整个系统。仅用于嵌入式系统。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/power/reset/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Provides a number of drivers which either reset a complete board or shut it down, by manipulating the main power supply on the board.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to enable board reset and power off  
  
</details>
  
<details>
<summary>[Y] Power supply class support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;允许用户空间程序通过sysfs/uevent接口对电源(电池、交流电、USB)进行监控。主要用于笔记本与嵌入式设备。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/power/supply/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to enable power supply class support. This allows power supply (batteries, AC, USB) monitoring by userspace via sysfs and uevent (if available) and/or APM kernel interface (if selected below).  
  
</details>
  
<details>
<summary><Y> Hardware Monitoring support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当前主板大多都有一个监控硬件温度/电压/风扇转速等状况的设备，请按照主板实际使用的芯片选择相应的子项。如果你不知道究竟需要使用哪个驱动，可以使用Superiotool和sensors-detect工具进行检测。另外,某些子项可能还需要CONFIG_I2C的支持。更多详情参见"Documentation/hwmon/userspace-tools"文档。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/hwmon/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hardware monitoring devices let you monitor the hardware health of a system. Most modern motherboards include such a device. It can include temperature sensors, voltage sensors, fan speed sensors and various additional features such as the ability to control the speed of the fans.  If you want this support you should say Y here and also to the specific driver(s) for your sensors chip(s) below.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To find out which specific driver(s) you need, use the sensors-detect script from the lm_sensors package.  Read <file:Documentation/hwmon/userspace-tools> for details.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This support can also be built as a module.  If so, the module will be called hwmon.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<Y> GPIO fan`  
* `<Y> Hwmon driver that uses channels specified via iio maps`  
* `<Y> National Semiconductor LM90 and compatibles`  
* `<Y> National Semiconductor LM95245 and compatibles`  
* `<M> NTC thermistor support from Murata`  
* `<M> PWM fan`  
* `<M> Texas Instruments INA219 and compatibles`  
* `<Y> Texas Instruments TMP102`  
  
</details>
  
<details>
<summary>-Y- Generic Thermal sysfs driver ---> </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为ACPI规范中定义的"thermal"(发热控制)提供一个通用的sysfs接口，以方便与诸如温度传感器和风扇之类的设备通信。由于目前所有PC和服务器都已支持ACPI，并且发热控制也越来越重要，所以建议选"Y"。详情参见"Documentation/thermal/sysfs-api.txt"文档。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/thermal/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Generic Thermal Sysfs driver offers a generic mechanism for thermal management. Usually it's made up of one or more thermal zone and cooling device. Each thermal zone contains its own temperature, trip points, cooling devices.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;All platforms with ACPI thermal support can use this driver.If you want this support, you should say Y or M here.  
  
</details>
  
<details>
<summary>[Y] Watchdog Timer Support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选"Y"并选中下面相应的驱动之后，再创建一个主/次设备号为10/130的字符设备"/dev/watchdog"，即可拥有一只看门狗。其工作原理是：当/dev/watchdog设备被打开后，如果喂狗守护进程超过60秒没有喂狗(写入"/dev/watchdog")，那么底层的看门狗硬件将会触发整个机器硬重启(相当于按下面板上的"RESET"按钮)。这对于提高服务器的在线率来说意义重大。详情参见"Documentation/watchdog/watchdog-api.txt"文档。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/watchdog/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say Y here (and to one of the following options) and create a character special file /dev/watchdog with major number 10 and minor number 130 using mknod ("man mknod"), you will get a watchdog, i.e.: subsequently opening the file and then failing to write to it for longer than 1 minute will result in rebooting the machine. This could be useful for a networked machine that needs to come back on-line as fast as possible after a lock-up. There's both a watchdog implementation entirely in software (which can sometimes fail to reboot the machine) and a driver for hardware watchdog boards, which are more robust and can also keep track of the temperature inside your computer. For details, read <file:Documentation/watchdog/watchdog-api.txt> in the kernel source.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The watchdog is usually used together with the watchdog daemon which is available from <ftp://ibiblio.org/pub/Linux/system/daemons/watchdog/>. This daemon can also monitor NFS connections and can reboot the machine when the process table is full.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N.  
  
</details>
  
<details>
<summary>Sonics Silicon Backplane  ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SSB(Sonics Silicon Backplane)是一种仅在嵌入式环境中使用的总线。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/ssb/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Support for the Sonics Silicon Backplane bus. You only need to enable this option, if you are configuring a kernel for an embedded system with this bus.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;It will be auto-selected if needed in other environments.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The module will be called ssb.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N.  
  
</details>
  
<details>
<summary>{Y} Broadcom specific AMBA ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Broadcom特有的AMBA(Advanced Microcontroller Bus Architecture)总线支持，仅用于嵌入式环境。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/bcma/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Bus driver for Broadcom specific Advanced Microcontroller Bus Architecture.  
  
</details>
  
<details>
<summary>[Y] ChipCommon-attached serial flash support</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/bcma/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Some cheap devices have serial flash connected to the ChipCommon instead of independent SPI controller. It requires using a separated driver that implements ChipCommon specific interface communication.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enabling this symbol will let bcma recognize serial flash and register it as platform device.  
  
</details>
  
<details>
<summary>[Y] BCMA Broadcom GBIT MAC COMMON core driver</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/bcma/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Driver for the Broadcom GBIT MAC COMMON core attached to Broadcom specific Advanced Microcontroller Bus.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N  
  
</details>
  
<details>
<summary>[Y] BCMA GPIO driver</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/bcma/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Driver to provide access to the GPIO pins of the bcma bus.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N  
  
</details>
  
<details>
<summary>[N] BCMA debugging</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/bcma/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This turns on additional debugging messages.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N  
  
</details>
  
<details>
<summary>Multifunction device drivers ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MFD(多功能设备)的含义是"在单个芯片上集成多个功能(GPIO、触摸屏、键盘、电流调节、电源管理...)"。此种芯片通常通过一个或多个IRQ线和低速数据总线(SPI/I2C/GPIO)与主CPU进行通信。对于主系统来说，它们通过数据总线显示为一个单独的MFD设备。但透过MFD框架，又可以拥有多个相互独立的子设备(子功能)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/mfd/Kconfig。  
  
</details>
  
<details>
<summary>-Y- Voltage and Current Regulator Support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通用的电压与电流调节器框架，除了提供通用的电压与电流调节接口外，还能通过sysfs向用户空间提供电压与电流的状态信息。目的在于通过动态调节电压和电流，降低能耗，延长电池寿命。主要用于嵌入式环境。一般由其他项选中。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/regulator/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Generic Voltage and Current Regulator support.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This framework is designed to provide a generic interface to voltage and current regulators within the Linux kernel. It's intended to provide voltage and current control to client or consumer drivers and also provide status information to user space applications through a sysfs interface.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The intention is to allow systems to dynamically control regulator output in order to save power and prolong battery life. This applies to both voltage regulators (where voltage output is controllable) and current sinks (where current output is controllable).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This framework safely compiles out if not selected so that client drivers can still be used in systems with no software controllable regulators.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say no.  
  
</details>
  
<details>
<summary><Y> Remote Controller support ---> </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/media/rc/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enable support for Remote Controllers on Linux. This is needed in order to support several video capture adapters, standalone IR receivers/transmitters, and RF receivers.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enable this option if you have a video capture board even if you don't need IR, as otherwise, you may not be able to compile the driver for your adapter.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y when you have a TV or an IR device.  
  
</details>
  
<details>
<summary><Y> Multimedia support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;多媒体设备：摄像头、视频采集、模拟电视、数字电视、机顶盒、收音机、遥控器、数字视频广播(DVB)...内核多媒体子系统由LinuxTV项目负责维护。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/media/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you want to use Webcams, Video grabber devices and/or TV devices enable this option and other options below. Additional info and docs are available on the web at <https://linuxtv.org>  
  
</details>
  
<details>
<summary>Graphics support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;图形设备/显卡支持，对于不需要使用图形界面的服务器环境来说，必须的最小选项集取决于平台(BIOS/UEFI)和引导程序(GRUB/LILO/GRUB4DOS)的设置(全选"N"则屏幕将无任何显示)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/gpu。  
  
</details>
  
<details>
<summary><Y> Sound card support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;声卡支持。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：sound/Kconfig  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you have a sound card in your computer, i.e. if it can say more than an occasional beep, say Y.  Be sure to have all the information about your sound card and its configuration down (I/O port, interrupt and DMA channel), because you will be asked for it.You want to read the Sound-HOWTO, available from <http://www.tldp.org/docs.html#howto>.   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;General information about the modular sound system is contained in the files <file:Documentation/sound/oss/Introduction>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The file <file:Documentation/sound/oss/README.OSS> contains some slightly outdated but still useful information as well.  Newer sound driver documentation is found in <file:Documentation/sound/alsa/*>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you have a PnP sound card and you want to configure it at boot time using the ISA PnP tools (read <http://www.roestock.demon.co.uk/isapnptools/>), then you need to compile the sound card support as a module and load that module after the PnP configuration is finished.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To do this, choose M here and read <file:Documentation/sound/oss/README.modules>; the module will be called soundcore.  
  
</details>
  
<details>
<summary>HID support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HID(人机接口设备)是一种定义计算机如何与人类交互的规范，常与USB或蓝牙搭配使用，常见的设备有：键盘、鼠标、触摸板、游戏杆、遥控器、蓝牙耳机、游戏手柄、手写板等等。不过HID设备不一定要有人机接口，只要符合HID规范，就是HID设备。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/hid/Kconfig  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A human interface device (HID) is a type of computer device that interacts directly with and takes input from humans. The term "HID" most commonly used to refer to the USB-HID specification, but other devices (such as, but not strictly limited to, Bluetooth) are designed using HID specification (this involves certain keyboards, mice, tablets, etc). This option adds the HID bus to the kernel, together with generic HID layer code. The HID devices are added and removed from the HID bus by the transport-layer drivers, such as usbhid (USB_HID) and hidp (BT_HIDP).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For docs and specs, see http://www.usb.org/developers/hidpage/  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say Y.  
  
</details>
  
<details>
<summary>[Y] USB support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通用串行总线(Universal Serial Bus)的目标是统一电脑的外设接口，目前几乎找不到没有USB接口的电脑，而且各种智能设备也大多带有USB接口。不要犹豫，选"Y"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/usb/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option adds core support for Universal Serial Bus (USB). You will also need drivers from the following menu to make use of it.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<M> Support for Host-side USB`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主机端(Host-side)USB支持。通用串行总线(USB)是一个串行总线子系统规范，它比传统的串口速度更快并且特性更丰富(供电、热插拔、最多可接127个设备等)，其目标是统一PC外设接口。USB总体上呈现一种树型结构，USB的"Host"(主设备)被称为"根"(也可以理解为是主板上的USB控制器)，USB的"Slave"(从设备)被称为"叶子"，而内部的节点则称为"hub"(集线器)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Universal Serial Bus (USB) is a specification for a serial bus subsystem which offers higher speeds and more features than the traditional PC serial port.  The bus supplies power to peripherals and allows for hot swapping.  Up to 127 USB peripherals can be connected to a single USB host in a tree structure.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The USB host is the root of the tree, the peripherals are the leaves and the inner nodes are special USB devices called hubs. Most PCs now have USB host ports, used to connect peripherals such as scanners, keyboards, mice, modems, cameras, disks, flash memory, network links, and printers to the PC.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if your computer has a host-side USB port and you want to use USB devices.  You then need to say Y to at least one of the Host Controller Driver (HCD) options below.  Choose a USB 1.1 controller, such as "UHCI HCD support" or "OHCI HCD support", and "EHCI HCD (USB 2.0) support" except for older systems that do not have USB 2.0 support.  It doesn't normally hurt to select them all if you are not certain.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If your system has a device-side USB port, used in the peripheral side of the USB protocol, see the "USB Gadget" framework instead.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;After choosing your HCD, then select drivers for the USB peripherals you'll be using.  You may want to check out the information provided in <file:Documentation/usb/> and especially the links given in <file:Documentation/usb/usb-help.txt>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile this driver as a module, choose M here: the module will be called usbcore.  
* `[Y] PCI based USB host interface`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A lot of embeded system SOC (e.g. freescale T2080) have both PCI and USB modules. But USB module is controlled by registers directly, it have no relationship with PCI module.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;When say N here it will not build PCI related code in USB driver.  
* `[Y] USB announce new devices`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在syslog中记录每个新接入系统的USB设备的详细标识信息，主要用于系统调试.不确定的选"N"。  
* `[Y] Enable USB persist by default`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据USB规范，当USB总线被挂起(休眠)后，它必须继续提供挂起电流(1-5毫安)，以确保USB设备能保持其内部状态，并且USB集线器(HUB)能够检测连接变化(设备插入和拔出)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这在技术上被称为"电力会话"(power session)。如果一个USB设备的电力会话被中断，那么系统必须按照该设备已经被拔出进行处理，这是一种保守的做法。因为没有挂起电流，计算机不可能知道外围设备究竟发生了什么变化：也许依然保持连接，也许已经被拔出并在同一端口上插入了一个新设备。系统必须做最坏的打算，默认情况下，Linux的行为符合USB规范的要求。当整个电脑进入休眠状态(例如挂起到硬盘)时，包括USB总线在内所有总线都将掉电，然后当系统被唤醒，所有USB设备都会被当做在休眠前就已经被拔出来处理。这样做始终是安全的，并且也是"官方正确"的做法。对于大多数USB设备来说，这样做没有任何问题，但是对于USB存储设备(例如移动硬盘/U盘)来说，如果在休眠前有尚未卸载的文件系统(特别是根文件系统)，当系统被唤醒之后，由于无法访问该文件系统，系统可能会立即崩溃！其实不只有掉电，只要"power session"被中断(例如BIOS在唤醒过程中重置了USB控制器)，都会导致这种故障。此选项(USB-persist)就是为了解决这个问题而设置的，虽然解决的不甚完美(参见"Documentation/usb/persist.txt")，但是依然推荐选"Y"，除非你确实有选"N"的理由。当然，最保险的做法是在休眠之前先卸载所有USB设备上的文件系统，而如果根文件系统位于USB设备上，就根本不使用任何休眠功能(不论是挂起到硬盘还是挂起到内存)。  
* `<M> xHCI HCD (USB 3.0) support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xHCI(eXtensible Host Controller Interface)就是当下大红大紫的USB3.0主机控制器规范。  
* `<M> EHCI HCD (USB 2.0) support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;EHCI(Enhanced Host Controller Interface)就是渐成昨日黄花的USB2.0(HighSpeed USB)主机控制器规范。  
* `<M> OHCI HCD (USB 1.1) support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OHCI(Open Host Controller Interface)是主要用于嵌入式环境的USB1.1(主机控制器规范，但也存在于某些老旧的SiS芯片组的PC上。  
* `<M> USB Modem (CDC ACM) support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;USB接口的猫或ISDN适配器，基本没人用的东西。  
* `<M> USB Mass Storage support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;USB存储设备(U盘、USB硬盘、USB软盘、USB光盘、USB磁带、记忆棒、数码相机、读卡器[包括某些笔记本内置的SD卡读卡器]等等)。该选项依赖于CONFIG_SCSI和CONFIG_BLK_DEV_SD选项。选"Y"，除非你确实知道自己在干什么。  
  
</details>
  
<details>
<summary><N> Ultra Wideband devices ----</summary>  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UWB(Ultra Wideband)是一种高带宽，低能耗，点对点,抗干扰性能强的无载波通信技术。UWB在较宽的频谱(3.1-10.6GHz)上，使用极低的功率(约为蓝牙的1/20)，以时间间隔极短(小于1ns)的脉冲信号进行通信。UWB主要应用于室内通信(2米范围内实现480Mbps速率，10米范围内实现110Mbps速率)，例如作为WUSB(Wireless USB)协议的传输层。如果你有UWB无线控制器，可以选"Y"，不确定的选"N"。详见"Documentation/usb/WUSB-Design-overview.txt"文档。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/uwb/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UWB is a high-bandwidth, low-power, point-to-point radio technology using a wide spectrum (3.1-10.6GHz). It is optimized for in-room use (480Mbps at 2 meters, 110Mbps at 10m). It serves as the transport layer for other protocols, such as Wireless USB (WUSB).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The topology is peer to peer; however, higher level protocols (such as WUSB) might impose a master/slave relationship.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if your computer has UWB radio controllers (USB or PCI) based. You will need to enable the radio controllers below.  It is ok to select all of them, no harm done.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For more help check the UWB and WUSB related files in <file:Documentation/usb/>.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To compile the UWB stack as a module, choose M here.  
  
</details>
  
<details>
<summary><Y> MMC/SD/SDIO card support ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MMC(MultiMediaCard)/SD(Secure Digital)/SDIO(Secure Digital I/O)主机控制器。[提示]虽然许多笔记本上有SD卡插槽,但其实它们大多使用的是CONFIG_USB_STORAGE驱动,而不是这里的驱动。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/mmc/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This selects MultiMediaCard, Secure Digital and Secure Digital I/O support.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you want MMC/SD/SDIO support, you should say Y here and also to your specific host controller driver.  
  
</details>
  
<details>
<summary><N> Sony MemoryStick card support ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sony记忆棒是一种Sony专用的存储设备。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/memstick/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sony MemoryStick is a proprietary storage/extension card protocol.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you want MemoryStick support, you should say Y here and also to the specific driver for your MemoryStick interface.  
  
</details>
  
<details>
<summary>[Y] LED Support ---></summary>  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;发光二级管(LED)支持.[提示]标准键盘上的LED灯不在此列(由input子系统控制)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/leds/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y to enable Linux LED support.  This allows control of supported LEDs from both userspace and optionally, by kernel events (triggers).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<Y> LED Class Support`  
* `<Y> LED Support for GPIO connected LEDs`  
* `<Y> PWM driven LED Support`  
* `<Y> LED driver for TLC59108 and TLC59116 controllers`  
  
</details>
  
<details>
<summary>[N] Accessibility support ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;无障碍(Accessibility)支持。各种帮助残疾人使用计算机的软硬件技术，例如：盲文设备、语音合成、键盘映射等等。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/accessibility/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Accessibility handles all special kinds of hardware devices or software adapters which help people with disabilities (e.g. blindness) to use computers.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;That includes braille devices, speech synthesis, keyboard remapping, etc.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to get to see options for accessibility.This option alone does not add any kernel code.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say N, all options in this submenu will be skipped and disabled.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N.  
  
</details>
  
<details>
<summary><N> InfiniBand support ----</summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;InfiniBand是一种低延迟/高带宽数据中心互联架构，采用远程直接内存存取(RDMA)实现高性能处理器间通信(IPC)，同时对虚拟化技术也提供了良好的支持。主要用于服务器集群与高性能计算(HPC)领域。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/infiniband/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Core support for InfiniBand (IB).  Make sure to also select any protocols you wish to use as well as drivers for your InfiniBand hardware.  
  
</details>
  
<details>
<summary><Y> EDAC (Error Detection And Correction) reporting ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在电磁环境比较恶劣的情况下，一些大规模集成电路常常会受到干扰，特别是像RAM这种利用双稳态进行存储的器件，往往会在强干扰下发生翻转，使原来存储的"0"变为"1"，或者"1"变为"0"，造成严重的后果(例如控制程序跑飞，关键数据出错)。随着芯片集成度的增加，发生错误的可能性也在增大，这已经成为一个不能忽视的问题。错误检测与纠正(EDAC)技术的目标就是发现并报告甚至纠正在计算机系统中发生的错误，这些错误是由CPU或芯片组报告的底层错误(内存错误/缓存错误/PCI错误/温度过高等等)，建议选"Y"。如果这些代码报告了一个错误，请到http://bluesmoke.sourceforge.net/和http://buttersideup.com/edacwiki查看更多信息。详见"Documentation/edac.txt"文档。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/edac/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;EDAC is a subsystem along with hardware-specific drivers designed to report hardware errors. These are low-level errors that are reported in the CPU or supporting chipset or other subsystems: memory errors, cache errors, PCI errors, thermal throttling, etc..  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, select 'Y'.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The mailing list for the EDAC project is linux-edac@vger.kernel.org.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `[Y] EDAC legacy sysfs`  
* `<M> Texas Instruments DDR3 ECC Controller`  
  
</details>
  
<details>
<summary>[Y] Real Time Clock ---></summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通用RTC(实时时钟)类支持。所有的PC机主板都包含一个电池动力的实时时钟芯片，以便在断电后仍然能够继续保持时间，RTC通常与CMOS集成在一起，因此BIOS可以从中读取当前时间(精度一般是秒级)。选中此项后你就可以在操作系统中使用一个或多个RTC设备(你还必须从下面启用一个或多个RTC接口)。[注意]Clock与Timer没有任何关系，Timer是定时器(用于计量时长)，Clock是时钟(用于记录当前的时刻"年-月-日 时：分：秒")。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/rtc/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Generic RTC class support. If you say yes here, you will be allowed to plug one or more RTCs to your system. You will probably want to enable one or more of the interfaces below.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为主要勾选项：  
* `[Y] Set system time from RTC on startup and resume`  
* `[Y] Set the RTC time based on NTP synchronization`  
* `[Y] RTC debug support`  
* `[Y] RTC non volatile storage support`  
* `[Y] /sys/class/rtc/rtcN (sysfs)`  
* `[Y] /proc/driver/rtc (procfs for rtcN)`  
* `[Y] /dev/rtcN (character devices)`  
* `<M> Dallas/Maxim DS1307/37/38/39/40/41, ST M41T00, EPSON RX-8025, ISL12057`  
* `<M> TI Palmas RTC driver`  
* `<M> TI TPS6586X RTC driver`  
* `<M> TI TPS65910 RTC driver`  
* `<M> EFI RTC`  
* `<M> TI OMAP Real Time Clock`  
  
</details>
  
<details>
<summary>[Y] DMA Engine support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DMA引擎(DMA Engine)可以看做是传统DMA控制器(DMA controller)的新生。在DMA引擎的协助下，CPU只需初始化一个传输动作，其余的动作就可以由DMA引擎独立完成(完成后以中断的方式通知CPU)，这对于高速传输大量数据以及"分散-收集"操作大有益处，可以节约大量的CPU资源(有时也可节约大量的内存操作)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;目前，DMA引擎有两个用途：(1)卸载高速网络栈中的内存COPY操作，(2)加速CONFIG_MD_RAID456驱动中的RAID操作。"DMA引擎"只是一个统称，在不同场合对应着不同的技术，例如Intel I/OAT(PC平台)和AHB(嵌入式)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/dma/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DMA engines can do asynchronous data transfers without involving the host CPU.  Currently, this framework can be used to offload memory copies in the network stack and RAID operations in the MD driver.  This menu only presents DMA Device drivers supported by the configured arch, it may be empty in some cases.  
  
</details>
  
<details>
<summary>DMABUF options --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以下为勾选项：  
* `-Y- Explicit Synchronization Framework`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/dma-buf/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The Sync File Framework adds explicit syncronization via userspace. It enables send/receive 'struct dma_fence' objects to/from userspace via Sync File fds for synchronization between drivers via userspace components. It has been ported from Android.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The first and main user for this is graphics in which a fence is associated with a buffer. When a job is submitted to the GPU a fence is attached to the buffer and is transferred via userspace, using Sync Files fds, to the DRM driver for example. More details at Documentation/sync_file.txt.  
  
</details>
  
<details>
<summary>[N] Auxiliary Display support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;辅助显示设备。例如基于KS0108控制器的Crystalfontz CFAG12864B单色液晶屏(分辨率:128x64)。仅用于嵌入式系统。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/auxdisplay/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to get to see options for auxiliary display drivers. This option alone does not add any kernel code.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say N, all options in this submenu will be skipped and disabled.  
  
</details>
  
<details>
<summary><M> Userspace I/O drivers--->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UIO(Userspace I/O)是运行在用户空间的I/O技术，它为开发用户空间的驱动提供了一个简单的架构(/dev/uioN)。使用uio的设备一般都属于嵌入式系统，不确定的选"N"。[提示]lsuio工具可以列出所有UIO的模块和其映射的内存地址。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/uio/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enable this to allow the userspace driver core code to be built.  This code allows userspace programs easy access to kernel interrupts and memory locations, allowing some drivers to be written in userspace.  Note that a small kernel driver is also required for interrupt handling to work properly.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you don't know what to do here, say N.  
  
</details>
  
<details>
<summary><N> VFIO Non-Privileged userspace driver framework ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VFIO是一套无特权用户空间I/O驱动框架，需要有IOMMU虚拟化硬件支持(AMD-Vi/Intel VT-d)。此选项仅用于宿主机内核，VFIO的目标是在IOMMU硬件的帮助下，取代CONFIG_UIO和CONFIG_KVM_DEVICE_ASSIGNMENT。VFIO主要用于编写高效的用户态驱动，以及在虚拟化环境的属主机中高效的实现设备直通(passthrough)且无须root特权，可用于详见"Documentation/vfio.txt"文档。[提示]QEMU 1.3以上版本才能利用VFIO特性。不玩KVM虚拟化的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/vfio/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VFIO provides a framework for secure userspace device drivers. See Documentation/vfio.txt for more details.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you don't know what to do here, say N.  
  
</details>
  
<details>
<summary>[N] Virtualization drivers ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个选项仅对PowerPC架构有意义。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/virt/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to get to see options for device drivers that support virtualization environments.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say N, all options in this submenu will be skipped and disabled.  
  
</details>
  
<details>
<summary>Virtio drivers --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;仅可用于客户机内核的Virtio驱动。Virtio的目标是为各种半虚拟化的虚拟机管理程序(特别是KVM)提供一组通用的模拟设备,目前已实现：network/block/balloon/console/hw_random，未来还会实现更多。下列驱动仅可用在基于lguest或QEMU的半虚拟化客户机中(一般是KVM或XEN)。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/virtio/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option is selected by any driver which implements the virtio bus, such as  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CONFIG_VIRTIO_PCI,   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CONFIG_VIRTIO_MMIO,   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CONFIG_RPMSG,  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CONFIG_S390_GUEST.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<Y> PCI driver for virtio devices`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;半虚拟化PCI设备驱动，VMM(虚拟机管理程序)必须要有相应的"PCI virtio backend"。基于QEMU的VMM(KVM,Xen)一般都支持该驱动。[提示]由于目前的ABI尚不稳定,建议使用时注意版本匹配。  
* `<Y> Platform bus driver for memory mapped virtio devices`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;支持使用内存映射机制的virtio设备驱动。  
  
</details>
  
<details>
<summary>Microsoft Hyper-V guest support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;仅在将此Linux内核作为微软Hyper-V虚拟机的来宾操作系统运行时，才需要开启这里的选项。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/hv/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Select this option to run Linux as a Hyper-V client operating system.  
  
</details>
  
<details>
<summary>[N] Staging drivers ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;尚在开发中或尚未完成的，目前尚不完善的驱动，切勿用于生产环境。仅供测试人员或者开发者试用。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/staging/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option allows you to select a number of drivers that are not of the "normal" Linux kernel quality level.  These drivers are placed here in order to get a wider audience to make use of them.  Please note that these drivers are under heavy development, may or may not work, and may contain userspace interfaces that most likely will be changed in the near future.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Using any of these drivers will taint your kernel which might affect support options from both the community, and various commercial support organizations.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you wish to work on these drivers, to help improve them, or to report problems you have with them, please see the driver_name.README file in the drivers/staging/ directory to see what needs to be worked on, and who to contact.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If in doubt, say N here.  
  
</details>
  
<details>
<summary>[N] Platform support for Goldfish virtual devices ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/platform/goldfish/Kconfig  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to get to see options for the Goldfish virtual platform.This option alone does not add any kernel code. Unless you are building for the Android Goldfish emulator say N here.  
  
</details>
  
<details>
<summary>[N] Platform support for Chrome hardware ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;专用于Google公司的Chromebook笔记本/Chromebox迷你机的设备驱动。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/platform/chrome/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to get to see options for platform support for various Chromebooks and Chromeboxes. This option alone does not add any kernel code.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say N, all options in this submenu will be skipped and disabled.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Common Clock Framework --->  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CCF(Common Clock Framework)是从3.4内核开始引入的新时钟框架，用于取代原有的"Clock Framework"。详见"Documentation/clk.txt"文档。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/clk/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The common clock framework is a single definition of struct clk, useful across many platforms, as well as an implementation of the clock API in include/linux/clk.h. Architectures utilizing the common struct clk should select this option.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<Y> Clock driver for TI Palmas devices`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This driver supports TI Palmas devices 32KHz output KG and KG_AUDIO using common clock framework.  
  
</details>
  
<details>
<summary>[Y] Hardware Spinlock drivers --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;硬件自旋锁驱动。目前仅出现在嵌入式处理器上，自旋锁是保护共享资源的一种锁机制，与互斥锁比较类似，都是为了解决对某项资源的互斥使用。无论是互斥锁，还是自旋锁，在任何时刻，最多只能有一个持有者。也就是说，在任何时刻最多只能有一个执行单元获得锁。但是两者在调度机制上略有不同，对于互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。但是自旋锁不会引起申请者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直在循环中"忙等"(占用CPU但无事可做)，直到该自旋锁被释放。"自旋"一词就是因此而得名。自旋锁的使用非常方便，但仅适用于需要极短时间锁定的场合(例如1毫秒)，以避免消耗太多的CPU空等时间。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/hwspinlock/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<Y> OMAP Hardware Spinlock device`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say y here to support the OMAP Hardware Spinlock device (firstly introduced in OMAP4).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N.  
  
</details>
  
<details>
<summary>Clock Source drivers --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"时钟源"驱动，主要面向嵌入式设备。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/clocksource/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `[Y] Enable ARM architected timer event stream generation by default`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option enables support by default for event stream generation based on the ARM architected timer. It is used for waking up CPUs executing the wfe instruction at a frequency represented as a power-of-2 divisor of the clock rate. The behaviour can also be overridden on the command line using the clocksource.arm_arch_timer.evtstream parameter. The main use of the event stream is wfe-based timeouts of userspace locking implementations. It might also be useful for imposing timeout on wfe to safeguard against any programming errors in case an expected event is not generated.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This must be disabled for hardware validation purposes to detect any hardware anomalies of missing events.  
  
</details>
  
<details>
<summary>-Y- Mailbox Hardware Support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mailbox硬件支持。这里的"Mailbox"是一个框架，通过消息队列和中断驱动信号，控制芯片上的多个处理器之间的通信。仅用于嵌入式环境。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/mailbox/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mailbox is a framework to control hardware communication between on-chip processors through queued messages and interrupt driven signals. Say Y if your platform supports hardware mailboxes.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `*{Y} OMAP2+ Mailbox framework support`  
* `(256) Mailbox kfifo default buffer size (bytes)`  
  
</details>
  
<details>
<summary>[Y] IOMMU Hardware Support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IOMMU硬件主要出现在带有I/O虚拟化技术的硬件上，例如带有AMD-Vi或VT-d技术的芯片。IOMMU主要作用：(1)内存地址转换(例如DMA地址转换，scatter-gather)，(2)中断重映射，(3)对设备读取和写入的进行权限检查。这对于提高虚拟化性能和安全性，以及在64位系统上更好的使用32位设备，意义重大。[提示]此选项仅对宿主机有意义，如果此内核要作为来宾操作系统运行，请选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/iommu/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if you want to compile device drivers for IO Memory Management Units into the kernel. These devices usually allow to remap DMA requests and/or remap interrupts from other devices on the system.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `[Y] OMAP IOMMU Support`  
* `[Y] Export OMAP IOMMU internals in DebugFS`  
  
</details>
  
<details>
<summary>Remoteproc drivers --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现代的SoC芯片一般都会以AMP(非对称多处理器)方式集成多个不同的处理器(例如OMAP5432就在单个芯片上集成了2个Cortex-A15处理器，2个Cortex-M4处理器，1个C64x DSP)，这样就可在不同的处理器上分别运行多个不同的操作系统实例(例如，在2个Cortex-A9处理器上以SMP(对称多处理器)方式运行Linux，在2个Cortex-M3和1个C64x上分别运行不同的实时操作系统)。而Remoteproc驱动对此种场合下的处理器间通信非常有用，详见"Documentation/remoteproc.txt"和"Documentation/rpmsg.txt"文档。目前仅对嵌入式系统有意义，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/remoteproc/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<M> Support for Remote Processor subsystem`  
* `<M> OMAP remoteproc support`  
* `<M> TI PRUSS remoteproc support`  
  
</details>
  
<details>
<summary>Rpmsg drivers --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此项和上面的Remoteproc紧密相关。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/rpmsg/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<M> Virtio RPMSG bus driver`  
* `<M> rpmsg Remote Procedure Call driver`  
* `<M> PRU RPMsg Communication driver`  
  
</details>
  
<details>
<summary>SOC (System On Chip) specific Drivers --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;专用于单片机的设备驱动，皆为嵌入式系统。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `[Y] TI SOC drivers support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/soc/ti/Kconfig  
  
</details>
  
<details>
<summary>[N] Generic Dynamic Voltage and Frequency Scaling (DVFS) support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DVFS(动态电压与频率调整)可以根据系统负载动态调节设备的运行频率和电压(对于同一芯片，频率越高，需要的电压也越高)，从而达到节能目的。此选项提供了一个类似CPUfreq(CONFIG_CPU_FREQ)的通用DVFS框架(devfreq)。目前DVFS技术进在嵌入式设备(例如Exynos4/Exynos5)上普遍存在，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/devfreq/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A device may have a list of frequencies and voltages available. devfreq, a generic DVFS framework can be registered for a device in order to let the governor provided to devfreq choose an operating frequency based on the device driver's policy.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Each device may have its own governor and policy. Devfreq can reevaluate the device state periodically and/or based on the notification to "nb", a notifier block, of devfreq.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Like some CPUs with CPUfreq, a device may have multiple clocks. However, because the clock frequencies of a single device are determined by the single device's state, an instance of devfreq is attached to a single device and returns a "representative" clock frequency of the device, which is also attached to a device by 1-to-1. The device registering devfreq takes the responsibility to "interpret" the representative frequency and to set its every clock accordingly with the "target" callback given to devfreq.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;When OPP is used with the devfreq device, it is recommended to register devfreq's nb to the OPP's notifier head.  If OPP is used with the devfreq device, you may use OPP helper functions defined in devfreq.h.  
  
</details>
  
<details>
<summary>-Y- External Connector Class (extcon) support  --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;extcon(外部连接器类)允许用户空间通过sysfs和uevent监控外部连接器，同时也支持多状态外部连接器(也就是拥有多个连接线缆的外部连接器)。例如，一端连接到主机USB端口的多状态外部连接器，一端可以同时连接一条HDMI线缆和一个AC适配器。30针的PDMI连接器也是多状态外部连接器的常见例子。不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/extcon/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to enable external connector class (extcon) support. This allows monitoring external connectors by userspace via sysfs and uevent and supports external connectors with multiple states; i.e., an extcon that may have multiple cables attached. For example, an external connector of a device may be used to connect an HDMI cable and a AC adaptor, and to host USB ports. Many of 30-pin connectors including PDMI are also good examples.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<M> Palmas USB EXTCON support`  
* `<M> USB GPIO extcon support`  
  
</details>
  
<details>
<summary>-Y- Memory Controller drivers --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内存控制器驱动。这里所说的"内存控制器"仅指嵌入式SoC系统中的各种控制器，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/memory/Kconfig。  
  
</details>
  
<details>
<summary><Y> Industrial I/O support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IIO子系统为各种不同总线(i2c、spi等)的嵌入式传感器驱动提供了一个统一的框架。例如：(1)模数转换器，(2)加速度传感器，(3)陀螺仪，(4)惯性测量仪，(5)电容-数字转换器，(6)压力/温度/光线传感器等等。主要用于工业领域和嵌入式领域，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：Industrial I/O support rivers。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The industrial I/O subsystem provides a unified framework for drivers for many different types of embedded sensors using a number of different physical interfaces (i2c, spi, etc).  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为主要勾选项：  
* `-Y- Enable buffer support within IIO`  
* `-Y- Industrial I/O buffering based on kfifo`  
* `-Y- Enable IIO configuration via configfs`  
* `-Y- Enable triggered sampling support`  
* `<Y> Enable software triggers support`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其他勾选项以各类芯片为主。  
  
</details>
  
<details>
<summary><N> Non-Transparent Bridge support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PCI-E非透明桥是一个点对点PCI-E总线，用于连接两条对等的PCI-E总线。例如英特尔 Atom S1200处理器， Xeon C5500/C3500 嵌入式处理器等。具体支持的设备号(PCI_DEVICE_ID)可以查看"drivers/ntb/ntb_hw.h"文件或NTB驱动数据库，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/ntb/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The PCI-E Non-transparent bridge hardware is a point-to-point PCI-E busconnecting 2 systems. When configured, writes to the device's PCImapped memory will be mirrored to a buffer on the remote system.  Thentb Linux driver uses this point-to-point communication as a method totransfer data from one system to the other.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say N.  
  
</details>
  
<details>
<summary>[N] VME bridge support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VME(VersaModule Eurocard)总线是一种通用的计算机总线，主要用于工业控制/军用系统/航空航天/交通运输/医疗等嵌入式领域。而VME桥则是其他总线(例如PCI/PCI-E)到VME总线之间的转换芯片，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/vme/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you say Y here you get support for the VME bridge Framework.  
  
</details>
  
<details>
<summary>-Y- Pulse-Width Modulation (PWM) Support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PWM(脉宽调制)是将模拟信号转换为脉波的一种技术，在计算机领域，这项技术常被用于控制风扇转速和背光显示器的亮度。很多微型处理器内部都包含有PWM控制器，此选项为所有PWM控制器驱动提供了一个统一的框架，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/pwm/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Generic Pulse-Width Modulation (PWM) support.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;In Pulse-Width Modulation, a variation of the width of pulses in a rectangular pulse signal is used as a means to alter the average power of the signal. Applications include efficient power delivery and voltage regulation. In computer systems, PWMs are commonly used to control fans or the brightness of display backlights.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This framework provides a generic interface to PWM devices within the Linux kernel. On the driver side it provides an API to register and unregister a PWM chip, an abstraction of a PWM controller, that supports one or more PWM devices. Client drivers can request PWM devices and use the generic framework to configure as well as enable and disable them.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This generic framework replaces the legacy PWM framework which allows only a single driver implementing the required API. Not all legacy implementations have been ported to the framework yet. The framework provides an API that is backward compatible with the legacy framework so that existing client drivers continue to work as expected.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say no.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<M> Freescale FlexTimer Module (FTM) PWM support`  
* `<Y> ECAP PWM support`  
* `<Y> EHRPWM PWM support`  
  
</details>
  
<details>
<summary>IRQ chip support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：/drivers/irqchip/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;没有选择项。  
  
</details>
  
<details>
<summary><N> IndustryPack bus support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IndustryPack是工业控制领域常用的一种总线，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/ipack/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option provides support for the IndustryPack framework.  There are IndustryPack carrier boards, which interface another bus (such as PCI) to an IndustryPack bus, and IndustryPack modules, that are hosted on these buses.  While IndustryPack modules can provide a large variety of functionality, they are most often found in industrial control applications.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say N if unsure.  
  
</details>
  
<details>
<summary>[Y] Reset Controller Support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为GPIO总线或者芯片内置的重启控制器提供通用支持，仅用于嵌入式设备。不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/reset/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Generic Reset Controller support.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This framework is designed to abstract reset handling of devices via GPIOs or SoC-internal reset controller modules.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say no.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `<Y> TI SYSCON Reset Driver`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This enables the reset driver support for TI devices with memory-mapped reset registers as part of a syscon device node. If you wish to use the reset framework for such memory-mapped devices, say Y here. Otherwise, say N.  
  
</details>
  
<details>
<summary><N> FMC support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FMC(FPGA Mezzanine Carrier)是一个定义如何将FPGA夹层卡(FPGA Mezzanine Card)连接到主机电路板的接口标准，仅用于嵌入式环境。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/fmc/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FMC (FPGA Mezzanine Carrier) is a mechanical and electrical standard for mezzanine cards that plug into a carrier board. This kernel subsystem supports the matching between carrier and mezzanine based on identifiers stored in the internal I2C EEPROM, as well as having carrier-independent drivers.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The framework was born outside of the kernel and at this time the off-tree code base is more complete.  Code and documentation is at git://ohwr.org/fmc-projects/fmc-bus.git .  
  
</details>
  
<details>
<summary>PHY Subsystem --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PHY子系统。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/phy/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `-Y- PHY Core`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为内核中的所有PHY设备提供一个通用的PHY框架，不确定的选"N"。内核中若有其他部分依赖它，会自动选上。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This framework is designed to provide a generic interface for PHY devices present in the kernel. This layer will have the generic API by which phy drivers can create PHY using the phy framework and phy users can obtain reference to the PHY. All the users of this framework should select this config.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其他为部分芯片的选项。  
  
</details>
  
<details>
<summary>[N] Generic powercap sysfs driver ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"power capping"的意思是允许用户把设备的总功耗限定在指定的范围内，此选项允许内核子系统将"power capping"的设置以sysfs的方式导出到用户空间，不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/powercap/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The power capping sysfs interface allows kernel subsystems to expose power capping settings to user space in a consistent way.  Usually, it consists of multiple control types that determine which settings may be exposed and power zones representing parts of the system that can be subject to power capping.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If you want this code to be compiled in, say Y here.  
  
</details>
  
<details>
<summary><N> MCB support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MCB(MEN Chameleon Bus)是专用于德国MEN Mikro Elektronik公司的FPGA设备的总线，MEN Mikro Elektronik公司的嵌入式计算主要是为航空/航海/铁路及陆地车辆的应用，以及自动化/电力/能源和医疗用途。不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/mcb/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;The MCB (MEN Chameleon Bus) is a Bus specific to MEN Mikroelektronik  FPGA based devices. It is used to identify MCB based IP-Cores within  an FPGA and provide the necessary framework for instantiating drivers for these devices.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If build as a module, the module is called mcb.ko  
  
</details>
  
<details>
<summary>Performance monitor support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;专用于ARM架构的CPU性能监控框架。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/perf/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下列为勾选项：  
* `[Y] ARM PMU framework`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say y if you want to use CPU performance monitors on ARM-based systems.  
  
</details>
  
<details>
<summary>-Y- Reliability, Availability and Serviceability (RAS) features ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;RAS(可靠、可用、可维护)是一个计算机硬件术语，可靠性描述系统能够持续正确工作多长时间，可用性描述系统能够正确工作的时间百分比，可维护性描述系统从错误恢复到正常需要多长时间。具有高等级RAS的硬件会有一系列额外的技术保障数据的可靠性与正确性。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/ras/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Reliability, availability and serviceability (RAS) is a computer hardware engineering term. Computers designed with higher levels of RAS have a multitude of features that protect data integrity and help them stay available for long periods of time without failure.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Reliability can be defined as the probability that the system will produce correct outputs up to some given time. Reliability is enhanced by features that help to avoid, detect and repair hardware faults.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Availability is the probability a system is operational at a given time, i.e. the amount of time a device is actually operating as the percentage of total time it should be operating.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Serviceability or maintainability is the simplicity and speed with which a system can be repaired or maintained; if the time to repair a failed system increases, then availability will decrease.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Note that Reliability and Availability are distinct concepts: Reliability is a measure of the ability of a system to function correctly, including avoiding data corruption, whereas Availability measures how often it is available for use, even though it may not be functioning correctly. For example, a server may run forever and so have ideal availability, but may be unreliable, with frequent data corruption.  
  
</details>
  
<details>
<summary>Android --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;安卓平台专用驱动。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/android/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Enable support for various drivers needed on the Android platform。  
  
</details>
  
<details>
<summary><N> NVDIMM (Non-Volatile Memory Device) Support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NVDIMM(非易失性内存)支持。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/nvdimm/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Generic support for non-volatile memory devices including ACPI-6-NFIT defined resources.  On platforms that define an NFIT, or otherwise can discover NVDIMM resources, a libnvdimm bus is registered to advertise PMEM (persistent memory) namespaces (/dev/pmemX) and BLK (sliding mmio window(s)) namespaces (/dev/ndblkX.Y). A PMEM namespace refers to a memory resource that may span multiple DIMMs and support DAX (see CONFIG_DAX).  A BLK namespace refers to an NVDIMM control region which exposes an mmio register set for windowed access mode to non-volatile memory.  
  
</details>
  
<details>
<summary><N> DAX: direct access to differentiated memory ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/dax/Kconfig。  
  
</details>
  
<details>
<summary>-Y- NVMEM Support ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NVMEM(非易失性存储器)设备支持，包括：EEPROM、EFUSES……不确定的选"N"。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/nvmem/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Support for NVMEM(Non Volatile Memory) devices like EEPROM, EFUSES...  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This framework is designed to provide a generic interface to NVMEM from both the Linux Kernel and the userspace.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This driver can also be built as a module. If so, the module will be called nvmem_core.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;If unsure, say no.  
  
</details>
  
<details>
<summary><N> System Trace Module devices  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;仅供调试使用。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/hwtracing/stm/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A System Trace Module (STM) is a device exporting data in System Trace Protocol (STP) format as defined by MIPI STP standards.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Examples of such devices are Intel(R) Trace Hub and Coresight STM.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to enable System Trace Module device support.  
  
</details>
  
<details>
<summary><N> Intel(R) Trace Hub controller  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;仅供调试使用。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/hwtracing/intel_th/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Intel(R) Trace Hub (TH) is a set of hardware blocks (subdevices) that produce, switch and output trace data from multiple hardware and software sources over several types of trace output ports encoded in System Trace Protocol (MIPI STPv2) and is intended to perform full system debugging.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This option enables intel_th bus and common code used by TH subdevices to interact with each other and hardware and for platform glue layers to drive Intel TH devices.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here to enable Intel(R) Trace Hub controller support.  
  
</details>
  
<details>
<summary><N> FPGA Configuration Framework ----  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FPGA配置框架支持，仅用于嵌入式系统。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/fpga/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Say Y here if you want support for configuring FPGAs from the kernel.  The FPGA framework adds a FPGA manager class and FPGA manager drivers.  
  
</details>
  
<details>
<summary>FSI support --->  </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/fsi/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FSI - the FRU Support Interface - is a simple bus for low-level access to POWER-based hardware.  
  
</details>
  
<details>
<summary><Y> Trusted Execution Environment support    </summary>  
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;针对移动端的安全威胁产生的可信执行环境技术。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/tee/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This implements a generic interface towards a Trusted Execution Environment (TEE).  
* `*TEE drivers --->`  
  + `<Y> OP-TEE`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考：drivers/tee/optee/Kconfig。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;This implements the OP-TEE Trusted Execution Environment (TEE) driver.  
