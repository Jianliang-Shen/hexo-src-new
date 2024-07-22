---
title: ARM Linux开发命令
date: 2019-07-22 17:45:47
tags:
    - Arm
    - Linux
categories: 
    - IoT
---

交叉编译内核编译命令

<!-- more -->

使用开发板版本：TI AM5728 EVM  
使用SDK版本：ti-processor-sdk-linux-rt-am57xx-evm-05.02.00.10  
官方软件支持：[RT-Linux-software](http://software-dl.ti.com/processor-sdk-linux-rt/esd/AM57X/latest/index_FDS.html)  
官方软件使用说明(更新至6.00.00.07,2019年7月)：[TISDK](http://software-dl.ti.com/processor-sdk-linux/esd/docs/06_00_00_07/linux/index.html)  
TI中文社区：[e2echina.ti](https://e2echina.ti.com/)  
创龙论坛：[51ele](http://www.51ele.net/)  

- [内核编译](#内核编译)
  - [命令](#命令)
  - [内核源码](#内核源码)
  - [内核功能裁剪思路](#内核功能裁剪思路)
- [设备树改写](#设备树改写)
  - [命令](#命令-1)
- [编译模块](#编译模块)
  
## 内核编译

### 命令

``` s
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- tisdk_am57xx-evm-rt_defconfig 
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage -j16
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
```

生成的zImage替换/rootfs/boot下的zImage文件。

### 内核源码

### 内核功能裁剪思路

## 设备树改写

### 命令

``` s
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- am57xx-evm-reva3.dtb -j8
```

源码位置：$tisdk/arch/arm/boot/dts/am57xx-beagle-x15-common.dtsi等。生成的am57xx-evm-reva3.dtb替换/rootfs/boot下的设备树文件。  

## 编译模块

``` s
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- modules -j16
sudo make ARCH=arm INSTALL_MOD_PATH=../rootfs modules_install
```

模块的版本要与内核一致，所以要编译模块。加载模块指令：insmod。卸载模块指令。
