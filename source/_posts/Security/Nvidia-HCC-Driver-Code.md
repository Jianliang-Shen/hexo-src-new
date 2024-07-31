---
layout: post
title: "机密计算: HCC 驱动代码"
index_img: /img/post_pics/HCC/hcc2_index.png
date: 2024-07-02 14:43:10
archive: true
tags:
    - TEE
    - Confidential Compute
    - GPU
    - Security
categories: 
    - Security
---

Hopper GPU驱动代码CC部分分析。

<!-- more -->

## 概述

535.43.02

新增了Hopper CC，主架构为H100
新增mbedtls和spdm库:

```txt
src/common/mebedtls

src/nvidia/generated/g_spdm_nvoc.c
src/nvidia/generated/g_spdm_nvoc.h

src/common/sdk/nvidia/inc/ctrl/ctrlcb33.h
src/common/sdk/nvidia/inc/class/clcb33.h

机密计算API ConfidentialComputeApi
src/nvidia/generated/g_conf_compute_api_nvoc.h
src/nvidia/generated/g_conf_compute_api_nvoc.c

src/nvidia/generated/g_conf_compute_nvoc.h
src/nvidia/generated/g_conf_compute_nvoc.c


src/nvidia/src/kernel/gpu/conf_compute/ccsl.c
src/nvidia/src/kernel/gpu/conf_compute/conf_compute_api.c
src/nvidia/src/kernel/gpu/conf_compute/conf_compute.c
src/nvidia/inc/kernel/gpu/conf_compute/conf_compute_keystore.h 密钥管理
src/nvidia/src/kernel/gpu/conf_compute/conf_compute_keystore.c
src/nvidia/src/kernel/gpu/conf_compute/arch/hopper/conf_compute_gh100.c
src/nvidia/src/kernel/gpu/conf_compute/arch/hopper/conf_compute_keystore_gh100.c


src/nvidia/inc/kernel/gpu/spdm/libspdm_includes.h
src/nvidia/inc/kernel/gpu/spdm/spdm.h

src\nvidia\src\kernel\gpu\mmu\arch\hopper\kern_gmmu_gh100.c (eb5c766)
src/nvidia/src/kernel/gpu/spdm/arch/hopper/spdm_gh100.c
src/nvidia/src/kernel/gpu/spdm/arch/hopper/spdm_module.c

src/nvidia/src/libraries/libspdm/2.3.1/
src/nvidia/src/libraries/libspdm/2.3.1/include/hal/library/cryptlib/
```

<!-- 判断是否开启CC mode:
gpuIsCCFeatureEnabled, gpuIsCCEnabledInHw_HAL
NvBool PDB_PROP_GPU_CC_FEATURE_CAPABLE; -->

SEC2（Security Engine 2） 是一种集成在GPU或其他硬件中的安全模块，专门用于处理加密、解密和认证等安全操作。它通常用于确保数据在传输和存储过程中的机密性和完整性。
CSL（Crypto Security Layer）

## 相关头文件

### nv_uvm_interface.h

kernel-open/common/inc/nv_uvm_interface.h

```c
/*******************************************************************************

    CSL 接口和锁定

    以下函数不获取 RM API 或 GPU 锁定，并且不能在不同线程中与相同的 UvmCslContext 参数同时调用。调用者必须保证这种排除。

    * nvUvmInterfaceCslLogDeviceEncryption
    * nvUvmInterfaceCslRotateIv
    * nvUvmInterfaceCslEncrypt
    * nvUvmInterfaceCslDecrypt
    * nvUvmInterfaceCslSign
    * nvUvmInterfaceCslQueryMessagePool
*/

/*******************************************************************************
    nvUvmInterfaceCslInitContext

    为给定的安全通道分配并初始化 CSL 上下文。

    上下文的生命周期与其配对的安全通道的生命周期相同。

    参数：
        uvmCslContext[IN/OUT] - CSL 上下文。
        channel[IN] - 安全通道句柄。

    错误代码：
        NV_ERR_INVALID_STATE - 系统未在机密计算模式下运行。
        NV_ERR_INVALID_CHANNEL - 关联通道不是安全通道。
        NV_ERR_IN_USE - 上下文已初始化。
*/
NV_STATUS nvUvmInterfaceCslInitContext(UvmCslContext *uvmCslContext,
                                       uvmGpuChannelHandle channel);

/*******************************************************************************
    nvUvmInterfaceDeinitCslContext

    安全地取消初始化并清除上下文的内容。

    如果上下文已取消初始化，则函数立即返回。

    参数：
        uvmCslContext[IN] - CSL 上下文。
*/
void nvUvmInterfaceDeinitCslContext(UvmCslContext *uvmCslContext);


/*******************************************************************************
    nvUvmInterfaceCslLogDeviceEncryption

    返回一个 IV，稍后可在 nvUvmInterfaceCslEncrypt 方法中使用。IV 包含一个“freshness bit”，其
    值由此方法设置，随后由 nvUvmInterfaceCslEncrypt 弄脏，以防止 IV 的非恶意重用。

    有关锁定要求，请参阅“CSL 接口和锁定”。
    此函数不执行动态内存分配。

    参数：
        uvmCslContext[IN/OUT] - CSL 上下文。
        encryptIv[OUT] - 设备加密成功之前存储的参数。它用作 nvUvmInterfaceCslEncrypt 的输入。

    错误代码：
        NV_ERR_INSUFFICIENT_RESOURCES - 新的 IV 会导致计数器溢出。
*/
NV_STATUS nvUvmInterfaceCslAcquireEncryptionIv(UvmCslContext *uvmCslContext,
                                               UvmCslIv *encryptIv);

/*******************************************************************************
    nvUvmInterfaceCslLogDeviceEncryption

    记录并检查有关设备加密的信息。

    有关锁定要求，请参阅“CSL 接口和锁定”。

    此函数不执行动态内存分配。

    参数：
        uvmCslContext[IN/OUT] - CSL 上下文。
        decryptIv[OUT] - 设备加密成功之前存储的参数。它用作 nvUvmInterfaceCslDecrypt 的输入。

    错误代码：
        NV_ERR_INSUFFICIENT_RESOURCES - 设备加密会导致计数器溢出。
*/
NV_STATUS nvUvmInterfaceCslLogDeviceEncryption(UvmCslContext *uvmCslContext,
                                               UvmCslIv *decryptIv);

/*******************************************************************************
    nvUvmInterfaceCslRotateIv

    旋转给定通道和方向的 IV。

    此函数将在 CPU 和 GPU 上旋转 IV。

    应首先解密已由 GPU 加密的未完成消息，然后调用方向等于 UVM_CSL_DIR_GPU_TO_CPU 的此函数。
    同样，应首先解密已由 CPU 加密的未完成消息，然后调用方向等于 UVM_CSL_DIR_CPU_TO_GPU
    的此函数。对于给定方向，在调用此函数之前通道必须处于空闲状态。无论 IV 的消息计数器的值如何，
    都可以调用此函数。

    有关锁定要求，请参阅“CSL 接口和锁定”。
    此函数不执行动态内存分配。

    参数：
        uvmCslContext[IN/OUT] - CSL 上下文。
        direction[IN] - 任一
            - UVM_CSL_DIR_CPU_TO_GPU
            - UVM_CSL_DIR_GPU_TO_CPU

    错误代码：
        NV_ERR_INSUFFICIENT_RESOURCES - 旋转操作会导致计数器溢出。
        NV_ERR_INVALID_ARGUMENT - 方向值无效。
*/
NV_STATUS nvUvmInterfaceCslRotateIv(UvmCslContext *uvmCslContext,
                                    UvmCslDirection direction);

/*******************************************************************************
    nvUvmInterfaceCslEncrypt

    加密数据并生成身份验证标签。

    身份验证、输入和输出缓冲区不得重叠。如果重叠，则调用此函数会产生未定义的行为。当输入和输出
    缓冲区按 16 字节对齐时，性能通常会最大化。这是 AES 块的自然对齐。
    encryptIV 可以从 nvUvmInterfaceCslAcquireEncryptionIv 获得。但是，它是可选的。如果
    它为 NULL，则将使用行中的下一个 IV。

    有关锁定要求，请参阅“CSL 接口和锁定”。
    此函数不执行动态内存分配。

    参数：
        uvmCslContext[IN/OUT] - CSL 上下文。
        bufferSize[IN] - 输入和输出缓冲区的大小（以字节为单位）。值的范围可以从 1 字节到 (2^32) - 1 字节。
        inputBuffer[IN] - 明文输入缓冲区的地址。
        encryptIv[IN/OUT] - 用于加密的 IV。可以为 NULL。
        outputBuffer[OUT] - 密文输出缓冲区的地址。
        authTagBuffer[OUT] - 身份验证标签缓冲区的地址。其大小为 UVM_CSL_CRYPT_AUTH_TAG_SIZE_BYTES。

    错误代码：
        NV_ERR_INVALID_ARGUMENT 
            - 数据大小为 0 字节。
            - encryptIv 已被使用。
*/
NV_STATUS nvUvmInterfaceCslEncrypt(UvmCslContext *uvmCslContext,
                                   NvU32 bufferSize,
                                   NvU8 const *inputBuffer,
                                   UvmCslIv *encryptIv,
                                   NvU8 *outputBuffer,
                                   NvU8 *authTagBuffer);

/*******************************************************************************
    nvUvmInterfaceCslDecrypt

    验证身份验证标签并解密数据。

    身份验证、输入和输出缓冲区不得重叠。如果重叠，则调用此函数会产生未定义的行为。当输入和输出
    缓冲区按 16 字节对齐时，性能通常会最大化。这是 AES 块的自然对齐。

    有关锁定要求，请参阅“CSL 接口和锁定”。
    此函数不执行动态内存分配。

    参数：
        uvmCslContext[IN/OUT] - CSL 上下文。
        bufferSize[IN] - 输入和输出缓冲区的大小（以字节为单位）。值的范围可以从 1 字节到 (2^32) - 1 字节。
        decryptIv[IN] - nvUvmInterfaceCslLogDeviceEncryption 给出的参数。
        inputBuffer[IN] - 密文输入缓冲区的地址。
        outputBuffer[OUT] - 明文输出缓冲区的地址。
        authTagBuffer[IN] - 身份验证标签缓冲区的地址。其大小为 UVM_CSL_CRYPT_AUTH_TAG_SIZE_BYTES。

    错误代码：
        NV_ERR_INSUFFICIENT_RESOURCES - 解密操作会导致计数器溢出。
        NV_ERR_INVALID_ARGUMENT - 数据大小为 0 字节。
        NV_ERR_INVALID_DATA - 身份验证标签验证失败。
*/
NV_STATUS nvUvmInterfaceCslDecrypt(UvmCslContext *uvmCslContext,
                                   NvU32 bufferSize,
                                   NvU8 const *inputBuffer,
                                   UvmCslIv const *decryptIv,
                                   NvU8 *outputBuffer,
                                   NvU8 const *authTagBuffer);

/*******************************************************************************
    nvUvmInterfaceCslSign

    为安全工作启动生成身份验证标签。

    身份验证和输入缓冲区不得重叠。如果重叠，则调用此函数会产生未定义的行为。

    有关锁定要求，请参阅“CSL 接口和锁定”。

    此函数不执行动态内存分配。

    参数：
        uvmCslContext[IN/OUT] - CSL 上下文。
        bufferSize[IN] - 输入缓冲区的大小（以字节为单位）。值的范围可以从 1 字节到 (2^32) - 1 字节。
        inputBuffer[IN] - 明文输入缓冲区的地址。
        authTagBuffer[OUT] - 身份验证标签缓冲区的地址。其大小为 UVM_CSL_SIGN_AUTH_TAG_SIZE_BYTES。

    错误代码：
        NV_ERR_INSUFFICIENT_RESOURCES - 签名操作会导致计数器溢出。
        NV_ERR_INVALID_ARGUMENT - 数据大小为 0 字节。
*/
NV_STATUS nvUvmInterfaceCslSign(UvmCslContext *uvmCslContext,
                                NvU32 bufferSize,
                                NvU8 const *inputBuffer,
                                NvU8 *authTagBuffer);


/*******************************************************************************
    nvUvmInterfaceCslQueryMessagePool

    返回在消息计数器溢出之前可以加密的消息数量。

    有关锁定要求，请参阅“CSL 接口和锁定”。
    此函数不执行动态内存分配。

    参数：
        uvmCslContext[IN/OUT] - CSL 上下文。
        direction[IN] - UVM_CSL_DIR_CPU_TO_GPU 或 UVM_CSL_DIR_GPU_TO_CPU。
        messageNum[OUT] - 溢出前剩余的消息数量。

    错误代码：
        NV_ERR_INVALID_ARGUMENT - direction 参数的值非法。
*/
NV_STATUS nvUvmInterfaceCslQueryMessagePool(UvmCslContext *uvmCslContext,
                                            UvmCslDirection direction,
                                            NvU64 *messageNum);
```

