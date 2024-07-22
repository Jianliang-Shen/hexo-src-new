---
title: 使用VScode和Cmake搭建嵌入式开发环境
date: 2019-07-22 17:45:53
tags:
    - Linux
    - VSCode
categories: 
    - 嵌入式
---

安装系统和相关软件，使用远程服务。

<!-- more -->

## 配置主机开发环境

1) 安装Ubuntu 16.04操作系统，双系统、虚拟机、单系统或windows子系统  
2) 安装TI SDK并配置交叉编译环境，并配置环境变量  
3) 安装SecureCRT或者minicom等串口软件，运行secureCRT或者minicom均需要超级权限sudo。
4) 配置有线以太网地址（ipv4）  

```bash
Address:192.168.111.101
Netmask:255.255.255.0
Gateway:192.168.111.1
DNS:4.4.4.4
```
<!-- more -->

## 配置项目工程

Linux可以使用许多IDE，甚至可以脱离集成开发环境通过撰写Makefile管理项目文件。  

### Makefile

假设工程文件中包括main.c、spi.c以及head.h，为了生成可执行文件，需要生成main.c和spi.c的链接文件main.o和spi.o，而main.o和spi.o也需要编译生成。这个过程就是makefile的执行过程。以下为Makefile的内容：  

```C
CC=gcc                  //gcc或者交叉编译gcc
OBJ=main.o spi.o          //OBJ是生成的链接文件目标，作为变量方便
main: $(OBJ)             //main可执行文件的生成需要OBJ中的两个文件
 $(CC) -o $@ $^      // -o表示生成可执行文件 $@表示目标文件（main）；$^表示所有依赖文件（OBJ）
spi.o: spi.c head.h 
 $(CC) -c $<         // $< 表示第一个依赖文件（spi.c）
main.o: main.c head.h
 $(CC) -c $<
clean:                  //make clean删除通过make指令生成的文件
 rm main $(OBJ)
```
  
### VScode + Cmake

当项目内容比较多时，可以通过CMake工具自动生成Makefile。Cmake主要配置CMakeLists.txt实现工程管理。推荐通过VScode生成CMake项目模板，过程如下：安装CMake插件；Ctrl+Shift+P，输入CMAKE QUICK START，选择gcc的kit，一般会自己查询到gcc或者交叉编译工具gcc，输入工程名称，再选择生成可执行文件。这样VScode就会生成项目模板。项目主要分为src、include、bin、build等文件夹，文件夹内容如下：  
  
```
|——zhx_cmake_prj   #ZHX项目主要软件  
    |——bin        #可执行文件夹目录  
    |——build      #build目录  
    |——include    #头文件目录  
    |——src        #项目源文件  
        |——driver  #设备驱动源文件目录  
            |——spi  
            |——uart  
            |...  
        |——main.c    
        |——CMakeLists.txt 
    |——CMakeLists.txt  
```

#### CMakeLists

主目录下的配置：  

```C
cmake_minimum_required (VERSION 3.0.0)
SET(CMAKE_C_COMPILER "/usr/local/arm_gcc/bin/arm-linux-gnueabihf-gcc")      //配置CMAKE GCC
SET(CMAKE_CXX_COMPILER "/usr/local/arm_gcc/bin/arm-linux-gnueabihf-g++")
project(ZHX VERSION 0.1.0)
MESSAGE (STATUS  "This is the binary dir:  " ${PROJECT_BINARY_DIR})
MESSAGE (STATUS  "This is the source dir:  " ${PROJECT_SOURCE_DIR})
INCLUDE_DIRECTORIES (include)   //添加include路径
ADD_SUBDIRECTORY(src)           //添加src子目录
```

二级CMakeLists.txt（src文件夹下）：  

```C
AUX_SOURCE_DIRECTORY(.  SRC_LIST)   //添加当前目录为源码编译目录
AUX_SOURCE_DIRECTORY(./driver/spi/  SRC_LIST)
AUX_SOURCE_DIRECTORY(./driver/i2c/  SRC_LIST)
AUX_SOURCE_DIRECTORY(../include/  SRC_LIST)     //添加include为源码编译目录
ADD_EXECUTABLE(main ${SRC_LIST} )               //通过源码目录中的所有依赖文件生成可执行文件main
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin) //将main输出至bin文件夹下
```

#### 文件编译方式

```
cd ($PRJ)/build  
cmake ..  
make
cd ($PRJ)/bin
```

## 配置开发板

### 制作SD卡

运行$tisdk/bin/creat-sdcard.sh，替换zImage以及设备树文件。  

### 远程登录与传递文件

```bash
ifconfig eth0 192.168.111.100
ssh root@192.168.111.100
scp file root@192.168.111.100:~
```

如果显示ssh无法登陆，原因是当前ip地址与开发板的上一次刷的系统配置的ip地址冲突，使用以下命令删除旧的配置：  

```bash
$: ssh-keygen –f “/home/usr-name/.ssh/known_hosts” –R “192.168.111.100”  
```

### 固定目标板的IP地址

修改/etc/systemd/network/目录下的10-eth.network文件：  

```
[Match]
Name=eth0
KernelCommandLine=!root=/dev/nfs

[Network]
Address=192.168.111.100/24
Gateway=192.168.111.1
```

### 串口收发文件

目标板没有安装ssh时，通过TCP/IP无法传输文件，在文件容量较小的情况下可以使用lrzsz工具。Lrzsz工具解压后使用./configure命令配置makefile，在主机中需要将bin、src以及根目录下的makefile中的CC均改为交叉编译链，再在主目录中make，最后将src/lrz和src/lsz拷贝至目标板的/bin文件夹下。  

1) 主机向从机发送文件：从机运行lrz，主机选择通过zModem发送，（minicom中通过Ctrl+A-Z-S进入）文件将保存在从机当前目录下；
2) 从机向主机发送文件：主机运行lrz（使用主机的gcc编译），从机运行：

```bash
lsz file
```
