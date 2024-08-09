---
layout: post
title: 国产 RISC-V
index_img: /img/riscv/index.jpg
date: 2024-08-07 11:26:27
archive: true
tags:
    - RISCV
    - RTOS
    - IoT
categories: 
    - IoT
---

国产C906/907/908和兼容RTOS.

<!-- more -->
# 硬件介绍

## 玄铁C906

玄铁C906处理器基于RV64GCV指令集实现，且包含了特定的算术增强扩展、Load Store指令以及TLB/Cache操作指令；同时，支持5-8级顺序流水线操作。

RV64GCV 指令集包括标准的整型指令集RV64I、乘除法指令集RV64M、原子指令集RV64A、单精度浮点指令集RV64F、压缩指令集RVC和矢量指令集RVV。

### C906特点

![](/img/riscv/1.png)

### C906 子系统 Overview

![](/img/riscv/2.png)

FPU（Float Point Unit）：

- 支持半精度、单精度以及双精度浮点计算；
- 用户可配置的rounding modes

Vector Unit

- 支持64bit或者128bit硬件数据链路
- 支持INT8-INT64，FP16-FP64以及BFP16
- 最高可达4GFLOPS（single core in 1GHz）

Branch predictor

- 支持分支预测

### 软件生态

- Compiler，assembler，linker，debugger以及binary tools支持GNU工具链
- C906支持官方的RISC-V Linux Kernel以及其应用软件生态
- Qemu已发布，并得到官方支持

### PPA特性

![](/img/riscv/3.png)

## 玄铁C907

玄铁C907处理器基于RV64GCB[V]以及RV32GCB[V]指令集实现，同时兼容RVB32，C907支持9级双问题顺序流水线。

### C907特点

支持硬件cache一致性，每个cluster包含了1-4个core，C907支持AXI4/ACE总线接口

![](/img/riscv/4.png)

### 玄铁C907子模块Overview

![](/img/riscv/5.png)

Vector Unit

- RISC-V Extension Version1.0
- 支持 128/256 bit vGPR
- 支持INT8-INT64，FP32 FP16 BF16
- 支持分段Load/Store

L1 Caches可配16KB/32KB/64KB，4个cores共用一个L2 cache，其可配大小128KB~4MB

Security

- TEE与REE隔离、TA与TEE隔离、TA与TA之间隔离
- 支持Secure boot

### PPA特性

![](/img/riscv/6.png)

### 软件生态

- Linux发布包：ubuntu Debian等
- 多媒体：ffmpeg，gstreamer等
- 容器：docker等
- OS kernel：linux，rtos
- 开发语言：C、C++、Python等

## 玄铁C908

C908处理器基于RV64GCB[V]指令集实现，兼容RVA22，实现了XIE技术（XuanTie Instruction Extension）；C907支持9级双问题顺序流水线。每个Cluster包含了1-4个cores，C908支持AXI4/ACE总线接口。

### C908特点

![](/img/riscv/7.png)

Vector Unit：

- RISC-V V Extension（Version 1.0）
- 支持128/256 bit vGPR
- 支持INT8/INT16/INT32/INT64/BF16/FP16/FP32
- 支持分段Load/Store

Security：

- Secure boot
- TEE与TEE之间隔离，TA与TEE之间隔离、TA与TA之间隔离

软件生态：

- Linux发布包：ubuntu Debian等
- 多媒体：ffmpeg，gstreamer等
- 容器：docker等
- OS kernel：linux，rtos
- 开发语言：C、C++、Python等

### PPA特性

![](/img/riscv/8.png)

# 使用介绍

RISC-V可选择使用OS也可选择不使用OS执行任务的分发与调度。在DCU应用场景中，需要从任务的实时性、调度效率以及功耗问题进行分析。

## OS介绍（主要针对调度方面进行调研）

主流RISC-V使用的OS主要为RTOS与Linux，RTOS与Linux的区别：

### 应用场景

- RTOS（Real Time Operating System）是实时操作系统，其特点是确定性调度，中断处理可抢占，相应速度快。它通常用在对时间敏感的应用中，如嵌入式系统、工业控制、航空航天等领域。RTOS的主要目标是提供快速且一致的系统响应，且具有较强的可预测性。
- Linux是一个通用操作系统，设计目标是为了提供一个稳定、多功能、多用户的环境，但通常不具备严格的实时性。