### uvm_conf_computing.h

kernel-open/nvidia-uvm/uvm_conf_computing.h

```c
#define UVM_CONF_COMPUTING_AUTH_TAG_SIZE (UVM_CSL_CRYPT_AUTH_TAG_SIZE_BYTES)

// An authentication tag pointer is required by HW to be 16-bytes aligned.
// 硬件要求认证标签指针必须 16 字节对齐。
#define UVM_CONF_COMPUTING_AUTH_TAG_ALIGNMENT 16

// An IV pointer is required by HW to be 16-bytes aligned.
// HW 要求 IV 指针为 16 字节对齐。
//
// Use sizeof(UvmCslIv) to refer to the IV size.
// 使用 sizeof(UvmCslIv) 来引用 IV 大小。
#define UVM_CONF_COMPUTING_IV_ALIGNMENT 16

// SEC2 decrypt operation buffers are required to be 16-bytes aligned. CE
// encrypt/decrypt can be unaligned if the buffer lies in a single 32B segment.
// Otherwise, they need to be 32B aligned.
// SEC2 解密操作缓冲区需要 16 字节对齐。如果缓冲区位于单个 32B 段中，则 CE 加密/解密可以
// 不对齐。否则，它们需要 32B 对齐。
#define UVM_CONF_COMPUTING_BUF_ALIGNMENT 32

#define UVM_CONF_COMPUTING_DMA_BUFFER_SIZE UVM_VA_BLOCK_SIZE

// SEC2 supports at most a stream of 64 entries in the method stream for
// signing. Each entry is made of the method address and method data, therefore
// the maximum buffer size is: UVM_METHOD_SIZE * 2 * 64 = 512.
// UVM, however, won't use this amount of entries, in the worst case scenario,
// we push a semaphore_releases or a decrypt. A SEC2 semaphore_release uses 6 1U
// entries, whereas a SEC2 decrypt uses 10 1U entries. For 10 entries,
// UVM_METHOD_SIZE * 2 * 10 = 80.
// SEC2 最多支持 64 个方法流条目进行签名。每个条目由方法地址和方法数据组成，因此最大缓冲区
// 大小为：UVM_METHOD_SIZE * 2 * 64 = 512。但是 UVM 不会使用这么多条目，在最坏的情况下，
// 我们会推送一个 semaphore_releases 或一个解密。SEC2 semaphore_release 使用 6 个
// 1U 条目，而 SEC2 解密使用 10 个 1U 条目。对于 10 个条目，UVM_METHOD_SIZE * 2 * 10 = 80。
#define UVM_CONF_COMPUTING_SIGN_BUF_MAX_SIZE 80

// All GPUs derive confidential computing status from their parent.
// By current policy all parent GPUs have identical confidential
// computing status.
// 所有 GPU 都从其父 GPU 获得机密计算状态。根据当前政策，所有父 GPU 都具有相同的机密计算状态。
NV_STATUS uvm_conf_computing_init_parent_gpu(const uvm_parent_gpu_t *parent);
bool uvm_conf_computing_mode_enabled_parent(const uvm_parent_gpu_t *parent);
bool uvm_conf_computing_mode_enabled(const uvm_gpu_t *gpu);
bool uvm_conf_computing_mode_is_hcc(const uvm_gpu_t *gpu);

typedef struct
{
    // List of free DMA buffers (uvm_conf_computing_dma_buffer_t).
    // A free DMA buffer can be grabbed anytime, though the tracker
    // inside it may still have pending work.
    // 空闲 DMA 缓冲区列表（uvm_conf_computing_dma_buffer_t）。空闲的 DMA 缓冲区可以随时获取，但其中的跟踪器可能仍有待处理的工作。
    struct list_head free_dma_buffers;

    // Used to grow the pool when full.
    // 用于在池满时增大池子。
    size_t num_dma_buffers;

    // Lock protecting the dma_buffer_pool
    // 保护 dma_buffer_pool 的锁
    uvm_mutex_t lock;
} uvm_conf_computing_dma_buffer_pool_t;

typedef struct
{
    // Backing DMA allocation
    // 支持 DMA 分配
    uvm_mem_t *alloc;

    // Used internally by the pool management code to track the state of
    // a free buffer.
    // 由池管理代码内部使用，以跟踪空闲缓冲区的状态。
    uvm_tracker_t tracker;

    // When the DMA buffer is used as the destination of a GPU encryption, SEC2
    // writes the authentication tag here. Later when the buffer is decrypted
    // on the CPU the authentication tag is used again (read) for CSL to verify
    // the authenticity. The allocation is big enough for one authentication
    // tag per PAGE_SIZE page in the alloc buffer.
    // 当 DMA 缓冲区用作 GPU 加密的目标时，SEC2 会在此处写入身份验证标签。稍后，当缓冲区
    // 在 CPU 上解密时，身份验证标签将再次用于（读取）CSL 以验证真实性。分配缓冲区中每个
    // PAGE_SIZE 页的分配足够大，可以容纳一个身份验证标签。
    uvm_mem_t *auth_tag;

    // CSL supports out-of-order decryption, the decrypt IV is used similarly
    // to the authentication tag. The allocation is big enough for one IV per
    // PAGE_SIZE page in the alloc buffer. The granularity between the decrypt
    // IV and authentication tag must match.
    // CSL 支持无序解密，解密 IV 的使用方式与身份验证标签类似。分配缓冲区中每个 PAGE_SIZE
    // 页的分配足够大，可以容纳一个 IV。解密 IV 和身份验证标签之间的粒度必须匹配。
    UvmCslIv decrypt_iv[(UVM_CONF_COMPUTING_DMA_BUFFER_SIZE / PAGE_SIZE)];

    // 后备分配中的加密页面的位图
    uvm_page_mask_t encrypted_page_mask;

    // See uvm_conf_computing_dma_pool lists
    // 参见 uvm_conf_computing_dma_pool 列表
    struct list_head node;
} uvm_conf_computing_dma_buffer_t;

// Retrieve a DMA buffer from the given DMA allocation pool.
// NV_OK                Stage buffer successfully retrieved
// NV_ERR_NO_MEMORY     No free DMA buffers are available for grab, and
//                      expanding the memory pool to get new ones failed.
//
// out_dma_buffer is only valid if NV_OK is returned. The caller is responsible
// for calling uvm_conf_computing_dma_buffer_free once the operations on this
// buffer are done.
// When out_tracker is passed to the function, the buffer's dependencies are
// added to the tracker. The caller is guaranteed that all pending tracker
// entries come from the same GPU as the pool's owner. Before being able to use
// the DMA buffer, the caller is responsible for either acquiring or waiting
// on out_tracker. If out_tracker is NULL, the wait happens in the allocation
// itself.
// Upon success the encrypted_page_mask is cleared as part of the allocation.
// 从给定的 DMA 分配池中检索 DMA 缓冲区。
//      NV_OK 阶段缓冲区已成功检索
//      NV_ERR_NO_MEMORY 没有可供抓取的空闲 DMA 缓冲区，扩展内存池以获取新缓冲区失败。
//
// out_dma_buffer 仅在返回 NV_OK 时才有效。调用者负责在完成此缓冲区上的操作后调用
// uvm_conf_computing_dma_buffer_free。
// 当 out_tracker 传递给函数时，缓冲区的依赖项将添加到跟踪器。调用者保证所有待处理的跟踪
// 器条目都来自与池所有者相同的 GPU。在能够使用 DMA 缓冲区之前，调用者负责获取或等待
// out_tracker。如果 out_tracker 为 NULL，则等待发生在分配本身中。
// 成功后，encrypted_pa​​ge_mask 将作为分配的一部分被清除。
NV_STATUS uvm_conf_computing_dma_buffer_alloc(uvm_conf_computing_dma_buffer_pool_t *dma_buffer_pool,
                                              uvm_conf_computing_dma_buffer_t **out_dma_buffer,
                                              uvm_tracker_t *out_tracker);

// Free a DMA buffer to the DMA allocation pool. All DMA buffers must be freed
// prior to GPU deinit.
//
// The tracker is optional and a NULL tracker indicates that no new operation
// has been pushed for the buffer. A non-NULL tracker indicates any additional
// pending operations on the buffer pushed by the caller that need to be
// synchronized before freeing or re-using the buffer.
// 将 DMA 缓冲区释放到 DMA 分配池。在 GPU 取消初始化之前，必须释放所有 DMA 缓冲区。
//
// 跟踪器是可选的，NULL 跟踪器表示尚未为缓冲区推送任何新操作。非 NULL 跟踪器表示调用者推送
// 的缓冲区上任何其他待处理操作，这些操作需要在释放或重新使用缓冲区之前进行同步。
void uvm_conf_computing_dma_buffer_free(uvm_conf_computing_dma_buffer_pool_t *dma_buffer_pool,
                                        uvm_conf_computing_dma_buffer_t *dma_buffer,
                                        uvm_tracker_t *tracker);

// Synchronize trackers in all entries in the GPU's DMA pool
// 同步 GPU 的 DMA 池中的所有条目中的跟踪器
void uvm_conf_computing_dma_buffer_pool_sync(uvm_conf_computing_dma_buffer_pool_t *dma_buffer_pool);


// Initialization and deinitialization of Confidential Computing data structures
// for the given GPU.
// 初始化和取消初始化给定 GPU 的机密计算数据结构。
NV_STATUS uvm_conf_computing_gpu_init(uvm_gpu_t *gpu);
void uvm_conf_computing_gpu_deinit(uvm_gpu_t *gpu);

// Logs encryption information from the GPU and returns the IV.
// 记录来自 GPU 的加密信息并返回 IV。
void uvm_conf_computing_log_gpu_encryption(uvm_channel_t *channel, UvmCslIv *iv);

// Acquires next CPU encryption IV and returns it.
// 获取下一个CPU加密IV并返回。
void uvm_conf_computing_acquire_encryption_iv(uvm_channel_t *channel, UvmCslIv *iv);

// CPU side encryption helper with explicit IV, which is obtained from
// uvm_conf_computing_acquire_encryption_iv. Without an explicit IV
// the function uses the next IV in order. Encrypts data in src_plain and
// write the cipher text in dst_cipher. src_plain and dst_cipher can't overlap.
// The IV is invalidated and can't be used again after this operation.
// CPU 端加密辅助程序，具有显式 IV，可从 uvm_conf_computing_acquire_encryption_iv 获取。
// 如果没有显式 IV，该函数将按顺序使用下一个 IV。加密 src_plain 中的数据并将密文写入
// dst_cipher。src_plain 和 dst_cipher 不能重叠。此操作后，IV 无效，无法再次使用。
void uvm_conf_computing_cpu_encrypt(uvm_channel_t *channel,
                                    void *dst_cipher,
                                    const void *src_plain,
                                    UvmCslIv *encrypt_iv,
                                    size_t size,
                                    void *auth_tag_buffer)
{
    NV_STATUS status;

    UVM_ASSERT(size);

    uvm_mutex_lock(&channel->csl.ctx_lock);
    status = nvUvmInterfaceCslEncrypt(&channel->csl.ctx,
                                      size,
                                      (NvU8 const *) src_plain,
                                      encrypt_iv,
                                      (NvU8 *) dst_cipher,
                                      (NvU8 *) auth_tag_buffer);
    uvm_mutex_unlock(&channel->csl.ctx_lock);

    // nvUvmInterfaceCslEncrypt fails when a 64-bit encryption counter
    // overflows. This is not supposed to happen on CC.
    UVM_ASSERT(status == NV_OK);
}

// CPU side decryption helper. Decrypts data from src_cipher and writes the
// plain text in dst_plain. src_cipher and dst_plain can't overlap. IV obtained
// from uvm_conf_computing_log_gpu_encryption() needs to be be passed to src_iv.
// CPU 端解密助手。从 src_cipher 解密数据并将纯文本写入 dst_plain。src_cipher 和
// dst_plain 不能重叠。从 uvm_conf_computing_log_gpu_encryption() 获得的 IV 需要传递给 src_iv。
NV_STATUS uvm_conf_computing_cpu_decrypt(uvm_channel_t *channel,
                                         void *dst_plain,
                                         const void *src_cipher,
                                         const UvmCslIv *src_iv,
                                         size_t size,
                                         const void *auth_tag_buffer)
{
    NV_STATUS status;

    uvm_mutex_lock(&channel->csl.ctx_lock);
    status = nvUvmInterfaceCslDecrypt(&channel->csl.ctx,
                                      size,
                                      (const NvU8 *) src_cipher,
                                      src_iv,
                                      (NvU8 *) dst_plain,
                                      (const NvU8 *) auth_tag_buffer);
    uvm_mutex_unlock(&channel->csl.ctx_lock);

    return status;
}

```

