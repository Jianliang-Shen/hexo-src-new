---
title: PCI设备命令
date: 2024-05-27 19:11:42
index_img: /img/gpu/pcie_index.png
tags:
    - Linux
    - Driver
categories:
    - Software
---

分享：读取GPU设备信息的常用指令

<!-- more -->

## lspci

- -v,--verbose选项开启后，将提供更为详尽的信息，其中涵盖了每个设备的内存地址区间以及相应的IRQ信息。 
- -nn：揭示设备的硬件标识符（供应商ID与设备ID），助您辨识未知设备。 
- -s<槽位号>:仅显示指定槽位号对应的设备信息。
- -t:以树状结构显示PCI设备之间的连接关系。
- -k:显示每个设备所使用的内核模块名称。
- -xxx:显示更加详细的寄存器级别信息。

```bash
lspci -t
-[0000:00]-+-00.0
           +-01.0-[01-03]----00.0-[02-03]----00.0-[03]--+-00.0
           |                                            \-00.1
           +-02.0
           +-14.0
           +-16.0
           +-17.0
           +-1b.0-[04]--
           +-1c.0-[05]--
           +-1c.2-[06]----00.0
           +-1c.7-[07]----00.0
           +-1d.0-[08]--
           +-1f.0
           +-1f.2
           +-1f.3
           +-1f.4
           \-1f.6

lspci -nn
03:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Vega 20 [Radeon VII] [1002:66af] (rev c1)
03:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Vega 20 HDMI Audio [Radeon VII] [1002:ab20]

# 显示设备上所以pcie设备的vendor id 和device id
lspci -n -s 03:00.0
03:00.0 0300: 1002:66af (rev c1)

# 显示相当详细的信息，所有能够解析出来的pcie信息都会显示出来
lspci -vvv -s 03:00.0
# 或者
lspci -vvv -d 1002:66af
03:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Vega 20 [Radeon VII] (rev c1) (prog-if 00 [VGA controller])
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Vega 20 [Radeon VII]
        Control: I/O+ Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 0, Cache Line Size: 64 bytes
        Interrupt: pin A routed to IRQ 144
        Region 0: Memory at 2400000000 (64-bit, prefetchable) [size=16G]
        Region 2: Memory at 2200000000 (64-bit, prefetchable) [size=2M]
        Region 4: I/O ports at e000 [size=256]
        Region 5: Memory at f7b00000 (32-bit, non-prefetchable) [size=512K]
        Expansion ROM at f7b80000 [disabled] [size=128K]
        Capabilities: <access denied>
        Kernel driver in use: amdgpu
        Kernel modules: amdgpu

lspci -k -s 03:00.0
03:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Vega 20 [Radeon VII] (rev c1)
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Vega 20 [Radeon VII]
        Kernel driver in use: amdgpu
        Kernel modules: amdgpu

# 显示设备上所有pcie设备的配置空间的标准部分（前 64 字节或 CardBus 桥接器的 128 字节）
lspci -x -s 03:00.0
03:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Vega 20 [Radeon VII] (rev c1)
00: 02 10 af 66 07 04 10 00 c1 00 00 03 10 00 80 00
10: 0c 00 00 00 24 00 00 00 0c 00 00 00 22 00 00 00
20: 01 e0 00 00 00 00 b0 f7 00 00 00 00 02 10 1e 08
30: 00 00 b8 f7 48 00 00 00 00 00 00 00 0b 01 00 00

# 显示更加详细的寄存器级别信息
lspci -xxx -s 03:00.0
03:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Vega 20 [Radeon VII] (rev c1)
00: 02 10 af 66 07 04 10 00 c1 00 00 03 10 00 80 00
10: 0c 00 00 00 24 00 00 00 0c 00 00 00 22 00 00 00
20: 01 e0 00 00 00 00 b0 f7 00 00 00 00 02 10 1e 08
30: 00 00 b8 f7 48 00 00 00 00 00 00 00 0b 01 00 00
```