### 占用资源

- RTOS通常更轻量级，占用资源更少，这使得它们特别适用于资源有限的嵌入式环境。
- Linux作为一个功能更为丰富的系统，其资源消耗相对较大，但也因此提供了更多的功能和更广泛的硬件支持。

### 使用生态

- RTOS由于其专业性和应用范围的限制，其社区支持和文档资源可能没有Linux那么丰富。
- Linux拥有庞大的全球社区，提供丰富的学习资源、文档和支持，使得开发者更易于入门和解决问题。

显然在DCU场景中，需要选择支持硬实时的操作系统，以满足任务下发的高吞吐，低延迟特性。

## RTOS介绍

当前常用的RTOS有FreeRTOS与RT-Thread，两者均开源可免费使用，其中RT-Thread是一款完全由国内团队开发维护的嵌入式RTOS，具有完全的自主知识产权。

## FreeRTOS

FreeRTOS中一共有四种task状态，其流转图如下：

![](/img/riscv/9.png)

### 调度算法

FreeRTOS中默认的time slice为1ms（称之为tick），硬件timer可配置每1ms产生一个tick中断。tick中断处理函数中，调度器选择一个task执行（如果是multi-core，可以选择多个task执行）。优先级抢占调度算法中，高优先级抢占低优先级任务，会立马执行高优先级任务（而不是下一个tick中断），相同优先级进行RR顺序执行。用户可以通过vTaskPrioritySet()配置task的优先级。

![](/img/riscv/10.png)

每个任务分配一个从0~configMAX_PRIORITIES-1的优先级，其中configMAX_PRIORITIES是FreeRTOS的最大优先级配置，优先级值越小表示任务优先级越低。

大小可扩展，可用程序内存占用低至 9KB。FreeRTOS系统提供的低功耗模式，当处理器进入空闲任务周期以后就关闭系统节拍中断(滴答定时器中断)，只有当其他中断发生或者其他任务需要处理的时侯处理器才会从低功耗模式中唤醒。

## RT-Thread

RT-Thread是一个嵌入式实时多线程操作系统，支持多任务执行，且任务通过线程调度实现。

特点：

- 主要由C语言实现，可裁剪性好
- RT-Thread功耗低，实用性强，占用资源小，最小占用3KB ROM，1.2KB RAM

### 线程调度

线程是RT-Thread操作系统中最小的调度单位，线程调度算法是基于优先级的全抢占式多线程调度算法，包括线程调度器自身。支持256个线程优先级，也可通过配置文件更改为最大支持线程优先级。0优先级代表最高优先级，最低优先级留给空闲线程使用。同时它也支持创建多个具有相同优先级的线程，相同优先级的线程间采用时间片的轮转调度算法进行调度，使每个线程运行相应时间。不限制线程数量的多少，线程数目只和硬件平台的具体内存相关。

![](/img/riscv/11.png)

## Linux介绍

Linux作为一个通用操作系统，提供了良好的多任务处理能力和高吞吐量，但是不能满足严格的实时性要求：

- 中断和内核抢占：尽管 Linux 支持可抢占内核（Preemptible Kernel），但在某些情况下，内核代码和关键部分仍然可能禁用抢占，这可能导致实时任务的延迟。
- 中断处理：Linux 内核在处理中断时可能不会立即响应，因为某些中断处理程序可能会在内核中执行较长时间，从而延迟实时任务的执行。
- 内核调度器：Linux 的调度器可能不足以保证在所有情况下都能满足硬实时任务的截止时间，尤其是在系统负载较高时。
- 内存管理：Linux 使用的是分页内存管理机制，这可能导致不可预测的页面错误，从而增加任务的延迟。
- 外部因素：硬件中断、系统负载、多任务竞争等因素都可能影响任务的执行时间，Linux 内核并不能完全控制这些外部因素。

对此，为了让Linux满足硬实时，衍生出RT-Linux和RTAI（Real Time Application Interface）。

## RT-Linux

RT-Linux是Linux内核的实时操作系统扩展，可以在Linux环境中开发实时和嵌入式系统。它基于一个实时微内核，将整个Linux操作系统作为一个完全抢占式的实时进程运行。RT-Linux架构图如下所示。