### nvrm_registry.h

src/nvidia/interface/nvrm_registry.h

```c


//
// Add the conditions to exclude these macros from Orin build, as CONFIDENTIAL_COMPUTE
// is a guardword. The #if could be removed when nvRmReg.h file is trimmed from Orin build.
//
// Enable Disable Confidential Compute and control its various modes of operation
// 0 - Feature Disable
// 1 - Feature Enable
//
// 添加条件以从 Orin 构建中排除这些宏，因为 CONFIDENTIAL_COMPUTE 是一个保护字。当从 Orin 构建中修剪 nvRmReg.h 文件时，可以删除 #if。
//
// 启用禁用机密计算并控制其各种操作模式
// 0 - 功能禁用
// 1 - 功能启用
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE                              "RmConfidentialCompute"
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_ENABLED                      0:0
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_ENABLED_NO                   0x00000000
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_ENABLED_YES                  0x00000001
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_DEV_MODE_ENABLED             1:1
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_DEV_MODE_ENABLED_NO          0x00000000
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_DEV_MODE_ENABLED_YES         0x00000001
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_GPUS_READY_CHECK             2:2
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_GPUS_READY_CHECK_DISABLED    0x00000000
#define NV_REG_STR_RM_CONFIDENTIAL_COMPUTE_GPUS_READY_CHECK_ENABLED     0x00000001

#define NV_REG_STR_RM_CONF_COMPUTE_EARLY_INIT                            "RmConfComputeEarlyInit"
#define NV_REG_STR_RM_CONF_COMPUTE_EARLY_INIT_DISABLED                   0x00000000
#define NV_REG_STR_RM_CONF_COMPUTE_EARLY_INIT_ENABLED                    0x00000001
```

### uvm_hal.h

```c
void uvm_hal_maxwell_sec2_init_noop(uvm_push_t *push);
void uvm_hal_hopper_sec2_init(uvm_push_t *push);

// Encrypts the contents of the source buffer into the destination buffer, up to
// the given size. The authentication tag of the encrypted contents is written
// to auth_tag, so it can be verified later on by a decrypt operation.
//
// The addressing modes of the destination and authentication tag addresses
// should match. If the addressing mode is physical, then the address apertures
// should also match.
typedef void (*uvm_hal_ce_encrypt_t)(uvm_push_t *push,
                                     uvm_gpu_address_t dst,
                                     uvm_gpu_address_t src,
                                     NvU32 size,
                                     uvm_gpu_address_t auth_tag);

// Decrypts the contents of the source buffer into the destination buffer, up to
// the given size. The method also verifies the integrity of the encrypted
// buffer by calculating its authentication tag, and comparing it with the one
// provided as argument.
//
// The addressing modes of the source and authentication tag addresses should
// match. If the addressing mode is physical, then the address apertures should
// also match.
typedef void (*uvm_hal_ce_decrypt_t)(uvm_push_t *push,
                                     uvm_gpu_address_t dst,
                                     uvm_gpu_address_t src,
                                     NvU32 size,
                                     uvm_gpu_address_t auth_tag);

void uvm_hal_maxwell_ce_encrypt_unsupported(uvm_push_t *push,
                                            uvm_gpu_address_t dst,
                                            uvm_gpu_address_t src,
                                            NvU32 size,
                                            uvm_gpu_address_t auth_tag);
void uvm_hal_maxwell_ce_decrypt_unsupported(uvm_push_t *push,
                                            uvm_gpu_address_t dst,
                                            uvm_gpu_address_t src,
                                            NvU32 size,
                                            uvm_gpu_address_t auth_tag);
void uvm_hal_hopper_ce_encrypt(uvm_push_t *push,
                               uvm_gpu_address_t dst,
                               uvm_gpu_address_t src,
                               NvU32 size,
                               uvm_gpu_address_t auth_tag);
void uvm_hal_hopper_ce_decrypt(uvm_push_t *push,
                               uvm_gpu_address_t dst,
                               uvm_gpu_address_t src,
                               NvU32 size,
                               uvm_gpu_address_t auth_tag);

// 源地址和目标地址必须是16字节对齐的。注意，最佳性能是在256字节对齐时实现的。解密大小必须大于0，并且是4字节的倍数。
//
// 认证标签地址也必须是16字节对齐的。
// 认证标签缓冲区大小在uvm_conf_computing.h中定义为UVM_CONF_COMPUTING_AUTH_TAG_SIZE字节。
//
// 将src缓冲区解密到给定大小的dst缓冲区中。
// 该方法还通过计算src缓冲区的认证标签并与提供的标签进行比较来验证其完整性。
//
// 注意：SEC2不支持加密。
typedef void (*uvm_hal_sec2_decrypt_t)(uvm_push_t *push, NvU64 dst_va, NvU64 src_va, NvU32 size, NvU64 auth_tag_va);

void uvm_hal_maxwell_sec2_decrypt_unsupported(uvm_push_t *push,
                                              NvU64 dst_va,
                                              NvU64 src_va,
                                              NvU32 size,
                                              NvU64 auth_tag_va);
void uvm_hal_hopper_sec2_decrypt(uvm_push_t *push, NvU64 dst_va, NvU64 src_va, NvU32 size, NvU64 auth_tag_va);


struct uvm_ce_hal_struct
{
    // ...
    uvm_hal_ce_memcopy_type_t memcopy_copy_type;
    // ...
    uvm_hal_ce_encrypt_t encrypt;
    uvm_hal_ce_decrypt_t decrypt;
};

struct uvm_sec2_hal_struct
{
    uvm_hal_init_t init;
    uvm_hal_sec2_decrypt_t decrypt;
    uvm_hal_semaphore_release_t semaphore_release;
    uvm_hal_semaphore_timestamp_t semaphore_timestamp;
};

typedef struct
{
    // id is either a hardware class or GPU architecture
    NvU32 id;
    NvU32 parent_id;
    union
    {
        // host_ops: id is a hardware class
        uvm_host_hal_t host_ops;

        // ce_ops: id is a hardware class
        uvm_ce_hal_t ce_ops;

        // arch_ops: id is an architecture
        uvm_arch_hal_t arch_ops;

        // fault_buffer_ops: id is an architecture
        uvm_fault_buffer_hal_t fault_buffer_ops;

        // access_counter_buffer_ops: id is an architecture
        uvm_access_counter_buffer_hal_t access_counter_buffer_ops;

        // sec2_ops: id is an architecture
        uvm_sec2_hal_t sec2_ops;
    } u;
} uvm_hal_class_ops_t;
```

#### Copy Engine

uvm_hal.c

```c

// Table for copy engine functions.
// Each entry is associated with a copy engine class through the 'class' field.
// By setting the 'parent_class' field, a class will inherit the parent class's
// functions for any fields left NULL when uvm_hal_init_table() runs upon module
// load. The parent class must appear earlier in the array than the child.
static uvm_hal_class_ops_t ce_table[] =
{
    {
        .id = MAXWELL_DMA_COPY_A,
        .u.ce_ops = {
            // ...
            .encrypt = uvm_hal_maxwell_ce_encrypt_unsupported,
            .decrypt = uvm_hal_maxwell_ce_decrypt_unsupported,
        }
    },
    // ...
    {
        .id = HOPPER_DMA_COPY_A,
        .parent_id = AMPERE_DMA_COPY_B,
        .u.ce_ops = {
            // ...
            .memcopy_copy_type = uvm_hal_hopper_ce_memcopy_copy_type,
            // ...
            .encrypt = uvm_hal_hopper_ce_encrypt,
            .decrypt = uvm_hal_hopper_ce_decrypt,
        },
    },
};
```

## C program

### uvm_gpu.c: uvm_ioctl

kernel-open/nvidia-uvm/uvm_gpu.c

```c
uvm_ioctl()
--> UVM_ROUTE_CMD_STACK_INIT_CHECK(UVM_REGISTER_GPU,                   uvm_api_register_gpu);
    --> uvm_va_space_register_gpu()
        --> uvm_gpu_retain_by_uuid()
            --> gpu_retain_by_uuid_locked()
                --> add_gpu()
                    --> init_gpu()
                        --> status = uvm_conf_computing_gpu_init(gpu);
                            // Initialization and deinitialization of Confidential Computing data structures
                            // for the given GPU.
                            if (status != NV_OK) {
                                UVM_ERR_PRINT("Failed to initialize Confidential Compute: %s for GPU %s\n",
                                            nvstatusToString(status),
                                            uvm_gpu_name(gpu));
                                return status;
                        }
```

### uvm_conf_computing_gpu_init

kernel-open/nvidia-uvm/uvm_conf_computing.c

```c
NV_STATUS uvm_conf_computing_gpu_init(uvm_gpu_t *gpu)
{
    NV_STATUS status;

    if (!uvm_conf_computing_mode_enabled(gpu))
        return NV_OK;

    status = conf_computing_dma_buffer_pool_init(&gpu->conf_computing.dma_buffer_pool);
    if (status != NV_OK)
        return status;

    status = dummy_iv_mem_init(gpu);
    if (status != NV_OK)
        goto error;

    return NV_OK;

error:
    uvm_conf_computing_gpu_deinit(gpu);
    return status;
}
```

### uvm_conf_computing_mode_enabled

```c
uvm_conf_computing_mode_enabled()
--> return uvm_conf_computing_get_mode(parent) != UVM_GU_CONF_COMPUTE_MODE_NONE;
    --> return parent->rm_info.gpuConfComputeCaps.mode;

typedef enum
{
    UVM_GPU_CONF_COMPUTE_MODE_NONE,
    UVM_GPU_CONF_COMPUTE_MODE_APM,
    UVM_GPU_CONF_COMPUTE_MODE_HCC,
    UVM_GPU_CONF_COMPUTE_MODE_COUNT
} UvmGpuConfComputeMode;

typedef struct UvmGpuInfo_tag
{
    // ...

    // Confidential Compute capabilities of this GPU
    UvmGpuConfComputeCaps gpuConfComputeCaps;

    // ...
} UvmGpuInfo;

nvGpuOpsGetGpuInfo()
--> status = nvGpuOpsQueryGpuConfidentialComputeCaps(clientHandle, &pGpuInfo->gpuConfComputeCaps);

static NV_STATUS
nvGpuOpsQueryGpuConfidentialComputeCaps(NvHandle hClient,
                                        UvmGpuConfComputeCaps *pGpuConfComputeCaps)
{
    NV_CONFIDENTIAL_COMPUTE_ALLOC_PARAMS confComputeAllocParams = {0};
    NV_CONF_COMPUTE_CTRL_CMD_SYSTEM_GET_CAPABILITIES_PARAMS confComputeParams = {0};
    RM_API *pRmApi = rmapiGetInterface(RMAPI_EXTERNAL_KERNEL);
    NvHandle hConfCompute = 0;
    NV_STATUS status = NV_OK;

    // ...

    if (confComputeParams.ccFeature == NV_CONF_COMPUTE_SYSTEM_FEATURE_APM_ENABLED)
    {
        pGpuConfComputeCaps->mode = UVM_GPU_CONF_COMPUTE_MODE_APM;
    }
    else if (confComputeParams.ccFeature == NV_CONF_COMPUTE_SYSTEM_FEATURE_HCC_ENABLED)
    {
        pGpuConfComputeCaps->mode = UVM_GPU_CONF_COMPUTE_MODE_HCC;
    }

cleanup:
    pRmApi->Free(pRmApi, hClient, hConfCompute);
    return status;
}
```

### conf_computing_dma_buffer_pool_init

kernel-open/nvidia-uvm/uvm_conf_computing.c

