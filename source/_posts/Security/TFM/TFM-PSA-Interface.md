---
title: TF-M PSA Interface
date: 2021-05-29 11:48:09
tags:
    - TF-M
    - Firmware
    - TEE
    - PSA
    - Arm
    - Security
categories:
    - Security
---

## PSA Interface
PSA接口完成从NS的client到secure服务的调用。

<!-- more --> 

<details>
<summary>server.c</summary>  
```c
//in psa_client.c
__attribute__((naked))
uint32_t psa_framework_version(void)
{
    __ASM volatile("SVC %0           \n"
                   "BX LR            \n"
                   : : "I" (TFM_SVC_PSA_FRAMEWORK_VERSION));
}

//in tfm_core_svcalls_ipc.c
static int32_t SVC_Handler_IPC(tfm_svc_number_t svc_num, uint32_t *ctx,
                               uint32_t lr)
{
    bool ns_caller = false;
    struct partition_t *partition = NULL;
    uint32_t veneer_base =
        (uint32_t)&REGION_NAME(Image$$, TFM_UNPRIV_CODE, $$RO$$Base);
    uint32_t veneer_limit =
        (uint32_t)&REGION_NAME(Image$$, TFM_UNPRIV_CODE, $$RO$$Limit);

    /*
     * The caller security attribute detection bases on LR of state context.
     * However, if SP calls PSA APIs based on its customized SVC, the LR may be
     * occupied by general purpose value while calling SVC.
     * Check if caller comes from non-secure: return address (ctx[6]) is belongs
     * to veneer section, and the bit0 of LR (ctx[5]) is zero.
     */
    if (ctx[6] = veneer_base && ctx[6] < veneer_limit &&
        !(ctx[5] & TFM_VENEER_LR_BIT0_MASK)) {
        ns_caller = true;
    }

    partition = tfm_spm_get_running_partition();
    if (!partition) {
        tfm_core_panic();
    }

    tfm_spm_validate_caller(partition, ctx, lr, ns_caller);

    switch (svc_num) {
    case TFM_SVC_PSA_FRAMEWORK_VERSION:
        return tfm_spm_psa_framework_version();
    case TFM_SVC_PSA_VERSION:
        return tfm_spm_psa_version(ctx, ns_caller);
    case TFM_SVC_PSA_CONNECT:
        return tfm_spm_psa_connect(ctx, ns_caller);
    case TFM_SVC_PSA_CALL:
        return tfm_spm_psa_call(ctx, ns_caller, lr);
    case TFM_SVC_PSA_CLOSE:
        tfm_spm_psa_close(ctx, ns_caller);
        break;
    case TFM_SVC_PSA_WAIT:
        return tfm_spm_psa_wait(ctx);
    case TFM_SVC_PSA_GET:
        return tfm_spm_psa_get(ctx);
    case TFM_SVC_PSA_SET_RHANDLE:
        tfm_spm_psa_set_rhandle(ctx);
        break;
    case TFM_SVC_PSA_READ:
        return tfm_spm_psa_read(ctx);
    case TFM_SVC_PSA_SKIP:
        return tfm_spm_psa_skip(ctx);
    case TFM_SVC_PSA_WRITE:
        tfm_spm_psa_write(ctx);
        break;
    case TFM_SVC_PSA_REPLY:
        tfm_spm_psa_reply(ctx);
        break;
    case TFM_SVC_PSA_NOTIFY:
        tfm_spm_psa_notify(ctx);
        break;
    case TFM_SVC_PSA_CLEAR:
        tfm_spm_psa_clear();
        break;
    case TFM_SVC_PSA_EOI:
        tfm_spm_psa_eoi(ctx);
        break;
    case TFM_SVC_PSA_PANIC:
        tfm_spm_psa_panic();
        break;
    case TFM_SVC_SPM_REQUEST:
        tfm_spm_request_handler((const struct tfm_state_context_t *)ctx);
        break;
    case TFM_SVC_PSA_LIFECYCLE:
        return tfm_spm_get_lifecycle_state();
#if (TFM_SPM_LOG_LEVEL TFM_SPM_LOG_LEVEL_SILENCE)
    case TFM_SVC_OUTPUT_UNPRIV_STRING:
        return tfm_hal_output_spm_log((const char *)ctx[0], ctx[1]);
#endif
    case TFM_SVC_PSA_IRQ_ENABLE:
        tfm_spm_irq_enable(ctx);
        break;
    case TFM_SVC_PSA_IRQ_DISABLE:
        return tfm_spm_irq_disable(ctx);
    default:
#ifdef PLATFORM_SVC_HANDLERS
        return (platform_svc_handlers(svc_num, ctx, lr));
#else
        ERROR_MSG("Unknown SVC number requested!");
        return PSA_ERROR_GENERIC_ERROR;
#endif
    }
    return PSA_SUCCESS;
}

uint32_t tfm_core_svc_handler(uint32_t *msp, uint32_t *psp, uint32_t exc_return)
{
    tfm_svc_number_t svc_number = TFM_SVC_PSA_FRAMEWORK_VERSION;
    uint32_t *svc_args = msp;

    if (!(exc_return & EXC_RETURN_MODE)) {
        /* Calling SVC from Handler Mode is not supported */
        tfm_core_panic();
    }

    if ((exc_return & EXC_RETURN_MODE) && (exc_return & EXC_RETURN_SPSEL)) {
        /* Use PSP when both EXC_RETURN.MODE and EXC_RETURN.SPSEL are set */
        svc_args = psp;
    } else {
        svc_args = msp;
    }

    /*
     * Stack contains:
     * r0, r1, r2, r3, r12, r14 (lr), the return address and xPSR
     * First argument (r0) is svc_args[0]
     */
    if (is_return_secure_stack(exc_return)) {
        /* SV called directly from secure context. Check instruction for
         * svc_number
         */
        svc_number = ((tfm_svc_number_t *)svc_args[6])[-2];
    } else {
        /* Secure SV executing with NS return.
         * NS cannot directly trigger S SVC so this should not happen. This is
         * an unrecoverable error.
         */
        tfm_core_panic();
    }
    switch (svc_number) {
    case TFM_SVC_HANDLER_MODE:
        tfm_arch_clear_fp_status();
        exc_return = tfm_spm_init();
        break;
    case TFM_SVC_GET_BOOT_DATA:
        tfm_core_get_boot_data_handler(svc_args);
        break;
    default:
        svc_args[0] = SVC_Handler_IPC(svc_number, svc_args, exc_return);
        break;
    }

    return exc_return;
}
```

