---
layout: post
title: "机密计算: Arm CCA"
index_img: /img/cc/arm/index.png
date: 2024-08-05 10:13:06
archive: true
tags: 
    - TEE
    - Confidential Compute
    - Security
categories: 
    - Security
---

介绍 Arm 机密计算架构 (Arm CCA) 如何在 Arm 计算平台中实现机密计算。

<!-- more -->

转自：[Arm机密计算架构技术白皮书](https://community.arm.com/management/archive/cn/b/blog/posts/arm-1203521971)

更多参考：

- [Arm CCA](https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture)
- [Introducing Arm Confidential Compute Architecture](https://developer.arm.com/documentation/den0125/0300/)
- [PDF: Introducing Arm Confidential Compute Architecture Version 3.0](/pdf/cc/introducing_arm_confidential_compute_architecture_den0125_0300_02_en.pdf)
- [【TEE】ARM CCA 可信计算架构](https://blog.csdn.net/qq_43543209/article/details/135659463)

## 什么是机密计算

机密计算是指在基于硬件支持的可信的安全环境中运行计算任务，以此对使用中的数据进行保护。这种保护能让代码和数据免于被特权软件和硬件代理所监控和修改。

在机密计算环境中执行的任何应用程序或操作系统都将独立于系统的其他部分单独执行。任何由此种独立执行所生成或消费的数据，在没有明确许可的情况下，都不会被同一平台上的任何其他角色监视。

本文假设您已经了解 Arm 异常模型和内存管理。若您对这些内容不熟悉，请阅读我们的 [AArch64 异常模型](https://developer.arm.com/documentation/102412/latest)和 [AArch64 内存管理](https://developer.arm.com/documentation/101811/latest)文章。Arm 机密计算架构 (CCA) 中的有些操作涉及到虚拟机和虚拟化。若您对这些概念不熟悉，请参考 [AArch64 虚拟化](https://developer.arm.com/documentation/102142/latest/)。若您还不熟悉关于 Arm 安全性的概念，请参阅[安全性介绍](https://developer.arm.com/documentation/102406/latest/)。

## Arm CCA 相关要求

通过Arm CCA 系统所执行的代码不需要信任所处环境中执行的大型复杂软件栈或者任何可能对其产生影响的外围设备，比如具有DMA功能的设备。Arm CCA 免去了与软件栈或硬件开发者建立许多关联的必要。

安全架构师在云服务器系统上部署负载时，可能并不知道系统 hypervisor 的开发者是谁，这时也许就会考虑 Arm CCA，原因是 hypervisor 是未知的，如果没有 Arm CCA，那么平台上的执行就缺少信任。

Arm CCA 使应用程序开发者能安全可靠地部署工作负载，无需信赖底层的软件基础设施，比如hypervisor，内核，和运行在安全环境的代码

平台若想使用 Arm CCA，必须具备以下条件：

- 一个能隔离所有不可信任者的执行环境
- 一个能将该执行环境初始化为值得信任的状态的机制。此初始化要求这个执行环境有自己的信任链 (Chain of Trust)，而此信任链是独立于平台上并行使用的不可信任环境的信任链。

本文将解释 Arm CCA 是如何通过硬件实施和软件使用来满足这些要求的。

## Arm CCA 扩展

正如什么是机密计算章节中所述，Arm CCA 让您能在部署应用程序或虚拟机的同时，阻止例如 hypervisor 等特权软件的访问。但是，内存等资源通常由这些特权软件来管理。在这种情况下，特权软件（例如 hypervisor ）就具有访问应用程序或虚拟机内存的权限。

Arm CCA 让您能控制虚拟机，但移除了访问虚拟机中使用的代码，寄存器状态和数据的权限。这种隔离是**通过创建受保护的虚拟机执行空间来实现的，又称为 Realm （ 机密领域）。**在代码执行和数据访问方面，Realm 能与 Normal world 完全隔离。Arm CCA 通过将架构硬件扩展和固件相结合实现了这种隔离。

**在 Arm CCA 中，Arm 应用 PE 上的硬件扩展被称为 Realm Management Extension (RME)。 RME 会和被称之为 Realm Management Monitor (RMM) 用来控制 Realm 的专门固件，及在 Exception level 3 中的 Monitor 代码交互。**在文章中的 Arm CCA 硬件架构和 Arm CCA 软件架构章节有这些部件的介绍。

### Realms

Realm 是一种 Arm CCA 环境，能被 Normal world 主机动态分配。主机是指能管理应用程序或虚拟机的监控软件。Realm 及其所在平台的初始化状态都可以得到验证。这一过程使 Realm 的所有者能在向它提供任何机密前就建立信任。因此，Realm 不必继承来自控制它的 Non-secure hypervisor 的信任。

**主机可以分配和管理资源配置。主机还可以管理调度 Realm 虚拟机。然而，主机不可以监控或修改 Realm 执行的指令。**在主机控制下，Realm 可以被创建并被销毁。通过主机请求，可以增加或移除页面，这与 hypervisor 管理任何其他非机密虚拟机的操作方式类似。为了运行 CCA 系统，需要对主机进行修改。主机可以继续控制非机密虚拟机，但需要与 Arm CCA 固件交互，特别是 Realm Management Monitor (RMM)。RMM 的运行可以参考 Arm CCA 软件扩展章节。

### Realm world 和 Root world

Armv8-A TrustZone 扩展拥有两个独立的环境，分别是 Secure world 和 Normal world ，支持代码的安全执行和数据的隔离。这里 World 是由 PE 的安全状态和物理地址空间组成。PE 执行时的安全状态会决定 PE 能访问哪种物理地址空间。**在安全状态下，PE 可以访问安全和非安全的物理地址空间，但是在非安全的状态下，它只能访问非安全的物理地址空间。** Normal world 一般用来指非安全的状态和非安全的物理地址空间的组合。

Arm CCA 是 Armv9-A 的一部分，引入了 Realm Management Extension (RME)。该扩展引入了两个额外的 World，分别是 Realm world 和 Root world：

- Root world 引入了 Root 安全状态和 Root 物理地址空间。PE 在 Exception level 3运行时，处于 Root 安全状态。Root PA 与 Secure PA 是分开的。这是与 Armv8-A TrustZone 的主要区别，Exception level 3 代码之前是没有私有地址空间的，而是使用的 Secure PA。Secure PA仍然用于 S_EL2/1/0。Monitor 代码运行在 Root World。
- Realm world 与 TrustZone 技术下的安全环境类似。Realm world 包括 Realm 安全状态和 Realm PA。Realm 状态代码可以在 R_EL2, R_EL1 和 R_EL0 上执行。在 Realm world 中运行的控制固件可以访问 Normal world 中的内存，支持共享缓冲。

下图说明了四个基于 RME 的 world 以及它们与 SCR_EL3 NS 和 NSE 位之间的关系：

![Arm CCA 四个 world 的位置](/img/cc/arm/cc1.png)

Root world 支持可信的启动执行和不同环境之间的切换。PE 复位到 Root world中。

Realm world 为虚拟机提供了一个与 Normal 和 Secure world 隔离的执行环境。虚拟机需要 Normal world 中主机的控制。为能全面控制 Realm 创建和执行，Arm CCA 系统将提供：

- Realm Management Extension：这是底层架构所要求的硬件扩展，让隔离的 Realm 虚拟机得以运行
- Realm Management Monitor：这属于固件的一部分，用于按照 Normal world 主机的请求来管理 Realm 的创建和执行

在 Arm CCA 硬件架构和 Arm CCA 软件架构的章节中，我们会对这些部件进行更详细的介绍。

在不支持 RME 的 PE 中的 world 切换由 SCR_EL3.NS 位控制。在切换为 Secure world 时，Exception level 3 软件设置 NS 为 0，切换为 Normal world 时设置 NS 为 1。支持 RME 的 PE 中进行 world 切换是通过在 SCR_EL3 寄存器增加一个新的 SCR_EL3.NSE 位来扩展的。

下表显示了“这些寄存器位”是如何控制四个环境之间的执行和访问：

![表 1：World 控制和状态位](/img/cc/arm/cc2.png)

### Arm TrustZone 扩展和 Arm RME 之间的区别

所有 Arm A系列的处理器都可以选择实施 Arm TrustZone 架构扩展。这些扩展支持开发隔离的执行和数据环境。可信操作系统（Trusted Operating System, TOS）等组件可以服务在隔离环境中执行的可信应用程序，响应 Normal world 中运行的 Rich OS 的安全服务器请求。

为 Armv8.4-A 中的 Secure world 增加虚拟化技术，使您能够管理 Secure world 的多个安全分区。这项功能可以支持多个 TOS 被应用于一个系统中。安全分区管理器（Secure Partition Manager, SPM）在 S_EL2 上执行，是安全分区的管理者。SPM 的功能类似于 Normal world 中的 hypervisor 。

操作上，TOS 经常作为信任链的一部分，它由更高特权的固件进行验证，这在有些系统中可能是 SPM。这意味着 TOS 依赖于与更高特权固件的开发者的关联。

有两种方法可以用来触发 TOS 的执行：

- Rich OS 让出，Rich OS 进入闲置循环，并执行 SMC 指令通过 Monitor 调用 TOS
- 指定给 Trusted OS 的中断。Secure type 1 中断被用于 TOS 的执行。Secure type 1 中断在 Normal world 执行期间通过 Monitor 来调用 TOS。

Realm 虚拟机不同于 TOS 或可信应用程序，因为 Realm 虚拟机是通过 Normal world 主机控制的。在创建和内存配置等方面，Realm 虚拟机就像被主机控制的任何其他虚拟机一样。

Realm 虚拟机执行和 TOS 执行之间的区别在于 Realm 并没有启用任何物理中断。所有 Realm 中断都通过 hypervisor 来进行虚拟化，然后通过传给 RMM 的命令来传送信号给 Realm。这就意味着受到破坏的 hypervisor 会阻止 Realm 虚拟机的执行，这样一来，并不能保证 Realm 的执行。

Realm 执行和内存访问由负责控制的主机软件进行初始化，例如 hypervisor 。Realm 并不一定要由主机进行验证。Realm 可以绕过任何信任链，因为它可以使用 RME 初始化证明。有关更多信息，请参阅本文的证明章节。Realm 能完全独立于控制软件。当 Realm 被主机初始化后，主机无法看见它的数据或数据内存。

Realm 和 TOS 在使用上的主要区别在于“**Secure 执行**”和“**Realm 执行**”两者设计目的不同。

可信应用程序用于平台的特定服务，这些服务都是与系统开发密切相关的参与者（如芯片供应商和OEM）所拥有的。

Realm 执行的目的是允许一般开发者在系统上执行代码时，无需涉及与计算系统开发者之间复杂的商业关系。

Arm CCA 使 Realm 在 Normal world 主机的控制下按照需求被创建和销毁。可以从 Realm 动态增加或收回资源。这使得隔离的好处能够惠及更多应用程序和用户，而不只是与设备绑定的服务。

信任的定义体现在机密性、完整性和真实性，具体解释如下：

- 在机密性方面，Arm CCA 环境的代码数据或状态无法被同一设备上运行的其他软件监视，即使这个软件具有更高的特权
- 在完整性方面，Arm CCA 环境的代码数据或状态无法被同一设备上运行的其他软件修改，即便这个软件具有更高的特权
- 在真实性方面，代码或数据可被运行在同一设备上的其他软件修改，但任何改动都能被识别

可信应用程序和 TOS 为系统提供机密性、完整性和真实性。 Realm 执行可为系统提供机密性和完整性。

Arm CCA 提供的四个 world 使 Secure world 和 Realm world 之间完全隔离开。这意味着可信应用程序无需担心任何 Realm 虚拟机的执行， Realm虚拟机也无需担心任何被执行的可信应用程序。

## Arm CCA 硬件架构

这一章节主要介绍 Realm Management Extension，即对 PE 架构的改动，使 PE 能运行 Realm。

### Realm world 的要求

下图完整说明了 Realm 是如何配合 Arm CCA 系统的：

![Realm world 软件执行](/img/cc/arm/cc3.png)

Realm world 必须能执行代码并访问内存和可信的设备，与所有其他非 Root world 和设备完全隔离开。

就像其他 world 或安全状态一样， Realm world 也有三个异常等级，分别是 R_EL0、R_EL1 和 R_EL2。 Realm虚拟机在 R_EL1 和 R_EL0 上运行。 Realm Management Monitor (RMM) 在 R_EL2 上运行。RMM 的细节可以参照本文 Arm CCA 软件栈的章节。

隔离是由 Realm Management Extension 构架扩展的硬件强制的，允许对内存管理，执行，Realm 的上下文和数据隔离进行控制。隔离是指通过 PE 的 fault 异常，或为 Realm 和 Root world 加密来阻止访问。在下图中，隔离的 Realm 虚拟机在 Normal world 由 hypervisor 生成并控制，但物理执行则在 Realm world：

![Realm 虚拟机执行](/img/cc/arm/cc4.png)

Realm 虚拟机的执行通过 hypervisor 命令初始化，这些命令被传达到 Monitor，然后通过 Monitor 推送到 RMM。

Monitor 在不同的 world 之间扮演着守门人的角色。它控制着 Normal，Secure 和 Realm world 之间的通道，确保各个 world 之间的隔离能一直得以维持，同时在需要的地方支持通信和控制。

## Arm CCA 内存管理

实施 TrustZone 安全扩展的 Arm A 系列处理器具有两个物理地址空间 (PAS)：

- Non-Secure 物理地址空间
- Secure 物理地址空间

Realm Management Extension 增加了另外两个物理地址空间：

- Realm 物理地址空间
- Root 物理地址空间

下图描述了这些物理地址空间以及如何在工作系统中实施这些空间：

![物理地址空间](/img/cc/arm/cc5.png)

不同 world 之间的 PAS 访问由硬件强制执行，见下表：

![物理地址访问权限](/img/cc/arm/cc6.png)

正如上图所示，**Root 状态可访问所有物理地址空间**。Root 状态支持非安全 PAS 与安全或 Realm PAS 之间所需的内存转换。

为确保这些针对所有 world 的隔离规则被强制执行，前文表格中提到的物理内存访问控制由 MMU 强制执行，在任何地址转换后执行。此一过程称为 Granule Protection Check (GPC)。

Granule Protection Table (GPT) 描述了每个内存粒度的PAS分配信息。Exception level 3 中的 Monitor 可以动态更新 GPT，支持物理内存在各个 world 间移动。

任何访问控制违规都会导致一种新的 fault，称为 Granule Protection Fault (GPF)。GPC 的使能、GPT 的内容和 GPF 的定向都受 Root 状态的控制。

属于 Realm 的资源必须存放于 Realm 的私有内存中，表示作为 Realm PAS 的一部分来确保隔离。然而，Realm 也许会需要访问非安全内存中的一些资源，比如支持消息传递。这意味着 Realm 需要能同时访问 Realm PAS 和 Non-Secure PAS 中的物理地址。

任何 Realm 虚拟机都在 Realm 环境中执行，但受到 Normal world 主机的控制。 Realm 虚拟机将需要能访问 Normal world PAS 和 Realm world PAS 中的内存。访问不同的 PAS 由 Realm stage 2 页表中 NS 位的状态控制。

下图显示了虚拟地址 (VA) 到物理地址 (PA) 链中 GPC 的全部阶段和位置。图中，TTD 是指 Translation Table Descriptor ，而 GPTD 指的是 Granule Protection Table Descriptor：

![Granule protection check](/img/cc/arm/cc7.png)

此图显示了具有 RME 的平台的虚拟地址到物理地址的转换阶段。

如需了解 stage 1 和 stage 2 转换的详细信息，请参阅 [AARCH64 内存管理](https://developer.arm.com/documentation/101811/latest)。但是，在这个例子中，Exception level 3 的 stage 2 页表条目定义了一个额外的位，目的是允许通过 Monitor 访问四个 PAS。这就是转换表定义中的非安全扩展（Non-secure Extension, NSE）位。

Normal world，Realm world 和 Secure world 都可以有 Exception level 1，Exception level 2 的 stage 1 转换，如有必要，Exception level 2 还可以有 stage 2 转换。

RME 在 stage 1 和 stage 2 转换过程后增加了GPC。GPC 对照 GPT 检查所有物理地址和 PAS，目的是允许访问或是产生 fault。GPT 保存Root 内存中，以确保它能与所有其他 world 隔离开。GPT 只可以通过在 Root world 运行的代码从 Monitor 代码或可信固件创建和修改。

非 PE 请求者如果连接到系统内存管理单元 (SMMU) 这样的请求者侧筛选器上，也将被包括在这项检查中。

### 证明

Realm 内运行的代码将管理机密数据或运行机密算法。因此，这些代码需要确保正在运行真正的 Arm CCA 平台，而不是冒充者。这些代码还需要知道自己已经被正确地加载，没有遭到篡改。此外，这些代码还需要知道整个平台或 Realm 并不处于可能导致机密泄露的调试状态。建立这种信任的过程被称为“证明”。

证明分成两个关键部分：

- 平台的证明
- Realm 初始化状态的证明

这两部分共同形成证明报告，表明 Realm 代码可以随时提出请求。之后这些报告可被用来鉴别平台和 Realm 代码的有效性。在平台证明中，需要证明芯片和固件这些构成 Realm 的基础部件是真实可靠的。而这便对硬件产生要求。需要向硬件分配一个身份。同样地，硬件需要支持关键固件镜像进行度量，如 Monitor、RMM 和平台上任何其它能极大影响安全性的控制器固件，如功耗控制器。

## Arm CCA 软件架构

Arm CCA平台是由许多额外的硬件组合（如 PE 中的 RME）和固件组件（特别是 Monitor 和 Realm Management Monitor ）组成。这一章节主要介绍面向 Arm CCA 平台的软件栈。

### 软件栈概述

Realm 虚拟机的执行旨在与 Normal world 隔离，它由 Normal world 主机负责初始化和控制。为支持 Realm 虚拟机的隔离执行，需要实现一个 hypervisor（运行在 Normal world Exception Level 2 的主机上）和 Realm 虚拟机（运行在 Realm world Exception level 2 或 R_EL2 上）之间的通讯栈。

RMM 负责管理通信和上下文切换。RMM 并不作策略决定，如将要运行哪种 Realm 或给 Realm 分配什么内存。这些依然是由主机 hypervisor 发出的命令。

RMM 通过 Realm world  stage 2 页表来支持各个 Realm 的彼此隔离。

RMM 直接与 Monitor 对接，后者又与 Secure world 和 Normal world 对接。在 Exception level 3 上运行的Monitor 具有平台特有的代码，这些必须服务于系统的所有可信功能。RMM 响应特定的接口，将通过全面定义的功能来管理来自主机和 Monitor 的请求。由于这个接口将定义明确，RMM 因此可以成为所有 Arm CCA 系统的通用代码。

下图展示了在 Realm world 中运行机密性的 Realm 虚拟机的整个 Arm CCA 平台：

![Realm 虚拟机执行](/img/cc/arm/cc8.png)

RMM 是 Realm world 的控制软件，能响应 Normal world hypervisor 的请求，支持对 Realm 虚拟机执行进行管理。RMM 通过 Root world 中的 Monitor 进行通信，从而控制 Realm 内存管理功能，实现 NS PAS 和 Realm PAS 之间的内存转换。

SMC 指令允许 RMM、 hypervisor 和 SPM 将控制权交给 Monitor 将控制器交给 Monitor，支持在所有 Exception level 2 软件和 Monitor 之间实现的通道。下图说明了Monitor 和各个 world 的不同控制软件间的通道：

![通过 SMC 通道实现 world 之间的通信](/img/cc/arm/cc9.png)

在每个 world 中，Exception level 2 执行可以通过执行 SMC 指令对 Exception level 3 的 Monitor 进行调用。SMC 指令的使用为个别 Exception 二级控制主机和 Exception level 3 Monitor之间的通信信道建立了基础。这就是 Normal world  Exception level 2上的主机通过 Monitor 与 Realm world Exception level 2 上的 RMM 建立通信的方法。

### Realm Management Monitor

RMM 是 Realm world 固件，用来管理 Realm虚 拟机的执行以及它们与 Normal world hypervisor 的交互。RMM 在 Realm world 的 Exception level 2 运行，也就是 R_EL2。

RMM 在 Arm CCA 系统中有两个职责，一是为主机提供服务，使主机能管理 Realm；二是 RME 直接向 Realm 提供服务。

主机服务可分为策略和机制两方面。

在策略的功能方面，RMM 对下列情况拥有全面的决策：

- 何时创建或销毁 Realm
- 何时为 Realm 增加或移除内存
- 调入调出 Realm 的时机

RMM 通过提供下列功能来支持主机的策略：

- 提供 Realm 页表操作服务，用于创建或销毁 Realm，以及 Realm 内存的添加或移除
- Realm 上下文的管理，用于调度过程中的上下文保存和恢复。
- 中断支持
- PSCI 调用截获，属于功耗管理请求。RMM 也向 Realm 提供服务，主要是证明和加密服务

此外，RMM 还支持以下针对 Realm 的安全性原语：

- RMM 验证主机的请求是否正确
- RMM 实现 Realm 彼此间的隔离

RMM 规范定义了两个通信通道，允许所有功能在 Normal world 主机和 Realm 虚拟机之间进行请求和控制。从主机到 RMM 的通信信道被称为 **Realm management Interface (RMI)**。RMM 和 Realm 虚拟机之间定义的第二个信道称为 **Realm Service Interface (RSI)**。RSI 是针对 RMM 请求服务的通信通道。

在下面的章节中，我们将介绍每个接口。

### Realm Management Interface

- RMI 是 RMM 和 Normal world 主机之间的接口。
- RMI 允许 Normal world hypervisor 向 RMM 发出命令来管理 Realm。RMI 利用主机 hypervisor 的 SMC 调用来请求 RMM 的管理控制。
- RMI 支持对 Realm 管理的控制，包括 Realm 的创建、填充、执行和销毁。

下图说明了 RMI 用在 Normal world 主机、Monitor 和 RMM 之间的哪些地方：

![Realm Management Interface 的路径](/img/cc/arm/cc10.png)

### Realm Services Interface

RSI 是 Realm 虚拟机和 RMM 之间的接口。RSI 为外部服务提供了一个通道，一些 Realm 管理操作需要从 RMM 传给 Realm。这些服务可以包括加密服务和证明。RSI 还为从 Realm 虚拟机到 RMM 的内存管理请求提供了通道。

下图标出了 RSI 在 RMM 和每个独立的 Realm 虚拟机之间的位置：

![Realm Service Interface](/img/cc/arm/cc11.png)