Allocate and map a new DMA stage buffer to CPU and GPU (VA)
分配并映射新的 DMA 阶段缓冲区到 CPU 和 GPU（VA）
主体调用了 `dma_buffer_create()`

### dma_buffer_create

这段代码定义了一个静态函数 `dma_buffer_create`，用于分配并映射新的DMA缓冲区到CPU和GPU的虚拟地址空间。以下是对这段代码的详细分析：

#### 函数签名

```c
static NV_STATUS dma_buffer_create(uvm_conf_computing_dma_buffer_pool_t *dma_buffer_pool,
                                   uvm_conf_computing_dma_buffer_t **dma_buffer_out)
```

- **参数**：
  - `dma_buffer_pool`：指向DMA缓冲区池的指针，用于管理DMA缓冲区的分配。
  - `dma_buffer_out`：指向指针的指针，用于输出分配的DMA缓冲区。

- **返回值**：
  - `NV_STATUS`：表示操作的状态，可以是成功（`NV_OK`）或失败（如`NV_ERR_NO_MEMORY`）。

#### 变量定义和初始化

```c
uvm_gpu_t *dma_owner;
uvm_conf_computing_dma_buffer_t *dma_buffer;
uvm_mem_t *alloc = NULL;
NV_STATUS status = NV_OK;
size_t auth_tags_size = (UVM_CONF_COMPUTING_DMA_BUFFER_SIZE / PAGE_SIZE) * UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
```

- **变量**：
  - `dma_owner`：指向拥有DMA缓冲区的GPU。
  - `dma_buffer`：指向DMA缓冲区的指针。
  - `alloc`：指向内存分配结构的指针。
  - `status`：表示操作状态，初始值为`NV_OK`。
  - `auth_tags_size`：计算认证标签所需的内存大小。

#### 分配并初始化DMA缓冲区结构

```c
dma_buffer = uvm_kvmalloc_zero(sizeof(*dma_buffer));
if (!dma_buffer)
    return NV_ERR_NO_MEMORY;
```

- **操作**：
  - 使用 `uvm_kvmalloc_zero` 分配并初始化DMA缓冲区结构。
  - 如果分配失败，返回`NV_ERR_NO_MEMORY`。

#### 获取DMA缓冲区所有者并初始化相关结构

```c
dma_owner = dma_buffer_pool_to_gpu(dma_buffer_pool);
uvm_tracker_init(&dma_buffer->tracker);
INIT_LIST_HEAD(&dma_buffer->node);
```

- **操作**：
  - 通过 `dma_buffer_pool_to_gpu` 函数获取DMA缓冲区的所有者GPU。
  - 初始化DMA缓冲区的追踪器和链表头。

#### 分配和映射DMA缓冲区到CPU内核空间

```c
status = uvm_mem_alloc_sysmem_dma_and_map_cpu_kernel(UVM_CONF_COMPUTING_DMA_BUFFER_SIZE, dma_owner, NULL, &alloc);
if (status != NV_OK)
    goto err;

dma_buffer->alloc = alloc;
```

- **操作**：
  - 调用 `uvm_mem_alloc_sysmem_dma_and_map_cpu_kernel` 分配系统内存并映射到CPU内核空间。
  - 如果分配失败，跳转到错误处理部分。
  - 将分配的内存指针赋值给 `dma_buffer->alloc`。

#### 将分配的内存映射到GPU内核空间

```c
status = uvm_mem_map_gpu_kernel(alloc, dma_owner);
if (status != NV_OK)
    goto err;
```

- **操作**：
  - 调用 `uvm_mem_map_gpu_kernel` 将分配的内存映射到GPU内核空间。
  - 如果映射失败，跳转到错误处理部分。

#### 分配和映射认证标签内存到CPU内核空间

```c
status = uvm_mem_alloc_sysmem_dma_and_map_cpu_kernel(auth_tags_size, dma_owner, NULL, &alloc);
if (status != NV_OK)
    goto err;

dma_buffer->auth_tag = alloc;
```

- **操作**：
  - 分配用于认证标签的系统内存并映射到CPU内核空间。
  - 如果分配失败，跳转到错误处理部分。
  - 将分配的内存指针赋值给 `dma_buffer->auth_tag`。

#### 将认证标签内存映射到GPU内核空间

```c
status = uvm_mem_map_gpu_kernel(alloc, dma_owner);
if (status != NV_OK)
    goto err;
```

- **操作**：
  - 将认证标签的内存映射到GPU内核空间。
  - 如果映射失败，跳转到错误处理部分。

#### 成功分配并返回

```c
*dma_buffer_out = dma_buffer;
return status;
```

- **操作**：
  - 将分配的DMA缓冲区指针赋值给输出参数 `dma_buffer_out`。
  - 返回操作状态 `status`。

#### 错误处理部分

```c
err:
dma_buffer_destroy_locked(dma_buffer_pool, dma_buffer);
return status;
```

- **操作**：
  - 如果任何一步失败，跳转到错误处理部分，销毁已经分配的DMA缓冲区。
  - 返回操作状态 `status`。

#### 总结

这段代码的主要目的是分配并初始化一个新的DMA缓冲区，包括认证标签，确保缓冲区既映射到CPU内核空间，也映射到GPU内核空间。整个过程包括：

1. 分配DMA缓冲区结构。
2. 初始化追踪器和链表头。
3. 分配并映射DMA缓冲区到CPU和GPU。
4. 分配并映射认证标签内存到CPU和GPU。
5. 错误处理，确保在任何一步失败时都能清理已分配的资源。

理解这段代码的工作原理，有助于确保在实际应用中正确使用和管理DMA缓冲区。

### dummy_iv_mem_init

这段代码的功能是为指定的GPU分配和映射用于存储初始化向量（IV）的内存。如果GPU处于特定的计算模式（例如HCC模式），则执行这些操作。

```c
static NV_STATUS dummy_iv_mem_init(uvm_gpu_t *gpu)
```

#### 参数

- **gpu**：指向表示GPU的结构体的指针。

#### 返回值

- **NV_STATUS**：表示操作的状态，可以是`NV_OK`（成功）或其他错误代码。

#### 检查GPU计算模式

```c
if (!uvm_conf_computing_mode_is_hcc(gpu))
    return NV_OK;
```

- **操作**：
  - 检查GPU是否处于特定的计算模式（例如HCC模式）。如果不处于该模式，则不需要进行任何初始化，直接返回成功状态`NV_OK`。

#### 分配系统内存用于初始化向量（IV）

```c
status = uvm_mem_alloc_sysmem_dma(sizeof(UvmCslIv), gpu, NULL, &gpu->conf_computing.iv_mem);
if (status != NV_OK)
    return status;
```

- **操作**：
  - 调用 `uvm_mem_alloc_sysmem_dma` 函数分配系统内存，并将其映射为DMA内存。
  - 分配的内存大小为 `sizeof(UvmCslIv)`，存储在 `gpu->conf_computing.iv_mem` 中。
  - 如果分配失败，返回相应的错误状态 `status`。

#### 将分配的内存映射到GPU内核空间

```c
status = uvm_mem_map_gpu_kernel(gpu->conf_computing.iv_mem, gpu);
if (status != NV_OK)
    goto error;
```

- **操作**：
  - 调用 `uvm_mem_map_gpu_kernel` 函数将分配的内存映射到GPU内核空间。
  - 如果映射失败，跳转到错误处理部分 `error`。

#### 成功返回

```c
return NV_OK;
```

- **操作**：
  - 如果所有操作都成功，返回 `NV_OK`，表示初始化成功。

#### 错误处理部分

```c
error:
dummy_iv_mem_deinit(gpu);
return status;
```

- **操作**：
  - 如果在内存分配或映射过程中发生错误，跳转到 `error` 标签。
  - 调用 `dummy_iv_mem_deinit` 函数进行清理，释放已经分配的资源。
  - 返回错误状态 `status`。

1. **检查GPU计算模式**：如果GPU不处于特定计算模式，则直接返回成功状态。
2. **分配内存**：尝试为初始化向量（IV）分配系统内存。如果分配失败，返回错误状态。
3. **映射内存**：将分配的内存映射到GPU内核空间。如果映射失败，执行错误处理。
4. **错误处理**：在发生错误时，清理已分配的资源并返回错误状态。

这段代码的主要目的是在特定计算模式下为GPU初始化用于存储初始化向量（IV）的内存。通过检查计算模式、分配内存和映射内存，这个函数确保GPU能够正确处理中断向量。如果在任何步骤中发生错误，函数会清理已分配的资源，并返回相应的错误状态。理解这个流程有助于正确管理GPU资源和处理错误情况。

```c
static NV_STATUS dummy_iv_mem_init(uvm_gpu_t *gpu)
{
    NV_STATUS status;

    if (!uvm_conf_computing_mode_is_hcc(gpu))
        return NV_OK;

    status = uvm_mem_alloc_sysmem_dma(sizeof(UvmCslIv), gpu, NULL, &gpu->conf_computing.iv_mem);
    if (status != NV_OK)
        return status;

    status = uvm_mem_map_gpu_kernel(gpu->conf_computing.iv_mem, gpu);
    if (status != NV_OK)
        goto error;

    return NV_OK;

error:
    dummy_iv_mem_deinit(gpu);
    return status;
}

struct uvm_gpu_struct
{
    uvm_parent_gpu_t *parent;
    // ...

    struct
    {
        uvm_conf_computing_dma_buffer_pool_t dma_buffer_pool;

        // 在CE加密过程中用于存储IV内容的临时内存。
        // 这个内存位置只有在CE通道之后才可用，
        // 因为我们使用它们来写入分配所需的PTE（页表项）。
        // 当需要物理地址来访问IV缓冲区时使用这个位置。
        // 参考函数：uvm_hal_hopper_ce_encrypt()。
        uvm_mem_t *iv_mem;

        // 在CE加密过程中用于存储IV内容的临时内存。
        // 由于`iv_mem`的限制，并且需要在通道初始化时使用此类缓冲区，
        // 我们使用RM分配。
        // 当需要虚拟地址来访问IV缓冲区时使用这个位置。
        // 参考函数：uvm_hal_hopper_ce_encrypt()。
        uvm_rm_mem_t *iv_rm_mem;
    } conf_computing;

    // ...
};
```

### uvm_va_block.c

```c
block_copy_push()
--> conf_computing_block_copy_push_cpu_to_gpu() / conf_computing_block_copy_push_gpu_to_cpu()

block_copy_end_push()
--> conf_computing_copy_pages_finish()

UVM_ROUTE_CMD_STACK_INIT_CHECK(UVM_TOOLS_READ_PROCESS_MEMORY,      uvm_api_tools_read_process_memory);
UVM_ROUTE_CMD_STACK_INIT_CHECK(UVM_TOOLS_WRITE_PROCESS_MEMORY,     uvm_api_tools_write_process_memory);
--> tools_access_va_block()
    static NV_STATUS tools_access_va_block(uvm_va_block_t *va_block,
                                        uvm_va_block_context_t *block_context,
                                        NvU64 target_va,
                                        NvU64 size,
                                        bool is_write,
                                        uvm_mem_t *stage_mem)
    {
        if (is_write) {
            return UVM_VA_BLOCK_LOCK_RETRY(va_block,
                                        NULL,
                                        uvm_va_block_write_from_cpu(va_block, block_context, target_va, stage_mem, size));
        }
        else {
            return UVM_VA_BLOCK_LOCK_RETRY(va_block,
                                        NULL,
                                        uvm_va_block_read_to_cpu(va_block, stage_mem, target_va, size));

        }
    }

uvm_va_block_write_from_cpu()
--> va_block_write_cpu_to_gpu()
    --> encrypted_memcopy_cpu_to_gpu()
        --> uvm_conf_computing_cpu_encrypt()

uvm_va_block_read_to_cpu()
--> va_block_read_gpu_to_cpu()
    --> encrypted_memcopy_gpu_to_cpu()
        --> uvm_conf_computing_cpu_decrypt()
```

### uvm_hal_hopper_ce_encrypt

这段代码定义了一个函数 `uvm_hal_hopper_ce_encrypt`，用于在指定的GPU上执行加密操作。它将源地址的数据加密后存储到目标地址，并生成一个认证标签。以下是对这段代码的详细分析：

```c
void uvm_hal_hopper_ce_encrypt(uvm_push_t *push,
                               uvm_gpu_address_t dst,
                               uvm_gpu_address_t src,
                               NvU32 size,
                               uvm_gpu_address_t auth_tag)
```

