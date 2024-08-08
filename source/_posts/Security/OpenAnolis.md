---
layout: post
title: "龙蜥社区: 云原生机密计算最佳实践"
index_img: /img/openanolis/index.png
date: 2024-07-29 14:37:13
archive: true
tags:
    - TEE
    - Confidential Compute
    - Security
    - OS
categories: 
    - Security
---

龙蜥社区（OpenAnolis）成立于2020年9月，由阿里云、ARM、统信软件、龙芯、飞腾、中科方德、Intel等24家国内外头部企业共同成立龙蜥社区理事会，到目前有超过300家合作伙伴参与共建，是是国内领先的开源社区之一。

<!-- more -->

- [Hygon Arch](https://openanolis.cn/sig/Hygon-Arch)
- [Cloud Native Confidential Computing SIG](https://openanolis.cn/sig/coco)

<iframe src="/pdf/cc/2023云原生机密计算最佳实践白皮书.pdf" width="100%" height="800" name="topFrame" scrolling="yes"  noresize="noresize" frameborder="0" id="topFrame"></iframe>

# 龙蜥简介

龙蜥操作系统（AnolisOS）搭载了ANCK版本的内核，性能和稳定性经过历年“双11”历练，能为云上典型用户场景带来40%的综合性能提升，故障率降低50%，兼容CentOS生态，提供平滑的CentOS迁移方案，并提供全栈国密能力。最新的长期支持版本AnolisOS8.6已发布，更多龙蜥自研，支持X86_64、RISC-V、Arm64、LoongArch架构，完善适配Intel、飞腾、海光、兆芯、鲲鹏、龙芯等主流芯片。

介绍：<https://openanolis.cn/anolisos>
下载：<https://openanolis.cn/download>

# 机密计算简介与现状

## 数据安全与机密计算

数据在整个生命周期有三种状态：At-Rest（静态）、In-Transit（传输中）和In-Use（使用中）。

- **At-Rest**状态下，一般会把数据存放在硬盘、闪存或其他的存储设备中。保护At-Rest状态的数据有很多方法，比如对文件加密后再存放或者对存储设备加密。
- **In-Transit**是指通过公网或私网把数据从一个地方传输到其他地方，用户可以在传输之前对文件加密或者采用安全的传输协议保证数据在传输中的安全，比如HTTPS、SSL、TLS、FTPS等。
- **In-Use**是指正在使用的数据。即便数据在传输过程中是被加密的，但只有把数据解密后才能进行计算和使用。也就意味着，如果数据在使用时没有被保护的话，仍然有数据泄露和被篡改的风险。

如今被广泛使用的加密技术可以用来提供数据机密性（防止未经授权的访问）和数据完整性（防止或检测未经授权的修改），但目前这些技术主要被用于保护传输中和静止状态的数据，目前对数据的第三个状态“使用中”提供安全防护的技术仍旧属于新的前沿领域。

机密计算指使用基于硬件的可信执行环境（TrustedExecutionEnvironment，TEE）对使用中的数据提供保护。通过使用机密计算，我们现在能够针对“使用中”的数据提供保护。

机密计算的核心功能有：

- 保护In-Use数据的机密性。未经授权的实体（主机上的应用程序、主机操作系统和Hypervisor、系统管理员或对硬件具有物理访问权限的任何其他人。）无法查看在TEE中使用的数据，内存中的数据是被加密的，即便被攻击者窃取到内存数据也不会泄露数据。
- 保护In-Use数据的完整性。防止未经授权的实体篡改正在处理中的数据，度量值保证了数据和代码的完整性，使用中有任何数据或代码的改动都会引起度量值的变化。
- 可证明性。通常TEE可以提供其起源和当前状态的证据或度量值，以便让另一方进行验证，并决定是否信任TEE中运行的代码。最重要的是，此类证据是由硬件签名，并且制造商能够提供证明，因此验证证据的一方就可以在一定程度上保证证据是可靠的，而不是由恶意软件或其他未经授权的实体生成的。

## 机密计算的现状与困境

目前机密计算正处于百花齐发和百家争鸣的阶段，市场和商业化潜力非常巨大。但机密计算在云原生场景中还有一些不足：

- 用户心智不足。用户普遍对机密计算这项新技术的认知感不足，难以将其与自己的业务直接联系起来，导致需求不够旺盛。
- 技术门槛高。目前，相比传统开发方式，主流的机密计算技术的编程模型给人们对机密计算技术的印象是学习和使用门槛高，用户需要使用机密计算技术对业务进行改造，令很多开发者望而生畏。
- 应用场景缺乏普适性。目前，机密计算主要被应用于具有特定行业壁垒或行业特征的场景，如隐私计算和金融等。这些复杂场景让普通用户很难触达机密计算技术，也难以为普通用户打造典型应用场景。同同厂商的CPUTEE虽各自具有自身的特点，但都无法解决异构计算算力不足的问题，限制了机密计算的应用领域。
- 信任根和信任模型问题。在信创、数据安全和安全合规等政策性要求对CPUTEE的信任根存在自主可控的诉求；与此同时，虽然有部分用户愿意信任云厂商和第三方提供的解决方案，但多数用户对云厂商和第三方不完全信任，要求将机密计算技术方案从租户TCB中完全移除。

总之，目前已有的机密计算技术方案存在以上困境，不能够完全满足用户不同场景的安全需求。为了解决以上四个问题，云原生机密计算SIG应运而生，主要可概括为四点：

1. 推广机密计算技术。

   - 邀请参与方在龙蜥大讲堂介绍和推广机密计算技术与解决方案。
   - 与芯片厂商合作，未来可以通过龙蜥实验室让外部用户体验机密计算技术，对机密计算有一个更深入化的了解。

2. 提高机密计算技术的可用性。

   - 支持多种机密计算硬件。
   - 提供多种运行时底座和编程框架供用户选择。

3. 提升机密计算技术的泛用性

   - 为最有代表性的通用计算场景打造解决方案和案例（特性即产品）。
   - 积极拥抱并参与到机密计算前沿技术领域的探索与实践，加速创新技术的落地。

4. 澄清误会并增加用户信心

   - 发布机密计算技术白皮书。
   - 与社区和业界合作，未来提供结合了软件供应链安全的远程证明服务体系。

# 云原生机密计算SIG概述

如何在保障安全的前提下最大程度发挥数据的价值，是当前面临的重要课题。我们将此范式称为隐私保护云计算，而机密计算是实现隐私保护云计算的必由之路。为拥抱隐私保护云计算新范式，促进隐私保护云计算生态发展，云原生机密计算SIG应运而生

| 类型     | 内容                       |
| -------- | ----------------------------------------------------------- |
| 解决方案 | Intel CCZoo (TensorFlow 横向联邦学习、在线推理服务、隐私集合求交)     |
| 运行时   | Gramine / 海光 CSV 机密容器 & 机密虚拟机 / Inclavare Containers / KubeTEE Enclave Services / Occlum / SEV 机密容器 & 机密虚拟机 / SGX 虚拟化 / TDX 机密容器 & 机密虚拟机 |
| 编程框架 | Apache Teaclave Java TEE SDK / Intel HE Toolkit / Intel SGX & PSW & DCAP    |
| OS适配   | Anolis 8 + ANCK 5.10                           |
| 硬件支持 | AMD SEV(-ES)/ 海光 CSV 1+2 / Intel SGX 2.0 / Intel TDX 1.0       |

云原生机密计算SIG的愿景是：

- 构建安全、易用的机密计算技术栈
- 适配各种常见机密计算硬件平台
- 打造典型机密计算产品和应用案例

## 项目介绍

### 海光 CSV 机密容器

CSV 是海光研发的安全虚拟化技术。CSV1 实现了虚拟机内存加密能力，CSV2 增加了虚拟机状态加密机制，CSV3 进一步提供了虚拟机内存隔离支持。CSV 机密容器能够为用户提供虚拟机内存加密和虚拟机状态加密能力，主机无法解密获取虚拟机的加密内存和加密状态信息。CSV 虚拟机使用隔离的 TLB、Cache 等硬件资源，支持安全启动、代码验证、远程认证等功能。

- 主页：<https://openanolis.cn/sig/coco/doc/533508829133259244>

### Intel Confidential Computing Zoo

Intel Confidential Computing Zoo Intel 发起并开源了 Confidential Computing Zoo (CCZoo)，CCZoo 基于 Intel TEE（SGX,TDX）技术，提供了不同场景下各种典型端到端安全解决方案的参考案例，增加用户在机密计算方案实现上的开发体验，并引导用户结合参考案例快速设计自己特定的机密计算解决方案。 CCZoo 目前提供了基于 Libos + Intel TEE + OpenAnolis 容器的 E2E 安全解决方案参考案例，后续，CCZoo 计划基于 OpenAnolis ，提供更多的机密计算参考案例，为用户提供相应的容器镜像，实现敏捷部署。

- 主页：<https://cczoo.readthedocs.io>
- 代码库：<https://github.com/intel/confidential-computing-zoo>

### Intel HE Toolkit

Intel HE Toolkit 旨在为社区和行业提供一个用于实验、开发和部署同态加密应用的平台。目前 Intel HE Toolkit 包括了主流的 Leveled HE 库，如 SEAL、Palisade和 HELib，基于使能了英特尔最新指令集加速的的 Intel HEXL 库，在英特尔至强处理器平台上为同态加密业务负载提供了卓越的性能体验。同时，Intel HE Toolkit即将集成半同态 Paillier 加速库 IPCL，为半同态加密应用提供加速支持。此外，Intel HE Toolkit 还提供了示例内核、示例程序和基准测试 HEBench。这些示例程序演示了利用主流的同态加密库构建各种同态加密应用保护用户隐私数据的能力。HEBench 则为各类第三方同态加密应用提供了公允的评价基准，促进了同态加密领域的研究与创新。

- 主页：<https://www.intel.com/content/www/us/en/developer/tools/homoorphicencryption/>
- 代码库：
  - Intel HE Toolkit: <https://github.com/intel/he-toolkit>
  - Intel HEXL: <https://github.com/intel/hexl>
  - Intel Paillier Cryptosystem Library (IPCL): <https://github.com/intel/pailliercryptolib>
  - HE Bench: <https://github.com/hebench>

### Intel SGX Platform Software and Datacenter Attestation Primitives

在龙蜥生态中为数据中心和云计算平台提供 Intel SGX 技术所需的平台软件服务，如远程证明等。

- RPM包：<https://download.01.org/intel-sgx/latest/linux-latest/distro/Anolis86/>
- 代码库：<https://github.com/intel/SGXDataCenterAttestationPrimitives>

### Intel SGX SDK

在龙蜥生态中为开发者提供使用 Intel SGX 技术所需的软件开发套件，帮助开发者高效便捷地开发机密计算程序和解决方案。

- RPM包：<https://download.01.org/intel-sgx/latest/linux-latest/distro/Anolis86/>
- 代码库：<https://github.com/intel/linux-sgx>

### Occlum

Occlum 是一个 TEE LibOS，是机密计算联盟（CCC, Confidential Computing Consortium）的官方开源项目。目前 Occlum 支持 Intel SGX 和 HyperEnclave 两种 TEE。Occlum 在 TEE 环境中提供了一个兼容 Linux 的运行环境，使得 Linux 下的应用可以不经修改就在 TEE 环境中运行。Occlum 在设计时将安全性作为最重要的设计指标，在提升用户开发效率的同时保证了应用的安全性。Occlum 极大地降低了程序员开发 TEE 安全应用的难度，提升了开发效率。

- 主页：<https://occlum.io/>
- 代码库：<https://github.com/occlum/occlum>

提供TEE有关的Kubernetes基础服务(如集群规模的密钥分发和同步服务、集群远程证明服务等），使得用户可以方便地将集群中多台TEE机器当作一个更强大的TEE来使用。

- 代码库：<https://github.com/SOFAEnclave/KubeTEE>

### Apache Teaclave Java TEE SDK

Apache Teaclave Java TEE SDK(JavaEnclave)是一个面向Java生态的机密计算编程框架，它继承Intel SGX SDK所定义的 Host-Enclave 机密计算分割编程模型。JavaEnclave 提供一种十分优雅的模式，对一个完整的 Java 应用程序进行分割与组织。它将一个 Java 项目划分成三个子模块，Common 子模块定义 SPI 服务接口，Enclave 子模块实现 SPI 接口并以 Provider 方式提供服务，Host 子模块负责 TEE 环境的管理和 Enclave 机密服务的调用。整个机密计算应用的开发与使用模式符合 Java 经典的 SPI 设计模式，极大降低了 Java 机密计算开发门槛。此外，本框架创新性应用 Java 静态编译技术，将 Enclave 子模块 Java 代码编译成 Native 形态并运行在TEE环境，极大减小了 Enclave 攻击面，杜绝了 Enclave 发生注入攻击的风险，实现了极致安全的 Java 机密计算运行环境。

- 主页:<https://teaclave.apache.org>
- 代码库:<https://github.com/apache/incubator-teaclave-java-tee-sdk>

### Gramine

Gramine 是一个轻量级的 LibOS，旨在以最小的主机要求运行单个应用程序。Gramine 可以在一个隔离的环境中运行应用程序。其优点是可定制，易移植，方便迁移，可以媲美虚拟机。在架构上 Gramine 可以在任何平台上支持运行未修改的 Linux 二进制文件。目前，Gramine 可以在 Linux 和 Intel SGX enclave 环境中工作。

- 主页：<https://gramine.readthedocs.io/>
- 代码库：<https://github.com/gramineproject/gramine>

### TDX机密容器&机密虚拟机

Intel Trust Domain Extension(TDX) 基于虚拟化扩展机密计算的隔离能力，通过构建机密虚拟机，为业务负载提供了虚拟机级别的机密计算的运行环境。通过 Linux 社区对 TDX 机密虚拟机的生态支持,基于 Linux 的业务应用可以方便的迁移到机密计算环境中。此外，Intel TDX Pod级机密容器将TDX机密虚拟机技术同容器生态无缝集成，以云原生方式运行，保护敏感工作负载和数据的机密性和完整性。在机密虚拟机内部，默认集成了 image-rs 和 attestation-agent 等组件，实现了容器镜像的拉取、授权、验签、解密、远程证明以及秘密注入等安全特性。

# 机密计算平台

## 海光CSV：海光安全虚拟化技术

- <https://gitee.com/anolis/cloud-kernel>
- <https://gitee.com/anolis/hygon-edk2>
- <https://gitee.com/anolis/hygon-qemu>
- <https://github.com/inclavare-containers/librats>
- <https://github.com/inclavare-containers/rats-tls>

容器技术的出现，使应用程序的打包、分发变得非常简单易用，Kubernetes 等容器编排技术的出现，进一步加速了容器生态的普及和发展，目前容器已经逐渐成为云计算的主要运行单元。但是由于传统容器共享操作系统内核，在隔离性和安全性上比传统虚拟机差。为了解决这个问题，Kata 容器应运而生，Kata 容器运行在轻量级虚拟机里，比起传统容器提供了更好的隔离性和安全性，使 Kata 容器同时具有容器技术带来的易用性和虚拟机技术带来的安全性。随着机密计算需求的出现，CPU 厂商纷纷推出了硬件 TEE 技术，传统虚拟机技术已无法满足机密计算的需要，Kata 容器的安全性需要进一步增强以便应用于机密计算场景。

虚拟化是云计算的底层基础技术，随着云计算的发展而被广泛应用。由于虚拟机的全部资源被主机操作系统和虚拟机管理器管理和控制，虚拟化本身有较严重的安全缺陷，主机操作系统和虚拟机管理器可任意读取和修改虚拟机资源且虚拟机无法察觉。主机操作系统和虚拟机管理器有权读写虚拟机代码段，虚拟机内存数据，虚拟机磁盘数据，并有权重映射虚拟机内存等。攻击者可利用主机操作系统和虚拟机管理器的安全缺陷获取操作系统的权限后攻击虚拟机，给虚拟机最终用户造成重大损失。

CSV 是海光自主研发的安全虚拟化技术，采用国密算法实现，CSV 虚拟机在写内存数据时 CPU 硬件自动加密，读内存数据时硬件自动解密，每个 CSV 虚拟机使用不同的密钥。海光 CPU 内部使用 ASID（AddressSpaceID）区分不同的 CSV 虚拟机和主机，每个 CSV 虚拟机使用独立的 Cache、TLB 等 CPU 资源，实现 CSV 虚拟机、主机之间的资源隔离。CSV 虚拟机使用隔离的硬件资源，支持启动度量、远程认证等功能，是安全的硬件可信执行环境。

![海光CSV](/img/openanolis/hygon_csv.png)

CSV机密容器技术将安全虚拟化技术与Kata容器技术结合，实现容器运行环境的度量和加密，容器中的程序可以使用远程认证功能实现身份证明。CSV机密容器和普通容器的接口完全兼容，用户可以使用Docker或者Kubernetes启动机密容器，实现对容器数据的隔离和保护。CSV技术构建了以安全加密虚拟机为基础的可信执行环境。在安全加密虚拟机保证了虚拟机数据机密性的基础上，更进一步保证了虚拟机数据的完整性，主机操作系统和虚拟机管理无法通过改写虚拟机嵌套页表对虚拟机实施重映射攻击。

安全加密虚拟化可以保证最终用户数据的机密性和完整性，可用于实施机密计算，适用于云计算和隐私计算场景。

## ARM CCA: Arm安全加密虚拟化技术

- Veracuz: https://github.com/veracruz-project/veracruz
- VERAISON-VERificAtIon of atteStatiON: https://github.com/veraison/veraison

TrustZone是Arm为设备安全提供的一个安全架构，通过硬件隔离和权限分层的方式将系统内分为安全世界（Secureworld）和正常世界（Normal/Non-Secureworld）。

在安全环境中，通过底层硬件隔离，不同执行级别，安全鉴权方式等方式，从最根本的安全机制上提供基于信任根（Root of Trust）的可信执行环境TEE（Trusted Execution Environment）,通过可信服务（Trusted Services）接口与和通用环境REE（Rich Execution Environment）进行安全通信，可以保护 TEE 中的安全内容不能被非安全环境的任何软件，包括操作系统底层软件等所访问，窃取，篡改和伪造等。因此一些安全的私密数据，比如一些安全密钥，密码，指纹以及人脸数据等都是可以放在安全世界的数据区中进行保护。当前，Trustzone机制已经非常成熟稳定，并得到大规模的应用，并以开源的方式给业界提供实现参考。可以访问 https://www.trustedfirmware.org/ 获取更多信息。

然而，TrustZone 所提供的安全机制和 TEE 环境只能提供硬件级别的安全隔离。通常情况下，安全物理地址空间的内存在系统引导时静态分配，适用于数量有限的平台。对于大规模云服务器的机密计算，旨在允许任何第三方开发人员保护他们的虚拟机（VM）或应用程序，必须能够在运行时保护与VM或应用程序关联的任何内存，而不受限制或分割。

Arm CCA 引入了一种新的机密计算世界：机密领域（Realm）。在ArmCCA中，硬件扩展被称为Realm Management Extension(RME)，RME会和被称之为机密领域管理监控器(Realm Management Monitor,RMM)，用来控制机密领域的专用固件，及在Exception level3中的 Monitor 代码交互。Realm 是一种 Arm CCA 环境，能被 Normal world 主机动态分配。主机是指能管理应用程序或虚拟机的监控软件。Realm 及其所在平台的初始化状态都可以得到验证。这一过程使 Realm 的所有者能在向它提供任何机密前就建立信任。因此，Realm 不必继承来自控制它的 Non-secure hypervisor 的信任。主机可以分配和管理资源配置,管理调度 Realm 虚拟机。然而，主机不可以监控或修改 Realm 执行的指令。在主机控制下，Realm 可以被创建并被销毁。通过主机请求，可以增加或移除页面，这与 hypervisor 管理任何其他非机密虚拟机的操作方式类似。

![ARM CCA](/img/openanolis/cca.png)

ArmCCA技术能够从根本上解决用户敏感应用数据的安全计算问题。它充分利用软硬件实现的信任根提供的数据和程序的物理隔离、保护、通信和认证体系，并在传统TrustZone的基础上，增加了被称为领域（Realm）的隔离区域，从最底层的安全机制和原理上解决用户程序和数据的隔离需求。

Realm 内运行的代码将管理机密数据或运行机密算法，这些代码需要确保正在运行真正的 Arm CCA 平台，而不是冒充者。这些代码还需要知道自己已经被正确地加载，没有遭到篡改。并且，这些代码还需要知道整个平台或 Realm 并不处于可能导致机密泄露的调试状态。建立这种信任的过程被称为“证明”。ARM 正在与包括机密计算联盟成员在内的主要行业合作伙伴合作，定义这一证明机制的属性，确保在不同的产品和设备上使用常见的平台真实性和来源方法。Arm 主导的开源软件 Veracuz 是一个框架，用于在一组相互不信任的个人之间定义和部署协作的、保护隐私的计算；VERAISON-VERific Atlon of atteStatiON 构建可用于证明验证服务的软件组件。

Armv9-A 架构引入了Arm CCA 的 RME 功能特性，采用对应架构的芯片也将拥有此项功能。此外，基于 CCA 的软件支持已经在虚拟硬件平台上进行开发、测试和验证，将在硬件设备问世的同时实现同步支持。更多信息，可以访问https://arm.com/armcca获取更多信息。
