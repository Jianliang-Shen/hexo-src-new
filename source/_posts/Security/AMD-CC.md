---
layout: post
title: "机密计算: AMD 解决方案"
index_img: /img/cc/amd/amd-cc-index.jpeg
date: 2024-08-01 20:29:31
archive: true
tags: 
    - TEE
    - Confidential Compute
    - Security
categories: 
    - Security
---

转载：[AMD机密计算解决方案分析](https://blog.csdn.net/huang987246510/article/details/135707747)

<!-- more -->

更多：

- [龙蜥社区 Hygon Arch（需要QQ浏览器或者手机百度浏览器打开）](https://openanolis.cn/sig/Hygon-Arch)
- [**Administrative utility for AMD SEV -- VirTee/SevCtl Github**](https://github.com/virtee/sevctl)
- [Hygon hag](https://gitee.com/anolis/hygon-devkit)
- [AMD SEV基本原理](https://blog.csdn.net/huang987246510/article/details/135487665)
- [【TEE】【AMD SEV内存加密】 白皮书](https://blog.csdn.net/qq_43543209/article/details/135652011)
- [【TEE】AMD SEV- SNP和Intel TDX的概述](https://blog.csdn.net/qq_43543209/article/details/135675421)
- [水印去除](https://www.mindonmap.com/zh/watermark-remover-online/#)

# 数据结构

AMD SEV机密计算场景下，Hypervisor的职能稍有改变，可以概括为以下三点：

- 管理主机计算资源管理
- 代理可信组件间通信
- 准备机密计算环境

对于第一点，QEMU/KVM复用已有的逻辑，无需修改，对于二、三点，QEMU/KVM主要基于SEV API实现。具体实现时，KVM需要封装SEV API并对QEMU暴露IOCTL命令字，QEMU按照SEV API定义的流程实现对机密虚机的生命周期管理逻辑。

## KVM

分析KVM对QEMU提供的IOCTL命令字涉及的数据结构

![](/img/cc/amd/amd-cc-1.png)

### 虚机管理命令

虚机管理命令字提供与虚机相关操作，因此它打开的是`/dev/kvm`字符设备，通过`KVM_CREATE_DEVICE`创建虚机关联的fd，再通过`KVM_MEMORY_ENCRYPT_OP`命令字下发命令，所有的虚机管理命令作为`KVM_MEMORY_ENCRYPT_OP`命令字的参数，通过`id`字段的不同来区分，其命令格式如下：

#### 命令格式

linux/arch/x86/include/uapi/asm/kvm.h:

```c
struct kvm_sev_cmd {
    __u32 id;   /* sev cmd ID，取值为sev_cmd_id */
    __u64 data;   /* 每个sev cmd对应的参数 */
    __u32 error;
    __u32 sev_fd; /* 字符设备/dev/sev对应的fd */
};
```

`kvm_sev_cmd`结构体中`id`字段可选的命令集合。

#### 命令集合

linux/arch/x86/include/uapi/asm/kvm.h:

```c
/* Secure Encrypted Virtualization command */
enum sev_cmd_id {
    /* Guest initialization commands */
    KVM_SEV_INIT = 0,
    KVM_SEV_ES_INIT,
    /* Guest launch commands */
    KVM_SEV_LAUNCH_START, /* 启动虚机命令，通知PSP，Guest owner与PSP建立会话 */
    KVM_SEV_LAUNCH_UPDATE_DATA, /* 更新启动初始化时的虚机内存数据，通知PSP */
    KVM_SEV_LAUNCH_UPDATE_VMSA,
    KVM_SEV_LAUNCH_SECRET, /* 加载虚机密钥密文到虚机内存，通知PSP进行密钥注入 */
    KVM_SEV_LAUNCH_MEASURE, /* 请求PSP计算度量值 */
    KVM_SEV_LAUNCH_FINISH,
    /* Guest migration commands (outgoing) */
    KVM_SEV_SEND_START,
    KVM_SEV_SEND_UPDATE_DATA,
    KVM_SEV_SEND_UPDATE_VMSA,
    KVM_SEV_SEND_FINISH,
    /* Guest migration commands (incoming) */
    KVM_SEV_RECEIVE_START,
    KVM_SEV_RECEIVE_UPDATE_DATA,
    KVM_SEV_RECEIVE_UPDATE_VMSA,
    KVM_SEV_RECEIVE_FINISH,
    /* Guest status and debug commands */
    KVM_SEV_GUEST_STATUS,
    KVM_SEV_DBG_DECRYPT,
    KVM_SEV_DBG_ENCRYPT,
    /* Guest certificates commands */
    KVM_SEV_CERT_EXPORT,
    /* Attestation report */
    KVM_SEV_GET_ATTESTATION_REPORT,
    /* Guest Migration Extension */
    KVM_SEV_SEND_CANCEL,

    KVM_SEV_NR_MAX,
};
```

#### 命令参数

虚机管理命令目的不同，因此每条命令有其对应的参数，以`KVM_SEV_LAUNCH_START`为例，其参数格式如下：

```c
struct kvm_sev_launch_start {
    __u32 handle;
    __u32 policy;  /* 启动机密虚机时的策略 */

    /*
     * 机密环境准备过程中，涉及到Guest owner与PSP间的数据传输，因此需要加密
     * 加密密钥通过PDH密钥协商协议得到，该协议需要通信双方提供各自的PDH证书
     * 让对方计算共享密钥，这里的dh就是Guest owner提供的PDH证书，目的是让
     * PSP计算得到通信共享密钥
     */
    __u64 dh_uaddr;
    __u32 dh_len;
    __u64 session_uaddr;
    __u32 session_len;
};
```

### 平台管理命令字

#### 命令格式

平台管理命令主要是向PSP发送平台管理相关命令，与虚机并无关系，它打开的是`/dev/sev`字符设备并直接下发平台管理命令字，其格式如下：

linux/include/uapi/linux/psp-sev.h:

```c
/**
 * struct sev_issue_cmd - SEV ioctl parameters
 *
 * @cmd: SEV commands to execute
 * @opaque: pointer to the command structure
 * @error: SEV FW return code on failure
 */
struct sev_issue_cmd {
    __u32 cmd;              /* In */
    __u64 data;             /* In */
    __u32 error;            /* Out */
}
```

#### 命令集合

`sev_issue_cmd`结构体中cmd字段可选的命令集合

linux/include/uapi/linux/psp-sev.h:

```c
/**
 * SEV platform commands
 */
enum {
    SEV_FACTORY_RESET = 0,
    SEV_PLATFORM_STATUS, /* 查询PSP状态 */
    /*
     * 请求PSP重新生成PEK签发的所有证书，包括PEK公私钥对，PEK证书，PDH公私钥对，* PDH证书，OCA公私钥对，OCA证书
     */
    SEV_PEK_GEN, 
     /* 请求PSP生成PEK公钥及其它用户制作PEK证书的信息 */
    SEV_PEK_CSR,
    /* 请求PSP重新生成PDH，该PDH的公钥信息被PEK私钥签名，由PEK签发 */
    SEV_PDH_GEN,
    /* 导出PDH公钥及PEK对公钥的签名，用于Gueste owner对PDH的可信验证 */
    SEV_PDH_CERT_EXPORT,
    /* 往PSP导入PEK证书，声明PSP的Platform owner */
    SEV_PEK_CERT_IMPORT,
    /* 查询CPU ID，在AMD CA是，此ID将作为CEK下载的输入 */
    SEV_GET_ID, /* This command is deprecated, use SEV_GET_ID2 */
    SEV_GET_ID2,

    SEV_MAX,
};
```

#### 命令参数

每个平台命令都有其对应的参数，以`PLATFORM_STATUS`为例，在下发`PLATFORM_STATUS`时会传入结构体`sev_user_data_status`，用于接收PSP的输出

linux/include/uapi/linux/psp-sev.h:

```c
/**
 * struct sev_user_data_status - PLATFORM_STATUS command parameters
 *
 * @major: major API version
 * @minor: minor API version
 * @state: platform state
 * @flags: platform config flags
 * @build: firmware build id for API version
 * @guest_count: number of active guests
 */
struct sev_user_data_status {
    /* PSP固件版本号 */
    __u8 api_major;         /* Out */
    __u8 api_minor;         /* Out */
    /* PSP状态 */
    __u8 state;             /* Out */
    __u32 flags;            /* Out */
    __u8 build;             /* Out */
    __u32 guest_count;      /* Out */
}
```

## QEMU

### SevState

`SevState` 描述机密虚机在启动或者迁移过程中的整个状态，当虚机状态变为running时，Hypervisor可以切换CPU模式进入Guest态。

```c
##
# @SevState:
#
# An enumeration of SEV state information used during @query-sev.
# SEV机密虚机默认状态
# @uninit: The guest is uninitialized.
# QEMU完成初始化工作，API LAUNCH UPDATE调用完成后更新为launch-update状态
# @launch-update: The guest is currently being launched; plaintext data and
#                 register state is being imported.
# QEMU获取度量值完成后，等待密钥注入的状态
# @launch-secret: The guest is currently being launched; ciphertext data
#                 is being imported.
# QEMU密钥注入完成，API LAUNCH FINISH调用完成后更新为running状态
# @running: The guest is fully launched or migrated in.
#
# @send-update: The guest is currently being migrated out to another machine.
#
# @receive-update: The guest is currently being migrated from another machine.
#
# Since: 2.12
##
{ 'enum': 'SevState',
  'data': ['uninit', 'launch-update', 'launch-secret', 'running',
           'send-update', 'receive-update' ],
  'if': 'TARGET_I386' }
```

### SevGuestState

SevGuestState主要描述SEV机密虚机的配置参数和运行时状态。

```c
/**
 * SevGuestState:
 *
 * The SevGuestState object is used for creating and managing a SEV
 * guest.
 *
 * # $QEMU \
 *         -object sev-guest,id=sev0 \
 *         -machine ...,memory-encryption=sev0
 */
struct SevGuestState {
    ConfidentialGuestSupport parent_obj;

    /* configuration parameters */
    char *sev_device;
    uint32_t policy;  /* SEV API 中定义的虚机启动策略 */
    /* 用于协商Guest owner与PSP通信密钥的pdh证书，Guest owner提供 */
    char *dh_cert_file;
    /* 通信会话文件，Guest owner提供 */
    char *session_file;
    /* 使能内存加密的C-bit页表项位置，Guest owner通过CPUID获取后配置 */
    uint32_t cbitpos;
    /* 使能内存加密后减少的物理地址位宽，Guest owner通过CPUID获取后配置 */
    uint32_t reduced_phys_bits;
    /* 指示QEMU是否需要将kernel的hash值填入OVMF 的GUIDed table */
    bool kernel_hashes;

    /* runtime state */
    uint32_t handle;
    /* SEV API固件的版本号信息 */
    uint8_t api_major;
    uint8_t api_minor;
    /* TBD */
    uint8_t build_id;
    /* 打开/dev/sev字符设备获得的fd */
    int sev_fd;
    SevState state;
    gchar *measurement;
    
    uint32_t reset_cs;
    uint32_t reset_ip;
    bool reset_data_valid;
};
```

# 虚机启动流程

## 可信认证

![AMD 证书链](/img/cc/amd/amd-cc-2.png)

### 基本原理

**机密虚机与普通虚机最大的不同是机密虚机所有者即Guest owner假定Hypervisor不可信，因此机密虚机的整个生命周期都不能被Hypervisor监听或篡改，保证机密虚机一定是Guest owner期望的虚机。**

在启动流程中，Guest owner首先要做的就是对安全处理器PSP做可信认证，确认安全处理器一定是可信的。Guest owner对PSP的校验就是请求（通过`PDH_CERT_EXPORT`）其导出可以证明自己身份的证书，这些证书包括**CEK证书、PEK证书、OCA证书**以及计算共享密钥用的PDH。Guest owner拿到证书后，做以下处理：

- 对于**CEK证书**，首先获取签发CEK证书的**CA即ASK**，使用ASK的公钥解密CEK证书得到CEK公钥的明文，重新计算明文摘要并对比CEK证书的摘要，如果两者相等，说明**CEK证书**确实是ASK签发的。因为**ASK签发CEK**的实际动作就是使用ASK的私钥对CEK的公钥进行加密存放到证书中，而ASK的私钥只有ASK可以访问，如果CEK公钥由非ASK的私钥进行加密，那么使用ASK的公钥对CEK公钥解密后，得到的明文摘要一定与证书中的摘要不同。
- 对于**PEK证书**，获取**PEK证书的CA即OCA**，使用OCA的公钥解密PEK证书得到PEK公钥明文，重新计算名为摘要并对于PEK证书上的摘要，相等说明PEK证书是OCA签发的。检查OCA是否是自签的证书，如果OCA是自签证书，不做进一步处理，如果OCA是它签证书，找到签发OCA的CA，下载其公钥再对OCA进行验签，整个流程与其它证书验签类似。
- 对于**PDH证书**，获取签发PDH证书的CA即PEK，使用上述类似流程对其进行验签。

上述所有验签通过后，Guest owner认为PSP就是可信的，继而可以让Hypervisor开始启动虚机，整个流程示意图如下：

![证书认证](/img/cc/amd/amd-cc-3.png)

### 具体流程

由于可信认证涉及的角色是Guest owner和PSP，Hypervisor因此不需要介入。PSP的可信认证工具通常由CPU厂商提供，也就是AMD和Hygon（基于AMD架构的国产CPU），我们简单介绍两者的工具：

#### 工具

##### AMD SevCTL

1. 服务器出厂后，第一次启动时还没有OCA，通过以下命令可以创建一个自签名的OCA：

   ```bash
   sevctl generate oca.cert oca.key
   sevctl provision oca.cert oca.key
   ```

2. OCA创建完成后，通过以下命令验证当前服务器中的PSP是否可信，该工具会根据CPU ID从AMD官网查询对应的CEK证书并导入到PSP，最后做验证

   ```bash
   sevctl verify
   ```

3. 创建完自签OCA证书并完成CEK证书导入后，可以通过export命令导出PSP的整个证书链供Guest owner验签

   ```bash
   sevctl export --full /opt/sev/cert_chain.cert
   ```

##### Hygon hag

1. 服务器出厂后第一次上电，需要导入通用的安全证书，首先下载hygon机密计算工具包：

   ```bash
   cd /opt/
   git clone https://gitee.com/anolis/hygon-devkit.git
   mv hygon-devkit hygon
   cp /opt/hygon/bin/hag /usr/bin
   ```

2. 在线导入通用的安全证书

   ```bash
   hag general hgsc_import
   ```

   也可以离线导入通用的安全证书

   ```bash
   hag general get_id           /* 获取PSP芯片ID */
   hag general hgsc_version     /* 查询证书版本号 */
   ```

3. 访问[Hygon证书下载官网](https://cert.hygon.cn/hgsc)，输入芯片ID和证书版本号下载证书，名称类似为hygon-hgsc-certchain-v1.0-H905P0005040204.bin，通过以下命令导入到Hygon服务器:

   ```bash
   hag general hgsc_import -offline -in /path/to/hygon-hgsc-certchain-v1.0-H905P0005040204.bin
   ```

4. 最后查询证书导入状态，is HGSC imported为YES

   ```bash
   hag csv platform_status
   ```

完成以上步骤后，导出证书链包含的所有文件：

```bash
hag csv pdh_cert_export
```

证书链文件包括：

```bash
cert_chain.bin
cert_chain.cert
cert_chain_readable.txt 
pdh.bin  
pdh.cert  
pdh_readable.txt
```

其中证书链被Guest owner用于校验PSP是否可信，证书链包含三个证书，分别是CEK、OCA 和PEK：

```bash
cat cert_chain_readable.txt | grep Userid
Userid:        HYGON-SSD-PEK
Userid:        HYGON-SSD-OCA
Userid:        HYGON-SSD-CEK
```

PDH被Guest owner用于计算与PSP之间通信的会话密钥：

```bash
cat pdh_readable.txt |grep Userid
Userid:        HYGON-SSD-PDH
```

#### 接口

的确，证书的导出确实与Hypervisor不相关，但QEMU实际上也提供了虚机运行时导出其证书的接口，以便支持Libvirt等虚机管理工具对机密虚机的能力查询：

```c
// 1. qapi中对sev能力的定义

// @SevCapability:

// The struct describes capability for a Secure Encrypted
// Virtualization feature.

// @pdh: Platform Diffie-Hellman key (base64 encoded)

// @cert-chain: PDH certificate chain (base64 encoded)

// @cpu0-id: Unique ID of CPU0 (base64 encoded) (since 7.1)

// @cbitpos: C-bit location in page table entry

// @reduced-phys-bits: Number of physical Address bit reduction when
//     SEV is enabled

// Since: 2.12
{ 'struct': 'SevCapability',
  'data': { 'pdh': 'str',
            'cert-chain': 'str',
            'cpu0-id': 'str',
            'cbitpos': 'int',
            'reduced-phys-bits': 'int'},
  'if': 'TARGET_I386' }
// 2. 对应的C结构体  
struct SevCapability {
    char *pdh;   /* 用于计算会话密钥PDH证书 */
    char *cert_chain; /* 证书链 */
    char *cpu0_id;
    int64_t cbitpos;
    int64_t reduced_phys_bits;
};
// 3. 具体的查询流程
qmp_query_sev_capabilities
    sev_get_capabilities
        kvm_vm_ioctl(kvm_state, KVM_MEMORY_ENCRYPT_OP, NULL)
        fd = open(DEFAULT_SEV_DEVICE, O_RDWR) /* 打开/dev/sev字符设备 */
        sev_get_pdh_info(fd, &pdh_data, &pdh_len, &cert_chain_data, &cert_chain_len, errp)) /* 下发命令字导出证书 */
            sev_platform_ioctl(fd, SEV_PDH_CERT_EXPORT, &export, &err)
        cap = g_new0(SevCapability, 1);
        cap->pdh = g_base64_encode(pdh_data, pdh_len);
        cap->cert_chain = g_base64_encode(cert_chain_data, cert_chain_len);
```

## 静态度量

### 基本原理

Guest owner完成对PSP的可信认证后，就可以通知Hypervisor启动虚机了，对于要启动的虚机，Guest owner需要确认Hypervisor启动虚机使用的所有引导文件与自己期望的相同。为达此目的，Guest owner需要将启动虚机涉及的所有引导文件和命令行参数作为输入，计算一个度量值，然后在虚机启动后，让PSP将相同的文件和命令行参数也作为输入，重新计算一个度量值，两者对比之后，如果相同，则可以确认Hypervisor在虚机启动过程中并没有对引导文件做手脚，启动的虚机确实是Guest owner期望的虚机。

Guest owner对引导文件和命令行参数进行的度量计算，与PSP可信认证相互独立，因此也可以在最开始启动虚机的时候做，如下面流程图的第一步所示：

![](/img/cc/amd/amd-cc-4.png)

#### AMD SevCTL

sevctl工具提供了measurement命令计算度量值，使用方式如下：

```bash
$ sevctl measurement build \
    --api-major 01 --api-minor 40 --build-id 40 \
    --policy 0x05 \
    --tik /path/to/VM_tik.bin \
    --firmware /usr/share/edk2/ovmf/OVMF.amdsev.fd \
    --kernel /path/to/kernel \
    --initrd /path/to/initrd \
    --cmdline "my kernel cmdline" \
    --vmsa-cpu0 /path/to/vmsa0.bin \
    --vmsa-cpu1 /path/to/vmsa1.bin \
    --num-cpus 4
    --launch-measure-blob /o0nzDKE5XgtVnUZWPhUea/WZYrTKLExR7KCwuMdbActvpWfXTFk21KMZIAAhQny
```

>launch-measure-blob指定输出文件的位置

#### Hygon hag

Hygon的hag工具提供了generate_launch_blob命令计算度量值：

```bash
hag csv generate_launch_blob -help
Usage: generate_launch_blob [options]
Valid options are:
    -help            Display this summary
    -verbose         Enable debug log print
    -dir dir         [optional] Specify the file generation directory
    -build ulong     Input the build id of the Firmware
    -bios infile     Input bios file
    -kernel infile   [optional] Input kernel file
    -cmdline infile  [optional] Input cmdline file, default is '\0'
    -initrd infile   [optional] Input initrd file
```

这里我们将OVMF固件作为度量值的输入，计算度量值，示例如下：

```bash
hag csv generate_launch_blob \
    -build 1881 \        /* 固件ID，通过hag csv platform_status查询得到 */
    -bios /opt/hygon/csv/OVMF.fd /* 度量值的输入：bios文件 */
```

Guest owner完成对度量值的计算后，就可以启动虚机了，后续只要获取虚机启动时PSP计算的初始化内存度量值，对比两者是否相等，就可以确认Hypervisor是否启动了一个期望的虚机。这里需要解决的一个问题是，Hypervisor需要怎么做，才能既做到为PSP提供度量值计算的输入，又无法窃取PSP计算的度量值输出，或者无法代替PSP进行度量值计算呢？

我们知道 PSP 将 VMCB 中的 ASID 作为加密进程关联内存的密钥，对内存进行加密。为了让Hypervisor无法代替PSP做度量值计算，SEV的做法是：

- 让Hypervisor将计算度量值所需的材料加载到Guest内存地址空间，这里主要指OVMF、kernel和initrd等启动文件，此时虚机的内存地址空间内容还是明文，Hypervisor虽然可以进行度量值计算，但因为PSP与Guest owner通信是密文，要得到密文必须先获取Guest owner与PSP之前通过PDH协商出来的共享密钥，因此它无法得到度量值加密后的密文数据。
- 通过API ACTIVATE通知PSP将ASID与内存加密的密钥VEK（VM Encryption Key）关联，PSP的具体逻辑是以ASID为索引，在系统的所有密钥槽中找到对应的VEK，将其加载到内存
- 通过API LAUNCH_UPDATE_DATA通知PSP对虚机内存进行加密。此时虚机的内存地址空间变为密文。
- 通过API LAUNCH_MEASURE通知PSP对 ASID关联的内存地址空间内容进行度量值计算，并将加密后的明文返回给Guest owner

### 具体流程

- 我们知道QEMU命令行支持`BIOS+grub`或`OVMF+UEFI+grub`固件启动，但在SEV机密计算场景下，只对`OVMF+UEFI+grub`启动方案做了适配。所以在具体启动SEV机密计算虚机，只能以OVMF方式引导虚机；
- 另外QEMU命令行既支持以显式指定`kernel+initrd+command line`的方式启动虚机；也支持直接以虚机镜像的方式启动虚机。当用户通过前者启动虚机时，需要指定OVMF、kernel和initrd文件，command line可选，QEMU的传统做法是将OVMF载入虚机内存，将kernel和initrd文件添加到fw_cfg（Firmware Configuration）设备，然后启动虚机，虚机启动过程中，OVMF逻辑会从fw_cfg设备中读取kernel和initrd文件并加载到内存，准备工作完成之后，跳转到kernel，移交控制权，这里可以看出，PSP对虚机初始化内存的度量，只会包含OVMF固件，并没有kernel和initrd文件，因为kernel和initrd是在虚机运行（即Guest态）之后OVMF从设备加载到内存的。在机密计算场景下，由于Hypervisor是不可信的，静态度量并没有包含kernel和initrd的内容，它完全有可能将用户kernel和initrd文件替换为恶意镜像而不被Guest owner发现，机密计算的方案需要考虑如何处理这种情况。对于使用虚机镜像的方式启动虚机的场景，也存在Hypervisor直接将镜像替换为恶意镜像的可能。我们下面分别讨论这两种场景下机密计算给出的解决方案以及具体的实现流程。

#### 内核启动

内核启动指QEMU通过命令行参数显式指定kernel+initrd+cmdline启动虚机，kernel+initrd文件包含在虚机镜像之中

#### 镜像启动

镜像启动指QEMU指定启动盘的方式启动虚机，kernel+initrd包含在虚机镜像之中。OVMF读取磁盘内容，通过grub指定磁盘上的kernel+initrd+cmdline。