#### 参数

- **push**：指向用于推送命令的结构体。
- **dst**：目标地址，解密后的数据将存储在此地址。
- **src**：源地址，包含需要加密的数据。
- **size**：加密数据的大小，必须大于0且为4字节的倍数。
- **auth_tag**：认证标签的地址，用于验证源数据的完整性。

#### 获取GPU和参数验证

```c
uvm_gpu_t *gpu = uvm_push_get_gpu(push);

UVM_ASSERT(uvm_conf_computing_mode_is_hcc(gpu));
UVM_ASSERT(uvm_push_is_fake(push) || uvm_channel_is_secure(push->channel));
UVM_ASSERT(IS_ALIGNED(auth_tag.address, UVM_CONF_COMPUTING_AUTH_TAG_ALIGNMENT));

if (!src.is_virtual)
    UVM_ASSERT(src.aperture == UVM_APERTURE_VID);
```

- **获取GPU**：从 `push` 结构体中获取GPU指针。
- **参数验证**：
  - 确保GPU处于特定计算模式（例如HCC模式）。
  - 确保 `push` 是虚拟的或通道是安全的。
  - 确保认证标签地址是对齐的。
  - 如果源地址不是虚拟地址，确保其光圈类型是 `UVM_APERTURE_VID`。

#### 目的地址和认证标签验证

```c
UVM_ASSERT(dst.is_virtual == auth_tag.is_virtual);

if (!dst.is_virtual) {
    UVM_ASSERT(dst.aperture == UVM_APERTURE_SYS);
    UVM_ASSERT(auth_tag.aperture == UVM_APERTURE_SYS);
}
```

- **验证地址模式**：
  - 确保目标地址和认证标签的地址模式一致（都是虚拟或都是物理地址）。
  - 如果目标地址不是虚拟地址，确保其光圈类型和认证标签的光圈类型都是 `UVM_APERTURE_SYS`。

#### 设置加密模式

```c
NV_PUSH_1U(C8B5, SET_SECURE_COPY_MODE, HWCONST(C8B5, SET_SECURE_COPY_MODE, MODE, ENCRYPT));
```

- **操作**：
  - 通过 `NV_PUSH_1U` 设置安全复制模式为加密。

#### 设置认证标签和IV地址

```c
auth_tag_address_hi32 = HWVALUE(C8B5, SET_ENCRYPT_AUTH_TAG_ADDR_UPPER, UPPER, NvU64_HI32(auth_tag.address));
auth_tag_address_lo32 = HWVALUE(C8B5, SET_ENCRYPT_AUTH_TAG_ADDR_LOWER, LOWER, NvU64_LO32(auth_tag.address));

iv_address = encrypt_iv_address(push, dst);

iv_address_hi32 = HWVALUE(C8B5, SET_ENCRYPT_IV_ADDR_UPPER, UPPER, NvU64_HI32(iv_address));
iv_address_lo32 = HWVALUE(C8B5, SET_ENCRYPT_IV_ADDR_LOWER, LOWER, NvU64_LO32(iv_address));

NV_PUSH_4U(C8B5, SET_ENCRYPT_AUTH_TAG_ADDR_UPPER, auth_tag_address_hi32,
                     SET_ENCRYPT_AUTH_TAG_ADDR_LOWER, auth_tag_address_lo32,
                     SET_ENCRYPT_IV_ADDR_UPPER, iv_address_hi32,
                     SET_ENCRYPT_IV_ADDR_LOWER, iv_address_lo32);
```

- **操作**：
  - 计算并设置认证标签的高32位和低32位地址。
  - 计算初始化向量（IV）的地址。
  - 计算并设置IV的高32位和低32位地址。
  - 使用 `NV_PUSH_4U` 将这些地址推送到硬件寄存器。

#### 执行加密操作

```c
encrypt_or_decrypt(push, dst, src, size);
```

- **操作**：
  - 调用 `encrypt_or_decrypt` 函数执行实际的加密操作，将源地址的数据加密后存储到目标地址。

#### 总结

函数 `uvm_hal_hopper_ce_encrypt` 主要执行以下步骤：

1. **获取GPU和验证参数**：确保GPU处于正确的模式，地址对齐，并且源和目标地址模式一致。
2. **设置加密模式**：通过硬件寄存器设置加密模式。
3. **设置地址**：计算并设置认证标签和IV的地址，将这些地址推送到硬件寄存器。
4. **执行加密**：调用实际的加密函数，将源地址的数据加密后存储到目标地址。

#### 流程图

```txt
获取GPU和验证参数
    |
    v
设置加密模式
    |
    v
计算并设置认证标签地址和IV地址
    |
    v
执行加密操作
```

通过这些步骤，该函数确保在GPU上安全高效地执行加密操作，并生成必要的认证标签以验证数据的完整性。

### uvm_hal_hopper_ce_decrypt

这段代码定义了一个函数 `uvm_hal_hopper_ce_decrypt`，用于在指定的GPU上执行解密操作。它将源地址的数据解密后存储到目标地址，并验证源数据的完整性。以下是对这段代码的详细分析：

```c
void uvm_hal_hopper_ce_decrypt(uvm_push_t *push,
                               uvm_gpu_address_t dst,
                               uvm_gpu_address_t src,
                               NvU32 size,
                               uvm_gpu_address_t auth_tag)
```

#### 参数

- **push**：指向用于推送命令的结构体。
- **dst**：目标地址，解密后的数据将存储在此地址。
- **src**：源地址，包含需要解密的数据。
- **size**：解密数据的大小，必须大于0且为4字节的倍数。
- **auth_tag**：认证标签的地址，用于验证源数据的完整性。

#### 获取GPU和参数验证

```c
uvm_gpu_t *gpu = uvm_push_get_gpu(push);

UVM_ASSERT(uvm_conf_computing_mode_is_hcc(gpu));
UVM_ASSERT(!push->channel || uvm_channel_is_secure(push->channel));
UVM_ASSERT(IS_ALIGNED(auth_tag.address, UVM_CONF_COMPUTING_AUTH_TAG_ALIGNMENT));
```

- **获取GPU**：从 `push` 结构体中获取GPU指针。
- **参数验证**：
  - 确保GPU处于特定计算模式（例如HCC模式）。
  - 确保 `push` 没有通道或者通道是安全的。
  - 确保认证标签地址是对齐的。

#### 源地址和认证标签验证

```c
UVM_ASSERT(src.is_virtual == auth_tag.is_virtual);

if (!src.is_virtual) {
    UVM_ASSERT(src.aperture == UVM_APERTURE_SYS);
    UVM_ASSERT(auth_tag.aperture == UVM_APERTURE_SYS);
}
```

- **验证地址模式**：
  - 确保源地址和认证标签的地址模式一致（都是虚拟或都是物理地址）。
  - 如果源地址不是虚拟地址，确保其光圈类型和认证标签的光圈类型都是 `UVM_APERTURE_SYS`。

#### 目的地址验证

```c
if (!dst.is_virtual)
    UVM_ASSERT(dst.aperture == UVM_APERTURE_VID);
```

- **验证目的地址**：
  - 如果目标地址不是虚拟地址，确保其光圈类型是 `UVM_APERTURE_VID`。

#### 设置解密模式

```c
NV_PUSH_1U(C8B5, SET_SECURE_COPY_MODE, HWCONST(C8B5, SET_SECURE_COPY_MODE, MODE, DECRYPT));
```

- **操作**：
  - 通过 `NV_PUSH_1U` 设置安全复制模式为解密。

#### 设置认证标签地址

```c
auth_tag_address_hi32 = HWVALUE(C8B5, SET_DECRYPT_AUTH_TAG_COMPARE_ADDR_UPPER, UPPER, NvU64_HI32(auth_tag.address));
auth_tag_address_lo32 = HWVALUE(C8B5, SET_DECRYPT_AUTH_TAG_COMPARE_ADDR_LOWER, LOWER, NvU64_LO32(auth_tag.address));

NV_PUSH_2U(C8B5, SET_DECRYPT_AUTH_TAG_COMPARE_ADDR_UPPER, auth_tag_address_hi32,
                     SET_DECRYPT_AUTH_TAG_COMPARE_ADDR_LOWER, auth_tag_address_lo32);
```

- **操作**：
  - 计算并设置认证标签的高32位和低32位地址。
  - 使用 `NV_PUSH_2U` 将这些地址推送到硬件寄存器。

#### 执行解密操作

```c
encrypt_or_decrypt(push, dst, src, size);
```

- **操作**：
  - 调用 `encrypt_or_decrypt` 函数执行实际的解密操作，将源地址的数据解密后存储到目标地址。

#### 总结

函数 `uvm_hal_hopper_ce_decrypt` 主要执行以下步骤：

1. **获取GPU和验证参数**：确保GPU处于正确的模式，地址对齐，并且源和认证标签的地址模式一致。
2. **验证地址模式**：确保源地址和认证标签的光圈类型一致，如果目标地址不是虚拟地址，验证其光圈类型。
3. **设置解密模式**：通过硬件寄存器设置解密模式。
4. **设置认证标签地址**：计算并设置认证标签的高32位和低32位地址，将这些地址推送到硬件寄存器。
5. **执行解密操作**：调用实际的解密函数，将源地址的数据解密后存储到目标地址。

#### 流程图

```txt
获取GPU和验证参数
    |
    v
验证源地址和认证标签地址模式
    |
    v
验证目的地址模式
    |
    v
设置解密模式
    |
    v
计算并设置认证标签地址
    |
    v
执行解密操作
```

通过这些步骤，该函数确保在GPU上安全高效地执行解密操作，并验证数据的完整性。

### uvm_va_block.c: conf_computing_block_copy_push_cpu_to_gpu

该函数

```c
gpu->parent->ce_hal->decrypt(push, dst_address, staging_buffer, PAGE_SIZE, auth_tag_buffer);
```

调用了decrypt，间接调用了 `uvm_hal_hopper_ce_decrypt`

kernel-open/nvidia-uvm/uvm_va_block.c

这个函数 `conf_computing_block_copy_push_cpu_to_gpu` 实现了在启用机密计算功能时，CPU 端页面加密和 GPU 端解密操作。它使用推送命令（push）在 GPU 上执行这些操作，并且 GPU 操作会遵守调用者先前在推送命令中设置的内存屏障（membar）。以下是对这段代码的详细分析：

```c
static void conf_computing_block_copy_push_cpu_to_gpu(uvm_va_block_t *block,
                                                      block_copy_state_t *copy_state,
                                                      uvm_va_block_region_t region,
                                                      uvm_push_t *push)
```

#### 参数

- **block**：表示虚拟地址块的结构体指针。
- **copy_state**：表示块复制状态的结构体指针。
- **region**：表示虚拟地址块区域的结构体。
- **push**：用于推送命令的结构体指针。

#### 初始化变量

```c
uvm_push_flag_t membar_flag = 0;
uvm_gpu_t *gpu = uvm_push_get_gpu(push);
uvm_page_index_t page_index = region.first;
uvm_conf_computing_dma_buffer_t *dma_buffer = copy_state->dma_buffer;
struct page *src_page = uvm_cpu_chunk_get_cpu_page(block, page_index);
uvm_gpu_address_t staging_buffer = uvm_mem_gpu_address_virtual_kernel(dma_buffer->alloc, gpu);
uvm_gpu_address_t auth_tag_buffer = uvm_mem_gpu_address_virtual_kernel(dma_buffer->auth_tag, gpu);
char *cpu_auth_tag_buffer = (char *)uvm_mem_get_cpu_addr_kernel(dma_buffer->auth_tag) +
                                        (page_index * UVM_CONF_COMPUTING_AUTH_TAG_SIZE);
uvm_gpu_address_t dst_address = block_copy_get_address(block, &copy_state->dst, page_index, gpu);
char *cpu_va_staging_buffer = (char *)uvm_mem_get_cpu_addr_kernel(dma_buffer->alloc) + (page_index * PAGE_SIZE);
```

- **初始化 GPU 和 DMA 缓冲区相关的变量**。
- **获取源页面、目标地址和认证标签的相关信息**。

#### 参数和状态验证

```c
UVM_ASSERT(UVM_ID_IS_CPU(copy_state->src.id));
UVM_ASSERT(UVM_ID_IS_GPU(copy_state->dst.id));
UVM_ASSERT(uvm_conf_computing_mode_enabled(gpu));
UVM_ASSERT(uvm_tracker_is_completed(&block->tracker));
```