在运用lspci指令得出的结果中，含有诸多复杂字段，故需具备相关经验与知识方能准确理解。 

- 设备编号：每个PCI设备都有唯一编号。 
- 型式：标注此装置归属的类别linux lspci命令详解，例如VGA控制器、网络适配器、音频接口等等。
- 厂商：制造商名称。 
- 设备：具体产品型号名称。 
- 描述：对设备功能的简要描述。 

透彻剖析与解读以上字段后，可迅速查找到所需要的PCI设备相关信息及做出相应对策。


## devmem2

直接访问PCI设备地址空间

```bash
sudo devmem2 0xf7b00000
/dev/mem opened.
Memory mapped at address 0x7fc181384000.
Value at address 0xF7B00000 (0x7fc181384000): 0x54

sudo cat /proc/iomem
00000000-00000fff : Reserved
00001000-0009c3ff : System RAM
0009c400-0009ffff : Reserved
000a0000-000bffff : PCI Bus 0000:00
000c0000-000cffff : Video ROM
000e0000-000fffff : Reserved
  000f0000-000fffff : System ROM
00100000-a2ce0fff : System RAM
a2ce1000-a2d1afff : ACPI Tables
a2d1b000-a30bdfff : System RAM
a30be000-a30befff : ACPI Non-volatile Storage
a30bf000-a30bffff : Reserved
a30c0000-afe2ffff : System RAM
afe30000-b16ebfff : Reserved
b16ec000-b1700fff : ACPI Tables
b1701000-b17fefff : System RAM
b17ff000-b1b26fff : ACPI Non-volatile Storage
b1b27000-b2ffefff : Reserved
b2fff000-b2ffffff : System RAM
b3000000-b7ffffff : Reserved
  b4000000-b7ffffff : Graphics Stolen Memory
b8000000-f7ffffff : PCI Bus 0000:00
  b8000000-b81fffff : PCI Bus 0000:08
  d0000000-dfffffff : 0000:00:02.0
  f7b00000-f7cfffff : PCI Bus 0000:01
    f7b00000-f7bfffff : PCI Bus 0000:02
      f7b00000-f7bfffff : PCI Bus 0000:03
        f7b00000-f7b7ffff : 0000:03:00.0   # GPU设备物理地址空间
        f7b80000-f7b9ffff : 0000:03:00.0
        f7ba0000-f7ba3fff : 0000:03:00.1
...
```

## 查看内存信息

```txt
sudo cat /proc/meminfo
MemTotal:       32738388 kB
MemFree:        19459800 kB
MemAvailable:   30920932 kB
Buffers:         1230120 kB
Cached:         10143732 kB
SwapCached:          280 kB
Active:          7116292 kB
Inactive:        5155544 kB
Active(anon):     804940 kB
Inactive(anon):    97316 kB
Active(file):    6311352 kB
Inactive(file):  5058228 kB
Unevictable:         260 kB
Mlocked:               0 kB
SwapTotal:       2097148 kB
SwapFree:        2096368 kB
Dirty:               588 kB
Writeback:             0 kB
AnonPages:        898004 kB
Mapped:           421580 kB
Shmem:              4320 kB
KReclaimable:     557620 kB
Slab:             879912 kB
SReclaimable:     557620 kB
SUnreclaim:       322292 kB
KernelStack:       11456 kB
PageTables:        17284 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    18466340 kB
Committed_AS:    3800676 kB
VmallocTotal:   34359738367 kB
VmallocUsed:       37656 kB
VmallocChunk:          0 kB
Percpu:            19520 kB
HardwareCorrupted:     0 kB
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
FilePmdMapped:         0 kB
CmaTotal:              0 kB
CmaFree:               0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:      629708 kB
DirectMap2M:    15998976 kB
DirectMap1G:    17825792 kB
```

## 什么是PCIe的BAR？

