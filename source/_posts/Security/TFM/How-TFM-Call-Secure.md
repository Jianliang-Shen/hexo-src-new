---
title: TF-M是如何调用secure接口的
date: 2021-05-29 19:18:06
tags:
    - TF-M
    - 固件
    - Arm
categories:
    - 安全
---

回到问题之初，TF-M是怎么进入secure端的，在LIB model的时候，曾有过一个SG命令，看看其他模式怎么进入的。

<!-- more -->

## SG


Secure Gateway. Secure Gateway marks a valid branch target for branches from Non-secure code that call Secure code. This instruction sets the Security state to Secure if its address is in Secure memory. If the address of this instruction is in Non-secure memory the instruction behaves as a NOP. If the PACTBTI Extension is implemented, this instruction is always a valid BTI landing pad regardless of whether or not the instruction behaves as a NOP.

## 函数属性

定义为：

```c
#define __tz_naked_veneer \
        __attribute__((cmse_nonsecure_entry, noclone, naked, section("SFN")))
```

这里的`cmse_nonsecure_entry`可以参考[secure gateway](https://developer.arm.com/documentation/100067/0612/Compiler-specific-Function--Variable--and-Type-Attributes/--attribute----cmse-nonsecure-entry---function-attribute)

## Call routine

interface/src/tfm_psa_ns_api.c: psa_call --> tfm_ns_interface_dispatch

这个函数回调`tfm_psa_connect_veneer`进入secure，然后进入`spm_interface_cross_dispatcher`

```c
__tz_naked_veneer
psa_status_t tfm_psa_call_veneer(psa_handle_t handle,
                                 uint32_t ctrl_param,
                                 const psa_invec *in_vec,
                                 psa_outvec *out_vec)
{
    __ASM volatile(
#if !defined(__ICCARM__)
        ".syntax unified                                      \n"
#endif

        "   push   {r2, r3}                                   \n"
        "   ldr    r2, [sp, #8]                               \n"
        "   ldr    r3, ="M2S(STACK_SEAL_PATTERN)"             \n"
        "   cmp    r2, r3                                     \n"
        "   bne    reent_panic4                               \n"
        "   pop    {r2, r3}                                   \n"
        "   push   {r4, lr}                                   \n"
#if CONFIG_TFM_PSA_API_CROSS_CALL == 1
        "   push   {r0-r3}                                    \n"
        "   ldr    r0, =tfm_spm_client_psa_call               \n"
        "   mov    r1, sp                                     \n"
        "   bl     spm_interface_cross_dispatcher             \n"
        "   pop    {r0-r3}                                    \n"
#elif CONFIG_TFM_PSA_API_SFN_CALL == 1
        "   bl     psa_call_pack_sfn                          \n"
#else
        "   svc    "M2S(TFM_SVC_PSA_CALL)"                    \n"
#endif
        "   bl     clear_caller_context                       \n"
        "   pop    {r1, r2}                                   \n"
        "   mov    lr, r2                                     \n"
        "   mov    r4, r1                                     \n"
        "   bxns   lr                                         \n"

        "reent_panic4:                                        \n"
        "   svc    "M2S(TFM_SVC_PSA_PANIC)"                   \n"
        "   b      .                                          \n"
    );
}
```