- **验证源和目标是否为 CPU 和 GPU**。
- **确保 GPU 启用了机密计算模式**。
- **确保块追踪器已经完成**。

#### 调整缓冲区地址

```c
staging_buffer.address += page_index * PAGE_SIZE;
auth_tag_buffer.address += page_index * UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
```

- **根据页面索引调整 staging_buffer 和 auth_tag_buffer 的地址**。

#### 内存屏障标志设置

```c
if (uvm_push_get_and_reset_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_NONE))
    membar_flag = UVM_PUSH_FLAG_NEXT_MEMBAR_NONE;
else if (uvm_push_get_and_reset_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_GPU))
    membar_flag = UVM_PUSH_FLAG_NEXT_MEMBAR_GPU;
```

- **根据推送命令设置内存屏障标志**。

#### 循环处理每个页面

```c
for_each_va_block_page_in_region(page_index, region) {
    void *src_cpu_virt_addr;

    UVM_ASSERT(src_page == uvm_cpu_chunk_get_cpu_page(block, page_index));

    src_cpu_virt_addr = kmap(src_page);
    uvm_conf_computing_cpu_encrypt(push->channel,
                                   cpu_va_staging_buffer,
                                   src_cpu_virt_addr,
                                   NULL,
                                   PAGE_SIZE,
                                   cpu_auth_tag_buffer);
    kunmap(src_page);

    if (page_index > region.first)
        uvm_push_set_flag(push, UVM_PUSH_FLAG_CE_NEXT_PIPELINED);

    if (page_index < (region.outer - 1))
        uvm_push_set_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_NONE);
    else if (membar_flag)
        uvm_push_set_flag(push, membar_flag);

    gpu->parent->ce_hal->decrypt(push, dst_address, staging_buffer, PAGE_SIZE, auth_tag_buffer);

    src_page++;
    dst_address.address += PAGE_SIZE;
    cpu_va_staging_buffer += PAGE_SIZE;
    staging_buffer.address += PAGE_SIZE;
    cpu_auth_tag_buffer += UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
    auth_tag_buffer.address += UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
}
```

- **循环处理区域内的每个页面**。
- **对每个页面进行 kmap 和 kunmap 操作**。
- **执行 CPU 端加密，并使用 GPU 端解密**。

#### 中文注释

```c
// 当启用机密计算功能时，该函数执行 CPU 端页面加密和 GPU 端解密到 CPR。
// GPU 操作遵守调用者在推送命令中先前设置的内存屏障。
static void conf_computing_block_copy_push_cpu_to_gpu(uvm_va_block_t *block,
                                                      block_copy_state_t *copy_state,
                                                      uvm_va_block_region_t region,
                                                      uvm_push_t *push)
{
    uvm_push_flag_t membar_flag = 0;
    uvm_gpu_t *gpu = uvm_push_get_gpu(push);
    uvm_page_index_t page_index = region.first;
    uvm_conf_computing_dma_buffer_t *dma_buffer = copy_state->dma_buffer;
    struct page *src_page = uvm_cpu_chunk_get_cpu_page(block, page_index);
    uvm_gpu_address_t staging_buffer = uvm_mem_gpu_address_virtual_kernel(dma_buffer->alloc, gpu);
    uvm_gpu_address_t auth_tag_buffer = uvm_mem_gpu_address_virtual_kernel(dma_buffer->auth_tag, gpu);
    char *cpu_auth_tag_buffer = (char *)uvm_mem_get_cpu_addr_kernel(dma_buffer->auth_tag) +
                                        (page_index * UVM_CONF_COMPUTING_AUTH_TAG_SIZE);
    uvm_gpu_address_t dst_address = block_copy_get_address(block, &copy_state->dst, page_index, gpu);
    char *cpu_va_staging_buffer = (char *)uvm_mem_get_cpu_addr_kernel(dma_buffer->alloc) + (page_index * PAGE_SIZE);

    UVM_ASSERT(UVM_ID_IS_CPU(copy_state->src.id));
    UVM_ASSERT(UVM_ID_IS_GPU(copy_state->dst.id));

    UVM_ASSERT(uvm_conf_computing_mode_enabled(gpu));

    // 参见 block_copy_begin_push 中的注释。
    UVM_ASSERT(uvm_tracker_is_completed(&block->tracker));

    staging_buffer.address += page_index * PAGE_SIZE;
    auth_tag_buffer.address += page_index * UVM_CONF_COMPUTING_AUTH_TAG_SIZE;

    if (uvm_push_get_and_reset_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_NONE))
        membar_flag = UVM_PUSH_FLAG_NEXT_MEMBAR_NONE;
    else if (uvm_push_get_and_reset_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_GPU))
        membar_flag = UVM_PUSH_FLAG_NEXT_MEMBAR_GPU;

    // kmap() 只保证 PAGE_SIZE 的连续性，所有加密和解密必须在 PAGE_SIZE 的基础上进行。
    for_each_va_block_page_in_region(page_index, region) {
        void *src_cpu_virt_addr;

        // 调用者保证区域内的所有页面都是连续的，
        // 这意味着它们被保证是同一个复合页面的一部分。
        UVM_ASSERT(src_page == uvm_cpu_chunk_get_cpu_page(block, page_index));

        src_cpu_virt_addr = kmap(src_page);
        uvm_conf_computing_cpu_encrypt(push->channel,
                                       cpu_va_staging_buffer,
                                       src_cpu_virt_addr,
                                       NULL,
                                       PAGE_SIZE,
                                       cpu_auth_tag_buffer);
        kunmap(src_page);

        // 第一个 LCE 操作应该是非流水线的，以保证顺序，因为我们不知道上次非流水线复制的时间。
        // 最后一个应用最初为推送计划的 membar（如果有的话）
        // TODO: 3857691: 继承策略而不是强制第一次调用非流水线。
        if (page_index > region.first)
            uvm_push_set_flag(push, UVM_PUSH_FLAG_CE_NEXT_PIPELINED);

        if (page_index < (region.outer - 1))
            uvm_push_set_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_NONE);
        else if (membar_flag)
            uvm_push_set_flag(push, membar_flag);

        gpu->parent->ce_hal->decrypt(push, dst_address, staging_buffer, PAGE_SIZE, auth_tag_buffer);

        src_page++;
        dst_address.address += PAGE_SIZE;
        cpu_va_staging_buffer += PAGE_SIZE;
        staging_buffer.address += PAGE_SIZE;
        cpu_auth_tag_buffer += UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
        auth_tag_buffer.address += UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
    }
}
```

### uvm_va_block.c: conf_computing_block_copy_push_gpu_to_cpu

该函数

```c
gpu->parent->ce_hal->encrypt(push, staging_buffer, src_address, PAGE_SIZE, auth_tag_buffer);
```

调用了decrypt，间接调用了 `uvm_hal_hopper_ce_encrypt`

这个函数 `conf_computing_block_copy_push_gpu_to_cpu` 实现了在启用机密计算功能时，GPU 端页面加密和 CPU 端解密操作。它使用推送命令（push）在 GPU 上执行这些操作，并且 GPU 操作会遵守调用者先前在推送命令中设置的内存屏障（membar）。以下是对这段代码的详细分析：

```c
static void conf_computing_block_copy_push_gpu_to_cpu(uvm_va_block_t *block,
                                                      block_copy_state_t *copy_state,
                                                      uvm_va_block_region_t region,
                                                      uvm_push_t *push)
```

#### 参数

- **block**：表示虚拟地址块的结构体指针。
- **copy_state**：表示块复制状态的结构体指针。
- **region**：表示虚拟地址块区域的结构体。
- **push**：用于推送命令的结构体指针。

#### 初始化变量

```c
uvm_push_flag_t membar_flag = 0;
uvm_gpu_t *gpu = uvm_push_get_gpu(push);
uvm_page_index_t page_index = region.first;
uvm_conf_computing_dma_buffer_t *dma_buffer = copy_state->dma_buffer;
uvm_gpu_address_t staging_buffer = uvm_mem_gpu_address_virtual_kernel(dma_buffer->alloc, gpu);
uvm_gpu_address_t auth_tag_buffer = uvm_mem_gpu_address_virtual_kernel(dma_buffer->auth_tag, gpu);
uvm_gpu_address_t src_address = block_copy_get_address(block, &copy_state->src, page_index, gpu);
```

- **初始化 GPU 和 DMA 缓冲区相关的变量**。
- **获取源地址和认证标签的相关信息**。

#### 参数和状态验证

```c
UVM_ASSERT(UVM_ID_IS_GPU(copy_state->src.id));
UVM_ASSERT(UVM_ID_IS_CPU(copy_state->dst.id));

UVM_ASSERT(uvm_conf_computing_mode_enabled(gpu));
```

- **验证源和目标是否为 GPU 和 CPU**。
- **确保 GPU 启用了机密计算模式**。

#### 调整缓冲区地址

```c
staging_buffer.address += page_index * PAGE_SIZE;
auth_tag_buffer.address += page_index * UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
```

- **根据页面索引调整 staging_buffer 和 auth_tag_buffer 的地址**。

#### 内存屏障标志设置

```c
if (uvm_push_get_and_reset_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_NONE))
    membar_flag = UVM_PUSH_FLAG_NEXT_MEMBAR_NONE;
else if (uvm_push_get_and_reset_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_GPU))
    membar_flag = UVM_PUSH_FLAG_NEXT_MEMBAR_GPU;
```

- **根据推送命令设置内存屏障标志**。

#### 循环处理每个页面

```c
// 由于我们使用 kmap() 映射用于 CPU 端加密操作的页面，它只保证 PAGE_SIZE 的连续性，
// 所有加密和解密操作必须在 PAGE_SIZE 基础上进行。
for_each_va_block_page_in_region(page_index, region) {
    uvm_conf_computing_log_gpu_encryption(push->channel, &dma_buffer->decrypt_iv[page_index]);

    // 第一个 LCE 操作应该是非流水线的，以保证顺序，因为我们不知道上次非流水线复制的时间。
    // 最后一个应用最初为推送计划的 membar（如果有的话）
    // TODO: 3857691: 继承策略而不是强制第一次调用非流水线。
    if (page_index > region.first)
        uvm_push_set_flag(push, UVM_PUSH_FLAG_CE_NEXT_PIPELINED);

    if (page_index < (region.outer - 1))
        uvm_push_set_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_NONE);
    else if (membar_flag)
        uvm_push_set_flag(push, membar_flag);

    gpu->parent->ce_hal->encrypt(push, staging_buffer, src_address, PAGE_SIZE, auth_tag_buffer);

    src_address.address += PAGE_SIZE;
    staging_buffer.address += PAGE_SIZE;
    auth_tag_buffer.address += UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
}
```

- **循环处理区域内的每个页面**。
- **记录 GPU 加密操作**。
- **执行 GPU 端加密操作，并根据页面索引调整地址和认证标签地址**。

#### 填充已加密页面掩码

```c
uvm_page_mask_region_fill(&dma_buffer->encrypted_page_mask, region);
```

- **更新 DMA 缓冲区中的已加密页面掩码**。

#### 中文注释

