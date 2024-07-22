---
layout: post
title: GPU Note
date: 2024-04-24 10:28:15
tags: 
    - 并行计算
    - GPU
categories: 
    - GPU
---

## 目录

- [目录](#目录)
- [缩略词](#缩略词)
- [GPU基础](#gpu基础)
  - [纹理缓存](#纹理缓存)
  - [GPU中的Frame Buffer](#gpu中的frame-buffer)
    - [1. Frame Buffer的定义和作用](#1-frame-buffer的定义和作用)
    - [2. Frame Buffer的工作原理](#2-frame-buffer的工作原理)
    - [3. Frame Buffer的组成](#3-frame-buffer的组成)
    - [4. 应用场景](#4-应用场景)
    - [5. 性能和优化](#5-性能和优化)
  - [GPU中的DF和UMC](#gpu中的df和umc)
    - [1. DF（Data Fabric）](#1-dfdata-fabric)
    - [2. UMC（Unified Memory Controller）](#2-umcunified-memory-controller)
    - [DF和UMC的关系和作用](#df和umc的关系和作用)
  - [GPU的Doorbell机制](#gpu的doorbell机制)
    - [1. Doorbell机制的定义](#1-doorbell机制的定义)
    - [2. Doorbell机制的工作原理](#2-doorbell机制的工作原理)
    - [3. Doorbell机制的优势](#3-doorbell机制的优势)
    - [4. Doorbell机制的应用场景](#4-doorbell机制的应用场景)
  - [Doorbell的缺点](#doorbell的缺点)
    - [1. 硬件资源占用](#1-硬件资源占用)
    - [2. 同步开销](#2-同步开销)
    - [3. 处理负载](#3-处理负载)
    - [4. 延迟和吞吐量瓶颈](#4-延迟和吞吐量瓶颈)
    - [5. 复杂性和实现成本](#5-复杂性和实现成本)
    - [6. 数据一致性](#6-数据一致性)
    - [7. 错误处理](#7-错误处理)
  - [GC Command Queue](#gc-command-queue)
    - [1. GC Command Queue的定义](#1-gc-command-queue的定义)
    - [2. GC Command Queue的工作原理](#2-gc-command-queue的工作原理)
    - [3. GC Command Queue的优势](#3-gc-command-queue的优势)
    - [4. 应用场景](#4-应用场景-1)
    - [5. 具体示例](#5-具体示例)
  - [CR3寄存器（Control Register 3）](#cr3寄存器control-register-3)
    - [1. CR3寄存器的定义](#1-cr3寄存器的定义)
    - [2. CR3寄存器的作用](#2-cr3寄存器的作用)
    - [3. CR3寄存器在虚拟机中的使用](#3-cr3寄存器在虚拟机中的使用)
    - [具体示例：虚拟机启动和CR3初始化](#具体示例虚拟机启动和cr3初始化)
  - [什么是GPU中的MMHUB？](#什么是gpu中的mmhub)
    - [1. MMHUB的定义和作用](#1-mmhub的定义和作用)
    - [2. MMHUB的主要功能](#2-mmhub的主要功能)
    - [3. MMHUB在GPU架构中的位置](#3-mmhub在gpu架构中的位置)
    - [4. MMHUB的工作流程](#4-mmhub的工作流程)
    - [5. MMHUB的应用场景](#5-mmhub的应用场景)
  - [什么是SHUB？](#什么是shub)
    - [1. SHUB的定义和作用](#1-shub的定义和作用)
    - [2. SHUB的主要功能](#2-shub的主要功能)
    - [3. SHUB在系统架构中的位置](#3-shub在系统架构中的位置)
    - [4. SHUB的应用场景](#4-shub的应用场景)
    - [5. SHUB的优势](#5-shub的优势)
    - [总结](#总结)
  - [VF和PF的区别及互相访问和调用](#vf和pf的区别及互相访问和调用)
    - [1. PF（Physical Function）](#1-pfphysical-function)
      - [主要特点](#主要特点)
    - [2. VF（Virtual Function）](#2-vfvirtual-function)
      - [主要特点](#主要特点-1)
    - [3. PF和VF的区别](#3-pf和vf的区别)
    - [4. PF和VF之间的访问和调用](#4-pf和vf之间的访问和调用)
      - [1. PF管理VF](#1-pf管理vf)
      - [2. VF访问设备资源](#2-vf访问设备资源)
      - [3. 中断和通知机制](#3-中断和通知机制)
    - [5. 示例应用场景](#5-示例应用场景)
  - [什么是VF页表？](#什么是vf页表)
    - [SR-IOV概述](#sr-iov概述)
    - [VF页表的作用](#vf页表的作用)
    - [VF页表的工作原理](#vf页表的工作原理)
    - [VF页表的结构](#vf页表的结构)
    - [示例：VF页表的工作流程](#示例vf页表的工作流程)
    - [总结](#总结-1)
  - [TLB和ATS，ATC](#tlb和atsatc)
  - [DRM](#drm)
  - [管中窥"GPU ISA"](#管中窥gpu-isa)
    - [HIP](#hip)
    - [汇编](#汇编)
  - [VGPR](#vgpr)
  - [AMD GPU LLVM](#amd-gpu-llvm)
  - [HSAKMT](#hsakmt)
    - [HSAKMT的角色](#hsakmt的角色)
  - [什么是Wavefront](#什么是wavefront)
  - [Barrier in GPU](#barrier-in-gpu)
  - [Fence](#fence)
  - [什么是chiplet和die](#什么是chiplet和die)
  - [GPU架构/集群](#gpu架构集群)
  - [RDMA](#rdma)
  - [SDMA](#sdma)
  - [NVLink](#nvlink)
  - [RoCE（RDMA over Converged Ethernet）](#rocerdma-over-converged-ethernet)
- [开发](#开发)
  - [什么是内存地址对齐](#什么是内存地址对齐)
  - [类静态定义](#类静态定义)
- [驱动](#驱动)
  - [AMD GPU是怎么创建queue的](#amd-gpu是怎么创建queue的)

## 缩略词

| 缩略词  | 解释                                                         |
| ------ | ------------------------------------------------------------ |
| TCC    | Texture Cache per Channel                                    |
| UTC    | Unified Translation Cache                                    |
| ATC    | [Address Translation Cache 地址转换缓存](https://docs.amd.com/r/en-US/pg118-system-cache/Address-Translation-Service-and-Cache)                         |
| ATS    | [Address Translation Service](https://cloud.tencent.com/developer/article/1905267)    |
| RC     | Root complex                                                 |
| SE     | Sahder Engine                                                |
| SPI    | Shader Processor Input                                       |
| HWS    | Hardware Scheduling                                          |
| TOPS   | Tera Operations Per Second 每秒钟万亿次操作                     |
| FLOPS  | floating-point operations per second  每秒所执行的浮点运算次数   |
| NPU    | Neural Network Processing Unit                               |
| DPU    | Deep-Learning Processing Unit                                |
| die    | 裸芯片                                                        |
| SVC    | Supervisor Call                                              |
| GMII   | Gigabit Medium Independent Interface  千兆介质独立接口          |
| XGMII  | 10 Gigabit Media Independent Interfac  10Gb介质独立接口        |
| CU     | Compute Unit 计算单元 / Coding Unit 编码单元                    |
| DCU    | Deep-learning Computing Unit  深度计算处理器                    |
| RLC    | Runtime Low-Power Controller  运行时低功耗控制器 / Run List Controller  运行列表控制器               |
| SRM    | Save and Restore Machine                                     |
| GFX    | Graphics                                               |
| HSA    |  Heterogeneous System Architecture                       |
| AQL    | Architected Queuing Language                            |
| TPU    | Tensor Processing Unit  张量处理器                      |
| ROCm   | Radeon Open Computing platform                         |
| CUDA   | Compute Unified Device Architecture  统一计算设备架构  |
| EOP    | End of Pipe                                             |
| EOS    | End of Shader                                           |
| HIP    | Heterogeneous-Compute Interface for Portability         |
| IB     | Indirect Buffer                                          |
| CP     | COMMAND PROCESSOR                                       |
| DRM    | Direct Rendering Manager                                |
| CPC    | Command Processor Compute                               |
| CPG    | Command Processor Graphics                              |
| CPF    | Command Processor Fetcher                               |
| HDP    | Host Data Path  主机数据通路                            |
| PM4    | Promo4Lib                                               |
| ACE    | Asynchronies Compute Engines                            |
| MEC    | Micro-Engine Computing                                  |
| IH     | Interrupt Handler                                        |
| UMD    | User Mode Driver  用户模式驱动                          |
| KMD    | Kernel Mode Driver                                      |
| KFD    | Kernel Fusion Driver  内核融合驱动 / KFD: Kernel Driver For HSA  HSA内核驱动 |
| UMA    | Uniform Memory Access 统一内存架构                             |
| GDS    | Global Data Share                                            |
| LDS    | Local data share                                             |
| VGPR   | Vector General Purpose Register                             |
| SRB    | Special Register Block                                       |
| VM     | Virtual Memory / Virtual Machine                             |
| RB     | Ring Buffer                                                  |
| DS     | Data Sharing                                                 |
| UTCL1  | Unified Translation Cache L1                                 |
| UMC    | Unified Memory Controller                                    |
| GRBM   | Graphics Register Bus Manager 图形寄存器总线管理器              |
| DMA    | Direct Memory Access                                         |
| SDMA   | System Direct Memory Access  系统直接存储器访问                 |
| RDMA   | Remote Direct Memory Access                                  |
| SEV    | Secure Encrypted Virtualization  安全虚拟化                    |
| ECU    | Engine Control Unit   引擎控制单元                             |
| DC     | Dispatch Controller                                      |
| KIQ    | kernel interface queue                                  |
| KCQ    | KMD Compute Queue                                       |
| KGQ    | KMD graphics Queue                                      |
| HIQ    | HSA Interface Queue                                     |
| DIQ    | Debug Interface Queue                                   |
| SVM    | share virtual memory                                    |
| UTC    | Unified Translation Cache                               |
| EA     | Efficiency Arbiter                                       |
| VMID   | Virtual Memory Identifier                              |
| MQD    | Memory Queue Descriptor                                 |
| HQD    | Hardware Queue Descriptor                               |
| SEM    | Semaphore  信号量                                       |
| SPM    | Stream Performance Monitor                              |
| BMC    | Baseboard Management Controller  基板管理控制器         |
| HBM    | High Bandwidth Memory                                   |
| BLAS   | Basic Linear Algebra Subprograms  基础线性代数子程序库 |

## GPU基础

[NVIDIA Turing 架构深度介绍](https://developer.nvidia.com/zh-cn/blog/nvidia-turing-architecture-in-depth/)
[A卡和N卡的架构有什么区别？](https://www.zhihu.com/question/267104699)
[Nvidia 硬件架构](https://zhuanlan.zhihu.com/p/545764760)

### 纹理缓存

[GPU Texture - Mipmap, Bilinear and Cache](https://zhuanlan.zhihu.com/p/494964914)

### GPU中的Frame Buffer

**Frame Buffer** 是图形处理单元（GPU）中的一个关键组件，主要用于存储即将显示在屏幕上的图像数据。它是计算机图形系统中非常重要的一部分。以下是关于Frame Buffer的详细解释：

#### 1. Frame Buffer的定义和作用

**Frame Buffer** 是一种内存缓冲区，用于存储每一帧图像的像素数据，这些数据将被显示器显示。它的主要作用包括：

- **存储图像数据**：Frame Buffer存储从GPU生成的像素数据，包括颜色、深度和透明度信息。
- **显示输出**：显示器从Frame Buffer读取像素数据并将其显示出来，每秒多次刷新以形成连续的视频流。

#### 2. Frame Buffer的工作原理

1. **图像生成**：
   - GPU处理图形指令，执行顶点着色、像素着色等操作，生成每个像素的颜色值和深度值。
   - 这些像素数据被写入Frame Buffer。

2. **存储结构**：
   - Frame Buffer通常由多个缓冲区组成，如前缓冲区（Front Buffer）和后缓冲区（Back Buffer）。
   - 后缓冲区用于GPU写入新的帧数据，前缓冲区用于显示器读取当前显示的帧数据。

3. **双缓冲技术**：
   - GPU在后缓冲区生成新帧数据的同时，显示器从前缓冲区读取并显示上一帧数据。
   - 当新帧生成完毕时，两个缓冲区交换角色（即前后缓冲区交换），以减少屏幕撕裂现象。

4. **同步机制**：
   - 垂直同步（V-Sync）：显示器刷新和Frame Buffer更新同步，以避免图像撕裂。
   - 动态同步技术（如G-Sync和FreeSync）：动态调整显示刷新率与Frame Buffer更新频率匹配，提高显示效果。

#### 3. Frame Buffer的组成

- **颜色缓冲区（Color Buffer）**：存储每个像素的颜色信息（通常包含红、绿、蓝和透明度通道）。
- **深度缓冲区（Depth Buffer）**：存储每个像素的深度信息，用于深度测试和隐藏面消除。
- **模板缓冲区（Stencil Buffer）**：用于复杂的图形操作，如模板测试和阴影绘制。

#### 4. 应用场景

- **游戏和图形渲染**：实时生成和显示高质量图像和视频。
- **视频播放**：解码视频帧并显示在屏幕上。
- **用户界面（UI）渲染**：生成和显示应用程序的图形界面。

#### 5. 性能和优化

- **带宽要求**：Frame Buffer需要高带宽的内存访问，以支持高分辨率和高刷新率的显示输出。
- **存储优化**：使用压缩技术和高效的内存管理策略，以减少存储需求和提高数据传输效率。

### GPU中的DF和UMC

在GPU架构中，DF（Data Fabric）和UMC（Unified Memory Controller）是两个重要的组件，它们在数据传输和内存管理中扮演着关键角色。以下是对DF和UMC的详细解释：

#### 1. DF（Data Fabric）

**Data Fabric** 是一种用于高效数据传输和通信的架构，在GPU和其他处理单元之间起到桥梁作用。DF的主要功能包括：

1. **数据传输**：
   - 负责在GPU的不同处理单元（如计算核心、内存控制器、显示控制器）之间传输数据。
   - 提供高带宽、低延迟的数据路径，确保数据能够快速、可靠地在各单元之间传递。

2. **数据管理**：
   - 管理数据的路由和调度，确保数据在正确的时间到达正确的位置。
   - 优化数据流，减少瓶颈，提升整体系统性能。

3. **系统集成**：
   - 将不同类型的处理单元（如CPU、GPU、加速器）集成到一个统一的架构中，提供统一的数据访问和管理接口。
   - 支持异构计算环境，提升系统的灵活性和扩展性。

#### 2. UMC（Unified Memory Controller）

**Unified Memory Controller（UMC）** 是一种用于管理和控制系统内存访问的组件。UMC的主要功能包括：

1. **内存管理**：
   - 管理GPU对系统内存的访问，包括内存分配、释放和访问控制。
   - 处理内存请求，调度内存访问操作，确保高效的内存利用率。

2. **内存带宽优化**：
   - 提供高带宽的内存接口，支持多通道并发访问，提升数据传输效率。
   - 实现内存访问优化，减少访问延迟，提升系统性能。

3. **统一内存访问**：
   - 支持统一内存架构（UMA），使CPU和GPU可以共享同一片物理内存。
   - 提供一致的内存视图，简化编程模型，提高编程效率。

#### DF和UMC的关系和作用

在GPU架构中，DF和UMC共同协作，确保数据在处理单元之间高效传输，并优化内存访问：

1. **高效数据传输**：
   - DF负责在处理单元之间传输数据，UMC管理内存访问，确保数据能够及时传递到需要的位置。
   - 通过DF和UMC的协同工作，GPU可以高效地处理复杂的计算任务，提升整体性能。

2. **内存访问优化**：
   - UMC提供高带宽的内存接口和优化的内存访问策略，DF通过高效的数据路由和调度，确保数据传输的高效性。
   - 这种协同工作使得GPU能够在处理高负载任务时保持高效的内存访问和数据传输。

3. **系统集成和扩展性**：
   - DF提供统一的数据管理接口，将GPU与其他处理单元集成到一个统一的架构中。
   - UMC支持统一内存访问，使得不同处理单元可以共享内存资源，提升系统的灵活性和扩展性。

总结：

- **DF（Data Fabric）**：负责高效的数据传输和管理，确保GPU和其他处理单元之间的数据能够快速、可靠地传递。
- **UMC（Unified Memory Controller）**：管理和优化内存访问，提供高带宽、低延迟的内存接口，支持统一内存架构。

通过DF和UMC的协同工作，GPU能够高效地处理数据传输和内存访问任务，提升整体系统性能和灵活性。这两个组件在现代GPU架构中扮演着关键角色，是实现高性能计算和图形处理的基础。

### GPU的Doorbell机制

**Doorbell机制** 是现代GPU架构中用于高效通信和同步的一种硬件机制，特别是在主机（Host）和GPU之间的通信中起到了关键作用。以下是对GPU的Doorbell机制的详细解释：

#### 1. Doorbell机制的定义

**Doorbell** 是一种硬件通知机制，用于在主机和GPU之间进行快速、低延迟的通信。它通过特定的内存地址实现，当主机写入该地址时，GPU可以立即被通知到有新的工作需要处理。

#### 2. Doorbell机制的工作原理

1. **内存映射**：
   - Doorbell地址通常是内存映射的I/O地址，主机通过PCIe总线将这些地址映射到其地址空间。
   - 每个Doorbell地址对应一个特定的GPU队列或任务。

2. **主机通知GPU**：
   - 当主机有新的任务需要GPU处理时，会向对应的Doorbell地址写入数据（通常是任务描述符的地址或任务标识）。
   - 这种写操作触发了Doorbell机制，立即通知GPU有新的任务需要处理。

3. **GPU响应**：
   - GPU监控Doorbell地址，当检测到写操作时，会读取写入的数据，并根据任务描述符或任务标识获取具体任务。
   - GPU随后将新任务添加到其任务队列中，开始处理。

#### 3. Doorbell机制的优势

1. **低延迟**：
   - Doorbell机制通过直接内存写入实现通知，相比传统的中断或轮询机制，具有更低的通信延迟。

2. **高效通信**：
   - 主机与GPU之间的通信通过Doorbell机制可以实现快速高效，减少了CPU和GPU之间的等待时间，提高了系统整体性能。

3. **简单实现**：
   - Doorbell机制实现简单，主机只需通过内存写操作即可通知GPU，无需复杂的同步机制。

#### 4. Doorbell机制的应用场景

1. **任务调度**：
   - 在任务调度中，主机通过Doorbell机制通知GPU有新的任务需要处理，GPU立即响应并开始执行任务。

2. **数据传输**：
   - 当主机需要将数据传输给GPU处理时，使用Doorbell机制通知GPU开始数据传输和处理。

3. **同步操作**：
   - Doorbell机制也可用于同步操作，主机通知GPU完成某些准备工作或等待某个事件的发生。

以下是一个Doorbell机制在主机和GPU之间通信的简化示例：

1. **主机准备任务**：
   - 主机将任务描述符或任务标识写入内存。

2. **主机写入Doorbell地址**：
   - 主机向Doorbell地址写入任务描述符的地址或任务标识。
   - 例如，假设Doorbell地址为`0x1234`，主机执行如下写操作：

     ```c
     *(volatile uint32_t *)0x1234 = task_descriptor_address;
     ```

3. **GPU检测Doorbell**：
   - GPU监控Doorbell地址，当检测到写操作时，读取写入的数据：

     ```c
     uint32_t task_descriptor_address = *(volatile uint32_t *)0x1234;
     ```

4. **GPU处理任务**：
   - GPU根据任务描述符地址获取具体任务，并将其添加到任务队列中开始处理。

总结：

**Doorbell机制** 是一种高效的硬件通知机制，用于主机和GPU之间的快速通信和同步。通过直接内存写入实现低延迟通知，Doorbell机制显著提高了任务调度和数据传输的效率，在现代GPU架构中扮演着重要角色。

### Doorbell的缺点

虽然Doorbell机制在GPU与主机之间的通信中提供了高效的通知和同步手段，但它也存在一些缺点和局限性。以下是Doorbell机制的主要缺点：

#### 1. 硬件资源占用

**有限的Doorbell数量**：

- Doorbell机制依赖于特定的内存映射I/O地址，硬件资源有限。
- 如果系统中需要大量的Doorbell来支持多个任务队列或多个GPU，硬件资源可能不足，限制了扩展性。

#### 2. 同步开销

**同步操作**：

- 尽管Doorbell机制减少了轮询和中断的延迟，但仍需要处理同步开销。
- 多个线程或进程在访问同一个Doorbell地址时，需要进行同步，以避免竞态条件（Race Condition），这增加了实现的复杂性。

#### 3. 处理负载

**频繁通知**：

- 在高负载环境中，频繁的Doorbell通知可能导致CPU和GPU之间的频繁上下文切换，增加处理负载。
- 如果Doorbell通知频率过高，可能导致性能下降，尤其是在任务粒度较小时，频繁的通知和处理会显著增加系统开销。

#### 4. 延迟和吞吐量瓶颈

**PCIe总线延迟**：

- Doorbell通知通过PCIe总线传输，尽管PCIe具有高带宽，但仍存在一定的传输延迟。
- 在高并发情况下，PCIe总线可能成为瓶颈，影响Doorbell通知的延迟和系统的整体吞吐量。

#### 5. 复杂性和实现成本

**实现复杂性**：

- Doorbell机制的实现需要硬件和软件的紧密配合，增加了设计和实现的复杂性。
- 需要在硬件中实现特定的寄存器和内存映射逻辑，同时在软件层面处理Doorbell的读写和同步。

#### 6. 数据一致性

**一致性问题**：

- 在多处理器或多核系统中，确保Doorbell通知的一致性和顺序是一个挑战。
- 需要确保写入Doorbell的操作与对应任务数据的写入顺序一致，以避免数据不一致的问题。

#### 7. 错误处理

**错误处理复杂性**：

- Doorbell机制涉及底层硬件的直接操作，错误处理和恢复机制更加复杂。
- 在Doorbell通知过程中出现错误或失败时，系统需要有有效的机制来检测和处理这些问题，以保证系统的可靠性。

总结：

尽管Doorbell机制在高效通知和同步方面具有显著优势，但它也存在硬件资源占用、同步开销、频繁通知负载、延迟和吞吐量瓶颈、实现复杂性、数据一致性以及错误处理等方面的缺点。这些问题需要在设计和实现中仔细考虑，以确保系统的性能和可靠性。

解决这些缺点的措施：

1. **优化硬件设计**：增加Doorbell寄存器的数量和灵活性，以支持更多的任务和GPU。
2. **软件同步**：使用锁机制或原子操作来处理多个线程的同步问题，确保安全访问。
3. **批量通知**：在可能的情况下，合并多个任务的通知，减少Doorbell写操作的频率。
4. **异步处理**：采用异步任务处理和调度，减少Doorbell通知带来的上下文切换开销。
5. **高效总线使用**：优化PCIe总线的使用，减少传输延迟，提升整体吞吐量。
6. **一致性保证**：在多核系统中，使用内存屏障和同步机制确保数据一致性。
7. **健壮的错误处理**：设计有效的错误检测和恢复机制，以应对Doorbell通知过程中可能出现的问题。

通过这些优化措施，可以在一定程度上缓解Doorbell机制的缺点，提升系统的性能和可靠性。

### GC Command Queue

**GC Command Queue**（图形计算命令队列）是在GPU架构中用于管理和调度图形和计算命令的一种机制。它的主要功能是组织和协调来自CPU的各种图形和计算任务，使GPU能够高效地处理这些任务。

#### 1. GC Command Queue的定义

GC Command Queue是一个先进先出（FIFO）的命令队列，用于存储和管理待处理的图形和计算命令。命令队列的主要目的是将CPU发出的图形和计算命令有序地发送给GPU进行处理。

#### 2. GC Command Queue的工作原理

1. **命令提交**：
   - 应用程序通过API（如OpenGL、DirectX、Vulkan、CUDA等）向GPU提交图形或计算命令。
   - 这些命令被放入GC Command Queue中，等待GPU处理。

2. **命令队列管理**：
   - GC Command Queue按照FIFO的原则管理命令，确保命令按提交的顺序被执行。
   - 命令队列可以包含多种类型的命令，如绘图命令、计算任务、内存传输等。

3. **命令调度和执行**：
   - GPU从GC Command Queue中读取命令，并将其分配给适当的处理单元（如图形核心、计算核心）。
   - GPU执行这些命令，并在完成后将结果返回给应用程序或进行下一步处理。

#### 3. GC Command Queue的优势

1. **高效资源利用**：
   - 通过命令队列，可以实现命令的有序管理和调度，提高GPU资源的利用效率。
   - 命令队列可以减少CPU与GPU之间的同步开销，提升整体系统性能。

2. **任务并行处理**：
   - GC Command Queue支持命令的并行处理，GPU可以同时处理多个命令，提高并行计算能力。
   - 多个命令队列可以用于不同的任务，进一步提升任务调度的灵活性和效率。

3. **简化编程模型**：
   - 命令队列提供了一种统一的接口，简化了开发人员的编程模型。
   - 开发人员可以通过命令队列提交命令，而无需关心底层的命令调度和执行细节。

#### 4. 应用场景

1. **图形渲染**：
   - 在图形渲染中，GC Command Queue用于管理绘图命令，如绘制三角形、纹理映射、光照计算等。
   - 渲染引擎将这些命令提交到命令队列，GPU按照顺序处理这些命令，生成最终的图像。

2. **计算任务**：
   - 在计算密集型应用（如科学计算、机器学习）中，GC Command Queue用于管理计算任务。
   - 计算任务被分解为多个命令，提交到命令队列，GPU并行处理这些任务，提高计算效率。

3. **数据传输**：
   - GC Command Queue还用于管理数据传输命令，如从主机内存到GPU内存的传输。
   - 通过命令队列，可以实现数据传输与计算任务的并行处理，提高整体效率。

#### 5. 具体示例

以下是一个使用Vulkan API的GC Command Queue示例，展示了如何向GPU提交命令：

```cpp
// 创建命令队列
VkQueue graphicsQueue;
vkGetDeviceQueue(device, graphicsQueueFamilyIndex, 0, &graphicsQueue);

// 创建命令缓冲区
VkCommandBufferAllocateInfo allocInfo = {};
allocInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
allocInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
allocInfo.commandPool = commandPool;
allocInfo.commandBufferCount = 1;

VkCommandBuffer commandBuffer;
vkAllocateCommandBuffers(device, &allocInfo, &commandBuffer);

// 开始记录命令
VkCommandBufferBeginInfo beginInfo = {};
beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;

vkBeginCommandBuffer(commandBuffer, &beginInfo);

// 记录绘图命令（示例）
vkCmdBindPipeline(commandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, graphicsPipeline);
vkCmdDraw(commandBuffer, vertexCount, instanceCount, firstVertex, firstInstance);

// 结束记录命令
vkEndCommandBuffer(commandBuffer);

// 提交命令到命令队列
VkSubmitInfo submitInfo = {};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;
submitInfo.commandBufferCount = 1;
submitInfo.pCommandBuffers = &commandBuffer;

vkQueueSubmit(graphicsQueue, 1, &submitInfo, VK_NULL_HANDLE);

// 等待命令执行完成
vkQueueWaitIdle(graphicsQueue);
```

总结：

GC Command Queue是GPU架构中管理和调度图形和计算命令的关键机制。它通过FIFO队列的方式，将CPU提交的命令有序地发送给GPU处理，实现高效的资源利用和任务并行处理。GC Command Queue在图形渲染、计算任务和数据传输等应用场景中起到了重要作用，显著提高了系统的性能和灵活性。

### CR3寄存器（Control Register 3）

**CR3** 是一个在x86和x86-64架构的CPU中使用的重要控制寄存器。它在内存管理和虚拟化中扮演着关键角色，尤其是在页表管理方面。以下是关于CR3寄存器的详细解释：

#### 1. CR3寄存器的定义

**CR3** 寄存器（Control Register 3）用于存储页表的物理基地址，具体来说，它存储的是当前进程或虚拟机的根页表（Page Directory Table）的物理地址。在启用分页机制（Paging）时，CPU通过CR3寄存器找到当前的页表，以进行虚拟地址到物理地址的转换。

#### 2. CR3寄存器的作用

1. **页表基地址**：
   - CR3寄存器包含当前页目录表或页全局目录表的物理地址，CPU使用这个地址进行地址转换。

2. **上下文切换**：
   - 在多任务系统中，每次上下文切换时，操作系统会更新CR3寄存器的值，以切换到新的进程或虚拟机的页表。

3. **TLB刷新**：
   - 每次写入CR3寄存器时，通常会导致翻译后备缓冲区（TLB）的刷新，因为地址空间发生了变化，旧的地址翻译缓存失效。

#### 3. CR3寄存器在虚拟机中的使用

当虚拟机启动时，虚拟机监视器（VMM）或虚拟机管理程序（如KVM、VMware）负责初始化和管理虚拟机的内存，包括设置页表。以下是具体步骤：

1. **GPU驱动初始化CR3寄存器**：
   - GPU驱动程序负责设置和初始化与GPU相关的内存管理结构，包括页表。
   - 在虚拟机启动时，GPU驱动程序会初始化CR3寄存器，将其指向虚拟机的根页表（Page Directory Table）的物理地址。

2. **根页表的创建和管理**：
   - 根页表是内存分页机制的起点，包含指向页表的指针，管理虚拟地址到物理地址的映射。
   - GPU驱动在初始化过程中会创建根页表，并将其基地址写入CR3寄存器。

3. **虚拟地址到物理地址的转换**：
   - 当虚拟机或GPU需要访问内存时，CPU使用CR3寄存器中的地址作为起点，通过页表进行虚拟地址到物理地址的转换。
   - 这种机制确保了内存访问的正确性和隔离性。

#### 具体示例：虚拟机启动和CR3初始化

以下是虚拟机启动时初始化CR3寄存器的示例步骤：

1. **创建根页表**：
   - 虚拟机管理程序创建一个新的根页表结构（页目录表），并将其物理地址保存。

2. **初始化CR3寄存器**：
   - 在虚拟机启动过程中，GPU驱动程序或VMM将根页表的物理地址写入CR3寄存器。
   - 例如，在x86架构的汇编代码中，初始化CR3寄存器的操作如下：

     ```assembly
     mov eax, root_page_table_physical_address
     mov cr3, eax
     ```

3. **启用分页机制**：
   - 在CR3寄存器初始化后，CPU使用CR3寄存器中的地址进行虚拟地址到物理地址的转换，确保内存访问的正确性。

总结：

- **CR3寄存器**：存储当前页表的物理基地址，是内存分页机制的关键组成部分。
- **虚拟机启动**：在虚拟机启动过程中，GPU驱动程序初始化CR3寄存器，将其指向虚拟机的根页表，以管理虚拟地址到物理地址的转换。
- **页表管理**：通过CR3寄存器，CPU能够找到并使用当前的页表结构，确保虚拟机内存访问的正确性和隔离性。

理解CR3寄存器及其在虚拟机内存管理中的作用，对于深入了解虚拟化技术和内存分页机制具有重要意义。

### 什么是GPU中的MMHUB？

**MMHUB（Memory Management Hub）** 是现代GPU架构中一个重要的组件，负责管理和调度GPU对内存的访问。它在协调GPU内部的不同子系统和内存之间的数据传输中扮演着关键角色。以下是关于MMHUB的详细解释：

#### 1. MMHUB的定义和作用

**MMHUB** 是GPU内部一个集中的内存管理模块，旨在高效地协调和控制GPU对内存的访问。它通过提供内存地址映射、权限管理和数据传输控制等功能，确保GPU各个部分能够高效、安全地访问内存资源。

#### 2. MMHUB的主要功能

1. **内存地址映射**：
   - MMHUB负责将GPU的虚拟地址映射到物理内存地址。它维护和管理页表，处理地址转换请求。
   - 通过MMHUB，GPU可以高效地进行虚拟内存和物理内存的转换，支持复杂的内存管理操作。

2. **权限管理**：
   - MMHUB管理内存访问权限，确保只有授权的GPU子系统和任务可以访问特定的内存区域。
   - 这种权限控制机制有助于防止未经授权的内存访问，增强系统的安全性。

3. **数据传输控制**：
   - MMHUB协调GPU内部的不同子系统（如图形核心、计算核心、显示引擎）之间的数据传输。
   - 它优化数据流，确保数据能够快速、高效地在各子系统之间传递，减少数据传输的延迟。

4. **内存带宽优化**：
   - MMHUB通过智能调度和优化内存访问，最大化内存带宽利用率。
   - 这种优化有助于提升GPU的整体性能，特别是在处理大量并行任务时。

#### 3. MMHUB在GPU架构中的位置

MMHUB在GPU架构中起着中心枢纽的作用，连接GPU的各个计算单元和内存控制器。它作为一个中介，管理所有内存访问请求并协调这些请求，以确保内存资源被高效利用。

#### 4. MMHUB的工作流程

1. **地址转换请求**：
   - GPU的各个子系统向MMHUB发送内存访问请求，包括虚拟地址。
   - MMHUB根据页表将虚拟地址转换为物理地址。

2. **权限检查**：
   - MMHUB检查请求是否具有访问该内存区域的权限。
   - 如果权限检查通过，MMHUB允许内存访问，否则拒绝请求并返回错误。

3. **数据传输**：
   - MMHUB协调数据传输，将数据从内存传输到请求的子系统，或将数据写回内存。
   - 它优化数据传输路径，确保高效的数据流。

#### 5. MMHUB的应用场景

1. **图形渲染**：
   - 在图形渲染过程中，MMHUB管理图形核心对显存的访问，确保渲染数据的高效传输。

2. **计算任务**：
   - 在并行计算任务中，MMHUB协调计算核心对内存的访问，支持高性能计算。

3. **显示输出**：
   - MMHUB管理显示引擎对帧缓冲区的访问，确保图像数据能够快速传输到显示器。

总结：

**MMHUB（Memory Management Hub）** 是GPU架构中的关键组件，负责管理和优化GPU对内存的访问。通过提供内存地址映射、权限管理和数据传输控制等功能，MMHUB确保了GPU各个部分能够高效、安全地访问内存资源。它在提升GPU整体性能、支持复杂内存管理操作和增强系统安全性方面发挥了重要作用。了解MMHUB的工作原理和功能，有助于深入理解现代GPU架构及其高效内存管理机制。

### 什么是SHUB？

**SHUB（System Hub）** 是一种在计算机系统架构中用于管理和协调系统内各种数据传输和通信的关键组件。SHUB可以被认为是连接系统中多个处理单元（如CPU、GPU、内存、外设等）的中心枢纽，提供高效的互连和数据传输能力。虽然SHUB的具体实现和功能可能因不同系统架构而异，但其核心作用是统一管理和优化系统内的数据流。

#### 1. SHUB的定义和作用

**SHUB** 是一个集中的系统控制和数据管理模块，旨在提高系统的整体效率和性能。它通过提供高速互连、数据路由、内存管理和外设控制等功能，确保系统内各组件之间的通信顺畅、高效。

#### 2. SHUB的主要功能

1. **高速互连**：
   - SHUB为系统中的CPU、GPU、内存和其他关键组件提供高速数据传输通道。
   - 它支持高带宽、低延迟的通信，满足现代计算和图形处理的需求。

2. **数据路由和调度**：
   - SHUB负责智能路由和调度数据传输，优化数据流路径，避免瓶颈和冲突。
   - 它能够动态分配带宽和优先级，确保关键任务得到及时处理。

3. **内存管理**：
   - SHUB管理系统内存的分配和访问控制，确保内存资源的高效利用。
   - 它可以支持虚拟内存映射和地址转换，提高内存访问的灵活性和安全性。

4. **外设控制**：
   - SHUB管理和控制系统中的各种外设，包括存储设备、网络接口、输入输出设备等。
   - 它提供统一的外设接口，简化系统设计和设备集成。

5. **系统协调**：
   - SHUB协调系统内不同处理单元的操作，确保任务调度和资源分配的高效性。
   - 它在多处理器和多核系统中起到关键的协调作用，提升整体系统性能。

#### 3. SHUB在系统架构中的位置

在现代系统架构中，SHUB通常位于CPU和其他关键组件之间的核心位置，充当数据和控制信号的集中管理器。它通过高速总线或专用互连接口连接系统中的各个部分。

#### 4. SHUB的应用场景

1. **高性能计算（HPC）**：
   - 在高性能计算系统中，SHUB用于管理和优化大量数据的传输和处理，提高计算效率。

2. **图形处理**：
   - 在图形处理单元（GPU）中，SHUB用于协调图形渲染和计算任务，支持高分辨率和复杂图形的实时处理。

3. **数据中心**：
   - 在数据中心架构中，SHUB用于管理服务器节点之间的通信和数据流，提高数据中心的整体性能和可靠性。

4. **嵌入式系统**：
   - 在嵌入式系统中，SHUB用于简化系统设计，提供统一的数据管理和外设控制接口。

#### 5. SHUB的优势

1. **高效数据管理**：
   - SHUB通过智能路由和调度功能，优化数据传输路径，提高系统数据管理效率。

2. **扩展性和灵活性**：
   - SHUB支持多处理单元和外设的互连，提供良好的系统扩展性和灵活性。

3. **性能提升**：
   - 通过高带宽、低延迟的互连和数据管理，SHUB显著提升系统的整体性能。

4. **简化设计**：
   - 统一的管理接口和外设控制简化了系统设计和集成，降低了开发复杂性。

#### 总结

**SHUB（System Hub）** 是一种用于管理和优化计算机系统内数据传输和通信的关键组件。它通过提供高速互连、数据路由、内存管理和外设控制等功能，确保系统内各组件之间的高效通信和协调。SHUB在高性能计算、图形处理、数据中心和嵌入式系统等领域发挥着重要作用，显著提升了系统的整体性能和设计灵活性。

理解SHUB的功能和应用，有助于更好地设计和优化现代计算机系统架构。

### VF和PF的区别及互相访问和调用

**PF（Physical Function）** 和 **VF（Virtual Function）** 是SR-IOV（Single Root I/O Virtualization，单根I/O虚拟化）技术中的两个关键概念。SR-IOV允许一个物理PCIe设备在硬件层面上实现多个虚拟设备，使得虚拟机能够直接访问物理设备，从而提高性能。

#### 1. PF（Physical Function）

**PF（Physical Function）** 是指物理PCIe设备的功能实体，提供完整的配置、管理和控制功能。每个物理设备至少有一个PF，用于执行设备的基本操作和管理任务。

##### 主要特点

- **完整性**：PF包含设备的完整功能集，能够执行设备的配置、管理和控制。
- **管理功能**：PF用于配置和管理VF，包括创建、初始化和销毁VF。
- **直接访问**：PF可以直接访问物理设备的所有资源，包括寄存器、内存和I/O空间。

#### 2. VF（Virtual Function）

**VF（Virtual Function）** 是由PF创建的虚拟设备，提供物理设备的一部分功能。每个物理设备可以创建多个VF，使得多个虚拟机可以共享同一个物理设备。

##### 主要特点

- **有限功能**：VF提供物理设备的一部分功能，通常用于处理特定类型的工作负载。
- **隔离性**：每个VF在硬件层面上是隔离的，多个VF可以独立运行，互不干扰。
- **性能**：VF通过硬件直通技术（如Intel的VT-d或AMD的IOMMU）直接访问物理设备资源，具有接近原生的性能。

#### 3. PF和VF的区别

- **功能范围**：PF包含完整的设备功能集，而VF只包含部分功能。
- **管理能力**：PF具有配置和管理VF的能力，VF没有管理其他功能的能力。
- **资源访问**：PF可以访问设备的所有资源，而VF只能访问分配给它的资源。

#### 4. PF和VF之间的访问和调用

在SR-IOV架构中，PF和VF之间的访问和调用机制主要涉及以下几个方面：

##### 1. PF管理VF

- **创建和销毁**：PF负责创建和销毁VF。创建VF时，PF分配所需的资源（如内存、I/O空间）并初始化VF。

  ```c
  // 伪代码示例
  pf_create_vf(device_id, vf_id, resources);
  ```

- **配置和初始化**：PF初始化VF的配置空间和寄存器，使其能够正常工作。

  ```c
  // 伪代码示例
  pf_init_vf(vf_id, config);
  ```

##### 2. VF访问设备资源

- **直接资源访问**：VF通过硬件直通技术直接访问物理设备的资源，如寄存器和内存。VF的访问权限由PF配置和管理。

  ```c
  // 伪代码示例
  vf_access_resource(vf_id, resource_id);
  ```

##### 3. 中断和通知机制

- **中断**：VF可以生成中断，通知PF或主机操作系统（Hypervisor）某些事件的发生，如数据包到达或任务完成。PF或Hypervisor处理这些中断，并作出相应的响应。

  ```c
  // 伪代码示例
  vf_generate_interrupt(vf_id, event);
  ```

- **通知机制**：PF可以通过配置寄存器或消息传递通知VF执行特定操作。VF响应这些通知，执行相应任务。

  ```c
  // 伪代码示例
  pf_notify_vf(vf_id, command);
  ```

#### 5. 示例应用场景

- **网络适配器**：在SR-IOV网络适配器中，PF用于配置和管理多个VF，每个VF可以分配给不同的虚拟机，使得虚拟机能够直接处理网络数据包，提供高性能的网络通信。
- **存储设备**：在SR-IOV存储设备中，PF用于管理多个VF，每个VF处理不同虚拟机的存储请求，提高存储访问的效率和性能。

总结：

**PF（Physical Function）** 和 **VF（Virtual Function）** 是SR-IOV技术中的关键组件，用于在硬件层面实现I/O虚拟化。PF提供完整的设备功能和管理能力，负责创建、配置和管理VF。VF作为虚拟设备，提供物理设备的一部分功能，直接访问分配给它的资源，实现高效的I/O操作。PF和VF之间通过硬件直通技术、配置寄存器和中断机制实现相互访问和调用，确保虚拟化环境中的高性能和高效管理。理解PF和VF的区别及其交互机制，有助于更好地设计和优化虚拟化系统。

### 什么是VF页表？

**VF页表** 是在SR-IOV（Single Root I/O Virtualization）环境中，虚拟功能（VF，Virtual Function）使用的一种内存管理结构。它用于管理和映射虚拟机（VM）访问的虚拟地址到物理地址的转换，从而允许虚拟机高效、安全地访问分配给它的物理设备资源。

#### SR-IOV概述

在SR-IOV技术中，一个物理PCIe设备（如网络接口卡、存储控制器等）可以被分成多个虚拟功能（VF），每个VF可以分配给不同的虚拟机，使其看起来像独立的物理设备。这种虚拟化方法允许多个虚拟机共享同一个物理设备，提高资源利用率和性能。

#### VF页表的作用

**VF页表** 主要作用是管理虚拟地址到物理地址的映射，使虚拟机能够安全、高效地访问物理设备资源。以下是VF页表的主要功能和作用：

1. **地址转换**：
   - VF页表将虚拟机使用的虚拟地址转换为物理地址。虚拟机通过VF页表访问分配给它的设备内存和资源。
   - 地址转换通过硬件支持的IOMMU（Input-Output Memory Management Unit，输入输出内存管理单元）进行，确保高效和安全的地址映射。

2. **内存隔离**：
   - VF页表确保每个虚拟机只能访问分配给它的内存区域，防止不同虚拟机之间的内存干扰和数据泄露。
   - 通过页表机制，IOMMU能够隔离和保护各虚拟机的内存访问，增强系统的安全性。

3. **性能优化**：
   - VF页表通过硬件加速的地址转换，提高了内存访问的性能。IOMMU在设备和主内存之间提供了直接的数据路径，减少了地址转换的开销。
   - 支持大页（Huge Pages）和快速地址转换机制，进一步优化内存访问性能。

#### VF页表的工作原理

1. **虚拟地址生成**：
   - 虚拟机生成虚拟地址（VA），用于访问其分配的设备资源。
   - 虚拟地址通过VF页表进行地址转换，映射到物理地址（PA）。

2. **IOMMU地址转换**：
   - IOMMU接收来自VF的虚拟地址请求，通过查找VF页表，将虚拟地址转换为物理地址。
   - 地址转换过程包括多级页表查找，以找到最终的物理地址。

3. **内存访问**：
   - 转换后的物理地址用于访问实际的设备内存或系统内存，完成数据读写操作。
   - 内存访问通过IOMMU进行管理和控制，确保安全性和高效性。

#### VF页表的结构

VF页表的结构与传统的CPU页表类似，通常包括多级页表，以支持大范围的地址转换。典型的页表结构包括：

1. **页目录表（PDT，Page Directory Table）**：
   - 一级页表，指向页表目录的地址。

2. **页表目录（PD，Page Directory）**：
   - 二级页表，包含指向页表项的地址。

3. **页表项（PTE，Page Table Entry）**：
   - 三级页表，包含虚拟地址到物理地址的具体映射信息。

4. **页（Page）**：
   - 实际的数据存储单元，对应物理内存中的一个块。

#### 示例：VF页表的工作流程

假设一个虚拟机通过VF页表访问设备内存，其工作流程如下：

1. **虚拟机生成虚拟地址**：
   - 虚拟机生成一个虚拟地址VA，用于访问设备内存。

2. **IOMMU查找页表**：
   - IOMMU接收到VA请求，通过查找VF页表，找到对应的物理地址PA。
   - 页表查找过程可能涉及多级页表查找，逐级解析虚拟地址。

3. **物理内存访问**：
   - IOMMU将VA转换为PA后，使用PA访问实际的设备内存或系统内存，完成数据传输。

#### 总结

**VF页表** 是在SR-IOV环境中，用于管理和映射虚拟地址到物理地址的内存管理结构。通过IOMMU，VF页表实现了虚拟机内存访问的安全性和高效性，确保每个虚拟机只能访问其分配的内存区域。理解VF页表的作用和工作原理，有助于设计和优化虚拟化环境中的内存管理和设备资源共享。

### TLB和ATS，ATC

RC（Root Complex）进行DMA地址转换是需要时间的，相较于不进行地址转换，显然进行DMA地址转换会增加DMA访问的时间。尤其是访问驻留内存转换表时，采用地址转换的方案会大大增加DMA访问的时间。当单次传输需要多次内存访问时，地址转换无疑会大大降低传输效率。

为了减小地址转换的以上不良影响，设计人员常常在需要进行地址转换的地方添加地址转换缓存（Address Translation Cache, ATC）。在CPU中，这种地址转换缓存通常是指转译后备缓冲区（Translation Look-aside Bufer, TLB）；在IO地址转换中，我们常用ATC来跟CPU的TLB加以区分。TLB与ATC的区别：TLB一次只服务于CPU的单个线程，而ATC服务于PCIe设备的多个IO function，每个IO function都相当于一个独立的线程。

RC： 在PCI Express（PCIe）系统中，根复合体（root complex）设备将处理器和内存子系统连接到由一个或多个交换设备组成的PCI Express交换结构。

来自AMD：
There are various reasons for enabling System Cache address translation, including:

- Avoiding host device driver and letting accelerators work directly with addresses provided by the host application
- Limiting the impact of “memory leakage” or an incorrectly programmed endpoint
- Address space conversion (smaller endpoint address range to larger system virtual address space)
- Providing scatter/gather functionality
- Virtualization support
- The System Cache includes an ATC function with companion ATS to support virtual address handling.

### DRM

DRM是Linux内核中负责与显卡交互的管理架构，用户空间很方便的利用DRM提供的API，实现3D渲染、视频解码和GPU计算等工作。

Direct Rendering Manager 提供了一个抽象层，允许用户空间程序（如图形用户界面）直接访问和控制图形硬件，而无需通过 X Window System 或其他中间件。这样可以提高图形性能和响应速度，减少渲染的延迟。

在 AMD 驱动中，DRM 模块负责与 AMD 显卡通信，处理图形渲染任务，并提供与用户空间的接口，以便图形应用程序可以使用 GPU 进行加速渲染。通过 DRM，用户空间程序可以利用 AMD 显卡的功能，执行各种图形相关的任务，如 2D 和 3D 渲染、视频解码等。

### 管中窥"GPU ISA"

LDS指令格式：
![](/img/post_pics/gpu/LDS_ISA.png)

| Field Name |Bits |Format or Description|
|----|----|----|
| OFFSET0 | [7:0]  | First address offset |
| OFFSET1 | [15:8] | Second address offset. For some opcodes this is concatenated with OFFSET0. |
| GDS     | [16]   | 1=GDS, 0=LDS operation. |
| OP      | [24:17] | See Opcode table below. |
| ENCODING| [31:26] | Must be: 110110 |
| ADDR    | [39:32] | VGPR which supplies the address. |
| DATA0   | [47:40] | First data VGPR. |
| DATA1   | [55:48] | Second data VGPR. |
| VDST    | [63:56] | Destination VGPR when results returned to VGPRs. |

两个简单的DS opcode：

| OPCODE | Name | Description |
|----|----|----|
| 13     | DS_WRITE_B32 | MEM[ADDR] = DATA. // Write dword. 32bit |
|  54    | DS_READ_B32  | RETURN_DATA = MEM[ADDR]. // Dword read. |

#### HIP

```c
#include <hip/hip_runtime.h>

extern "C"
__global__ void ds_read_b32(int *out) {
  HIP_DYNAMIC_SHARED(int, sharedTmp);
  // init the lds
  sharedTmp[threadIdx.x] = threadIdx.x;
  __syncthreads();
  int ldsAddr = threadIdx.x * 4;
  int result;
  asm volatile(
      "ds_read_b32 %0 ,%1\n"
      "s_waitcnt lgkmcnt(0)"
      : "=v"(result)
      : "v"(ldsAddr)
      :);
  out[threadIdx.x] = result;
}
```

这段代码是使用 AMD 的 HIP（Heterogeneous-Compute Interface for Portability）编写的一个 CUDA 核函数。HIP 是一个用于编写可移植 GPU 加速代码的工具，它允许开发者在不同的 GPU 架构上编写相似的代码，并且可以通过编译时选择目标架构进行优化。

`asm volatile(...)`这是一个内联汇编语句，允许在 CUDA/HIP 程序中直接嵌入 GPU 汇编指令。在这里，使用了 `ds_read_b32` 汇编指令来从动态共享内存中读取一个 32 位整数，并将结果存储在 `result` 中。

这段内联汇编代码是针对 AMD GPU 架构中的数据寄存器（DS - Data Share）指令 `ds_read_b32` 的调用。DS 指令结构解释：

- `OFFSET0` 和 `OFFSET1`：用于指定地址的偏移量。
- `GDS`：用于指示是 GDS（Global Data Share）还是 LDS（Local Data Share）操作。在这里，`GDS` 为 0，表示是一个 LDS（本地数据共享）操作。
- `OP`：操作码，指定 DS 指令的具体操作类型。在这里，未给出具体的操作码，因此在这段代码中的 `ds_read_b32` 指令将会执行读取操作。
- `ENCODING`：固定为 110110，用于识别 DS 指令。
- `ADDR`：VGPR 寄存器，用于提供地址。
- `DATA0` 和 `DATA1`：VGPR 寄存器，用于存储读取的数据。
- `VDST`：目的 VGPR 寄存器，用于存储结果。

内联汇编代码解释：

```c
asm volatile(
    "ds_read_b32 %0 ,%1\n"
    "s_waitcnt lgkmcnt(0)"
    : "=v"(result)  // 输出操作数，将结果写入 result 变量
    : "v"(ldsAddr) // 输入操作数，使用 ldsAddr 变量作为地址
    :);            // 没有使用到任何 clobbered 寄存器
```

- `"ds_read_b32 %0 ,%1\n"`：这是内联汇编中的字符串指令，调用了 DS 指令 `ds_read_b32`。其中 `%0` 和 `%1` 分别对应输出操作数和输入操作数的位置。在这里，`%0` 对应输出结果 `result` 的位置，`%1` 对应输入参数 `ldsAddr` 的位置。这条指令的作用是从 `ldsAddr` 指定的本地数据共享内存中读取一个 32 位整数，并将结果存储到 `result` 变量中。

- `"s_waitcnt lgkmcnt(0)"`：这是一个 HIP 的函数调用，用于等待指令发出前面的所有访存请求完成。在这里，确保 `ds_read_b32` 指令执行完毕后再继续执行后续指令。

- `: "=v"(result)`：这部分是输出操作数（output operands），通过 `=v` 指定了将结果存储到 `result` 变量中，`v` 表示使用 VGPR。
  
- `: "v"(ldsAddr)`：这部分是输入操作数（input operands），指定了使用 `ldsAddr` 变量作为输入地址，同样使用了 VGPR。

- `:`：最后的 `:` 表示 clobbered 寄存器列表为空，即没有修改其他寄存器的需要。

综上所述，这段内联汇编代码通过 `ds_read_b32` 指令从本地数据共享内存中读取一个 32 位整数，并将结果存储到 `result` 变量中。然后使用 `s_waitcnt lgkmcnt(0)` 确保了前面的读取操作已经完成，然后程序继续执行。

#### 汇编

这段代码是一个使用 AMD GPU 架构的 HIP 编写的内核函数 `ds_read_b32` 的汇编代码。让我们一行一行地解释并添加注释：

```assembly
.text
.protected ds_read_b32     ; -- Begin function ds_read_b32
.globl ds_read_b32
.p2align 8
.type ds_read_b32,@function
ds_read_b32:                    ; @ds_read_b32
```

- `.text`: 这表示接下来的指令是代码段，包含程序的可执行指令。
- `.protected ds_read_b32`: 声明 `ds_read_b32` 函数为受保护的，表示其他文件可以使用这个函数。
- `.globl ds_read_b32`: 将 `ds_read_b32` 函数声明为全局可见，使其可以在其他文件中被调用。
- `.p2align 8`: 将下一个符号（symbol）对齐到 2^8 字节（即 256 字节）的边界。
- `.type ds_read_b32,@function`: 声明 `ds_read_b32` 是一个函数。

```assembly
; %bb.0:                                ; %entry
 s_load_dwordx2 s[0:1], s[4:5], 0x0
 v_lshlrev_b32_e32 v1, 2, v0
 v_add_u32_e32 v2, 0, v1
 ds_write_b32 v2, v0
 s_waitcnt lgkmcnt(0)
 v_mov_b32_e32 v3, s1
 v_add_co_u32_e32 v0, vcc, s0, v1
 s_barrier
 s_waitcnt lgkmcnt(0)
 ;;#ASMSTART
 ds_read_b32 v2 ,v1
s_waitcnt lgkmcnt(0)
 ;;#ASMEND
 v_addc_co_u32_e32 v1, vcc, 0, v3, vcc
 global_store_dword v[0:1], v2, off
 s_endpgm
```

这里是核函数的主体部分。解释如下：

- `s_load_dwordx2 s[0:1], s[4:5], 0x0`: 从内存中加载两个双字（32 位整数）到 `s[0:1]` 寄存器中。
- `v_lshlrev_b32_e32 v1, 2, v0`: 将寄存器 `v0` 中的值左移 2 位，并将结果存储到 `v1` 中。
- `v_add_u32_e32 v2, 0, v1`: 将寄存器 `v1` 的值与 0 相加，并将结果存储到 `v2` 中。
- `ds_write_b32 v2, v0`: 将寄存器 `v2` 的值写入数据段（data segment）中的 `v0` 地址。
- `s_waitcnt lgkmcnt(0)`: 等待指令发出前面的所有访存请求完成。
- `v_mov_b32_e32 v3, s1`: 将寄存器 `s1` 的值移动到 `v3` 中。
- `v_add_co_u32_e32 v0, vcc, s0, v1`: 将 `s0` 和 `v1` 相加，进位存放在 `vcc` 中，结果存放在 `v0` 中。
- `s_barrier`: 同步所有线程，等待所有线程完成之前的任务。
- `s_waitcnt lgkmcnt(0)`: 等待指令发出前面的所有访存请求完成。
- `ds_read_b32 v2, v1`: 从数据段读取 `v1` 地址处的数据到寄存器 `v2` 中。
- `s_waitcnt lgkmcnt(0)`: 等待指令发出前面的所有访存请求完成。
- `v_addc_co_u32_e32 v1, vcc, 0, v3, vcc`: 将 `v3` 与 0 相加，进位存放在 `vcc` 中，结果存放在 `v1` 中。
- `global_store_dword v[0:1], v2, off`: 将 `v2` 中的值存储到全局内存中的 `v[0:1]` 地址偏移 `off` 处。
- `s_endpgm`: 结束程序。

```assembly
.section .rodata,#alloc
.p2align 6
.amdhsa_kernel ds_read_b32
 .amdhsa_group_segment_fixed_size 0
 .amdhsa_private_segment_fixed_size 0
 .amdhsa_kernarg_size 64
 .amdhsa_user_sgpr_private_segment_buffer 1
 .amdhsa_user_sgpr_dispatch_ptr 0
 .amdhsa_user_sgpr_queue_ptr 0
 .amdhsa_user_sgpr_kernarg_segment_ptr 1
 .amdhsa_user_sgpr_dispatch_id 0
 .amdhsa_user_sgpr_flat_scratch_init 0
 .amdhsa_user_sgpr_private_segment_size 0
 .amdhsa_system_sgpr_private_segment_wavefront_offset 0
 .amdhsa_system_sgpr_workgroup_id_x 1
 .amdhsa_system_sgpr_workgroup_id_y 0
 .amdhsa_system_sgpr_workgroup_id_z 0
 .amdhsa_system_sgpr_workgroup_info 0
 .amdhsa_system_vgpr_workitem_id 0
 .amdhsa_next_free_vgpr 4
 .amdhsa_next_free_sgpr 6
 .amdhsa_reserve_vcc 0
 .amdhsa_reserve_flat_scratch 0
 .amdhsa_reserve_xnack_mask 1
 .amdhsa_float_round_mode_32 0
 .amdhsa_float_round_mode_16_64 0
 .amdhsa_float_denorm_mode_32 3
 .amdhsa_float_denorm_mode_16_64 3
 .amdhsa_dx10_clamp 1
 .amdhsa_ieee_mode 1
 .amdhsa_fp16_overflow 0
 .amdhsa_exception_fp_ieee_invalid_op 0
 .amdhsa_exception_fp_denorm_src 0
 .amdhsa_exception_fp_ieee_div_zero 0
 .amdhsa_exception_fp_ieee_overflow 0
 .amd

hsa_exception_fp_ieee_underflow 0
 .amdhsa_exception_fp_ieee_inexact 0
 .amdhsa_exception_int_div_zero 0
.end_amdhsa_kernel
```

这部分是 `.rodata` 段，用于存放只读数据和常量。这里包含了对 `ds_read_b32` 核函数的一些元数据和参数设置，例如：

- `amdhsa_kernel ds_read_b32`: 声明 `ds_read_b32` 是一个 AMD HSA 核函数。
- 设置了一系列 AMD HSA 核函数的参数，包括工作组段（group segment）大小、私有段（private segment）大小、Kernarg（kernel arguments）大小等。
- 针对寄存器的设置，包括下一个可用的 VGPR 和 SGPR 数量，保留的 VCC（向量寄存器控制）和 Flat Scratch 寄存器数量等。
- 设置了浮点数的舍入模式、IEEE 模式、异常处理等。

```assembly
.Lfunc_end0:
 .size ds_read_b32, .Lfunc_end0-ds_read_b32
                                        ; -- End function
```

这是函数的结束标记，表示 `ds_read_b32` 函数的大小为 `.Lfunc_end0 - ds_read_b32`。

```assembly
.section .AMDGPU.csdata
; Kernel info:
; codeLenInByte = 76
; NumSgprs: 8
; NumVgprs: 4
; ScratchSize: 0
; MemoryBound: 0
; FloatMode: 192
; IeeeMode: 1
; LDSByteSize: 0 bytes/workgroup (compile time only)
; SGPRBlocks: 0
; VGPRBlocks: 0
; NumSGPRsForWavesPerEU: 8
; NumVGPRsForWavesPerEU: 4
; Occupancy: 10
; WaveLimiterHint : 1
; COMPUTE_PGM_RSRC2:SCRATCH_EN: 0
; COMPUTE_PGM_RSRC2:USER_SGPR: 6
; COMPUTE_PGM_RSRC2:TRAP_HANDLER: 0
; COMPUTE_PGM_RSRC2:TGID_X_EN: 1
; COMPUTE_PGM_RSRC2:TGID_Y_EN: 0
; COMPUTE_PGM_RSRC2:TGID_Z_EN: 0
; COMPUTE_PGM_RSRC2:TIDIG_COMP_CNT: 0
.protected _ZN17__HIP_CoordinatesI15__HIP_ThreadIdxE1xE ; @_ZN17__HIP_CoordinatesI15__HIP_ThreadIdxE1xE
.type _ZN17__HIP_CoordinatesI15__HIP_ThreadIdxE1xE,@object
.section .rodata._ZN17__HIP_CoordinatesI15__HIP_ThreadIdxE1xE,#alloc
.weak _ZN17__HIP_CoordinatesI15__HIP_ThreadIdxE1xE
_ZN17__HIP_CoordinatesI15__HIP_ThreadIdxE1xE:
 .zero 1
 .size _ZN17__HIP_CoordinatesI15__HIP_ThreadIdxE1xE, 1
```

这部分包含了 AMDGPU 的一些元数据，例如内核信息、寄存器使用情况、内存占用等信息。也包含了对一些 HIP 相关的符号和数据的定义。

总的来说，这段代码是一个 AMD GPU 架构下的 HIP 内核函数 `ds_read_b32` 的汇编代码，用于在 GPU 上执行一些数据操作，包括从内存加载数据、进行计算、存储数据等。

### VGPR

"Vector General Purpose Register"（VGP 寄存器）是指向量处理器架构中的一种寄存器类型。向量处理器是一种专门设计用于高效处理向量（数组）数据的处理器。VGP 寄存器用于存储向量操作中的数据，它们与普通的整数或浮点寄存器不同，因为它们可以同时处理多个元素。在向量处理器架构中，VGP 寄存器通常有较大的宽度，以便同时处理多个数据元素。例如，在某些向量处理器中，一个 VGP 寄存器可能能够存储一组浮点数或整数，而不仅仅是单个浮点数或整数。使用 VGP 寄存器，程序员可以编写针对整个向量的操作，而不是逐个元素地处理。这种向量化的操作可以显著提高处理器的性能，特别是在处理大量数据时，因为它们允许并行处理多个元素。

### AMD GPU LLVM

[User Guide for AMDGPU Backend](https://llvm.org/docs/AMDGPUUsage.html)

### HSAKMT

HSAKMT（HSA Kernel Mode Thunk）是一种用于支持异构系统架构（HSA）的软件层。在异构计算中，多种处理器（如CPU、GPU、DSP等）共同工作，处理各种不同类型的计算任务。为了优化这些不同处理器之间的数据交换和处理效率，HSA旨在提供统一的内存视图，即统一内存寻址（Unified Memory Addressing，UMA），这是通过HSAKMT等机制实现的。

统一内存寻址是一种使CPU和GPU（以及其他可能的处理器）可以共享相同物理内存的技术。在传统的系统中，CPU和GPU通常拥有独立的内存空间，数据必须在这些内存空间之间显式复制，这会导致性能下降和编程复杂性增加。通过实现统一内存寻址，HSA架构允许所有的处理器看到一个连续且一致的内存地址空间，从而简化了数据共享和通信。处理器可以直接访问共享内存中的数据，无需进行数据复制，这显著提高了效率和响应速度。

#### HSAKMT的角色

HSAKMT是Linux下的一种内核模式驱动程序，提供了一套接口（thunks），允许用户空间应用程序与HSA兼容硬件进行交互。这些接口支持包括但不限于内存管理、队列管理、信号量和中断管理等功能。在内存管理方面，HSAKMT处理以下关键功能：

- **内存分配和释放：** 管理统一内存的分配和释放，确保不同处理器可以高效访问。
- **内存映射：** 允许不同的处理器单元映射相同的物理内存地址，实现真正的内存共享。
- **地址转换：** 在需要时提供地址转换服务，以支持老旧设备或特殊情况下的兼容性。

总体而言，HSAKMT在实现HSA架构的统一内存寻址中扮演着核心角色，通过提供底层的内存管理和控制，使得异构计算设备能够更加高效地协同工作。这种技术在需要大量数据处理和高并行计算能力的应用场景（如大数据分析、机器学习和科学计算等）中尤为重要。

### 什么是Wavefront

[现代渲染引擎开发-GPU架构 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/406096300)
对应NVIDIA是warp，即线程束

### Barrier in GPU

在GPU（图形处理单元）中，"barrier"（屏障）通常指的是内存屏障（memory barrier）或同步屏障（synchronization barrier）。

- **内存屏障（Memory Barrier）**：GPU中的内存屏障是用来确保对内存的访问操作的顺序性和可见性。GPU中的线程通常是并行执行的，可能会导致一些线程在其他线程之前完成了对内存的写入操作。内存屏障的作用是确保这些写入操作按照程序员指定的顺序被其他线程可见，从而避免数据的不一致性。
- **同步屏障（Synchronization Barrier）**：这种屏障用于确保在执行到该屏障之前的指令已经全部执行完成，然后才能继续执行之后的指令。这对于一些需要依赖先前指令结果的指令序列是非常重要的，确保程序的正确性和可靠性。

在GPU编程中，程序员可能需要显式地使用这些屏障来控制线程的执行顺序和访问内存的可见性，以避免数据竞争和其他并发问题。

### Fence

GPU中的“fence”是一种同步机制，用来确保在不同的GPU命令或不同的处理器（例如CPU和GPU）之间维持一定的执行顺序。在图形和计算应用程序中，正确的执行顺序是非常重要的，特别是在资源被多个任务共享时。

Fence的功能：

1. **同步命令队列：** 在多个GPU命令队列中使用时，fence可以确保一个队列中的命令在另一个队列的命令开始前完成。例如，在渲染图形前，确保所有的纹理加载完成。
2. **CPU与GPU同步：** 确保CPU端的数据操作完成后，GPU才开始执行相关的图形或计算任务。或者反过来，确保GPU任务完成后，CPU才处理GPU的输出数据。
3. **资源管理：** 在使用共享资源，如内存、缓冲区或纹理时，fence帮助管理访问权限，防止数据冲突和损坏。

工作原理：

- 当GPU或CPU达到执行过程中的某个特定点（例如，提交一个重要的渲染命令后）时，它会插入一个fence。
- 这个fence将会设置一个标记或标志，表明到达了某个执行阶段。
- 其他任务（无论是在GPU还是CPU上）在继续执行前，必须等待这个fence标志被触发，表明之前的任务已经完成了必需的处理。

应用示例：在一个典型的游戏或高性能计算应用中，可能需要加载大量的纹理数据到GPU。在这些纹理被加载之前开始渲染过程，将会导致错误或未完成的渲染输出。通过在纹理加载命令后设置一个fence，然后在渲染命令前检查这个fence，开发者可以确保渲染过程只在所有纹理加载完成后开始。

总的来说，fence是维护高效、可靠和正确的数据处理与计算顺序的关键工具，在高性能计算和复杂图形处理中尤其重要。

### 什么是chiplet和die

[多Die封装：Chiplet小芯片的研究报告_Tofino (sohu.com)](https://www.sohu.com/a/358664318_314773)
[被寄予厚望的Chiplet技术 (baidu.com)](https://baijiahao.baidu.com/s?id=1765588870976378387&wfr=spider&for=pc)

### GPU架构/集群

[GPU集群网络、集群规模、集群算力 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/678455569)
[NVIDIA GPU 架构梳理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/394352476)
[NVLink版与PCIe版GPU，究竟有什么区别？ (qq.com)](https://mp.weixin.qq.com/s?__biz=MzAxOTAwNjkyOA==&mid=2447911102&idx=1&sn=ce82e7d923adc65d280c6e9fedd1279d&chksm=8fd11c01b8a69517c827589702cec629bd509528198dcbd96659c6997a537a21a598ba5f285a&scene=21#wechat_redirect)
[简单说说算力网络：什么是InfiniBand？ (qq.com)](https://mp.weixin.qq.com/s?__biz=MzAxOTAwNjkyOA==&mid=2447911141&idx=1&sn=24ead8a11fdb212ab63c3c3b40bd2abd&chksm=8fd11c5ab8a6954c1d6f43ee642eac1cad102aaa37b489e76c413a45a47a860a16658a54f126&scene=21#wechat_redirect)
[简单说说算力网络：DGX A100如何组集群？ (qq.com)](https://mp.weixin.qq.com/s?__biz=MzAxOTAwNjkyOA==&mid=2447911165&idx=1&sn=a474a6224ba569a14b0e7321e8899b42&chksm=8fd11c42b8a69554ae8eeed9d89a5e73d76f0118adc6d56d3722550bf239cbe4f9029662e9e1&cur_album_id=2922338978153005057&scene=189#wechat_redirect)
[简单说说算力网络：集群互联，选RoCE还是InfiniBand？ (qq.com)](https://mp.weixin.qq.com/s/Z_5JX67WH2jyUxkPtm7x9g)
[简单说说算力网络：128台H100如何组集群？ (qq.com)](https://mp.weixin.qq.com/s?__biz=MzAxOTAwNjkyOA==&mid=2447911175&idx=1&sn=b14c51d56695557ce6d14a21f58e577a&chksm=8fd11db8b8a694ae50dd72acf5a77dd68cbd2b081fcf52f82fdd266529903d7ac50b5a35c356&cur_album_id=2922338978153005057&scene=189#wechat_redirect)
[如何估算大模型训练所需算力？ (qq.com)](https://mp.weixin.qq.com/s?__biz=MzAxOTAwNjkyOA==&mid=2447911183&idx=1&sn=0e9dc1678ccbd4cd3b5f1a6e4c120317&chksm=8fd11db0b8a694a6ff37349d93ffe1768fb774a85e316afaebd86a1f17c8be5702fbca53a5f2&cur_album_id=2922338978153005057&scene=189#wechat_redirect)
[简单说说算力网络：英伟达最新GPU互联架构 (qq.com)](https://mp.weixin.qq.com/s?__biz=MzAxOTAwNjkyOA==&mid=2447911210&idx=1&sn=d7a72f8b09d3ce8d3660ca301aa0ffa2&chksm=8fd11d95b8a694836f93c6d6303b559d1e56f29a1cb7cbcd54f7d9149c71fe3db2a7b4808a65&cur_album_id=2922338978153005057&scene=189#wechat_redirect)
[35页PPT，了解InfiniBand和RoCE (qq.com)](https://mp.weixin.qq.com/s/mgppjGqPBQTAQxSvkSvvAw)

### RDMA

[深入浅出全面解析RDMA技术 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/687200171)
[RDMA技术详解（一）：RDMA概述 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/55142557)
[(24 封私信 / 80 条消息) 在各互联网公司中，有将 RDMA 技术用于生产环境的实例吗？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/59122163/answer/208899370)
[从DPU开始到RDMA到CUDA - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/517176491)

### SDMA

SDMA（Stream Direct Memory Access）和 RDMA（Remote Direct Memory Access）是两种数据传输技术，它们在功能和应用领域有所不同。

SDMA通常指在一个设备内部，例如GPU或特定的硬件加速器中，用于管理内存与内存之间或内存与设备之间的数据传输的技术。SDMA使得数据传输可以绕过主处理器（CPU），从而减少延迟和CPU的负载，提高整体系统效率。特点和应用：

- **高效数据管理：** SDMA允许设备直接从内存中读写数据，无需通过CPU进行中间处理。
- **多用途：** 常见于GPU、DSP（数字信号处理器）等，用于处理大量数据流，如视频处理、图像渲染等。
- **改善性能：** 减少处理器的负载，使设备能更快地处理数据。

RDMA是一种网络技术，允许网络中的一台计算机直接访问另一台计算机的内存，无需通过操作系统处理数据，从而极大地提高数据传输的速度和效率。这种方式特别适合高性能计算和大规模数据中心。特点和应用：

- **低延迟、高吞吐量：** RDMA减少了数据传输过程中的延迟，提高了网络传输的吞吐量。
- **CPU卸载：** 数据传输过程中不占用CPU资源，允许CPU处理其他更为复杂的任务。
- **应用广泛：** 在数据中心、云计算、文件存储服务以及高性能计算中有广泛应用，如实现高效的大规模集群通信。

总结来说，SDMA关注的是设备内部或设备与内存之间的高效数据流动，而RDMA关注的是网络中不同计算机之间的高效数据交换。两者都旨在提高数据处理速度，减少CPU负载，但应用的具体环境和目标不同。

NVLink 和 RDMA 都是高速数据传输技术，但它们的设计目标和应用领域有所不同。然而，它们可以互补使用，以优化数据中心、高性能计算或深度学习应用的性能。

### NVLink

NVLink 是由 NVIDIA 开发的一种高带宽、低延迟的数据传输接口，主要用于连接 GPU 与 GPU 之间，或 GPU 与 CPU 之间。NVLink 的设计目标是为了超越传统的PCI Express（PCIe）总线的带宽限制，提供更快的数据传输速率，从而加速数据密集型应用，尤其是在需要多个 GPU 协同工作的场景中。特点：

- **高带宽：** NVLink 提供比 PCIe 更高的传输带宽，大大提升了 GPU 间的数据交换速度。
- **低延迟：** 直接连接 GPU 间或 GPU 与 CPU 之间的接口减少了通信延迟。
- **多连接配置：** 支持将多个 GPU 以链式或网格形式互连，提高并行处理能力。

NVLink 与 RDMA 的联系：虽然 NVLink 主要用于连接 GPU（或 GPU 与 CPU），而 RDMA 用于优化网络中的计算机间通信，但两者可以结合使用，以提高整体系统的数据处理能力和效率：

1. **数据中心和云环境：** 在包含多个 GPU 节点的数据中心，NVLink 可用于加速节点内部的 GPU 通信，而 RDMA 可以用来优化节点之间的通信。
2. **高性能计算：** 在需要大量数据交换的高性能计算应用中，NVLink 提升同一节点内多 GPU 之间的通信速度，RDMA 则加快不同计算节点间的数据交换。

通过这种方式，两种技术互补，共同提高了处理速度和效率，特别是在深度学习、科学计算等要求高并行处理能力的场景中。这种组合允许大规模系统中的快速数据传输，同时优化资源使用，实现高效的数据处理和计算性能。

### RoCE（RDMA over Converged Ethernet）

RoCE 是在传统以太网基础上实现的 RDMA 技术。它允许在以太网环境中实施 RDMA，这意味着可以在不需要专门硬件（如 InfiniBand）的情况下，利用现有的以太网基础设施实现 RDMA 的优势。特点：

- **以太网兼容：** 使用现有的以太网技术，降低实施 RDMA 的门槛和成本。
- **版本多样性：** 包括 RoCE v1（不依赖于特定的以太网技术）和 RoCE v2（支持路由，可以在多个网络间传输数据）。
- **灵活性和普及性：** 由于基于广泛使用的以太网，RoCE 在企业和数据中心中更易于部署和扩展。

RDMA 和 RoCE 的联系与区别：

RoCE（RDMA over Converged Ethernet）和 RDMA（Remote Direct Memory Access）之间存在直接的联系，但也有一些区别。RDMA 是一种通用的技术概念，而 RoCE 是这种技术的一种具体实现方式。

- **联系：** RoCE 是 RDMA 的一种实现，提供了在以太网上实施 RDMA 的方法。它保留了 RDMA 的所有优势，如低延迟、高吞吐量和低 CPU 利用率。
- **区别：** RDMA 是一种更广泛的技术概念，可以通过多种传输方式实现，包括 InfiniBand、iWARP（Internet Wide Area RDMA Protocol）以及 RoCE。RoCE 特别是指在以太网上实现的 RDMA。

总之，RoCE 使得在传统以太网基础上实施 RDMA 成为可能，这样不仅可以利用现有的网络基础设施，还能享受 RDMA 技术带来的低延迟和高效率优势。这种技术特别适合那些希望在不更换现有网络硬件的情况下，提升网络性能的应用场景。

## 开发

### 什么是内存地址对齐

DW aligned，最低两位为0，四字节对齐。QW aligned，最低三位为0，八字节（四字）对齐。为什么需要这样的对齐：

1. **硬件效率**：很多计算机架构设计中，CPU从内存中读取数据最为高效的方式是从某个固定边界开始。如果数据存储在对齐的地址上，CPU可以在一个或少数几个内存访问周期内读取完整的数据块，否则可能需要多次访问，增加延迟。
2. **防止硬件异常**：在某些硬件平台上，如x86和x64架构，如果尝试从非对齐的地址加载或存储Dword数据，可能导致硬件异常，如对齐检查异常（Alignment Check exception）。这种异常处理会降低程序的执行效率。
3. **简化内存管理**：对齐内存地址简化了内存的管理。系统不必考虑跨越多个内存页或缓存行来处理单个数据单位，降低了处理的复杂性和潜在的性能问题。

因此，对于32位数据（Dword），最低两位为0的地址对齐要求确保了数据处理的效率和程序的稳定运行。在编程和系统设计时，通常会采用专门的内存分配函数来确保所分配的内存满足对齐要求，例如在C语言中使用`aligned_alloc`函数或在其他高级语言中使用类似机制。

### 类静态定义

```c
#include <iostream>

using namespace std;

struct test {
    int a;
    int b;
    static int c;
};

/* should declare the static first */
int test::c = 3;

int main() {
    printf("sizeof(test) = %d\n", sizeof(struct test));

    struct test t1;
    t1.a = 1;
    t1.b = 2;
    printf("sizeof(t) = %d\n", sizeof(t1));
    printf("t1.c = %d\n", t1.c);

    struct test t2;
    printf("t2.c = %d\n", t2.c);

    t2.c = 4;
    printf("t1.c = %d\n", t1.c);

    // undefined reference to `test::c'
    // t.c = 3;

    return 0;
}

sizeof(test) = 8
sizeof(t) = 8
t1.c = 3
t2.c = 3
t1.c = 4
```

## 驱动

### AMD GPU是怎么创建queue的

<details>
<summary>代码</summary>

```c
// ----------------------------------------------------------------------------------------------------------------------------------
//                            kfdtest
// ----------------------------------------------------------------------------------------------------------------------------------
class BaseQueue
--> HSAKMT_STATUS Create(unsigned int NodeId, unsigned int size = DEFAULT_QUEUE_SIZE, HSAuint64 *pointers = NULL);
 --> memset(&m_Resources, 0, sizeof(m_Resources));
 --> hsaKmtCreateQueue(NodeId, type, DEFAULT_QUEUE_PERCENTAGE, DEFAULT_PRIORITY, m_QueueBuf->As<unsigned int*>(), m_QueueBuf->Size(), NULL, &m_Resources);
// ----------------------------------------------------------------------------------------------------------------------------------
//                            Thunk
// ----------------------------------------------------------------------------------------------------------------------------------
  --> struct kfd_ioctl_create_queue_args args = {0};
  --> handle_concrete_asic(q, &args, NodeId, Event, QueueResource->ErrorReason);
  --> args.read_pointer_address = QueueResource->QueueRptrValue;
   args.write_pointer_address = QueueResource->QueueWptrValue;
   args.ring_base_address = (uintptr_t)QueueAddress;
   args.ring_size = QueueSizeInBytes;
   args.queue_percentage = QueuePercentage;
   args.queue_priority = priority_map[Priority+3];
  --> err = kmtIoctl(kfd_fd, AMDKFD_IOC_CREATE_QUEUE, &args);
// ----------------------------------------------------------------------------------------------------------------------------------
//                            Driver
// ----------------------------------------------------------------------------------------------------------------------------------
   --> static int kfd_ioctl_create_queue(struct file *filep, struct kfd_process *p, void *data)
    --> set_queue_properties_from_user(&q_properties, args);
     --> q_properties->is_interop = false;
      q_properties->is_gws = false;
      q_properties->queue_percent = args->queue_percentage;
      q_properties->priority = args->queue_priority;
      q_properties->queue_address = args->ring_base_address;
      q_properties->queue_size = args->ring_size;
      q_properties->read_ptr = (uint32_t *) args->read_pointer_address;
      q_properties->write_ptr = (uint32_t *) args->write_pointer_address;
      q_properties->eop_ring_buffer_address = args->eop_buffer_address;
      q_properties->eop_ring_buffer_size = args->eop_buffer_size;
      q_properties->ctx_save_restore_area_address = args->ctx_save_restore_address;
      q_properties->ctx_save_restore_area_size = args->ctx_save_restore_size;
      q_properties->ctl_stack_size = args->ctl_stack_size;
      //...
    --> kfd_process_device_data_by_id()
    --> kfd_bind_process_to_device()
    --> pqm_create_queue(&p->pqm, dev, filep, &q_properties, &queue_id, NULL, NULL, NULL, &doorbell_offset_in_process);
     --> struct kfd_process_device *pdd = kfd_get_process_device_data()
      // PM4Queue.hpp: Type is  HSA_QUEUE_COMPUTE --> KFD_IOC_QUEUE_TYPE_COMPUTE --> KFD_QUEUE_TYPE_COMPUTE
      // SDMAQueue.hpp: Type is HSA_QUEUE_SDMA    --> KFD_IOC_QUEUE_TYPE_SDMA    --> KFD_QUEUE_TYPE_SDMA
     --> init_user_queue(pqm, dev, &q, properties, f, *qid);
     --> kfd_process_drain_interrupts(pdd);
     --> retval = dev->dqm->ops.create_queue(dev->dqm, q, &pdd->qpd, q_data, restore_mqd, restore_ctl_stack);
         // dqm->ops.create_queue = create_queue_cpsch;
      --> create_queue_cpsch()
       --> allocate_doorbell()
       --> mqd_mgr->init_mqd(mqd_mgr, &q->mqd, q->mqd_mem_obj, &q->gart_mqd_addr, &q->properties);
       --> list_add(&q->list, &qpd->queues_list);
       --> increment_queue_count(dqm, qpd, q);
       --> execute_queues_cpsch(dqm, KFD_UNMAP_QUEUES_FILTER_DYNAMIC_QUEUES, 0, USE_DEFAULT_GRACE_PERIOD);
        --> retval = unmap_queues_cpsch(dqm, filter, filter_param, grace_period, false);
         --> retval = pm_send_unmap_queue(&dqm->packet_mgr, filter, filter_param, reset);
         --> pm_send_query_status(&dqm->packet_mgr, dqm->fence_gpu_addr, KFD_FENCE_COMPLETED);
         --> retval = amdkfd_fence_wait_timeout(dqm->fence_addr, KFD_FENCE_COMPLETED, queue_preemption_timeout_ms);
         --> pm_release_ib(&dqm->packet_mgr);
        --> map_queues_cpsch(dqm);
         --> pm_send_runlist(&dqm->packet_mgr, &dqm->queues);
          --> retval = pm_create_runlist_ib(pm, dqm_queues, &rl_gpu_ib_addr, &rl_ib_size);
          --> retval = kq_acquire_packet_buffer(pm->priv_queue, ket_size_dwords, &rl_buffer);
          --> retval = pm->pmf->runlist(pm, rl_buffer, rl_gpu_ib_addr, rl_ib_size / sizeof(uint32_t), false);
           --> pm_runlist_v9()
          --> kq_submit_packet(pm->priv_queue);
           --> write_kernel_doorbell()
            --> writel(value, db);
             --> __io_bw();
              __raw_writel((u32 __force)__cpu_to_le32(value), addr);
              __io_aw();
         --> dqm->active_runlist = true;
       --> deallocate_doorbell(qpd, q);


/** Ioctl table */
static const struct amdkfd_ioctl_desc amdkfd_ioctls[] = {
 //...
 AMDKFD_IOCTL_DEF(AMDKFD_IOC_CREATE_QUEUE,
   kfd_ioctl_create_queue, 0),
 //...
}

amdgpu_amdkfd_device_init()
--> adev->kfd.init_complete = kgd2kfd_device_init(adev->kfd.dev, adev_to_drm(adev), &gpu_resources);
 --> kfd->dqm = device_queue_manager_init(kfd);
  --> case KFD_SCHED_POLICY_HWS_NO_OVERSUBSCRIPTION:
    /* initialize dqm for cp scheduling */
    dqm->ops.create_queue = create_queue_cpsch;
    dqm->ops.initialize = initialize_cpsch;
    dqm->ops.start = start_cpsch;
    dqm->ops.stop = stop_cpsch;
    dqm->ops.pre_reset = pre_reset;
    dqm->ops.destroy_queue = destroy_queue_cpsch;
    dqm->ops.update_queue = update_queue;
    dqm->ops.register_process = register_process;
    dqm->ops.unregister_process = unregister_process;
    dqm->ops.uninitialize = uninitialize;
    dqm->ops.create_kernel_queue = create_kernel_queue_cpsch;
    dqm->ops.destroy_kernel_queue = destroy_kernel_queue_cpsch;
    dqm->ops.set_cache_memory_policy = set_cache_memory_policy;
    dqm->ops.process_termination = process_termination_cpsch;
    dqm->ops.evict_process_queues = evict_process_queues_cpsch;
    dqm->ops.restore_process_queues = restore_process_queues_cpsch;
    dqm->ops.get_wave_state = get_wave_state;
    dqm->ops.reset_queues = reset_queues_cpsch;
    dqm->ops.get_queue_checkpoint_info = get_queue_checkpoint_info;
    dqm->ops.checkpoint_mqd = checkpoint_mqd;
    break;
 --> kfd_resume()
  --> kfd->dqm->ops.start(kfd->dqm);
   --> retval = pm_init(&dqm->packet_mgr, dqm);
    --> pm->priv_queue = kernel_queue_init(dqm->dev, KFD_QUEUE_TYPE_HIQ);
     --> kq_initialize(kq, dev, type, KFD_KERNEL_QUEUE_SIZE)
      --> kq->mqd_mgr->init_mqd(kq->mqd_mgr, &kq->queue->mqd,
             kq->queue->mqd_mem_obj,
             &kq->queue->gart_mqd_addr,
             &kq->queue->properties);
  --> set_sched_resources(dqm)
   --> return pm_send_set_resources(&dqm->packet_mgr, &res);
    --> retval = pm->pmf->set_resources(pm, buffer, res);
     --> pm_set_resources_v9()


static int pm_map_process_v9(struct packet_manager *pm,
  uint32_t *buffer, struct qcm_process_device *qpd)
{
 struct pm4_mes_map_process *packet;
 uint64_t vm_page_table_base_addr = qpd->page_table_base;
 struct kfd_dev *kfd = pm->dqm->dev;
 struct kfd_process_device *pdd =
   container_of(qpd, struct kfd_process_device, qpd);

 packet = (struct pm4_mes_map_process *)buffer;
 memset(buffer, 0, sizeof(struct pm4_mes_map_process));
 packet->header.u32All = pm_build_pm4_header(IT_MAP_PROCESS,
     sizeof(struct pm4_mes_map_process));
 packet->bitfields2.diq_enable = (qpd->is_debug) ? 1 : 0;
 packet->bitfields2.process_quantum = 10;
 packet->bitfields2.pasid = qpd->pqm->process->pasid;
 packet->bitfields14.gds_size = qpd->gds_size & 0x3F;
 packet->bitfields14.gds_size_hi = (qpd->gds_size >> 6) & 0xF;
 packet->bitfields14.num_gws = (qpd->mapped_gws_queue) ? qpd->num_gws : 0;
 packet->bitfields14.num_oac = qpd->num_oac;
 packet->bitfields14.sdma_enable = 1;
 packet->bitfields14.num_queues = (qpd->is_debug) ? 0 : qpd->queue_count;

 if (kfd->dqm->trap_debug_vmid && pdd->process->debug_trap_enabled &&
   pdd->process->runtime_info.runtime_state == DEBUG_RUNTIME_STATE_ENABLED) {
  packet->bitfields2.debug_vmid = kfd->dqm->trap_debug_vmid;
  packet->bitfields2.new_debug = 1;
 }

 packet->sh_mem_config = qpd->sh_mem_config;
 packet->sh_mem_bases = qpd->sh_mem_bases;
 if (qpd->tba_addr) {
  packet->sq_shader_tba_lo = lower_32_bits(qpd->tba_addr >> 8);
  /* On GFX9, unlike GFX10, bit TRAP_EN of SQ_SHADER_TBA_HI is
   * not defined, so setting it won't do any harm.
   */
  packet->sq_shader_tba_hi = upper_32_bits(qpd->tba_addr >> 8)
    | 1 << SQ_SHADER_TBA_HI__TRAP_EN__SHIFT;

  packet->sq_shader_tma_lo = lower_32_bits(qpd->tma_addr >> 8);
  packet->sq_shader_tma_hi = upper_32_bits(qpd->tma_addr >> 8);
 }

 packet->gds_addr_lo = lower_32_bits(qpd->gds_context_area);
 packet->gds_addr_hi = upper_32_bits(qpd->gds_context_area);

 packet->vm_context_page_table_base_addr_lo32 =
   lower_32_bits(vm_page_table_base_addr);
 packet->vm_context_page_table_base_addr_hi32 =
   upper_32_bits(vm_page_table_base_addr);

 return 0;
}

static int pm_runlist_v9(struct packet_manager *pm, uint32_t *buffer,
   uint64_t ib, size_t ib_size_in_dwords, bool chain)
{
 struct pm4_mes_runlist *packet;

 int concurrent_proc_cnt = 0;
 struct kfd_dev *kfd = pm->dqm->dev;

 /* Determine the number of processes to map together to HW:
  * it can not exceed the number of VMIDs available to the
  * scheduler, and it is determined by the smaller of the number
  * of processes in the runlist and kfd module parameter
  * hws_max_conc_proc.
  * Note: the arbitration between the number of VMIDs and
  * hws_max_conc_proc has been done in
  * kgd2kfd_device_init().
  */
 concurrent_proc_cnt = min(pm->dqm->processes_count,
   kfd->max_proc_per_quantum);

 packet = (struct pm4_mes_runlist *)buffer;

 memset(buffer, 0, sizeof(struct pm4_mes_runlist));
 packet->header.u32All = pm_build_pm4_header(IT_RUN_LIST,
      sizeof(struct pm4_mes_runlist));

 packet->bitfields4.ib_size = ib_size_in_dwords;
 packet->bitfields4.chain = chain ? 1 : 0;
 packet->bitfields4.offload_polling = 0;
 packet->bitfields4.chained_runlist_idle_disable = chain ? 1 : 0;
 packet->bitfields4.valid = 1;
 packet->bitfields4.process_cnt = concurrent_proc_cnt;
 packet->ordinal2 = lower_32_bits(ib);
 packet->ib_base_hi = upper_32_bits(ib);

 return 0;
}

static int pm_set_resources_v9(struct packet_manager *pm, uint32_t *buffer,
    struct scheduling_resources *res)
{
 struct pm4_mes_set_resources *packet;

 packet = (struct pm4_mes_set_resources *)buffer;
 memset(buffer, 0, sizeof(struct pm4_mes_set_resources));

 packet->header.u32All = pm_build_pm4_header(IT_SET_RESOURCES,
     sizeof(struct pm4_mes_set_resources));

 packet->bitfields2.queue_type =
   queue_type__mes_set_resources__hsa_interface_queue_hiq;
 packet->bitfields2.vmid_mask = res->vmid_mask;
 packet->bitfields2.unmap_latency = KFD_UNMAP_LATENCY_MS / 100;
 packet->bitfields7.oac_mask = res->oac_mask;
 packet->bitfields8.gds_heap_base = res->gds_heap_base;
 packet->bitfields8.gds_heap_size = res->gds_heap_size;

 packet->gws_mask_lo = lower_32_bits(res->gws_mask);
 packet->gws_mask_hi = upper_32_bits(res->gws_mask);

 packet->queue_mask_lo = lower_32_bits(res->queue_mask);
 packet->queue_mask_hi = upper_32_bits(res->queue_mask);

 return 0;
}

static int pm_map_queues_v9(struct packet_manager *pm, uint32_t *buffer,
  struct queue *q, bool is_static)
{
 struct pm4_mes_map_queues *packet;
 bool use_static = is_static;

 packet = (struct pm4_mes_map_queues *)buffer;
 memset(buffer, 0, sizeof(struct pm4_mes_map_queues));

 packet->header.u32All = pm_build_pm4_header(IT_MAP_QUEUES,
     sizeof(struct pm4_mes_map_queues));
 packet->bitfields2.num_queues = 1;
 packet->bitfields2.queue_sel =
  queue_sel__mes_map_queues__map_to_hws_determined_queue_slots_vi;

 packet->bitfields2.engine_sel =
  engine_sel__mes_map_queues__compute_vi;
 packet->bitfields2.gws_control_queue = q->gws ? 1 : 0;
 packet->bitfields2.extended_engine_sel =
  extended_engine_sel__mes_map_queues__legacy_engine_sel;
 packet->bitfields2.queue_type =
  queue_type__mes_map_queues__normal_compute_vi;

 switch (q->properties.type) {
 case KFD_QUEUE_TYPE_COMPUTE:
  if (use_static)
   packet->bitfields2.queue_type =
  queue_type__mes_map_queues__normal_latency_static_queue_vi;
  break;
 case KFD_QUEUE_TYPE_DIQ:
  packet->bitfields2.queue_type =
   queue_type__mes_map_queues__debug_interface_queue_vi;
  break;
 case KFD_QUEUE_TYPE_SDMA:
 case KFD_QUEUE_TYPE_SDMA_XGMI:
  use_static = false; /* no static queues under SDMA */
  if (q->properties.sdma_engine_id < 2 && !pm_use_ext_eng(q->device))
   packet->bitfields2.engine_sel = q->properties.sdma_engine_id +
    engine_sel__mes_map_queues__sdma0_vi;
  else {
   packet->bitfields2.extended_engine_sel =
    extended_engine_sel__mes_map_queues__sdma0_to_7_sel;
   packet->bitfields2.engine_sel = q->properties.sdma_engine_id;
  }
  break;
 default:
  WARN(1, "queue type %d", q->properties.type);
  return -EINVAL;
 }
 packet->bitfields3.doorbell_offset =
   q->properties.doorbell_off;

 packet->mqd_addr_lo =
   lower_32_bits(q->gart_mqd_addr);

 packet->mqd_addr_hi =
   upper_32_bits(q->gart_mqd_addr);

 packet->wptr_addr_lo =
   lower_32_bits((uint64_t)q->properties.write_ptr);

 packet->wptr_addr_hi =
   upper_32_bits((uint64_t)q->properties.write_ptr);

 return 0;
}

static int pm_unmap_queues_v9(struct packet_manager *pm, uint32_t *buffer,
   enum kfd_unmap_queues_filter filter,
   uint32_t filter_param, bool reset)
{
 struct pm4_mes_unmap_queues *packet;

 packet = (struct pm4_mes_unmap_queues *)buffer;
 memset(buffer, 0, sizeof(struct pm4_mes_unmap_queues));

 packet->header.u32All = pm_build_pm4_header(IT_UNMAP_QUEUES,
     sizeof(struct pm4_mes_unmap_queues));

 packet->bitfields2.extended_engine_sel = pm_use_ext_eng(pm->dqm->dev) ?
  extended_engine_sel__mes_unmap_queues__sdma0_to_7_sel :
  extended_engine_sel__mes_unmap_queues__legacy_engine_sel;

 packet->bitfields2.engine_sel =
  engine_sel__mes_unmap_queues__compute;

 if (reset)
  packet->bitfields2.action =
   action__mes_unmap_queues__reset_queues;
 else
  packet->bitfields2.action =
   action__mes_unmap_queues__preempt_queues;

 switch (filter) {
 case KFD_UNMAP_QUEUES_FILTER_BY_PASID:
  packet->bitfields2.queue_sel =
   queue_sel__mes_unmap_queues__perform_request_on_pasid_queues;
  packet->bitfields3a.pasid = filter_param;
  break;
 case KFD_UNMAP_QUEUES_FILTER_ALL_QUEUES:
  packet->bitfields2.queue_sel =
   queue_sel__mes_unmap_queues__unmap_all_queues;
  break;
 case KFD_UNMAP_QUEUES_FILTER_DYNAMIC_QUEUES:
  /* in this case, we do not preempt static queues */
  packet->bitfields2.queue_sel =
   queue_sel__mes_unmap_queues__unmap_all_non_static_queues;
  break;
 default:
  WARN(1, "filter %d", filter);
  return -EINVAL;
 }

 return 0;
}

static int pm_query_status_v9(struct packet_manager *pm, uint32_t *buffer,
   uint64_t fence_address, uint64_t fence_value)
{
 struct pm4_mes_query_status *packet;

 packet = (struct pm4_mes_query_status *)buffer;
 memset(buffer, 0, sizeof(struct pm4_mes_query_status));


 packet->header.u32All = pm_build_pm4_header(IT_QUERY_STATUS,
     sizeof(struct pm4_mes_query_status));

 packet->bitfields2.context_id = 0;
 packet->bitfields2.interrupt_sel =
   interrupt_sel__mes_query_status__completion_status;
 packet->bitfields2.command =
   command__mes_query_status__fence_only_after_write_ack;

 packet->addr_hi = upper_32_bits((uint64_t)fence_address);
 packet->addr_lo = lower_32_bits((uint64_t)fence_address);
 packet->data_hi = upper_32_bits((uint64_t)fence_value);
 packet->data_lo = lower_32_bits((uint64_t)fence_value);

 return 0;
}

```

</details>  