**BAR（Base Address Register，基址寄存器）** 是PCI Express（PCIe）设备的重要组成部分，用于指定设备在系统内存地址空间或I/O地址空间中的基址。每个PCIe设备可以有多个BAR，每个BAR定义了设备的一部分内存或I/O空间。主机（Host）使用这些地址与设备进行通信。

### PCIe BAR的功能

1. **地址映射**：
   - BAR用于映射设备的内存或I/O空间到系统地址空间，使主机能够访问设备的寄存器和内存。
   
2. **资源分配**：
   - 在系统启动时，操作系统或BIOS会扫描PCIe设备，读取BAR寄存器，并为每个BAR分配一个唯一的地址范围。
   
3. **设备通信**：
   - 主机通过读取和写入BAR映射的地址空间，与PCIe设备进行通信，控制设备的操作和传输数据。

### 不同的BAR类型及其差异

每个PCIe设备可以有多个BAR，每个BAR的类型和用途可能有所不同。以下是常见的BAR类型及其差异：

#### 1. 内存空间BAR（Memory Space BAR）

**内存空间BAR** 映射设备的内存空间，使主机能够访问设备的寄存器和内存缓冲区。

- **32位内存空间BAR**：
  - 地址范围：0x00000000到0xFFFFFFFF。
  - 用于映射设备的小容量内存空间。
  
- **64位内存空间BAR**：
  - 地址范围：64位地址空间。
  - 由两个连续的BAR寄存器组成（低32位和高32位），用于映射设备的大容量内存空间。

**特点**：
  - 适用于需要高带宽访问的大容量存储区域，如图形显存、网络缓冲区等。
  - 支持内存映射I/O（MMIO），主机通过直接内存访问（DMA）高效读取和写入数据。

#### 2. I/O空间BAR（I/O Space BAR）

**I/O空间BAR** 映射设备的I/O端口，使主机能够访问设备的I/O寄存器。

- **I/O地址范围**：0x0000到0xFFFF（16位地址空间）。

**特点**：
  - 适用于简单的设备寄存器访问，如控制寄存器、状态寄存器。
  - 通常用于低带宽设备，如串口、并口、键盘控制器等。

### BAR的分配和使用

#### 1. 系统启动和资源分配

- 在系统启动时，BIOS或操作系统扫描PCIe总线上的设备，读取每个设备的配置空间，找到BAR寄存器。
- 根据设备的需求，BIOS或操作系统为每个BAR分配一个唯一的地址范围，并将分配结果写回BAR寄存器。
- 这使得每个设备在系统地址空间中有一个固定的映射地址，主机可以通过这些地址与设备通信。

#### 2. 主机与设备通信

- 主机通过读取和写入BAR映射的地址空间，与设备进行数据交换。
- 内存空间BAR通常用于高速数据传输，通过DMA操作提高效率。
- I/O空间BAR用于低速控制和状态寄存器访问，通过I/O指令进行读写。

### 示例

假设有一个PCIe设备具有两个BAR：

1. **BAR0（32位内存空间BAR）**：
   - 地址范围：0x80000000到0x8000FFFF（64KB）。
   - 用于映射设备的控制寄存器和状态寄存器。
   
2. **BAR1（64位内存空间BAR）**：
   - 地址范围：0x900000000000到0x9000000FFFFF（1MB）。
   - 用于映射设备的大容量数据缓冲区。

在系统启动时，操作系统扫描PCIe设备，并为这两个BAR分配地址。主机可以通过这些地址与设备进行通信。例如，主机可以通过读取和写入0x80000000到0x8000FFFF的地址范围来配置设备的寄存器，或通过0x900000000000到0x9000000FFFFF的地址范围来传输数据。

### 总结

- **BAR（Base Address Register）** 是PCIe设备用于定义其内存或I/O空间基址的寄存器。
- **内存空间BAR** 用于映射设备的内存区域，适合高速数据传输。
- **I/O空间BAR** 用于映射设备的I/O端口，适合低速控制和状态访问。
- BIOS或操作系统在系统启动时为每个BAR分配唯一的地址范围，确保主机能够与设备进行有效通信。