```c
// 当启用机密计算功能时，该函数执行 GPU 端页面加密。GPU 操作遵守调用者在推送命令中先前设置的内存屏障。
static void conf_computing_block_copy_push_gpu_to_cpu(uvm_va_block_t *block,
                                                      block_copy_state_t *copy_state,
                                                      uvm_va_block_region_t region,
                                                      uvm_push_t *push)
{
    uvm_push_flag_t membar_flag = 0;
    uvm_gpu_t *gpu = uvm_push_get_gpu(push);
    uvm_page_index_t page_index = region.first;
    uvm_conf_computing_dma_buffer_t *dma_buffer = copy_state->dma_buffer;
    uvm_gpu_address_t staging_buffer = uvm_mem_gpu_address_virtual_kernel(dma_buffer->alloc, gpu);
    uvm_gpu_address_t auth_tag_buffer = uvm_mem_gpu_address_virtual_kernel(dma_buffer->auth_tag, gpu);
    uvm_gpu_address_t src_address = block_copy_get_address(block, &copy_state->src, page_index, gpu);

    UVM_ASSERT(UVM_ID_IS_GPU(copy_state->src.id));
    UVM_ASSERT(UVM_ID_IS_CPU(copy_state->dst.id));

    UVM_ASSERT(uvm_conf_computing_mode_enabled(gpu));

    staging_buffer.address += page_index * PAGE_SIZE;
    auth_tag_buffer.address += page_index * UVM_CONF_COMPUTING_AUTH_TAG_SIZE;

    if (uvm_push_get_and_reset_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_NONE))
        membar_flag = UVM_PUSH_FLAG_NEXT_MEMBAR_NONE;
    else if (uvm_push_get_and_reset_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_GPU))
        membar_flag = UVM_PUSH_FLAG_NEXT_MEMBAR_GPU;

    // 由于我们使用 kmap() 映射用于 CPU 端加密操作的页面，它只保证 PAGE_SIZE 的连续性，
    // 所有加密和解密操作必须在 PAGE_SIZE 基础上进行。
    for_each_va_block_page_in_region(page_index, region) {
        uvm_conf_computing_log_gpu_encryption(push->channel, &dma_buffer->decrypt_iv[page_index]);

        // 第一个 LCE 操作应该是非流水线的，以保证顺序，因为我们不知道上次非流水线复制的时间。
        // 最后一个应用最初为推送计划的 membar（如果有的话）
        // TODO: 3857691: 继承策略而不是强制第一次调用非流水线。
        if (page_index > region.first)
            uvm_push_set_flag(push, UVM_PUSH_FLAG_CE_NEXT_PIPELINED);

        if (page_index < (region.outer - 1))
            uvm_push_set_flag(push, UVM_PUSH_FLAG_NEXT_MEMBAR_NONE);
        else if (membar_flag)
            uvm_push_set_flag(push, membar_flag);

        gpu->parent->ce_hal->encrypt(push, staging_buffer, src_address, PAGE_SIZE, auth_tag_buffer);

        src_address.address += PAGE_SIZE;
        staging_buffer.address += PAGE_SIZE;
        auth_tag_buffer.address += UVM_CONF_COMPUTING_AUTH_TAG_SIZE;
    }

    uvm_page_mask_region_fill(&dma_buffer->encrypted_page_mask, region);
}
```

#### 总结

函数 `conf_computing_block_copy_push_gpu_to_cpu` 实现了在启用机密计算功能时，GPU 端页面加密的过程。通过初始化变量和验证参数，函数确保 GPU 和 CPU 间的数据传输和加密操作的正确性。通过处理区域内的每个页面，函数逐页执行加密操作，并更新相关的地址和认证标签地址。在完成所有页面的处理后，函数更新 DMA 缓冲区中的已加密页面掩码。

### uvm_va_block.c: conf_computing_copy_pages_finish

这个函数 `conf_computing_copy_pages_finish` 负责完成机密计算中的页面复制操作。它在推送命令完成后，处理从GPU到CPU的页面数据传输，并在CPU端解密这些数据。以下是对这段代码的详细分析：

```c
static NV_STATUS conf_computing_copy_pages_finish(uvm_va_block_t *block,
                                                  block_copy_state_t *copy_state,
                                                  uvm_push_t *push)
```

#### 参数

- **block**：表示虚拟地址块的结构体指针。
- **copy_state**：表示块复制状态的结构体指针。
- **push**：用于推送命令的结构体指针。

#### 返回值

- **NV_STATUS**：表示操作的状态，可能的值包括 `NV_OK`（成功）或其他错误代码。

#### 初始化变量

```c
NV_STATUS status;
uvm_page_index_t page_index;
uvm_conf_computing_dma_buffer_t *dma_buffer = copy_state->dma_buffer;
uvm_page_mask_t *encrypted_page_mask = &dma_buffer->encrypted_page_mask;
void *auth_tag_buffer_base = uvm_mem_get_cpu_addr_kernel(dma_buffer->auth_tag);
void *staging_buffer_base = uvm_mem_get_cpu_addr_kernel(dma_buffer->alloc);
```

- **初始化DMA缓冲区和相关内存地址**。
- **获取加密页面掩码、认证标签缓冲区基地址和暂存缓冲区基地址**。

#### 参数和状态验证

```c
UVM_ASSERT(uvm_channel_is_secure(push->channel));

if (UVM_ID_IS_GPU(copy_state->dst.id))
    return NV_OK;

UVM_ASSERT(UVM_ID_IS_GPU(copy_state->src.id));
```

- **确保推送命令的通道是安全的**。
- **如果目标是GPU，直接返回成功状态**。
- **确保源是GPU**。

#### 等待推送命令完成

```c
status = uvm_push_wait(push);
if (status != NV_OK)
    return status;
```

- **等待推送命令完成**。
- **如果等待失败，返回错误状态**。

#### 处理每个页面

```c
for_each_va_block_page_in_mask(page_index, encrypted_page_mask, block) {
    struct page *dst_page = uvm_cpu_chunk_get_cpu_page(block, page_index);
    void *staging_buffer = (char *)staging_buffer_base + (page_index * PAGE_SIZE);
    void *auth_tag_buffer = (char *)auth_tag_buffer_base + (page_index * UVM_CONF_COMPUTING_AUTH_TAG_SIZE);
    void *cpu_page_address = kmap(dst_page);

    status = uvm_conf_computing_cpu_decrypt(push->channel,
                                            cpu_page_address,
                                            staging_buffer,
                                            &dma_buffer->decrypt_iv[page_index],
                                            PAGE_SIZE,
                                            auth_tag_buffer);
    kunmap(dst_page);
    if (status != NV_OK) {
        // TODO: Bug 3814087: [UVM][HCC] 处理CSL认证标签验证失败和其他失败情况。
        // uvm_conf_computing_cpu_decrypt() 可能因为认证标签验证失败而失败。
        // 如果发生这种情况，认为是严重故障，无法恢复。
        uvm_global_set_fatal_error(status);
        return status;
    }
}
```

- **遍历加密页面掩码中的每个页面**。
- **获取目标页面和相关缓冲区地址**。
- **将页面映射到CPU地址空间**。
- **调用 `uvm_conf_computing_cpu_decrypt` 函数在CPU端解密页面数据**。
- **解除页面映射**。
- **如果解密失败，记录严重错误并返回错误状态**。

#### 中文注释

```c
// 当启用机密计算功能时，该函数完成 GPU 到 CPU 的页面复制操作。
// GPU 操作遵守调用者在推送命令中先前设置的内存屏障。
static NV_STATUS conf_computing_copy_pages_finish(uvm_va_block_t *block,
                                                  block_copy_state_t *copy_state,
                                                  uvm_push_t *push)
{
    NV_STATUS status;
    uvm_page_index_t page_index;
    uvm_conf_computing_dma_buffer_t *dma_buffer = copy_state->dma_buffer;
    uvm_page_mask_t *encrypted_page_mask = &dma_buffer->encrypted_page_mask;
    void *auth_tag_buffer_base = uvm_mem_get_cpu_addr_kernel(dma_buffer->auth_tag);
    void *staging_buffer_base = uvm_mem_get_cpu_addr_kernel(dma_buffer->alloc);

    UVM_ASSERT(uvm_channel_is_secure(push->channel));

    if (UVM_ID_IS_GPU(copy_state->dst.id))
        return NV_OK;

    UVM_ASSERT(UVM_ID_IS_GPU(copy_state->src.id));

    status = uvm_push_wait(push);
    if (status != NV_OK)
        return status;

    // kmap() 只保证 PAGE_SIZE 的连续性，所有加密和解密操作必须在 PAGE_SIZE 的基础上进行。
    for_each_va_block_page_in_mask(page_index, encrypted_page_mask, block) {
        struct page *dst_page = uvm_cpu_chunk_get_cpu_page(block, page_index);
        void *staging_buffer = (char *)staging_buffer_base + (page_index * PAGE_SIZE);
        void *auth_tag_buffer = (char *)auth_tag_buffer_base + (page_index * UVM_CONF_COMPUTING_AUTH_TAG_SIZE);
        void *cpu_page_address = kmap(dst_page);

        status = uvm_conf_computing_cpu_decrypt(push->channel,
                                                cpu_page_address,
                                                staging_buffer,
                                                &dma_buffer->decrypt_iv[page_index],
                                                PAGE_SIZE,
                                                auth_tag_buffer);
        kunmap(dst_page);
        if (status != NV_OK) {
            // TODO: Bug 3814087: [UVM][HCC] 处理 CSL 认证标签验证失败和其他失败情况。
            // uvm_conf_computing_cpu_decrypt() 可能因为认证标签验证失败而失败。
            // 如果发生这种情况，认为是严重故障，无法恢复。
            uvm_global_set_fatal_error(status);
            return status;
        }
    }

    return NV_OK;
}
```

#### 总结

函数 `conf_computing_copy_pages_finish` 负责完成 GPU 到 CPU 的页面复制操作，并在 CPU 端解密这些数据。通过初始化变量和验证参数，函数确保数据传输和加密操作的正确性。遍历加密页面掩码中的每个页面，逐页解密数据，并处理可能出现的错误。在完成所有页面的处理后，函数返回成功状态。

### encrypted_memcopy_cpu_to_gpu

这个函数 `encrypted_memcopy_gpu_to_cpu` 实现了在 GPU 和 CPU 之间进行同步加密复制的操作。它通过 GPU 端的加密（使用复制引擎）和 CPU 端的解密，将 GPU 上的数据传输到 CPU，最终在目标 CPU 缓冲区中得到解密后的明文数据。以下是对这段代码的详细分析：

```c
__attribute__ ((format(printf, 6, 7)))
static NV_STATUS encrypted_memcopy_gpu_to_cpu(uvm_gpu_t *gpu,
                                              void *dst_plain,
                                              uvm_gpu_address_t src_gpu_address,
                                              size_t size,
                                              uvm_tracker_t *tracker,
                                              const char *format,
                                              ...)
```

#### 参数

- **gpu**：指向 GPU 结构体的指针。
- **dst_plain**：目标 CPU 缓冲区的指针，用于存储解密后的明文数据。
- **src_gpu_address**：源 GPU 地址，包含需要加密的数据。
- **size**：需要复制的数据大小。
- **tracker**：用于跟踪操作的追踪器，可以为 NULL。
- **format**：格式化字符串，用于日志记录。
- **...**：可变参数列表，与格式化字符串对应。

#### 返回值

- **NV_STATUS**：表示操作的状态，可能的值包括 `NV_OK`（成功）或其他错误代码。

#### 初始化变量和检查条件

```c
NV_STATUS status;
UvmCslIv decrypt_iv;
uvm_push_t push;
uvm_conf_computing_dma_buffer_t *dma_buffer;
uvm_gpu_address_t dst_gpu_address, auth_tag_gpu_address;
void *src_cipher, *auth_tag;
va_list args;

UVM_ASSERT(uvm_conf_computing_mode_enabled(gpu));
UVM_ASSERT(size <= UVM_CONF_COMPUTING_DMA_BUFFER_SIZE);
```

- **初始化状态变量和加密相关变量**。
- **检查 GPU 是否启用了机密计算模式**。
- **确保要复制的数据大小不超过最大限制 `UVM_CONF_COMPUTING_DMA_BUFFER_SIZE`**。

#### 分配 DMA 缓冲区

```c
status = uvm_conf_computing_dma_buffer_alloc(&gpu->conf_computing.dma_buffer_pool, &dma_buffer, NULL);
if (status != NV_OK)
    return status;
```

- **从 GPU 的 DMA 缓冲池中分配一个 DMA 缓冲区**。
- **如果分配失败，返回错误状态**。

#### 开始推送命令并记录日志

```c
va_start(args, format);
status = uvm_push_begin_acquire(gpu->channel_manager, UVM_CHANNEL_TYPE_GPU_TO_CPU, tracker, &push, format, args);
va_end(args);

if (status != NV_OK)
    goto out;

uvm_conf_computing_log_gpu_encryption(push.channel, &decrypt_iv);
```

- **使用可变参数列表初始化推送命令**。
- **如果推送命令初始化失败，跳转到错误处理部分**。
- **记录 GPU 加密操作的日志**。

#### 设置地址并执行加密

```c
dst_gpu_address = uvm_mem_gpu_address_virtual_kernel(dma_buffer->alloc, gpu);
auth_tag_gpu_address = uvm_mem_gpu_address_virtual_kernel(dma_buffer->auth_tag, gpu);
gpu->parent->ce_hal->encrypt(&push, dst_gpu_address, src_gpu_address, size, auth_tag_gpu_address);

status = uvm_push_end_and_wait(&push);
if (status != NV_OK)
    goto out;
```

