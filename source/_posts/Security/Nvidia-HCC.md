---
layout: post
title: "机密计算: Hopper Confidential Compute"
date: 2024-04-04 10:26:14
index_img: /img/post_pics/HCC/7.png
archive: true
tags: 
    - 并行计算
    - TEE
    - GPU
    - TDX
categories: 
    - 安全
---

NV TEE指代Nvidia Hopper Confidential Compute技术。该实现基于CPU TEE硬件技术、操作系统内核扩展、虚拟机安全扩展、GPU TEE技术和GPU内存加密等技术。

<!-- more -->

## 目录

- [目录](#目录)
- [主要结论](#主要结论)
- [NV机密计算白皮书](#nv机密计算白皮书)
  - [机密计算定义](#机密计算定义)
    - [什么是值得信任的系统](#什么是值得信任的系统)
    - [信任链--硬件有效性](#信任链--硬件有效性)
    - [机密计算--安全系统特性](#机密计算--安全系统特性)
      - [H100如何集成至TVM的TEE](#h100如何集成至tvm的tee)
    - [安全和可信启动](#安全和可信启动)
  - [NVIDIA Hopper H100机密计算特性](#nvidia-hopper-h100机密计算特性)
    - [H100机密计算的目标](#h100机密计算的目标)
    - [威胁和缓解措施](#威胁和缓解措施)
      - [机密性](#机密性)
        - [带内攻击](#带内攻击)
      - [完整性](#完整性)
        - [修改负载](#修改负载)
        - [重放](#重放)
        - [侧信道攻击](#侧信道攻击)
        - [性能计数器](#性能计数器)
      - [可用性](#可用性)
  - [使用H100搭建机密计算环境](#使用h100搭建机密计算环境)
    - [步骤1-4 定义主机拓扑和配置gpu CC模式](#步骤1-4-定义主机拓扑和配置gpu-cc模式)
    - [步骤5-7 - CVM测试GPU并开始执行代码](#步骤5-7---cvm测试gpu并开始执行代码)
      - [建立安全会话](#建立安全会话)
    - [认证GPU](#认证gpu)
  - [在GPU上运行机密计算应用](#在gpu上运行机密计算应用)
    - [CC模式的数据流](#cc模式的数据流)
    - [开发人员注意事项](#开发人员注意事项)
  - [Performance Samples](#performance-samples)
  - [结论](#结论)
- [实践验证](#实践验证)
  - [CPU支持](#cpu支持)
  - [内核编译](#内核编译)
  - [NVTrust配置](#nvtrust配置)
    - [安装依赖](#安装依赖)
    - [查询CC mode](#查询cc-mode)
    - [单独开启/关闭CC mode](#单独开启关闭cc-mode)
  - [nvidia-smi相关命令](#nvidia-smi相关命令)

## 主要结论

依赖

- HCC实现依赖虚拟化技术，NV称之为Confidential Virtual Machine。因此需要CPU TEE技术实现内存隔离、内存加密，保证虚拟机整体的机密性和安全性。
  - CVM与一般的并行计算虚拟机实现相比，需要开启H100的机密特性，并且认证GPU后方可开始计算。
- HCC依赖安全启动，这同样由CPU厂商保证

性能：

- CUDA支持了异构内存的统一寻址技术UVM，NV对此处的内存管理进行了加密，因此会降低CPU/GPU内存传输的带宽。
- 在GPU内部的内存原始数据不会被加密，性能不会有变化。
- 多GPU NVlink吞吐量明显下降

NV GPU CC mode的关键技术：

- 物理连接的PCIe出入口均有256位AES-GCM加密，同时可以以此进行数字签名，确保数据来源的安全性
- 96位滚动IV保证同明文不会重复密文，可以缓解重放攻击
- 禁用JTAG以及约束BMC的访问权限，缓解侧信道攻击
- 在开发者模式之外，禁用性能分析，缓解侧信道攻击
- GPU提供ECC-384密钥对签名和设备标识符PDI实现认证，设备唯一私有密钥IK存放在反熔丝中。

[开发人员如何利用机密计算保护 NVIDIA H100 上的应用程序和数据](/pdf/Confidential-Computing-The-Developer’s-View-to-Secure-an-Application-and-Data-on-NVIDIA-H100.pdf)

<iframe src="/pdf/Confidential-Computing-The-Developer’s-View-to-Secure-an-Application-and-Data-on-NVIDIA-H100.pdf" width="100%" height="540" name="topFrame" scrolling="yes"  noresize="noresize" frameborder="0" id="topFrame"></iframe>

[Hopper 机密计算：其内部工作原理](/pdf/Hopper-Confidential-Computing-How-it-Works-under-the-Hood.pdf)

<iframe src="/pdf/Hopper-Confidential-Computing-How-it-Works-under-the-Hood.pdf" width="100%" height="540" name="topFrame" scrolling="yes"  noresize="noresize" frameborder="0" id="topFrame"></iframe>

机密计算协会：[Confidential Computing Consortium](https://confidentialcomputing.io/)
机密计算常用术语：[Common Terminology for Confidential Computing](/pdf/Common-Terminology-for-Confidential-Computing.pdf)
白皮书：[HCC-Whitepaper-v1.0](/pdf/HCC-Whitepaper-v1.0.pdf)
免费试用：[Confidential Computing | NVIDIA](https://www.nvidia.com/en-sg/data-center/solutions/confidential-computing/)
部署guide：
[NVIDIA Trusted Computing Solutions](https://docs.nvidia.com/nvtrust/index.html#guides)
[Confidential Computing Deployment Guide - (Intel TDX & KVM)](https://docs.nvidia.com/cc-deployment-guide-tdx.pdf)
[Confidential Computing Deployment Guide - (AMD SEV-SNP & KVM)](https://docs.nvidia.com/cc-deployment-guide-snp.pdf)
[nvtrust](https://github.com/nvidia/nvtrust)
[NVIDIA Hopper Confidential Computing Attestation Documentation](https://docs.attestation.nvidia.com/Intro/Introduction.html)


机密计算优势：

- 基于硬件的安全和隔离
  使用NVIDIA Blackwell保持合规性，同时保护数据的机密性和完整性，同时实现虚拟机从边缘到云的完全隔离。Confidential Computing提供了一个物理隔离的可信执行环境(TEE)，以在使用数据时保护整个工作负载。
- 性能安全选择
  NVIDIA Blackwell的机密计算将性能和安全性结合在一起，即使是最大的数据模型，也可以为使用中的数据、人工智能模型和应用程序提供保护。机密计算可在NVIDIA Blackwell和Hopper gpu上使用。
- 设备认证的可认证性
  支持零信任体系结构，使用认证计算资产可信性的认证服务。保持合规性，并确保无论平台或工作负载在何处运行，应用程序和数据在TEE内受到NVIDIA Blackwell和Hopper gpu的保护。
- 无需更改代码的性能
  利用机密计算的所有好处而无需更改代码。NVIDIA的GPU优化软件在Blackwell和Hopper gpu上加速端到端AI工作负载，使组织能够维护隐私、安全性和法规遵从性。

应用：解锁AI安全的新可能性

- 保护人工智能知识产权
- 人工智能训练和推理的安全性
- 人工智能训练安全

## NV机密计算白皮书

### 机密计算定义

机密计算通过在基于硬件的**经过认证的可信执行环境（TEE）**中执行计算来保护正在使用的数据。这些安全且隔离的环境可防止在使用过程中对应用程序和数据进行未经授权的访问或修改，从而提高管理敏感和受监管数据的组织的安全保证。如今，数据在存储和通过网络传输时通常会进行静态加密，但在内存中使用时不会进行加密。此外，在传统的计算基础设施中，保护正在使用的数据和代码的能力受到限制。处理**个人身份信息、财务数据或健康信息等敏感数据的组织**需要缓解针对应用程序或**内存中数据的机密性和完整性**的威胁。

NIVIDIA H100 GPU 是 NVIDIA 的旗舰产品，包含机密计算支持。

#### 什么是值得信任的系统

机密计算协会正在帮助推动行业走向标准化流程，以确保机密性。

系统中的机密性是一个分层的复杂系统：要实现最高级别的信任，必须从系统的基础硬件开始。

从数据中心节点的物理安全开始似乎就足够了，包括**限制进入、经过审查的技术人员和隔离的管理网络**，但保护系统实际上甚至在系统首次通电之前就开始了。

**值得信赖的系统由硬件和软件组成**，可以使用认证构成系统的技术的真实性的机制来认证其是否值得信赖。在使用机密计算的情况下，用户需要**先认证设备和固件**的可信度，然后才能运行其工作负载，同时保护正在使用的数据和代码。纵观构成系统的技术层，每一层都依赖于基础设施堆栈中后续层的技术。要认证系统的可信度，首先要认证**基础设施的最底层技术**的可信度，即硬件计算并向上传递，这一点很重要。

![](/img/post_pics/HCC/1.png)

#### 信任链--硬件有效性

信任链是描述从终端单元每一步认证系统可信度直到到达最终权威或本质上可信任的“Root”的过程。许多芯片提供商已经创建了自己的方法来提供真实性证明。例如，CPU 供应商提供芯片真实性证明或将其使用限制为特定 OEM，主板供应商为其协处理器（例如 BMC）和相关的基于固件的组件（如 BIOS、UEFI 或BMC 代码）。

在过去的四代中，NVIDIA 在不断提高其设备的安全性和完整性。第一个记录在案的工作是在 NVIDIA V100 GPU 中，在设备上运行的固件上提供了 AES 身份认证。这种身份认证确保用户可以相信启动固件既没有损坏也没有被篡改。

![](/img/post_pics/HCC/2.png)

从加密 T4 中的固件以使恶意攻击者无法轻松检查潜在的安全漏洞，到 Ampere 节点添加了固件的外部微控制器检查 (ERoT) 以确定它是否是仍然有效（撤销），并在 Hopper 达到顶峰。**Hopper是一款具有完全机密计算能力的 GPU**。在信任链中，组成链的每一层软件的可信度都由前一层保证，直到到达**信任根（Root of Trust， RoT）**。

不变性和形式认证为信任根奠定了基础。在CC中，我们使用On-Die RoT来认证融合到GPU上的密钥，以认证GPU身份的可信度，并认证所使用的固件的可信度。与 RoT 相结合，安全启动可认证 GPU 固件是否已由 NVIDIA 签名，并且仅允许执行在 GPU 启动期间使用的已签名和经过身份认证的固件。它是一种用于认证和加载 GPU 固件模块的机制。**硬件机制确保固件在加载过程之后无法被修改，直到随后的重置。这两种机制的结合可确保 GPU 始终仅运行经过身份认证且未损坏的固件。机密计算是安全硬件的一项功能，必须依赖经过认证的硬件才能实现机密性预期。**


#### 机密计算--安全系统特性

到目前为止，这些描述都严格确保硬件的有效性，尚未专门针对机密计算。这些功能是新功能，并且不断发展以满足开发人员的性能需求和组织制度规定的保密要求，因此，必须拥有特定的 CPU 硬件 SKU 才能使用 NVIDIA H100 启用保密计算。

- **Intel CPU 必须支持“可信域扩展”(TDX)，参考文档，[Documentation for Intel® Trust Domain Extensions](https://www.intel.cn/content/www/cn/zh/developer/tools/trust-domain-extensions/documentation.html)**
- AMD CPU 必须支持“具有安全嵌套分页的安全加密虚拟化”(SEV-SNP)
- ARM CPU 必须支持 ARM“机密计算架构”(CCA)。

上述主机具有支持机密计算的硬件功能。这些主机中的每一个都可以使用**访问控制检查、分页控制、地址转换和内存加密等技术**的组合来在数据使用时提供数据保护。

##### H100如何集成至TVM的TEE

要解决的第一个问题是**如何在CPU处理系统中的内存页面的现有框架内提供GPU访问加密数据**；处理器中的MMU已经配置为防止虚拟机之间未经授权的内存访问，但数据是加密的。

CPU供应商已经在保密模式下配置了非安全页面的分配方式，这意味着系统中的任何人都可以访问它。但是，只要这些Non-secure页面中的数据保持加密状态，就可以保持机密性。这些页面对GPU是可见的，GPU可以将有效载荷传输到其内部存储中。从硬件的角度来看，**虚拟机负责加密数据，将其发送到这个 bounce buffer，GPU可以将其复制到其内部存储器，解密并处理它。一旦完成，GPU将再次加密数据，将其放回 bounce buffer，并通知虚拟机数据已准备好使用。**

![](/img/post_pics/HCC/3.png)

与CPU供应商类似，H100中的CC功能需要许多新的创新硬件功能。下面将详细讨论这些内部特性、操作模式以及开发人员如何开始使用它，但是安全系统中还有最后一个主题需要考虑：安全引导。

#### 安全和可信启动

一旦系统到达并被接收到数据中心，来自不同硬件供应商的所有这些特性就被连接起来，启用Confidential系统的过程就可以真正开始了。然而，许多活动在操作系统准备好为虚拟机提供附加GPU之前就开始了。

当系统上电时：

- 许多组件将开始前面提到的自检(它们的RoT；在某些情况下，组件可能会相互通信)；
- 如果所有测试都通过了，系统将开始引导到主机操作系统的过程。

在俗称的“安全引导”中，主板在**非易失性存储器**中存储一组证书，这些证书用于认证试图加载执行的软件二进制文件的有效性。许多OEM将预装来自主要操作系统供应商(例如，Canonical、Red Hat和Microsoft)的证书。当UEFI固件获取这些操作系统引导加载程序二进制文件时，它们的签名将根据预加载的证书进行检查。如果二进制文件未签名或检查失败(例如，没有匹配证书的签名)，系统将停止其引导过程。假设第一个引导加载程序二进制文件通过了签名/证书检查，UEFI固件将加载并将执行传递给它；然后，这个经过身份认证的引导加载程序可以加载最终内核。由于该内核仍然具有对系统固件的完全访问权限，因此还必须根据系统中的证书对其进行认证。加载后，这些经过认证的内核将在继续引导时禁用此访问。内核驱动程序仍然可以访问引导固件，因此需要确保系统中的I/O设备被允许操作(再次通过检查签名；这次是在硬件驱动程序上)。它开始对系统中的硬件进行自己的查询，并通过与主板供应商握手，请求证书是否存储且有效，并与加载的驱动程序的签名相匹配。在任何时候，故障都可能停止引导过程，或者禁用驱动程序，但继续引导。

如前所述，带有NVIDIA H100的CC依赖于带有AMD SEV-SNP或IntelTDX的CPU。这些都是基于虚拟机的解决方案，因此需要一个几乎重复的步骤。在引导流程的这一点上，主机系统已经对系统中的硬件进行了基本认证，但是已经隐式地信任硬件供应商，认为他们“做了正确的事情”。应该做的额外步骤称为“认证”，其中主机可能要求某些硬件提供“证据”，证明它们“是它们所说的”，并且它们正在运行正确的固件和系统配置等。

硬件的认证需要贯彻“**信任但要认证**”。主机应该在所有可用硬件上运行认证，然后启动其**管理程序并为其最终的机密工作负载创建虚拟机**。主机在完成它们自己的认证检查之后，将把适当的硬件传递给它们的管理程序，并提供一个CVM。在这个CVM中，**内核启动必须再次重新认证任何试图直接访问系统硬件的驱动程序的签名**。一旦系统完成引导，CVM就可以传输给最终用户了。他们可以在主机的所有者-运营商(例如Azure或GCP认证服务)的基础设施中对CVM使用进一步的认证。

### NVIDIA Hopper H100机密计算特性

通过上电进行硬件检查，确保设备可以对支持/授权的硬件和就绪状态进行一些初始交叉检查。一旦CVM开始加载，NVIDIA GPU驱动程序将开始启动许多进程来准备GPU用于Confidential系统。为了拥有一个真正的生产就绪的机密加速机器，NVIDIA需要拥有强大的固件和软件堆栈，以及认证流程，以**提供一个完整的CC解决方案，其中包括代码和数据的保护和完整性。Hopper H100有几个新的基于硬件的功能添加到它的芯片中，以实现这种级别的机密性。**

#### H100机密计算的目标

NVIDIA为启用具有CC的H100设定了三个主要目标，与CC联盟的规则一致

- **数据和代码机密性：**保护VM实例中的所有应用程序代码和数据不被主机读取。
- **数据和代码完整性：**保护虚拟机实例中的所有应用程序代码和数据不被主机更改。
- **基本物理攻击：**PCIe和DDR内存等总线上的中间商无法泄漏数据或代码。

NVIDIA GPU的发放方式如下：

- 分配GPU
- 向虚拟机传递多个GPU
- 通过vGPU和SRIOV向虚拟机传递部分图形处理器。

![](/img/post_pics/HCC/4.png)

这些部署模式也可以直接映射到CC模式。每台Hopper H100设备都将支持用例，每个用例支持以下操作模式：

- CC OFF：H100标准操作。没有任何加密/身份认证路径处于活动状态，也没有任何其他特定于cc的系统块/防火墙/边带通道处于活动状态。
- **CC ON：H100与CPU上的驱动程序将完全激活所有可用的CC功能；所有防火墙都处于活动状态，所有边带通道都已关闭。**
- CC DevTools：许多开发人员依靠我们的开发人员工具来帮助调试他们的代码和配置文件，这样他们就可以了解系统瓶颈，以提高整体性能。在CC-Devtools模式下，GPU完全处于“CC-On”模式，如上所述，除了运行开发人员工具所需的数据路径。

#### 威胁和缓解措施

Hopper H100的CC模式的目标可以进一步细分。这里将依次讨论每个子类别，描述H100如何在高级别上减轻特定威胁。

- 范围内威胁向量
  - **软件攻击**
  - **基本物理攻击**
  - **软件回滚攻击**
  - **加密攻击**
  - **数据回滚和重放攻击**
- 范围外威胁向量
  - **复杂物理攻击**
  - **拒绝服务攻击DoS**

##### 机密性

机密性子类别描述了您的数据如何对恶意或其他未经授权的攻击者保证机密性。有许多方法可以考虑窥探或以其他方式获取机密代码，从基于硬件的物理接口窥探到试图访问未分配给它的内存或GPU的软件代码；其中一些缓解措施继承自CPU供应商为保持传统CVM安全所做的努力，而其他缓解措施则来自NVIDIA，专门用于保护GPU。

###### 带内攻击

第一种方法，也是最简单的可视化方法，是简单地使用物理PCIe或NVLink连接来读取租户数据。PCIe总线分析仪在整个行业中非常常见，特别是在调试新芯片或测试PCI SIG合规性时。

传统上，这些中间的设备在主机和目标设备之间物理连接，悄悄地记录通过连接器导线的所有活动，然后将数据重构为用户可读的格式。**NVIDIA在其所有入口/出口路径上都有内置的加密和解密引擎；这些引擎使用256位AES-GCM加密。任何进入或退出GPU的请求或响应都必须加密。**

但是，在不更改密钥的情况下，**多次发送相同的明文消息将在总线上产生相同的加密数据，这是不可接受的，因为它可以向攻击者提供有关系统、有效负载等的知识。**对此的一个解决方案是引入一个叫做改变**初始化向量(IV)**的进一步保护层，它可以用来有效地随机化每个消息；基本密钥不会改变，而是由IV在数学上进行修改，它会随着每次有效载荷的变化而变化。这种好处是双重的：

- 多个纯文本消息现在产生具有相同密钥的完全不同的密文。
- 为有效负载引入了一种强有序的行为，从而减轻了一种称为重放攻击的常见攻击向量。

在AES-GCM中，**需要一个96位IV。即使增加了IV， AES-GCM仍然容易重用(Key， IV)对来加密不同的数据块。 “唯一性”的总数被限制为(2^96)-1，超过此值后IV将“耗尽”，加密密钥必须轮换并替换。**NVIDIA软件栈自动支持这种轮换，并提供用户修改默认策略的选项。

所有从开发人员的TVM或从其配置的GPU传输的数据都将以这种方式加密。NVIDIA H100如何集成到TVM的TEE中提到的“bounce buffer”将只包含加密数据。**虚拟机中的数据存储在主机内存中，并使用CPU供应商提供的设施进行加密和保护。**

##### 完整性

###### 修改负载

与保密部分类似，我们讨论了用于防止对手读取您的数据的缓解措施，但是，在flight中修改数据不能仅通过加密来缓解。的确(尤其是滚动iv)，修改加密流量几乎不可能产生可确定的结果，但它绝对可能引入完全不被注意的错误或差异。

**AES-GCM算法还为创建数字签名(称为AuthTag)和密文提供了条件。它使用其密钥对密文进行“签名”，以创建有效载荷的唯一指纹；这是一种加密安全方法，攻击者实际上不可能修改密文及相关的AuthTag。**在远端接收到有效负载后，接收方将使用其密钥副本执行相同的操作，以计算其自己版本的AuthTag。如果它们不匹配，解密将失败，并抛出错误。

###### 重放

重放攻击，一个中间人拦截一个有效数据，并在未来的某个时候再次发送它。一个类比是，攻击者已经识别了一个传输队列的有效载荷： “Password accepted, continue onward and trust me” 。如果捕获了特定的有效载荷，则可以在适当的时候重播，从而使攻击者能够访问用户的密码。同样，H100上的AES-GCM滚动IV功能可以减轻这种威胁。之前，我们提到了滚动IV如何具有防止重放攻击的次要功能。当安全通道的两个端点都相关时，每个端点都可以保留一个消息计数器作为IV增量。**执行解密的接收端点不使用加密发送器发送的IV，而是使用自己的IV。**与加密后的IV增量类似，解密端点将在解密后增加IV。如果攻击者或恶意用户试图捕获和重放，解密和AuthTag比较都将失败，因为IV已经增加了。

###### 侧信道攻击

在许多企业/超大规模数据中心中，节点数量可以达到数千到数百万，因此远程管理访问、升级或调试硬件的需求是至关重要的。通常，这些服务器中的主板有一个次要的轻量级协处理器，称为**主板管理控制器(BMC)**。BMC有自己的网络接口、内存、网络等，并且通过系统管理总线(SMBus)，技术人员可以登录到这个单独的BMC，并获得对节点中所有内容的完整系统访问权限：

- 虚拟桌面(如插入本地显示器和键盘)到主CPU复核。
- 节点硬件访问和信息(UEFI/BIOS更新，温度，库存，电源控制等)
- 外设访问和信息(GPU库存，固件更新，温度，性能计数器等)

这个边带非常强大，如果访问落入攻击者手中可能是灾难性的。**H100将锁定对可能访问数据的任何路径的访问，并将剩余路径减少到仅限匿名“卡运行状况”的详细级别。GPU在保密模式下运行时JTAG调试也被禁用。**

###### 性能计数器

当使用NVIDIA开发者工具(DevTools)来分析应用程序的性能/行为时，特殊的设备端硬件计数器被用来帮助测量设备上的代码。然而，性能计数器可以用来推断设备的使用行为，并为侧信道攻击提供途径。这些在完全CC-On模式下被禁用。然而，由于分析是优化应用程序性能的关键，我们支持一种称为CC-Dev Tools的开发模式。而在CC-DevTools模式下，所有的加密路径，反弹缓冲区等都是启用的，但对这些计数器的访问是开放的，供开发人员使用。

##### 可用性

可用性涉及相关的系统的总体健康状况，这些系统与企图拒绝或干扰另一个用户或管理程序本身的操作或访问的攻击者有关。H100及其软件栈不需要与Hypervisor挂钩，因此不会干扰主机的操作。传统的隔离/保护Hypervisor与VM的方法被新的CPU特性(TDX和SEV-SNP)所补充，旨在将VM与潜在的恶意Hypervisor隔离开来。

一般H100只能访问其分配的VM拥有的内存位置：Hypervisor在IOMMU中配置的地址。TDX和SEV-SNP更进一步，只允许IOMMU允许的事务访问CVM中标记为**共享的**的内存页。此组合安全的目的如下：

- 提供标准的多用户保护，防止拒绝服务系统内其他用户(恶意的H100试图访问其他虚拟机，或恶意的虚拟机试图访问其他正常的虚拟机及其硬件)。
- 它可以防止恶意的H100破坏CVM不应该访问的内存空间。

NVIDIA长期以来一直与顶级公共云服务提供商(CSP)整合，这些提供商提供对其基础设施内硬件的半自主、直接访问。对可能访问这些系统的每个潜在用户进行全面审查是不切实际的。NVIDIA与这些CSP合作，以确保任何使用其服务的恶意用户不会通过主机崩溃、pcie总线锁定或物理损坏GPU或基础设施的其他部分使GPU处于无法使用的状态，以供下一个用户使用。任何对GPU的永久管理访问都被锁定在带内控制之外。

唯一超出可用性范围的攻击是恶意的Hypervisor管理程序阻止对CVM的访问。这些攻击可能包括移除用户网络访问、断开GPU电源等。虽然这些攻击不会有软件模块，但所有用于加密/解密虚拟机和GPU内存的私钥都在虚拟机管理程序无法访问的密钥槽中。这些密钥是临时的，在VM拆卸或GPU重置时被舍弃或擦除。

### 使用H100搭建机密计算环境

介绍使用H100时CVM如何操作。关于流程的具体细节、支持的版本、示例代码等，可以在我们的[NVIDIA/nvtrust： Ancillary open source software to support confidential computing on NVIDIA GPUs (github.com)](https://github.com/nvidia/nvtrust/)上找到的可信环境部署guide中找到。部署指南将定义多个角色：

- **硬件IT管理员, Hardware IT Administrator**
- **主机操作系统管理员, Host OS Administrator**
- **虚拟机管理员, Virtual Machine Administrator**
- **虚拟机用户, Virtual Machine User**
- **容器用户, Container User**

用于设置流程；由于安全系统（尤其是针对CC的系统）涉及系统堆栈的每个级别，不同的团队将负责或者掌握在这些环境中取得成功所需的不同经验、需求和步骤。

![](/img/post_pics/HCC/5.png)

#### 步骤1-4 定义主机拓扑和配置gpu CC模式

Hopper H100机密计算的目标中描述的Hopper H100 CC模式必须由虚拟化环境配置。CPU功能以及H100软件堆栈允许这些模式的组合配置。

![](/img/post_pics/HCC/6.png)

在上图中，系统中有4个gpu。管理程序可以根据gpu的配置需要动态更改任何gpu上的Confidential模式。

**启用/禁用CC的API是作为主机的带内PCIe命令或带外BMC命令提供的。**切换模式需要GPU进行FLR (Function Level Reset)复位才能生效。在这个重置过程中，内存锁被占用，它阻塞对GPU内存的访问直到它被清除（缓解冷启动攻击）。GPU固件在GPU移交给用户之前启动内存和寄存器和ram中的状态擦洗。同时配置GPU防火墙，防止GPU被非法访问。

#### 步骤5-7 - CVM测试GPU并开始执行代码

##### 建立安全会话

在CPU上加载安全虚拟机时会加载NVIDIA GPU驱动。这将触发VM与超出基本PCIe枚举设备的第一次通信。必须创建安全会话以允许在虚拟机和GPU之间传输的数据进行加密和解密。需要加密会话的原因如下：

- GPU无法访问虚拟机的保护页面。这意味着从GPU流入和流出的数据必须通过未受保护的bounce buffers，其中TCB之外的实体（例如，管理程序）可以读/写它们。
- PCIE不是VM和用户之间的安全专用通道，通过它传输的数据可能会被TCB之外的实体读取或破坏。

建立安全会话有两个部分：

- 用于建立共享对称会话密钥的Diffie-Hellman密钥交换。
- 检索包含硬件和软件组件测量值的设备认证报告。

在GPU上运行的所有固件和microcode都由NVIDIA编写和签名，认证在执行之前。检索和认证报告是在设备中建立信任的关键步骤。它认证系统版本是否符合设备用户的期望和信任。**一旦通过检查NVIDIA根CA的签名来认证认证报告，并且建立了对称密钥，设备就可以被运行在TEE中的CUDA应用程序使用了。设备之间已经建立信任并创建安全通道。**

#### 认证GPU

**证明是用户或信任方想要质疑 GPU 硬件及其关联驱动程序、固件和microcode，并在继续操作之前收到响应有效且真实的确认的过程。**

CVM在使用GPU之前必须先认证所有GPU都是正版的，然后才能将其纳入其trust boundary。**它通过从设备或NVIDIA设备身份服务检索设备身份证书（用设备唯一的ECC-384密钥对签名）来实现这一点。它可以使用嵌入到每个GPU芯片中的每个设备标识符（PDI）来检索。根据NVIDIA的证书颁发机构认证此证书将认证该设备是由NVIDIA制造的。该设备唯一的私有身份密钥(IK)被烧进每个H100的保险丝中；公钥被保留，但这些私钥的所有副本在制造过程中被销毁。**

此外，TVM还必须通过与OCSP （Online certificate Service Protocol）等服务获取的证书吊销列表(certificate Revocation List， CRL)进行比较，确保GPU证书没有被吊销。虽然使用NVIDIA远程认证服务（ NVIDIA Remote Attestation Service, NRAS）将是认证认证报告的主要方法，但将为真正的air-gapped场景（[什么是Air-gapped test or development environments？_air gapped-CSDN博客](https://blog.csdn.net/hadues/article/details/132768990)）提供选项，以提供本地认证。

以下是使用NVIDIA远程认证服务的认证工作流程的一般步骤：

1. 信赖方将使用NVIDIA认证SDK对GPU或NRAS api进行认证。
2. NVIDIA认证使用符合NIST SP800-90 a /B/C标准的安全随机数生成器生成至少128位的随机随机数。
3. NVIDIA认证SDK将nonce发送到使用cc_admin/nvml库的认证人员，以获得设备(GPU)的测量。
4. 证人将nonce嵌入背书证据中。
5. 此nonce也将在有效负载中传递给远程认证。
   该证据将使用名为“证明密钥”的私钥进行签名，并共享包含与该证明密钥对应的公钥的证书，以认证证据的签名。证据由认证密钥(AK)背书。对于GPU来说，AK是在每次全芯片复位时确定地生成的。为AK生成证书，并由每个设备的设备身份密钥签名。该证书链在身份密钥之上有一个或多个中间证书，以证明对AK的信任。
6. SDK将带设备身份、认证密钥证书和nonce的认可证据发送到HTTPS (TLS会话)上的远程认证服务
7. NRAS认证请求，解析证据并根据SPDM规范进行认证。
8. NRAS还认证为重放攻击检测存储在内存中的nonce。
9. NRAS使用设备证书公钥的散列维护nonce，并附加24小时的TTL。
10. 对于相同设备，具有相同nonce的其他请求将被相同的服务器实例(pod)拒绝，因为内存缓存中已经处理了相同的nonce。
11. NRAS认证证据背书证书链。
    如果证书链中的任何证书被撤销，NRAS将返回一个“证据无效”。消息。
12. NRAS使用经过认证的证书链中的公钥来认证证据的签名(参见RFC 5280) ，并检查证书链的颁发者是否属于NVIDIA PKI。
13. 如果无法认证证据的签名，则会产生错误，并停止认证评估。
    HTTP错误码400，提示签名无效。
14. NRAS使用证据中提供的驱动程序版本和GPU模型从RIM Service获取RIM Bundle (Golden Measurements) 。
15. 如果有任何连接错误，将重新尝试NRAS到RIM服务的通信。
    任何来自RIM服务的异常，例如，未找到RIM Bundle。将导致GPU认证失败和不支持的驱动程序版本。将被发送。考虑到同一个RIM Bundle不会被更新和更改，NRAS可以缓存频繁访问的RIM Bundle。
16. NRAS从RIMM Bundle获取证书链，并调用CertZapper OCSP端点来认证证书是否有效。如果证书被撤销，NRAS将发送一个认证失败，错误响应为“RIM Bundle被撤销”。
17. NRAS根据公共证书认证RIM包的签名。
18. NRAS将证据与从RIMM服务中获取的黄金测量值进行比较，并生成认证结果。
19. NRAS调用UAM (OPA rego policy engine)应用评估政策作为证据。这适用于某些度量不匹配的情况，但认证者知道这一点，并且作为所有者的认证者仍然可以决定认证是否正确。还有依赖方评估政策，依赖方适用于最终(EAT)认证结果。
20. NRAS使用临时证书L3对EAT进行签名。
    1. 此L3证书将是短期的，ttl为24小时。
21. NRAS以JWT格式为已认证的声明创建EAT(实体证明令牌)。
22. NRAS使用临时密钥和证书，并使用私钥对JWT令牌进行签名。
23. NRAS向SDK发送一个证明令牌(JWT)。
24. JWT具有防止重放攻击(jti)的JWT Id，以及结果有效的到期时间。
    步骤11-14中的任何失败都将导致内部服务器错误，并且在重试用尽后将实现内部重试，错误响应为内部服务器错误，稍后再试。消息。
25. NRAS将具有设备证书哈希值的nonce存储在内存中，其ttl为24小时，在将认证结果发送给依赖方之前，NRAS将其处理成功。
    需要这些数据来避免重放攻击。
26. NRAS将结果发送给认证SDK。
27. SDK将响应(EAT)发送回依赖方。
28. 依赖方使用作为JWT的一部分或作为发行者端点提供的OpenID公钥认证令牌。
    OpenID端点是JWKS端点，它将拥有用于认证EAT的公共证书，该端点由NRAS服务提供服务。
29. NRAS从Vault PKI引擎中检索证书。
30. 在认证成功后，依赖方将应用自己的评估策略来确定设备是否处于良好状态。
    根据决定，依赖方将允许设备通信或阻止它。

### 在GPU上运行机密计算应用

一旦 CPU TEE 的信任扩展到 GPU，运行计算应用程序就与常规 GPU 相同。使用正确的硬件、驱动程序和通过的证明报告完成“验证 GPU”中的步骤后，执行内核应该是透明公开的。

#### CC模式的数据流

在正确配置、引导和认证了H100的CVM之后，就可以开始在H100 GPU上安全地处理数据了。我们努力确保尽可能多的提升和改变编码风格。目标是在启用H100 CC模式时，让来自用户的现有代码和内核无需更改即可工作。

缺省情况下，不允许设备与CVM交互，不允许设备直接访问CVM内存。下面我们描述驱动程序如何使H100以CC模式安全地与CVM通信。

H100 GPU具有具有加密/解密功能的DMA引擎，负责将数据从CPU的内存中移出或移出。在机密环境中，DMA引擎只允许访问共享内存页面来检索和放置数据。为了确保有效负载、模型和数据的机密性和完整性，这些页面中的数据被加密并签名。这些共享内存区域称为弹跳缓冲区，因为它们将用于在数据传输到安全内存enclave之前暂存受保护的数据，然后对其进行解密和身份认证，然后进行处理。**NVIDIA长期以来一直为我们的开发人员提供一种称为统一虚拟内存(UVM)的解决方案，该解决方案基于称为cudaMallocManaged()的内存分配API自动处理GPU内存和CPU内存之间的页面迁移。当CPU访问数据时，UVM将页面迁移到CPU的系统内存中。当需要使用GPU上的数据时，UVM会将数据迁移到GPU内存中。对于CC，我们扩展了UVM，通过共享内存中的反弹缓冲区使用加密和身份认证的分页。对于多个GPU可能使用NVLINK相互移动数据的用例，我们通过GPU不受保护的内存中的反弹缓冲区使用硬件加密传输。**

#### 开发人员注意事项

这里总结了一些开发人员在CC模式下使用H100时必须注意的事项。

- 由于CPU厂商从外部资源隔离他们的CVM内存的方式，固定内存分配如cudaHostAlloc()和cudaMallocHost()不能由GPU直接访问。相反，它们由UVM使用加密分页处理，就好像它们是由cudaManagedAlloc()分配的一样。这意味着固定内存访问在CC模式下会变慢。
- cudaHostRegister()不受支持，因为这个API直接访问由CVM内部的malloc()或new()创建的内存。当GPU处于CC模式时，这个API将返回一个错误代码。cudaHostRegister()在NVIDIA库中没有广泛使用。
- 开发人员在CC模式下使用H100 GPU时必须使用NVIDIA - persistenced守护进程来保持驱动程序加载，即使在不使用时也是如此。
  - 在典型的操作中，当不再使用 NVIDIA 设备资源时，NVIDIA 内核驱动程序将拆除设备状态。然而，在 CC 模式下，这会导致破坏在驱动程序的设置 SPDM 阶段建立的共享秘密和共享密钥。为了保护用户数据，GPU 将不允许在没有 FLR 的情况下重新启动 SPDM 会话建立
  - **nvidia-persistenced** 提供了一个称为持久模式的配置选项，可以通过 NVIDIA 管理软件（例如 nvidia-smi）进行设置。启用持久模式后，将阻止 NVIDIA 内核驱动程序退出。 **nvidia-persistenced** 不使用任何设备资源。它只是休眠，同时保持对 NVIDIA 设备状态的引用。

### Performance Samples

CC的主要目标是CUDA应用程序可以在不改变的情况下运行，同时最大限度地提高底层硬件和软件的加速潜力。CUDA为将在CC模式下运行的应用程序提供了提升和转移的好处。因此，NVIDIA GPU CC架构与CPU架构兼容，还提供了从非机密环境到CC环境的应用程序可移植性。考虑到目前为止的描述，当计算量与输入数据量相比很大时，GPU上的CC工作负载执行接近非CC模式应该不足为奇。与输入数据相比，当计算量较低时，跨非安全互连的通信开销会限制应用程序吞吐量。以下是CC模式下的性能参考：

- 在CC模式下，以下性能基本要素与非机密模式相当：
  - GPU原始计算性能
    计算引擎对驻留在GPU内存中的未加密数据执行未加密代码。
  - GPU内存带宽
    封装上的HBM内存被认为是安全的，可以抵御常见的物理攻击工具，例如中间层，并且没有加密。
- 以下性能原语受到额外的加密和解密开销的影响：
  - CPU- gpu互连带宽受CPU加密性能限制，约为4gb /秒。
  - 跨非安全互连的数据传输吞吐量会导致通过未受保护内存中的加密反弹缓冲区传输的延迟开销。
  - 多GPU用例中的GPU对端内存带宽直接连接多GPU NVLINK拓扑在未受保护的GPU内存中使用加密的弹跳缓冲区，从而降低吞吐量

![](/img/post_pics/HCC/7.png)

对GPU命令缓冲区、同步原语、异常元数据以及GPU和运行在CPU上的机密虚拟机之间交换的其他内部驱动程序数据进行加密也会带来开销。对这些数据结构进行加密和身份认证可以防止对用户数据的侧信道攻击。CUDA统一虚拟内存长期以来一直允许开发人员使用来自CPU和GPU的相同虚拟地址指针，这确实简化了应用程序代码。在CC模式下，统一内存管理器必须对跨非安全互连迁移的所有页面进行加密。

### 结论

由于监管制度、隐私问题或加速其他敏感工作负载的需求，整个计算行业都认识到，在操作数据时需要改变传统的安全思维和操作方式。

## 实践验证

### CPU支持

部署参考：[Deployment Guide for Confidential Computing (nvidia.com)](https://docs.nvidia.com/confidential-computing-deployment-guide.pdf)
[nvtrust/host_tools/python at main · NVIDIA/nvtrust (github.com)](https://github.com/NVIDIA/nvtrust/tree/main/host_tools/python)
NV HCC的使用方式是Confidential Virtual Machine。Host端通过KVM或者Qemu创建虚拟机，在guest VM中认证H100并启动程序。

```bash
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
192  Intel(R) Xeon(R) Platinum 8468
```

**CVM需要支持TDX扩展的Intel至强处理器或者支持SEV-SNP扩展的AMD芯片。**

![](/img/post_pics/HCC/bios.png)

有关TDX，参考：

[英特尔®信任域扩展 （英特尔® TDX） 模块 (intel.cn)](https://www.intel.cn/content/www/cn/zh/download/738875/intel-trust-domain-extension-intel-tdx-module.html?wapkw=tdx)
[Intel® Trust Domain Extensions (Intel® TDX)](https://www.intel.cn/content/www/cn/zh/developer/tools/trust-domain-extensions/overview.html?wapkw=tdx)
相关新闻：[Seamless attestation of Intel TDX and NVIDIA H100 TEEs with Intel Trust Authority](https://community.intel.com/t5/Blogs/Products-and-Solutions/Security/Seamless-Attestation-of-Intel-TDX-and-NVIDIA-H100-TEEs-with/post/1525587)

如需进一步验证，需要**5th gen Xeon处理器芯片，或者4th代号“Emerald rapids”的处理器，联系intel售后 +86 800 820 1100获取更多信息。**

### 内核编译

如果顺利开启了硬件TDX支持，参考下列步骤编译内核补丁：

```bash
apt update
apt install --no-install-recommends --yes build-essential fakeroot \
devscripts wget git equivs liblz4-tool sudo python-is-python3 python3-dev \
pkg-config unzip help2man texinfo xfonts-unifont libfreetype6-dev libdevmapper-dev \
libsdl1.2-dev libfuse3-dev liblzma-dev liblzo2-dev mtools libefiboot-dev \
libefivar-dev qemu-system
 
pip3 install numpy flex bison
git clone https://github.com/intel/tdx-tools
git clone https://github.com/NVIDIA/nvtrust.git
 
cd tdx-tools
git checkout -b 2023ww15 refs/tags/2023ww15
 
cd build/ubuntu-22.04
./build-repo.sh
cd host_repo
apt -y --allow-downgrades install ./*.deb
reboot
```

验证内核：

```bash
uname -a
dmesg | grep -i tdx
cat /proc/cmdline
```

### NVTrust配置

Nvtrust主要是提供Host端的CC mode查询工具和guest VM的认证工具。
仓库：[NVIDIA/nvtrust: Ancillary open source software to support confidential computing on NVIDIA GPUs (github.com)](https://github.com/NVIDIA/nvtrust/tree/main)

#### 安装依赖

```bash
### python环境
python3.10 -m venv myenv
source myenv/bin/activate
 
### 安装verifier
cd nvtrust/guest_tools/gpu_verifiers/local_gpu_verifier
pip3 install . -i http://segitlab.hygon.cn:8081/repository/pypi-aliyun//simple --trusted-host segitlab.hygon.cn
 
### 安装 attestation SDK
cd nvtrust/guest_tools/attestation_sdk/dist
pip3 install nv_attestation_sdk-1.3.0-py3-none-any.whl -i http://segitlab.hygon.cn:8081/repository/pypi-aliyun//simple --trusted-host segitlab.hygon.cn
 
### 开启CC mode
apt install patchelf
cd nvtrust/host_tools/python
python3 gpu_cc_tool.py --gpu_name=H100 --set-cc-mode=on --reset-after-cc-mode-switch
```

#### 查询CC mode

```bash
$ python3 gpu_cc_tool.py --gpu-name=H100 --query-cc-settings
NVIDIA GPU Tools version 535.86.06
Topo:
  Intel root port 0000:00:1b.0
   PCI 0000:03:00.0 0x11f8:0x4128
    PCI 0000:04:00.0 0x11f8:0x4128
     NvSwitch 0000:05:00.0 ? 0x22a3 BAR0 0x9c000000
    PCI 0000:04:01.0 0x11f8:0x4128
     NvSwitch 0000:06:00.0 ? 0x22a3 BAR0 0x9a000000
    PCI 0000:04:02.0 0x11f8:0x4128
     NvSwitch 0000:07:00.0 ? 0x22a3 BAR0 0x98000000
    PCI 0000:04:03.0 0x11f8:0x4128
     NvSwitch 0000:08:00.0 ? 0x22a3 BAR0 0x96000000
  Intel root port 0000:15:01.0
   PCI 0000:16:00.0 0x1000:0xc030
    PCI 0000:17:00.0 0x1000:0xc030
     GPU 0000:18:00.0 H100-PCIE 0x2324 BAR0 0x52042000000
  Intel root port 0000:37:01.0
   PCI 0000:38:00.0 0x1000:0xc030
    PCI 0000:39:00.0 0x1000:0xc030
     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000
  Intel root port 0000:48:01.0
   PCI 0000:49:00.0 0x1000:0xc030
    PCI 0000:4a:01.0 0x1000:0xc030
     GPU 0000:4c:00.0 H100-PCIE 0x2324 BAR0 0x76042000000
  Intel root port 0000:59:01.0
   PCI 0000:5a:00.0 0x1000:0xc030
    PCI 0000:5b:01.0 0x1000:0xc030
     GPU 0000:5d:00.0 H100-PCIE 0x2324 BAR0 0x86042000000
  Intel root port 0000:97:01.0
   PCI 0000:98:00.0 0x1000:0xc030
    PCI 0000:99:00.0 0x1000:0xc030
     GPU 0000:9a:00.0 H100-PCIE 0x2324 BAR0 0xaa042000000
  Intel root port 0000:b7:01.0
   PCI 0000:b8:00.0 0x1000:0xc030
    PCI 0000:b9:00.0 0x1000:0xc030
     GPU 0000:ba:00.0 H100-PCIE 0x2324 BAR0 0xbe042000000
  Intel root port 0000:c7:01.0
   PCI 0000:c8:00.0 0x1000:0xc030
    PCI 0000:c9:01.0 0x1000:0xc030
     GPU 0000:cb:00.0 H100-PCIE 0x2324 BAR0 0xce042000000
  Intel root port 0000:d7:01.0
   PCI 0000:d8:00.0 0x1000:0xc030
    PCI 0000:d9:01.0 0x1000:0xc030
     GPU 0000:db:00.0 H100-PCIE 0x2324 BAR0 0xde042000000
2024-03-29,02:45:34.994 INFO     Selected GPU 0000:18:00.0 H100-PCIE 0x2324 BAR0 0x52042000000
2024-03-29,02:45:34.994 WARNING  GPU 0000:18:00.0 H100-PCIE 0x2324 BAR0 0x52042000000 has CC mode on, some functionality may not work
2024-03-29,02:45:35.065 INFO     GPU 0000:18:00.0 H100-PCIE 0x2324 BAR0 0x52042000000 CC settings:
2024-03-29,02:45:35.065 INFO       enable = 1
2024-03-29,02:45:35.065 INFO       enable-devtools = 0
2024-03-29,02:45:35.065 INFO       enable-allow-inband-control = 1
2024-03-29,02:45:35.065 INFO       enable-devtools-allow-inband-control = 1
```

#### 单独开启/关闭CC mode

```bash
$ python3 gpu_cc_tool.py --gpu-bdf=3a:00.0 --set-cc-mode=on --reset-after-cc-mode-switch
NVIDIA GPU Tools version 535.86.06
Topo:
  Intel root port 0000:37:01.0
   PCI 0000:38:00.0 0x1000:0xc030
    PCI 0000:39:00.0 0x1000:0xc030
     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000
2024-03-29,05:28:52.728 INFO     Selected GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000
2024-03-29,05:28:52.880 INFO     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000 CC mode set to on. It will be active after GPU reset.
2024-03-29,05:28:54.533 INFO     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000 was reset to apply the new CC mode.
 
$ python3 gpu_cc_tool.py --gpu-bdf=3a:00.0 --query-cc-settings
NVIDIA GPU Tools version 535.86.06
Topo:
  Intel root port 0000:37:01.0
   PCI 0000:38:00.0 0x1000:0xc030
    PCI 0000:39:00.0 0x1000:0xc030
     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000
2024-03-29,05:26:08.123 INFO     Selected GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000
2024-03-29,05:26:08.123 WARNING  GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000 has CC mode on, some functionality may not work
2024-03-29,05:26:08.194 INFO     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000 CC settings:
2024-03-29,05:26:08.194 INFO       enable = 1
2024-03-29,05:26:08.194 INFO       enable-devtools = 0
2024-03-29,05:26:08.194 INFO       enable-allow-inband-control = 1
2024-03-29,05:26:08.194 INFO       enable-devtools-allow-inband-control = 1
 
$ python3 gpu_cc_tool.py --gpu-bdf=3a:00.0 --set-cc-mode=off --reset-after-cc-mode-switch
NVIDIA GPU Tools version 535.86.06
Topo:
  Intel root port 0000:37:01.0
   PCI 0000:38:00.0 0x1000:0xc030
    PCI 0000:39:00.0 0x1000:0xc030
     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000
2024-03-29,05:28:21.814 INFO     Selected GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000
2024-03-29,05:28:21.815 WARNING  GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000 has CC mode on, some functionality may not work
2024-03-29,05:28:21.906 INFO     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000 CC mode set to off. It will be active after GPU reset.
2024-03-29,05:28:23.558 INFO     GPU 0000:3a:00.0 H100-PCIE 0x2324 BAR0 0x66042000000 was reset to apply the new CC mode.
```

> 注意：每开启一个卡CC mode，会导致nvidia-smi丢失该卡的信息。

### nvidia-smi相关命令

```bash
nvidia-smi -r
systemctl restart nvidia-fabricmanager.service
nvidia-smi conf-compute -f
 
nvidia-smi conf-compute -h
 
    conf-compute -- Display Confidential Compute information.
 
    Usage: nvidia-smi conf-compute [options]
 
    Options include:
    [-h | --help]: Display help information
    [-i | --id]: Enumeration index, PCI bus ID or UUID.
                 Provide comma separated values for more than one device.
    [-gc | --get-cpu-caps]: Display Confidential Compute CPU Capability
    [-gg | --get-gpus-caps]: Display Confidential Compute GPUs Capability
    [-d | --get-devtools-mode]: Display Confidential Compute DevTools Mode
    [-e | --get-environment]: Display Confidential Compute Environment
    [-f | --get-cc-feature]: Display Confidential Compute CC feature status
    [-gm | --get-mem-size-info]: Display Confidential Compute GPU Protected/Unprotected Memory Sizes
    [-sm | --set-unprotected-mem-size]: Set Confidential Compute GPU Unprotected Memory Size in KiB
    [-srs | --set-gpus-ready-state]: Set Confidential Compute GPUs Ready State
    [-grs | --get-gpus-ready-state]: Display Confidential Compute GPUs Ready State
```