![](/img/riscv/12.png)

RT-Linux特点：

- 硬实时：RTLinux提供了硬实时功能，这意味着它可以保证关键任务以极低的延迟和抖动满足其时间限制。这对于精确定时至关重要的应用程序是必不可少的。
- 协核/双核架构：RTLinux采用双核架构，由标准Linux内核和RTLinux内核组成。RTLinux内核与Linux内核一起运行，负责实时任务调度和管理。
- 兼容性：RTLinux被设计为与标准Linux发行版兼容。这允许在同一系统上运行实时和非实时应用程序，充分利用Linux广泛的软件，RTLinux支持其本地API (RTAI API)和posix兼容的API，用于实时应用程序开发。

与其他一些实时扩展一样，RT-Linux使用co-kernel架构。它由管理应用程序级用户进程的标准Linux内核与处理实时任务的实时内核微内核一起组成。RT-Linux内核作为可加载模块运行，它由一个实时调度程序(RT-Scheduler)组成，负责调度和管理实时任务。标准linux内核作为一个整体，执行一个低优先级的实时作业时由RT-Scheduler负责调度。

## RTAI

RTAI是针对Linux操作系统的一个实时扩展，通过在Linux内核之上添加一个实时调度层来实现实时功能，这个调度层可以处理实时任务，并保证它们的执行优先于普通的Linux进程和线程，以确保它们能够满足硬实时系统的严格时间约束。

RTAI提供了硬实时调度、实时内存管理、实时通信机制（如实时FIFOs和共享内存）以及中断处理和实时定时器等功能。

RTAI特点：

- 硬实时性能：能够保证在特定的、可预测的时间内响应事件。
- 与Linux内核的集成：允许用户在保持Linux操作系统功能的同时，获得实时性能。
- 用户友好的API：提供了一套API，使得开发实时应用程序变得简单。
- 模块化：开发者可以根据需要加载和卸载实时模块。

>什么是Real-Time：实时应用程序是那些设计用于在保证的时间范围或截止日期内响应请求的应用程序。

## 调度算法介绍

任务调度主要有两种类型：抢占调度与非抢占调度

- 抢占调度：允许高优先级抢占低优先级任务，确保高优先级先执行完成，保证了高优先级任务不会被低优先级任务延迟。然而，抢占过程会存在额外的上下文切换，其过程增加了额外的时间与计算资源，从而影响系统性能。
- 非抢占调度：非抢占调度在允许另一个任务运行之前完成一个任务的执行，避免了额外的抢占过程的上下文切换。然而，如果低优先级任务执行时间超出了预期，可能导致高优先级任务执行推迟。

### No-OS（Bare Machine）介绍

正常的裸机应用代码架构，由一个大while循环以及一些中断服务函数构成。如下图所示，中断服务函数叫前台程序，大while叫后台程序。这种前后台式的工作系统，本质上还是一个单任务的系统。

![](/img/riscv/13.png)

相对而言，单任务的系统实时性较差，后台的各个任务，不管紧急程序有多高，都要排着队轮循。相应的，无额外资源占用的优势使得在MCU资源受限的单一任务应用场景下，使用裸机更为合理。

# 总结

![](/img/riscv/14.png)

# 参考

参考：

risc-v中c906,c907,c908文档
RTOS and RT-linux：<https://medium.com/@ghosalarjun/rtos-and-rt-linux-da20d994ae3e>
FreeRTOS调度算法介绍：<https://www.elecfans.com/d/2804757.html>
RT-Linux使用指南：<http://cs.uccs.edu/~cchow/pub/rtl/doc/html/GettingStarted/>
FreeRTOS使用手册：<https://www.freertos.org/fr-content-src/uploads/2018/07/161204_Mastering_the_FreeRTOS_Real_Time_Kernel-A_Hands-On_Tutorial_Guide.pdf>
FreeRTOS官方指南：<https://www.freertos.org/zh-cn-cmn-s/FreeRTOS-quick-start-guide.html>
RT-Thread学习教程：<https://bbs.huaweicloud.com/blogs/375535>
RT-Thread官方指南：<https://www.rt-thread.org/document/site/#/rt-thread-version/rt-thread-standard/README>
<https://github.com/RT-Thread-packages/FreeRTOS-Wrapper>