- **设置目标 GPU 地址和认证标签地址**。
- **使用复制引擎在 GPU 上执行加密操作**。
- **等待推送命令完成**。
- **如果等待失败，跳转到错误处理部分**。

#### 获取加密数据并在 CPU 上解密

```c
src_cipher = uvm_mem_get_cpu_addr_kernel(dma_buffer->alloc);
auth_tag = uvm_mem_get_cpu_addr_kernel(dma_buffer->auth_tag);
status = uvm_conf_computing_cpu_decrypt(push.channel, dst_plain, src_cipher, &decrypt_iv, size, auth_tag);
```

- **获取加密数据和认证标签的 CPU 地址**。
- **在 CPU 上执行解密操作**。
- **如果解密失败，跳转到错误处理部分**。

#### 错误处理和释放资源

```c
out:
uvm_conf_computing_dma_buffer_free(&gpu->conf_computing.dma_buffer_pool, dma_buffer, NULL);
return status;
```

- **释放分配的 DMA 缓冲区**。
- **返回操作状态**。

#### 中文注释

```c
// 启动 GPU 和 CPU 之间的同步加密复制操作。
//
// 该复制操作包括 GPU 端加密（依赖于复制引擎）和 CPU 端解密步骤，
// 使得目标 CPU 缓冲区（由 dst_plain 指向）将包含未加密的（明文）内容。
// 目标缓冲区可以在受保护或未受保护的系统内存中，而源缓冲区必须在受保护的显存中。
//
// 允许的最大复制大小是 UVM_CONF_COMPUTING_DMA_BUFFER_SIZE。
//
// 如果输入追踪器不为 NULL，则由负责加密复制的推送命令内部获取。
__attribute__ ((format(printf, 6, 7)))
static NV_STATUS encrypted_memcopy_gpu_to_cpu(uvm_gpu_t *gpu,
                                              void *dst_plain,
                                              uvm_gpu_address_t src_gpu_address,
                                              size_t size,
                                              uvm_tracker_t *tracker,
                                              const char *format,
                                              ...)
{
    NV_STATUS status;
    UvmCslIv decrypt_iv;
    uvm_push_t push;
    uvm_conf_computing_dma_buffer_t *dma_buffer;
    uvm_gpu_address_t dst_gpu_address, auth_tag_gpu_address;
    void *src_cipher, *auth_tag;
    va_list args;

    UVM_ASSERT(uvm_conf_computing_mode_enabled(gpu));
    UVM_ASSERT(size <= UVM_CONF_COMPUTING_DMA_BUFFER_SIZE);

    status = uvm_conf_computing_dma_buffer_alloc(&gpu->conf_computing.dma_buffer_pool, &dma_buffer, NULL);
    if (status != NV_OK)
        return status;

    va_start(args, format);
    status = uvm_push_begin_acquire(gpu->channel_manager, UVM_CHANNEL_TYPE_GPU_TO_CPU, tracker, &push, format, args);
    va_end(args);

    if (status != NV_OK)
        goto out;

    uvm_conf_computing_log_gpu_encryption(push.channel, &decrypt_iv);

    dst_gpu_address = uvm_mem_gpu_address_virtual_kernel(dma_buffer->alloc, gpu);
    auth_tag_gpu_address = uvm_mem_gpu_address_virtual_kernel(dma_buffer->auth_tag, gpu);
    gpu->parent->ce_hal->encrypt(&push, dst_gpu_address, src_gpu_address, size, auth_tag_gpu_address);

    status = uvm_push_end_and_wait(&push);
    if (status != NV_OK)
        goto out;

    src_cipher = uvm_mem_get_cpu_addr_kernel(dma_buffer->alloc);
    auth_tag = uvm_mem_get_cpu_addr_kernel(dma_buffer->auth_tag);
    status = uvm_conf_computing_cpu_decrypt(push.channel, dst_plain, src_cipher, &decrypt_iv, size, auth_tag);

 out:
    uvm_conf_computing_dma_buffer_free(&gpu->conf_computing.dma_buffer_pool, dma_buffer, NULL);
    return status;
}
```

#### 总结

函数 `encrypted_memcopy_gpu_to_cpu` 实现了在 GPU 和 CPU 之间的同步加密复制操作。通过 GPU 端的加密和 CPU 端的解密，该函数确保数据在传输过程中保持机密性和完整性。通过初始化变量、分配 DMA 缓冲区、推送和等待命令、获取和解密数据，函数完成了从 GPU 到 CPU 的数据复制操作。错误处理和资源释放确保了在任何步骤失败时都能正确清理资源并返回相应的错误状态。

### encrypted_memcopy_cpu_to_gpu

这个函数 `encrypted_memcopy_cpu_to_gpu` 实现了在 CPU 和 GPU 之间进行同步加密复制的操作。它通过 CPU 端的加密和 GPU 端的解密，将 CPU 上的明文数据传输到 GPU，最终在目标 GPU 缓冲区中得到解密后的密文数据。以下是对这段代码的详细分析：

```c
__attribute__ ((format(printf, 6, 7)))
static NV_STATUS encrypted_memcopy_cpu_to_gpu(uvm_gpu_t *gpu,
                                              uvm_gpu_address_t dst_gpu_address,
                                              void *src_plain,
                                              size_t size,
                                              uvm_tracker_t *tracker,
                                              const char *format,
                                              ...)
```

#### 参数

- **gpu**：指向 GPU 结构体的指针。
- **dst_gpu_address**：目标 GPU 地址，存储解密后的密文数据。
- **src_plain**：源 CPU 缓冲区的指针，包含未加密的明文数据。
- **size**：需要复制的数据大小。
- **tracker**：用于跟踪操作的追踪器，可以为 NULL。
- **format**：格式化字符串，用于日志记录。
- **...**：可变参数列表，与格式化字符串对应。

#### 返回值

- **NV_STATUS**：表示操作的状态，可能的值包括 `NV_OK`（成功）或其他错误代码。

#### 初始化变量和检查条件

```c
NV_STATUS status;
uvm_push_t push;
uvm_conf_computing_dma_buffer_t *dma_buffer;
uvm_gpu_address_t src_gpu_address, auth_tag_gpu_address;
void *dst_cipher, *auth_tag;
va_list args;

UVM_ASSERT(uvm_conf_computing_mode_enabled(gpu));
UVM_ASSERT(size <= UVM_CONF_COMPUTING_DMA_BUFFER_SIZE);
```

- **初始化状态变量和加密相关变量**。
- **检查 GPU 是否启用了机密计算模式**。
- **确保要复制的数据大小不超过最大限制 `UVM_CONF_COMPUTING_DMA_BUFFER_SIZE`**。

#### 分配 DMA 缓冲区

```c
status = uvm_conf_computing_dma_buffer_alloc(&gpu->conf_computing.dma_buffer_pool, &dma_buffer, NULL);
if (status != NV_OK)
    return status;
```

- **从 GPU 的 DMA 缓冲池中分配一个 DMA 缓冲区**。
- **如果分配失败，返回错误状态**。

#### 开始推送命令并记录日志

```c
va_start(args, format);
status = uvm_push_begin_acquire(gpu->channel_manager, UVM_CHANNEL_TYPE_CPU_TO_GPU, tracker, &push, format, args);
va_end(args);

if (status != NV_OK)
    goto out;
```

- **使用可变参数列表初始化推送命令**。
- **如果推送命令初始化失败，跳转到错误处理部分**。

#### CPU 端加密操作

```c
dst_cipher = uvm_mem_get_cpu_addr_kernel(dma_buffer->alloc);
auth_tag = uvm_mem_get_cpu_addr_kernel(dma_buffer->auth_tag);
uvm_conf_computing_cpu_encrypt(push.channel, dst_cipher, src_plain, NULL, size, auth_tag);
```

- **获取加密数据和认证标签的 CPU 地址**。
- **在 CPU 上执行加密操作**。

#### 设置地址并执行解密

```c
src_gpu_address = uvm_mem_gpu_address_virtual_kernel(dma_buffer->alloc, gpu);
auth_tag_gpu_address = uvm_mem_gpu_address_virtual_kernel(dma_buffer->auth_tag, gpu);
gpu->parent->ce_hal->decrypt(&push, dst_gpu_address, src_gpu_address, size, auth_tag_gpu_address);

status = uvm_push_end_and_wait(&push);
```

- **设置源 GPU 地址和认证标签地址**。
- **使用复制引擎在 GPU 上执行解密操作**。
- **等待推送命令完成**。

#### 错误处理和释放资源

```c
out:
uvm_conf_computing_dma_buffer_free(&gpu->conf_computing.dma_buffer_pool, dma_buffer, NULL);
return status;
```

- **释放分配的 DMA 缓冲区**。
- **返回操作状态**。

#### 中文注释

```c
// 启动 CPU 和 GPU 之间的同步加密复制操作。
//
// 源 CPU 缓冲区（由 src_plain 指向）包含未加密的明文内容；
// 该函数在内部执行 CPU 端加密步骤，然后启动 GPU 端 CE 解密。
// 源缓冲区可以在受保护或未受保护的系统内存中，而目标缓冲区必须在受保护的显存中。
//
// 允许的最大复制大小是 UVM_CONF_COMPUTING_DMA_BUFFER_SIZE。
//
// 如果输入追踪器不为 NULL，则由负责加密复制的推送命令内部获取。
__attribute__ ((format(printf, 6, 7)))
static NV_STATUS encrypted_memcopy_cpu_to_gpu(uvm_gpu_t *gpu,
                                              uvm_gpu_address_t dst_gpu_address,
                                              void *src_plain,
                                              size_t size,
                                              uvm_tracker_t *tracker,
                                              const char *format,
                                              ...)
{
    NV_STATUS status;
    uvm_push_t push;
    uvm_conf_computing_dma_buffer_t *dma_buffer;
    uvm_gpu_address_t src_gpu_address, auth_tag_gpu_address;
    void *dst_cipher, *auth_tag;
    va_list args;

    // 确认 GPU 启用了机密计算模式
    UVM_ASSERT(uvm_conf_computing_mode_enabled(gpu));
    // 确认数据大小不超过最大限制
    UVM_ASSERT(size <= UVM_CONF_COMPUTING_DMA_BUFFER_SIZE);

    // 分配 DMA 缓冲区
    status = uvm_conf_computing_dma_buffer_alloc(&gpu->conf_computing.dma_buffer_pool, &dma_buffer, NULL);
    if (status != NV_OK)
        return status;

    // 初始化推送命令
    va_start(args, format);
    status = uvm_push_begin_acquire(gpu->channel_manager, UVM_CHANNEL_TYPE_CPU_TO_GPU, tracker, &push, format, args);
    va_end(args);

    if (status != NV_OK)
        goto out;

    // 获取加密数据和认证标签的 CPU 地址
    dst_cipher = uvm_mem_get_cpu_addr_kernel(dma_buffer->alloc);
    auth_tag = uvm_mem_get_cpu_addr_kernel(dma_buffer->auth_tag);
    // 在 CPU 上执行加密操作
    uvm_conf_computing_cpu_encrypt(push.channel, dst_cipher, src_plain, NULL, size, auth_tag);

    // 设置源 GPU 地址和认证标签地址
    src_gpu_address = uvm_mem_gpu_address_virtual_kernel(dma_buffer->alloc, gpu);
    auth_tag_gpu_address = uvm_mem_gpu_address_virtual_kernel(dma_buffer->auth_tag, gpu);
    // 在 GPU 上执行解密操作
    gpu->parent->ce_hal->decrypt(&push, dst_gpu_address, src_gpu_address, size, auth_tag_gpu_address);

    // 等待推送命令完成
    status = uvm_push_end_and_wait(&push);

out:
    // 释放 DMA 缓冲区
    uvm_conf_computing_dma_buffer_free(&gpu->conf_computing.dma_buffer_pool, dma_buffer, NULL);
    return status;
}
```

#### 总结

函数 `encrypted_memcopy_cpu_to_gpu` 实现了在 CPU 和 GPU 之间的同步加密复制操作。通过 CPU 端的加密和 GPU 端的解密，该函数确保数据在传输过程中保持机密性和完整性。通过初始化变量、分配 DMA 缓冲区、推送和等待命令、获取和加密数据，函数完成了从 CPU 到 GPU 的数据复制操作。错误处理和资源释放确保了在任何步骤失败时都能正确清理资源并返回相应的错误状态。
