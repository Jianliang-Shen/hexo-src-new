---
title: 常见开发问题汇总
date: 2020-05-04 14:31:21
tags: 
    - C/C++
    - Algorithm
    - Data Structure
    - OS
categories: 
    - Algorithm
---

整理常见必考知识点。

<!-- more -->

## arm过往面经总结

1. 多态继承
2. 链表插入，素数筛选，位运算，翻转链表
[判断质数](https://leetcode-cn.com/problems/count-primes/solution/ru-he-gao-xiao-pan-ding-shai-xuan-su-shu-by-labula/)
3. linux shell使用，考英语

  

6. a++和++a区别
   - `i++`，先执行其他操作，再i自加；  
   - `i--`，先执行其他操作，再i自减；  
   - `++i`，先i自加，再执行其他操作；  
   - `--i`，先i自减，再执行其他操作；  
  
7. typedef语句解释
语句：
typedef int MazeType[25][25];
解释：
typedef   基本数据类型   数组类型名[常量表达式];
这样为这种数组定义了一个别名。
应用：
MazeType M; //等价于 int M[25][25];
typedef和define不同，typedef发生在编译，有检查，define发生在预处理，没有检查
8. C语言如何调用C++第三方库
在头文件加入extern "C"
9. 二分搜索，素数检验（输出第100个素数）
10. ARM的基础知识

12. 中断上半部和下半部区别及作用
13. cache shader
14. 全局变量和局部变量




  
## 网卡收集数据传递给上层应用的过程，网卡型号及接口

### 网络驱动硬件主要组成

1. MAC和PHY
有集成mac的soc+phy解决方案和soc+mac/phy一体芯片解决方案，前者带MAC的SOC与PHY相连的接口包括MII/RMII的网络数据接口和MDIO控制接口，优势在于：
   - 内部 MAC 外设会有专用的加速模块，比如专用的 DMA，加速网速数据的处理。
   - 网速快，可以支持 10/100/1000M 网速。
   - 外接 PHY 可选择性多，成本低。
1. MII/RMII/MDIO接口
MII表示一种介质独立接口，后者是精简的意思。
MDIO表示管理数据输入输出接口，包括MDIO数据线和MDC时钟线。
3. 物理接口RJ45及变压器
  
### 驱动架构

1. Linux 内核使用 net_device 结构体表示一个具体的网络设备，net_device 是整个网络驱动的灵魂。网络驱动的核心就是初始化 net_device 结构体中的各个成员变量，然后将初始化完成以后的 net_device 注册到 Linux 内核中。net_device 结构体定义在 include/linux/netdevice.h 中。
2. 事实上网络设备有多种，大家不要以为就只有以太网一种。Linux 内核内核支持的网络接口有很多，比如光纤分布式数据接口(FDDI)、以太网设备(Ethernet)、红外数据接口(InDA)、高性能并行接口(HPPI)、CAN 网络等。
3. net_device 有个非常重要的成员变量：netdev_ops，为 net_device_ops 结构体指针类型，这就是网络设备的操作集。net_device_ops 结构体定义在 include/linux/netdevice.h 文件中，net_device_ops 结构体里面都是一些以“ndo_” 开头的函数，这些函数就需要网络驱动编写人员去实现，不需要全部都实现，根据实际驱动情况实现其中一部分即可。
4. 网络是分层的，对于应用层而言不用关系具体的底层是如何工作的，只需要按照协议将要发送或接收的数据打包好即可。打包好以后都通过 dev_queue_xmit 函数将数据发送出去，接收数据的话使用 netif_rx 函数即可，我们依次来看一下这两个函数。
5. sk_buff 是 Linux 网络驱动中一个非常重要的结构体，网络数据就是以 sk_buff 保存的，各个协议层在 sk_buff 中添加自己的协议头，最终由底层驱动讲 sk_buff 中的数据发送出去。 网络数据的接收过程恰好相反， 网络底层驱动将接收到的原始数据打包成 sk_buff，然后发送给上层协议，上层会取掉相应的头部，然后将最终的数据发送给用户。
6. Linux 里面的网络数据接收也轮询和中断两种，中断的好处就是响应快，数据量小的时候处理及时，速度快，但是一旦当数据量大，而且都是短帧的时候会导致中断频繁发生，消耗大量的 CPU 处理时间在中断自身处理上。轮询恰好相反，响应没有中断及时，但是在处理大量数据的时候不需要消耗过多的 CPU 处理时间。Linux 在这两个处理方式的基础上提出了另外一种网络数据接收的处理方法：NAPI(New API)，NAPI 是一种高效的网络处理技术。NAPI 的核心思想就是不全部采用中断来读取网络数据，而是采用中断来唤醒数据接收服务程序，在接收服务程序中采用 POLL 的方法来轮询处理数据。这种方法的好处就是可以提高短数据包的接收效率，减少中断处理的时间。目前 NAPI 已经在 Linux 的网络驱动中得到了大量的应用，NXP 官方编写的网络驱动都是采用的 NAPI 机制。
  
## 网络服务器的高速并发处理，降低传输时延

## 内存管理的方式，什么是内存管理，其作用是什么

## 内核如何管理内存的页

参考[内核如何管理内存的页](https://www.jianshu.com/p/3f2f8910c559)

## TCP/IP协议是否了解，Linux怎么做网络路由



## 操作系统的任务调度方式，你的进程是怎么提高实时性或者优先级的

抢占的原理参考[抢占](https://blog.csdn.net/gracioushe/article/details/6436502?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param)




## 内核的裁剪和配置方式

## 数据库是否支持并发，数据库ID是否可以任意数据类型，数据库一定要ID吗

支持并发，但还是需要锁；
可以用其他类型，但int方便自增；
不一定需要，但必须要有主键，ID作为一种范式，是比较好的一种索引的方式。

## 如何管理数据库的脏数据，掉电未写完的数据

[数据库中的并发操作带来的一系列问题](https://blog.csdn.net/Jessica__sun/article/details/69815623?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-3.channel_param)
[mysql解决脏读、不可重复读、虚读的办法](https://blog.csdn.net/jjkang_/article/details/54925661?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.channel_param)

## 视频的编解码方式

## 进程和线程的区别

## 虚拟地址与物理地址


  
## 进程的资源分为哪些？漏了代码段

## 多进程如何共享硬件中断

## 进程和线程的区别

## 内核态中断和线程同步的方式，不能用信号量

## uboot的作用

## arm启动进入操作系统的步骤

## const变量在程序的哪个段

静态存储区

## 驱动中断的响应包括哪些内容

参考[Linux设备驱动中断机制](https://www.cnblogs.com/reality-soul/p/6543999.html)

## 双核的ARM在uboot里面用了吗

## strlen和sizeof区别

## 重写和重载区别

参考[重写(Override)与重载(Overload)的区别（面试题）](https://www.cnblogs.com/luckyjcx/p/12268669.html)

## memcpy和strcpy区别

## gdb调试

## 命令行如何查看进程打开的文件

lsof -p PID

## select和epoll区别

参考[select、poll、epoll之间的区别(搜狗面试)](https://www.cnblogs.com/aspirant/p/9166944.html)

## char int char char double在32位结构体大小

## 手撕代码，倒装句子的单词，单词顺序不变，不使用额外的存储

## CAN和UART区别，CAN是帧结构吗

参考[IIC、SPI、UART、USART、USB、CAN等通讯协议原理及区别](https://blog.csdn.net/heda3/article/details/89053635)

## CAN芯片怎么写的，数据怎么与内核交互的，属于网络设备还是字符设备

## 你觉得最体现你项目能力的是哪个部分，你在其中参与的核心解决的难题是啥，你给我说说三取二的原理，以及你遇到什么困难

## 描述你的项目我来复述并评估拟堆项目的理解，项目背景和项目分工以及你的主要工作和难点


  
## 堆和栈的区别，函数栈、线程栈的区别

## 你在微电子学与固体电子学专业里面，你觉得自己能力算中上还是最好的那一批？

## 最开始的：自我介绍，说你最大的和别人不同的

## 有名管道的父节点和子节点

## 共享内存的使用注意事

回答锁和不能随意释放





## 寄存器修饰的关键字的理解

volatile，参考[C语言中volatile关键字的作用](https://www.cnblogs.com/hjh-666/p/11148119.html)

## arm处理器的模式



## kmalloc和vmalloc的区别，内核怎么分配128M连续内存

kmalloc物理地址连续
vmalloc物理地址不连续



## 原子操作的底层是怎么实现的