通过理解BAR的类型和作用，可以更好地设计和调试PCIe设备的通信和数据传输。

## 什么是NBIF？

**NBIF（Northbridge Interconnect Fabric）** 是一种用于连接和管理CPU、GPU、内存和其他关键系统组件的互连架构。NBIF通常用于高性能计算系统、服务器和图形处理单元（GPU）中，以提供高带宽、低延迟的数据传输通道。以下是关于NBIF的详细解释：

### 1. NBIF的定义和作用

**NBIF（Northbridge Interconnect Fabric）** 是一种先进的互连架构，负责在系统的不同处理单元（如CPU、GPU）和内存之间进行高效的数据传输和通信。它作为北桥的一部分或替代北桥功能，提供了连接CPU和其他系统组件的高性能数据路径。

### 2. NBIF的主要功能

1. **高带宽数据传输**：
   - NBIF提供高带宽的数据通道，支持大量数据在CPU、GPU和内存之间的快速传输。
   - 这种高带宽能力对于处理复杂的计算任务和高分辨率图形渲染至关重要。

2. **低延迟通信**：
   - NBIF设计为低延迟架构，确保数据在不同系统组件之间快速传输，减少通信延迟。
   - 低延迟通信提高了系统的整体响应速度和性能。

3. **多设备互连**：
   - NBIF支持连接多个CPU、GPU、内存模块和其他外设，形成一个高效的互连网络。
   - 这种多设备互连能力使得系统能够扩展和集成更多的处理单元，提高计算和处理能力。

4. **内存一致性**：
   - NBIF维护系统内存的一致性，确保所有处理单元访问的内存数据是最新的。
   - 内存一致性机制对于多核处理和并行计算环境尤为重要。

5. **数据路由和调度**：
   - NBIF智能路由和调度数据传输，优化数据流，避免瓶颈和冲突。
   - 这种智能路由功能提高了数据传输的效率和可靠性。

### 3. NBIF在GPU中的应用

在现代GPU架构中，NBIF用于连接GPU与CPU、内存和其他外设，提供高效的数据传输和通信通道。以下是具体的应用场景：

1. **CPU-GPU通信**：
   - NBIF提供高带宽、低延迟的通道，用于CPU与GPU之间的数据交换。
   - 这种高效通信对于图形渲染、科学计算和机器学习任务尤为关键。

2. **内存访问**：
   - NBIF管理GPU对系统内存的访问，确保GPU能够快速读取和写入内存数据。
   - 高效的内存访问提高了GPU处理大规模数据集的能力。

3. **多GPU互连**：
   - 在多GPU系统中，NBIF提供多个GPU之间的高效互连，支持并行计算和任务分配。
   - 多GPU互连提高了系统的计算能力和处理速度。

### 4. NBIF的优势

1. **高性能**：
   - NBIF的高带宽和低延迟特性使得系统能够处理复杂的计算和图形任务，提高整体性能。

2. **扩展性**：
   - NBIF支持多设备互连，系统可以轻松扩展，增加更多的处理单元和内存资源。

3. **可靠性**：
   - 智能路由和调度功能确保数据传输的可靠性和效率，减少数据传输中的瓶颈和冲突。

### 总结

**NBIF（Northbridge Interconnect Fabric）** 是一种高性能的互连架构，负责连接和管理CPU、GPU、内存及其他关键系统组件。它提供高带宽、低延迟的数据传输通道，支持多设备互连，确保系统内存的一致性，并优化数据传输的路由和调度。NBIF在高性能计算、图形处理和并行计算环境中发挥着重要作用，显著提高了系统的整体性能和扩展性。

通过了解NBIF的功能和应用，可以更好地理解现代计算系统和GPU架构的高效数据管理和通信机制。


## 北桥和南桥

在传统的PC架构中，**北桥（Northbridge）** 和 **南桥（Southbridge）** 是两个重要的芯片，它们负责连接和管理不同的系统组件。这两个桥在计算机的主板上分别处理不同类型的数据传输和通信。以下是对北桥和南桥的详细解释：