</details> 

代码中有一些重点：
* `__attribute__((naked))`

  This attribute tells the compiler that the function is an embedded assembly function. You can write the body of the function entirely in assembly code using `__asm` statements.
  
  The compiler does not generate prologue and epilogue sequences for functions with `__attribute__((naked))`.
  
  The compiler only supports basic `__asm` statements in `__attribute__((naked))` functions. Using extended assembly, parameter references or mixing C code with `__asm` statements might not work reliably.

* `__ASM`

__asm 关键字用于调用内联汇编程序，并且可在 C 或 C++ 语句合法时出现

* `volatile`

`volatile` tells the compiler not to optimize anything that has to do with the `volatile` variable.

不只是内嵌汇编操纵栈"这种方式属于编译无法识别的变量改变，另外更多的可能是多线程并发访问共享变量时，一个线程改变了变量的值，怎样让改变后的值对其它线程 visible。一般说来，volatile用在如下的几个地方：

* 中断服务程序中修改的供其它程序检测的变量需要加 volatile；
* 多任务环境下各任务间共享的标志应该加 volatile；
* 存储器映射的硬件寄存器通常也要加 volatile 说明，因为每次对它的读写都可能由不同意义；

* `SVC`

ARM有两种模式，handler处理器模式和thread线程模式，前者能够访问更多的资源并始终处于特权模式下，SVC实现thread模式向handler模式的请求，请求通过imm立即数跳转。参考：[ cortex-M3 的SVC、PendSV异常，与操作系统(ucos实时系统)_@角色扮演#-CSDN博客](https://blog.csdn.net/guozhongwei1/article/details/49544671?utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-3.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromMachineLearnPai2~default-3.control)

In ARMv8-M document:
![](/img/TF-M/tfm_if.png)

## Supervisor calls

看一下ARMV8-ARM中对SVC的解释：

The `SVC` instruction generates an SVC. A typical use for SVCs is to request privileged operations or access to system resources from an operating system.

The `SVC` instruction has a number embedded within it, often referred to as the SVC number. On most ARM processors, the SVC number indicates the service that is being requested. On microcontroller profiles, the processor saves the argument registers to the stack on the initial exception entry.

A late-arriving exception, taken before the first instruction of the SVC handler executes, might corrupt the copy of the arguments still held in R0 to R3. This means that the stack copy of the arguments must be used by the SVC handler. Any return value must also be passed back to the caller by modifying the stacked register values. In order to do this, a short piece of assembly code must be implemented at the start of the SVC handler. This identifies where the registers are saved, extracts the SVC number from the instruction, and passes the number, and a pointer to the arguments, to the main body of the handler written in C.

The following example shows an example SVC handler. This code tests the EXC_RETURN value set by the processor to determine which stack pointer was in use when the `SVC` was called. This can be useful for reentrant SVCs, but is unnecessary on most systems because in a typical system design, SVCs are only called from user code that uses the process stack. In such cases, the assembly code can consist of a single `MSR` instruction followed by a tail calling branch (`B` instruction) to the C body of the handler.

## Example SVC Handler

```c
__asm void SVCHandler(void)
{
    IMPORT SVCHandler_main
    TST lr, #4
    ITE EQ
    MRSEQ R0, MSP
    MRSNE R0, PSP
    B SVCHandler_main
}
void SVCHandler_main(unsigned int * svc_args)
{
    unsigned int svc_number;
    /*
    * Stack contains:
    * R0, R1, R2, R3, R12, R14, the return address and xPSR
    * First argument (R0) is svc_args[0]
    */
    svc_number = ((char *)svc_args[6])[-2];
    switch(svc_number)
    {
        case SVC_00:
            /* Handle SVC 00 */
            break;
        case SVC_01:
            /* Handle SVC 01 */
            break;
        default:
            /* Unknown SVC */
            break;
    }
}
```

The following example shows how you can make different declarations for a number of SVCs. `__svc` is a compiler keyword that replaces a function call with an `SVC` instruction containing the specified number.

## Example of calling an SVC from C code

```c
#define SVC_00 0x00
#define SVC_01 0x01
void __svc(SVC_00) svc_zero(const char *string);
void __svc(SVC_01) svc_one(const char *string);
int call_system_func(void)
{
    svc_zero("String to pass to SVC handler zero");
    svc_one("String to pass to a different OS function");
}
```
