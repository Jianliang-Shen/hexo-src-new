---
title: ARM Cortex-M FPU 模块
date: 2022-07-11 21:14:39
tags:
    - Arm
    - FPU
    - TEE
    - 固件
    - TF-M
    - 中断
categories: 
    - 嵌入式
---

测试安全模式和非安全模式下中断的FPU寄存器保护

<!-- more -->

- [FPU基本介绍](#fpu基本介绍)
  - [寄存器](#寄存器)
    - [CPACR](#cpacr)
    - [Floating point register bank](#floating-point-register-bank)
    - [FPSCR](#fpscr)
    - [FPCCR](#fpccr)
    - [FPCAR](#fpcar)
    - [FPDSCR](#fpdscr)
  - [编译命令](#编译命令)
  - [相关指令](#相关指令)
- [中断](#中断)
  - [寄存器](#寄存器-1)
  - [配置和使用](#配置和使用)
- [测试方案和结果](#测试方案和结果)

## FPU基本介绍

Float point unit，浮点计运算元，在ARM Cortex-M同样广泛应用。C语言中浮点数有单精度和双精度之分：

```bash
float pi = 3.141592F;
double pi = 3.1415926535897932384626433832795;
```

### 寄存器

#### CPACR

Coprocessor Access Control Register，指定协处理器和浮点扩展的访问权限。如果实现了MVE，则此寄存器指定MVE的访问权限。MVE是指基于M-Profile Vector Extension矢量扩充方案的Arm Helium技术。MVE可以分为2大类，MVE-I和MVE-F。MVE-I仅对整型向量提供支持，MVE-F则对浮点数据向量提供支持。要包含MVE-F那么处理器核心就需要支持MVE-I以及浮点扩展。
寄存器组成：

| 31-24 | 23-22 | 21-20 | 19-16 | 15-14 | ... | 3-2 | 1-0 |
| ----- | ----- | ----- | ----- | ----- | --- | --- | --- |
| RES0  | CP11  | CP10  | RES0  | CP7   | ... | CP1 | CP0 |

`CP10`定义浮点功能的访问权限，`00b`表示对FP/MVE导致NOCP UsageFault；`01b`表示只能特权访问；`11b`表示可以在特权非特权访问FP Extension和MVE。`CP11`则和CP10同步，如果`CP10`是`RAZ/WI`，则`CP11`也是；如果`CP11`值编程和`CP10`一致，则该值为`UNKNOWN`。`CPm`，控制协处理器m的访问权限，同`CP10`。
> `RES0`, Reserved，保留字段。
> `RAZ/WI`，读为0，写忽略。
> `NOCP`，No Co-processor。

使处理器能完全访问CP10和CP11：

```c
#if (__FPU_PRESENT == 1) && (__FPU_USED == 1)
    SCB->CPACR |= ((3UL << 20U)|(3UL << 22U));  /* set CP10 and CP11 Full Access */
#endif
```

> `SCB`，System Control Block，系统控制块。
> `__FPU_PRESENT`，表示当前处理器是否有FPU
> `__FPU_USED`，表示当前处理器是否使用FPU

#### Floating point register bank

分为`S0-S31`或者`D0-D15`，后者为双字64位寄存器，D0指S1+S0。

- S0-S15是caller saved（调用者保护）寄存器，如果函数A调用函数B，A函数必须保存这些寄存器（例如入栈），因为这些寄存器有可能被B破坏，类似R0-R3寄存器。
- S16-S31是callee saved（被调用者保护）寄存器，如果函数A调用函数B，并且B函数需要使用超过16个寄存器（指caller），B函数需要首先保存callee在堆栈上，并且在返回函数A前还原callee寄存器。

#### FPSCR

Float point status and control register，浮点状态和控制寄存器。类似程序状态寄存器应用`xPSR`（APSR, Application Program Status Register）。可以将状态复制给APSR：

```c
VMRS APSR_nzcv, FPSCR;
```

#### FPCCR

Float point context control register浮点上下文控制寄存器，用于控制[惰性压栈(lazy stacking)](https://developer.arm.com/documentation/dai0298/a/)特性等异常处理行为。TF-M初始化时对`FPCCR`的修改：

```c
#if (CONFIG_TFM_FP >= 1)

#ifdef CONFIG_TFM_LAZY_STACKING
    /* Enable lazy stacking. */
    FPU->FPCCR |= FPU_FPCCR_LSPEN_Msk;
#else
    /* Disable lazy stacking. */
    FPU->FPCCR &= ~FPU_FPCCR_LSPEN_Msk;
#endif

    /*
     * If the SPE will ever use the floating-point registers for sensitive
     * data, then FPCCR.ASPEN, FPCCR.TS, FPCCR.CLRONRET and FPCCR.CLRONRETS
     * must be set at initialisation and not changed again afterwards.
     * Let SPE decide the S/NS shared setting (LSPEN and CLRONRET) to avoid the
     * possible side-path brought by flexibility. This is not needed
     * if the SPE will never use floating-point but enables the FPU only for
     * avoiding NOCP faults during interrupted NSPE to SPE calls.
     */
    FPU->FPCCR |= FPU_FPCCR_ASPEN_Msk
                  | FPU_FPCCR_TS_Msk
                  | FPU_FPCCR_CLRONRET_Msk
                  | FPU_FPCCR_CLRONRETS_Msk
                  | FPU_FPCCR_LSPENS_Msk;

```

- `ASPEN`：置位时，使能`CONTROL.FPCA`，caller saved（S0-S15，FPSR和VPR）寄存器在异常入口和推出时的状态自动保存
- `LSPEN`：置位时，异常流程使用惰性压栈特性以保持中断等待
- `LSPENS`：控制协处理器mLazy状态保存启用Secure的访问权限。该位控制LSPEN位是否可从非安全状态写入。置位时，LSPEN可从两种安全状态读取，但从非安全状态忽略对LSPEN的写入。
- `CLRONRET`：如果 ASPEN/CONTROL.FPCA = 1 并且 FPCCR_S.LSPACT = 0，在异常返回时，清空caller saved寄存器，CLRONRETS置位时，NS不能写入
- `CLRONRETS`：仅限Secure能清空caller saved寄存器，置位时，NS只能读CLRONRET，而不能写入
- `TS`：当设置为0时即使PE处于安全状态，浮点寄存器也被视为非安全，因此，callee saved寄存器永远不会被推入堆栈。如果浮点寄存器从不包含需要保护的数据，清除该标志可以减少中断延迟。由于该字段改变了安全堆栈帧的解释方式，如果该位的状态与当前安全堆栈不一致，则可能导致不可预测的行为。 因此，固件在修改此值时必须小心。该字段在非安全状态下表现为 RAZ/WI。**简而言之，TS=1，则在secure中，caller and callee均会保存。**

#### FPCAR

Float point context address register，浮点上下文地址寄存器。和惰性压栈机制有关，即`FPCCR.LSPEN`。目的是为了降低中断等待，为长栈帧包括整型的寄存器组R0-R3,R12,LR,Return Address，xPSR以及FPU寄存器S0-S15和FPSCR预留栈空间从而快速返回。

#### FPDSCR

Float point default status control register，存放默认浮点状态控制数据的配置信息，这些数据在异常入口处被复制到了FPSCR。

### 编译命令

gcc:

```bash
-mfloat-abi=hard -mfpu=fpv5-d16 -march=cortex-m55 -mcpu=armv8.1-m.main
```

hard和soft abi（应用程序二进制接口）的影响：

- 是否使用浮点单元进行计算
- 调用函数和被调用函数如何传递参数（使用整数寄存器组还是FP寄存器组）

gcc浮点ABI选项

- -mfloat-abi=soft，0，此时所有浮点运算都由runtime库函数实现，数据通过整数寄存器组实现
- -mfloat-abi=softfp，1，代码可以访问FPU，如需访问runtime库函数，使用软件浮点调用规则，即使用整数寄存器组实现
- -mfloat-abi=hard，2，代码可以访问FPU，调用runtime库函数使用FPU相关的调用规则

> 如果所有浮点运算都是单精度的，则硬件浮点ABI的性能最高，如果双精度，软件ABI性能更好一些，因为Cortex-M4的FPU不支持双精度的浮点运算，数值仍需从FPU寄存器复制后由软件处理，带来额外开销。
>
### 相关指令

| 指令        | 说明                             | 示例                      |
| ----------- | -------------------------------- | ------------------------- |
| VADD        | 浮点加法                         | /                         |
| VSUB        | 浮点减法                         | /                         |
| VMUL        | 浮点乘法                         | /                         |
| VDIV        | 浮点除法                         | /                         |
| VABS        | 浮绝对值                         | /                         |
| VSQRT       | 浮点平方根                       | /                         |
| VCVT(R/B/F) | 数值转换                         | /                         |
| VCMP        | 比较两个浮点寄存器               | `vcmp.f16     s6,#0`      |
| VSTR        | 将浮点寄存器的值存入存储器       | /                         |
| VLDR        | 从存储中加载一个浮点数存入寄存器 | /                         |
| VPUSH/VPOP  | 压栈出栈                         | `vpush     {s0-s15}`      |
| VMOV        | 复制                             | `vmov      s1,r0`         |
| VSTM        | 将多个寄存器读出到地址中         | `vstm      r0, {s16-s31}` |
| VLDM        | 将地址中的数组写入多个寄存器     | `vldm      r0,{s16-s31}`  |

## 中断

### 寄存器

除了STIR以外，所有寄存器都只能特权访问。STIR默认特权访问，但也可以设置`CCR.USERSETMPEND=1`实现非特权访问。

| Address    | Register   | Description                           | NVIC API                                    |
| ---------- | ---------- | ------------------------------------- | ------------------------------------------- |
| 0xE002E100 | NVIC_ISERn | Interrupt Set Enable Register         | NVIC_EnableIRQ                              |
| 0xE002E180 | NVIC_ICERn | Interrupt Clear Enable Register       | NVIC_DisableIRQ                             |
| 0xE002E200 | NVIC_ISPRn | Interrupt Set Pending Register        | NVIC_SetPendingIRQ                          |
| 0xE002E280 | NVIC_ICPRn | Interrupt Clear Pending Register      | NVIC_ClearPendingIRQ                        |
| 0xE002E300 | NVIC_IABRn | Interrupt Active Bit Register         | NVIC_GetActive                              |
| 0xE002E400 | NVIC_IPRn  | Interrupt Priority Register           | NVIC_SetPriority                            |
| 0xE000E380 | NVIC_ITNSn | Interrupt Target Non-secure Register  | NVIC_SetTargetState/NVIC_ClearTargetState   |
| 0xE000EF00 | STIR       | Software Triggered Interrupt Register | NVIC->STIR = 3  <==> NVIC_SetPendingIRQ(3); |
| 0xE000ED08 | VTOR       | Vector Table Offset Register          | NVIC_SetVector                              |

> 若要在下一个操作前立即执行中断，则要设置存储器屏障`_DSB()`。
> 设置ITNS，1在NS中执行，0在S中执行
> 注：`S可以触发S、NS的中断，NS可以触发NS的中断，但是无法触发S的中断，后者是架构不允许的`

以下汇编实现NVIC的功能，仅限前32中断。

```c
__attribute__((naked)) uint32_t ClearTargetState(uint32_t IRQ_NUM){
    __asm volatile(
        "push    {r6-r9, lr}                  \n"
        "ldr     r7, =0xE000E380              \n"
        "ldr     r8, [r7]                     \n"
        "ldr     r9, =0x1                     \n"
        "lsl     r9, r0                       \n"
        "bic     r8, r8, r9                   \n"
        "str     r8, [r7]                     \n"
        "pop     {r6-r9, pc}                  \n"
    );
}

__STATIC_INLINE uint32_t NVIC_ClearTargetState(IRQn_Type IRQn)
{
  if ((int32_t)(IRQn) >= 0)
  {
    NVIC->ITNS[(((uint32_t)IRQn) >> 5UL)] &= ~((uint32_t)(1UL << (((uint32_t)IRQn) & 0x1FUL)));
    return((uint32_t)(((NVIC->ITNS[(((uint32_t)IRQn) >> 5UL)] & (1UL << (((uint32_t)IRQn) & 0x1FUL))) != 0UL) ? 1UL : 0UL));
  }
  else
  {
    return(0U);
  }
}

__attribute__((naked)) uint32_t SetTargetState(uint32_t IRQ_NUM){
    __asm volatile(
        "push    {r6-r9, lr}                  \n"
        "ldr     r7, =0xE000E380              \n"
        "ldr     r8, [r7]                     \n"
        "ldr     r9, =0x1                     \n"
        "lsl     r9, r0                       \n"
        "orr     r8, r8, r9                   \n"
        "str     r8, [r7]                     \n"
        "pop     {r6-r9, pc}                  \n"
    );
}
__attribute__((naked)) uint32_t EnableIRQ(uint32_t IRQ_NUM){
    __asm volatile(
        "push    {r6-r9, lr}                  \n"
        "ldr     r7, =0xE000E100              \n"
        "ldr     r8, [r7]                     \n"
        "ldr     r9, =0x1                     \n"
        "lsl     r9, r0                       \n"
        "orr     r8, r8, r9                   \n"
        "str     r8, [r7]                     \n"
        "pop     {r6-r9, pc}                  \n"
    );
}

__STATIC_INLINE void __NVIC_EnableIRQ(IRQn_Type IRQn)
{
  if ((int32_t)(IRQn) >= 0)
  {
    __COMPILER_BARRIER();
    NVIC->ISER[(((uint32_t)IRQn) >> 5UL)] = (uint32_t)(1UL << (((uint32_t)IRQn) & 0x1FUL));
    __COMPILER_BARRIER();
  }
}
```

触发中断的代码：

```c
__attribute__((naked)) uint32_t interrupt_trigger(uint32_t IRQ_NUM){
    __asm volatile(
        "push    {r7, lr}                  \n"
        /* Software Trigger Interrupt Register address is 0xE000EF00. */
        "ldr     r7, =0xE000EF00           \n"
        "str     r0, [r7]                  \n"
        "dsb     0xF                       \n"
        "pop     {r7, pc}                  \n"
    );
}
```

### 配置和使用

```c
typedef enum _IRQn_Type {
    NonMaskableInt_IRQn                = -14,  /* Non Maskable Interrupt */
    HardFault_IRQn                     = -13,  /* HardFault Interrupt */
    MemoryManagement_IRQn              = -12,  /* Memory Management Interrupt */
    BusFault_IRQn                      = -11,  /* Bus Fault Interrupt */
    UsageFault_IRQn                    = -10,  /* Usage Fault Interrupt */
    SecureFault_IRQn                   = -9,   /* Secure Fault Interrupt */
    SVCall_IRQn                        = -5,   /* SV Call Interrupt */
    DebugMonitor_IRQn                  = -4,   /* Debug Monitor Interrupt */
    PendSV_IRQn                        = -2,   /* Pend SV Interrupt */
    SysTick_IRQn                       = -1,   /* System Tick Interrupt */
    NONSEC_WATCHDOG_RESET_REQ_IRQn     = 0,    /* Non-Secure Watchdog Reset
                                                * Request Interrupt
                                                */
    NONSEC_WATCHDOG_IRQn               = 1,    /* Non-Secure Watchdog Interrupt */
    SLOWCLK_TIMER_IRQn                 = 2,    /* SLOWCLK Timer Interrupt */
    ...
    /* Reserved                        = 128:130   Reserved */
} IRQn_Type;
```

1. 分配IRQn，这个枚举类型里，前面为系统异常，>=0的为中断。定义一个新的`IRQn`，例如`Example_IRQn`。
2. 设置Handler函数。在BSP中，中断函数可以写为回调函数的模式，嵌入到中断调用的函数。这里的Handler函数是ISPR或者STIR触发中断后，CPU直接执行的函数，不通过回调的上层模式，在寄存器层面实现中断。
3. 绑定。有两种方式，一是在内存中通过`NVIC_SetVector`，实现`IRQn`和Handler的绑定；如果RAM是受到保护的，采用另一种方式直接改startup文件中的配对表。
4. 设置执行的S/NS环境。通过`INTS`寄存器配置对应中断是在S还是NS中执行。
5. 使能中断，通过`NVIC_EnableIRQ`打开中断使能。
6. 触发中断，通过asm函数修改`STIR`寄存器的值或者通过NVIC的接口set pending。（后者只能特权模式）

## 测试方案和结果

测试平台：ARM AN521/AN552 FVP

1. 测试NS调用NS的中断保护，NS中断修改所有的寄存器，返回后只有caller恢复
2. 测试S调用S的中断保护，由于`FPCCR.TS=1`，所以caller callee在中断结束后都恢复
   1. 中断发生前，`FPCCR_S.LSPACT`=1，`CONTROL_S.FPCA`=1
   2. 进入Handler时，`FPCCR_S.LSPACT`=0，`CONTROL_S.FPCA`=0，因此返回时不清caller，如果此时修改为1，则测试用例失败（caller被清无法恢复）
3. 测试S调用NS的中断保护，在进入NS IRQ Handler时，验证所有寄存器清零；修改寄存器值后中断返回后Secure中的caller callee均恢复

> If FPCCR_S.TS is 1 when the Floating-point context and additional Floating-point context are both pushed to the stack, S0-S31 and the FPSCR are set to zero after stacking