### 1. 北桥（Northbridge）

**北桥** 是主板上的一个关键芯片，负责连接CPU、内存和高速图形接口（如AGP或PCIe）。它主要处理高速和高带宽的通信任务。

#### 主要功能：
1. **CPU连接**：
   - 连接CPU并处理其与系统内存和其他高速设备（如显卡）之间的数据传输。
   - 管理系统总线，确保CPU能够高效访问内存和外部设备。

2. **内存控制**：
   - 连接和管理系统内存（RAM），处理内存地址的映射和数据传输。
   - 现代北桥通常集成了内存控制器，支持DDR、DDR2、DDR3等不同类型的内存。

3. **高速图形接口**：
   - 连接显卡，通过AGP或PCIe接口提供高带宽的数据传输通道。
   - 处理图形数据的传输和管理，支持高性能图形渲染。

#### 演变：
- 在现代计算机系统中，北桥的功能逐渐被集成到CPU中，特别是内存控制器和PCIe控制器，减少了芯片的数量和系统的复杂性。

### 2. 南桥（Southbridge）

**南桥** 是主板上的另一个关键芯片，负责连接和管理低速设备和外设。它处理较低带宽的通信任务，并管理系统中的输入输出设备。

#### 主要功能：
1. **外围设备接口**：
   - 连接和管理低速外设，如硬盘（通过SATA或IDE接口）、光驱、USB设备、音频设备等。
   - 提供多种接口，支持各种外设的连接和数据传输。

2. **I/O控制**：
   - 管理系统中的I/O设备，包括键盘、鼠标、串口和并口。
   - 提供GPIO（通用输入输出）接口，用于连接和控制各种I/O设备。

3. **系统管理**：
   - 处理系统管理相关的功能，如电源管理、系统时钟、BIOS等。
   - 提供CMOS和RTC（实时时钟）支持，管理系统设置和时间。

#### 演变：
- 随着技术的发展，南桥的功能也逐渐被集成到单一的芯片组（如PCH，Platform Controller Hub）中，进一步简化了主板设计。

### 北桥和南桥的区别

1. **功能划分**：
   - **北桥** 主要处理高速通信，连接CPU、内存和显卡。
   - **南桥** 主要处理低速通信，管理外围设备和I/O接口。

2. **位置和连接**：
   - 北桥通常位于靠近CPU的位置，通过高速总线直接连接CPU。
   - 南桥通过北桥与CPU连接，负责连接其他外围设备。

### 现代计算机架构的变化

在现代计算机系统中，传统的北桥和南桥架构逐渐被整合和简化。以下是一些关键变化：

1. **北桥功能集成**：
   - 现代CPU通常集成了北桥的主要功能，如内存控制器和PCIe控制器，减少了系统芯片的数量。
   - 这种集成提高了系统的性能和效率，减少了延迟。

2. **PCH（Platform Controller Hub）**：
   - 南桥的功能被整合到PCH中，PCH集成了所有I/O控制和外围设备管理功能。
   - PCH通过DMI（Direct Media Interface）或类似接口与CPU连接，简化了系统架构。

### 总结

- **北桥（Northbridge）** 和 **南桥（Southbridge）** 是传统PC架构中的两个关键芯片，分别处理高速和低速的系统通信任务。
- 北桥负责连接CPU、内存和显卡，处理高带宽的数据传输；南桥负责管理外围设备和I/O接口。
- 随着技术的发展，这两个功能逐渐被集成到现代CPU和PCH中，简化了主板设计，提高了系统性能和效率。

通过理解北桥和南桥的角色和演变，可以更好地理解计算机系统的架构及其发展方向。

## RDMA的BDF

**RDMA（Remote Direct Memory Access）** 是一种允许计算机直接通过网络访问另一台计算机内存的技术，极大地提高了数据传输的效率。RDMA消除了对CPU的干扰、缓存拷贝和上下文切换，因此在高性能计算和数据中心应用中非常流行。

