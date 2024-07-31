---
title: Windows系统配置Linux子系统开发ARM
date: 2019-07-11 16:42:17
index_img: /img/index_img/wsl_ubuntu.JPG
tags: 
    - Arm
    - Linux
categories: 
    - OS
---

Win10可以支持Ubuntu子系统，再也不需要虚拟机!

<!-- more -->

## 安装系统

系统要求：最新版win10，打开应用商店安装Ubuntu应用软件，需要系统开启虚拟机功能以及LINUX子系统功能。 
安装完成后如下：  
![](/img/index_img/wsl_ubuntu.JPG)
<!-- more -->
### 配置 Ubuntu

换源等操作。

``` bash
$ cd /etc/apt
$ sudo cp sources.list sources.list.bak
$ sudo vi sources.list
$ add 
  deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
  deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
  deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
$ sudo apt-get update
$ sudo apt-get upgrade 
```

### 安装 gcc 交叉编译工具

#### 命名规则
交叉编译工具链的命名规则为：`arch [-vendor] [-os] [-(gnu)eabi]`
* arch - 体系架构，如ARM，MIPS
* vendor - 工具链提供商
* os - 目标操作系统
* eabi - 嵌入式应用二进制接口（Embedded ApplicationBinary Interface）  

根据对操作系统的支持与否，ARM  GCC可分为支持和不支持操作系统，如  
arm-none-eabi：这个是没有操作系统的，自然不可能支持那些跟操作系统关系密切的函数，比如fork()。使用的是newlib这个专用于嵌入式系统的C库。  
arm-none-linux-eabi：用于Linux的，使用Glibc。  
比如:  
arm-none-eabi-gcc：用于编译 ARM 架构的裸机系统（包括 ARM Linux 的 boot、kernel，不适用编译 Linux 应用 Application）。  
arm-none-linux-gnueabi-gcc：主要用于基于 ARM 架构的 Linux 系统，可用于编译 ARM 架构的 u-boot、Linux内核、Linux应用等。  
arm-eabi-gcc：Android ARM 编译器。  
armcc ARM：公司推出的编译工具，功能和 arm-none-eabi 类似。  
arm-none-uclinuxeabi-gcc：用于uCLinux，使用Glibc。  
arm-none-symbianelf-gcc：用于symbian。  
下载链接: [gcc-toolchain](https://releases.linaro.org/components/toolchain/binaries/7.2-2017.11/arm-linux-gnueabihf/)
``` bash
$ tar xvf gcc-linaro-7.2.1-2017.11-x86_64_arm-linux-gnueabihf.tar.xz -C /usr/local/arm_gcc
$ sudo vim /etc/profile
$ export PATH=/usr/local/arm_gcc/bin/:$PATH
```

### 安装 Minicom

``` bash
$ sudo apr-get install minicom
$ sudo minicom -D /dev/ttySx
```

### 编译代码

需要在CMakelists.txt中set gcc和g++路径，其余与Linux正常。