**BDF（Bus, Device, Function）** 是一个标识PCIe设备的地址命名格式，用于唯一标识系统中的每个PCIe设备。以下是对BDF在RDMA环境中的详细解释：

### 1. 什么是BDF？

**BDF** 代表了PCIe设备的总线号、设备号和功能号，是一个三元组，用于唯一标识系统中的每个PCIe设备。

#### BDF的组成部分：
- **Bus（总线号）**：一个系统中可以有多个PCIe总线，每个总线都有一个唯一的编号。
- **Device（设备号）**：每个PCIe总线上可以连接多个设备，每个设备在该总线上有一个唯一的编号。
- **Function（功能号）**：每个设备可以有多个功能，每个功能在该设备中有一个唯一的编号。通常，功能号为0表示主功能，其他功能号表示设备的其他功能。

#### BDF格式：
- **标准表示**：Bus:Device.Function，例如`02:00.0`表示总线号为2，设备号为0，功能号为0的PCIe设备。

### 2. BDF在RDMA中的作用

在RDMA环境中，BDF用于识别和管理网络适配器（如NIC，Network Interface Controller），这些适配器支持RDMA操作。以下是BDF在RDMA中的几个关键作用：

1. **设备识别**：
   - 在配置RDMA网络时，系统需要识别支持RDMA功能的网络适配器。BDF用于唯一标识和选择这些设备。
   - 管理工具和软件使用BDF来识别和管理RDMA适配器，确保数据传输路径的正确性。

2. **资源分配**：
   - RDMA操作需要高效地分配和管理硬件资源（如内存和网络带宽）。BDF用于分配和管理这些资源，确保每个RDMA适配器能够获得所需的资源。
   - 在多租户环境中，BDF用于分配和隔离不同租户的RDMA资源，确保资源的公平和安全使用。

3. **故障排除**：
   - 在RDMA网络中发生故障时，BDF用于精确定位故障设备，便于快速诊断和修复问题。
   - 管理工具可以通过BDF查看设备的状态和性能指标，帮助管理员优化网络配置和性能。

4. **性能优化**：
   - 使用BDF，系统可以针对不同的RDMA设备进行性能调优，确保每个设备在其特定环境中的最佳性能。
   - 在多设备环境中，通过BDF进行负载均衡和路径优化，提高整体网络性能。

### 3. BDF在系统中的使用

在实际操作中，系统管理员和RDMA配置工具使用BDF来进行设备配置和管理。例如：

- **查看设备BDF**：使用工具如`lspci`查看系统中所有PCIe设备的BDF：
  ```sh
  lspci
  ```
  输出示例：
  ```
  02:00.0 Ethernet controller: Mellanox Technologies MT27620 Family [ConnectX-3 Pro]
  ```

- **配置RDMA设备**：使用BDF进行设备配置：
  ```sh
  rdma_configure -d 02:00.0
  ```
  这条命令会配置BDF为`02:00.0`的RDMA设备。

- **监控设备性能**：使用BDF监控设备性能和状态：
  ```sh
  rdma_stat -d 02:00.0
  ```

### 总结

**BDF（Bus, Device, Function）** 是用于唯一标识PCIe设备的地址命名格式。在RDMA环境中，BDF用于识别、配置和管理RDMA网络适配器，通过精确定位和管理这些设备，确保高效的数据传输和网络性能。理解BDF的作用及其在RDMA中的应用，对于配置和维护高性能计算和数据中心网络具有重要意义。

参考
[lspci 命令详解及常用命令](https://blog.csdn.net/qq_28499879/article/details/123804789)
[深入探索Linux系统中lspci命令的基本介绍、用法和实例讲解](https://www.linuxcool.com/srtslxtzlmld)
[17.Linux驱动基础-devmem2工具调试工具使用](https://blog.csdn.net/weixin_43824344/article/details/121450879)
[用devmem2读写设备IO内存](https://blog.csdn.net/happen23/article/details/113700200)
