---
title: 快速了解zephyr
date: 2023-06-25 21:52:36
index_img: /img/post_pics/index_img/Zephyr-logo.png
tags: 
    - Arm
    - RTOS
    - Zephyr
categories: 
    - 操作系统
---

zephyr是一个强大的嵌入式跨平台开源操作系统，有丰富的模块和完善的文档。

<!-- more -->

本篇旨在挑选文档重点内容和代码演示。为后续的开发提供基础。

- [安装](#安装)
- [编译和运行](#编译和运行)
  - [使用Kconfig关闭编译选项](#使用kconfig关闭编译选项)
  - [musca\_s1开发板](#musca_s1开发板)
  - [Nucleo开发板](#nucleo开发板)
- [应用开发](#应用开发)
  - [概述](#概述)
  - [调试](#调试)
    - [使用工具连接调试硬件板](#使用工具连接调试硬件板)
- [API](#api)
- [内核系统开发](#内核系统开发)
  - [点灯](#点灯)
  - [线程开发](#线程开发)
    - [生命周期](#生命周期)
    - [线程状态](#线程状态)
    - [线程特性](#线程特性)
    - [创建线程](#创建线程)
  - [调度](#调度)
  - [CPU idle](#cpu-idle)
  - [系统线程](#系统线程)
  - [工作队列](#工作队列)
  - [中断](#中断)
  - [其他内核功能](#其他内核功能)
- [数据传输](#数据传输)

## 安装
参考[zephyr安装指南](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)

## 编译和运行
参考west使用指南：
 * [Building, Flashing and Debugging](https://docs.zephyrproject.org/latest/develop/west/build-flash-debug.html)
   * 嵌入CMAKE：[Permanent CMake Arguments](https://docs.zephyrproject.org/latest/develop/west/build-flash-debug.html#permanent-cmake-arguments)
 * [Built-in commands](https://docs.zephyrproject.org/latest/develop/west/built-in.html)
 * [Signing Binaries](https://docs.zephyrproject.org/latest/develop/west/sign.html)
 * [Using Zephyr without west](https://docs.zephyrproject.org/latest/develop/west/without-west.html)


以qemu an521为例
```bash
west update
west build -p auto -b mps2_an521_ns -t run samples/hello_world
```

选项`run`表示编译后自动运行，参考west usage，输出
```bash
[INF] Beginning TF-M provisioning
[WRN] TFM_DUMMY_PROVISIONING is not suitable for production! This device is NOT SECURE
[Sec Thread] Secure image initializing!
Booting TF-M a8313be08
Creating an empty ITS flash layout.
Creating an empty PS flash layout.
[INF][Crypto] Provisioning entropy seed... complete.
*** Booting Zephyr OS build zephyr-v3.3.0-1195-g9a759025d912 ***
Hello World! mps2_an521_ns
```

显然，这里打开了默认的trusted firmware-M
### 使用Kconfig关闭编译选项
参考：[Configuration System (Kconfig)](https://docs.zephyrproject.org/latest/build/kconfig/index.html)

命令：
```bash
west build -p auto -b mps2_an521_ns samples/hello_world -t menuconfig
```

为了编译不含TF-M的版本，选择打开`boards/arm/mps2_an521/Kconfig.defconfig`看到：
```python
config BOARD
	default "mps2_an521_ns" if TRUSTED_EXECUTION_NONSECURE
	default "mps2_an521_remote" if BOARD_MPS2_AN521_CPU1
	default "mps2_an521"

# By default, if we build for a Non-Secure version of the board,
# force building with TF-M as the Secure Execution Environment.
config BUILD_WITH_TFM
	default y if TRUSTED_EXECUTION_NONSECURE
```
运行
```bash
west build -p auto -b mps2_an521 samples/hello_world -t run
```
打开menuconfig看到
![](/img/post_pics/os/menuconfig_zephyr.png)
此时输出为：
```
*** Booting Zephyr OS build zephyr-v3.3.0-1195-g9a759025d912 ***
Hello World! mps2_an521
```

### musca_s1开发板
![](/img/post_pics/os/v2m_musca_s1.jpg)
板载资源可以在[Musca S1](https://docs.zephyrproject.org/latest/boards/arm/v2m_musca_s1/doc/index.html)找到，下载镜像，直接拷贝到存储区

```
west build -p auto -b v2m_musca_s1 samples/hello_world
cp build/zephyr/zephyr.hex /Volumes/MUSCA_S
minicom -D /dev/tty.usbmodem11302

output:
Musca-S1 Dual Firmware Version 1.9
*** Booting Zephyr OS build zephyr-v3.3.0-1195-g9a759025d912 ***
Hello World! musca_s1
```

### Nucleo开发板
![](/img/post_pics/os/nucleo_l552ze_q.jpg)
板载资源参考：[ST Nucleo L552ZE Q](https://docs.zephyrproject.org/latest/boards/arm/nucleo_l552ze_q/doc/nucleol552ze_q.html)，因为mac下载pyocd太慢装不了该板子的pack，选择runner openocd来下载镜像。编译时打开：
```
brew install openocd
west build -b nucleo_l552ze_q_ns samples/tfm_integration/tfm_ipc -- -DBOARD_FLASH_RUNNER=openocd
./build/tfm/regression.sh  #在下载不包含tfm的镜像时，需要额外跑一次恢复为normal模式
west flash
```

打开串口：
```
[WRN] This device was provisioned with dummy keys. This device is NOT SECURE
[Sec Thread] Secure image initializing!
Booting TF-M 79a6115d3
*** Booting Zephyr OS build zephyr-v3.4.0-483-g6050a10f8b29 ***
TF-M IPC on nucleo_l552ze_q
The version of the PSA Framework API is 257.
The PSA Crypto service minor version is 1.
Generating 256 bytes of random data:
11 50 AA DA 03 54 2E DB 0F B5 D3 79 EA BD C9 97 
8C F1 73 AB 3C DF 5E B2 FB C0 24 1E 34 32 02 A2 
B1 17 CD CF DE 25 E9 F2 B0 72 82 42 58 77 63 A7 
B0 30 5D 84 BF EA F4 6F D1 A3 6D 7A 23 27 EF 82 
9C 25 59 9B 31 43 9B 5A E5 FA F6 76 DE B2 0A 38 
2B 65 C1 45 53 61 27 7C 32 AC 7A 51 42 54 BD 92 
F9 8A CF 5B 64 FE B6 41 EB EC 08 CC 1E 8C 46 2A 
A7 16 6F A2 FE 4A 66 B4 D3 23 EF F9 6B 4A 40 84 
75 CE F8 AB 5F 61 52 C9 00 7B 10 9E CB CB D5 70 
35 1B E9 95 C7 71 F8 66 58 7B 7D 2A EB FF C1 2B 
9A 75 CD 34 39 8B 19 D1 98 24 3A 82 61 78 60 E1 
88 7C 6D A6 FE 4C AE 27 EF 24 1F DD 05 1A AF 04 
44 70 5D 5E 13 A1 DE C6 10 4C 15 13 CD DE EE E5 
29 81 C9 D3 9F 5E 7C 03 33 E7 F1 86 C1 57 2D 48 
F7 78 83 C0 39 B1 D3 ED 07 EA FD A0 B0 10 FE 8C 
C0 17 F4 74 96 67 D8 29 C4 1B 47 9F C3 69 06 0E
```





## 应用开发

### 概述
应用既可以在zephyrproject/zephyr里面创建编译，也可以在外部建立prj文件夹，项目文件中应当包含：
```
<app>
├── CMakeLists.txt
├── app.overlay
├── prj.conf
└── src
    └── main.c
```
以`samples/hello_world`为例：
* cmake文件，环境中一定要设置`ZEPHYR_BASE`
* 设备树文件
* Kconfig配置文件，defconfig
* C/C++源文件

### 调试
Zephyr运行qemu的命令是：
```
qemu-system-arm -cpu cortex-m33 \
                -machine mps2-an521 \
                -nographic -m 16 -vga none -net none \
                -pidfile qemu.pid \
                -chardev stdio,id=con,mux=on \
                -serial chardev:con \
                -mon chardev=con,mode=readline \
                -icount shift=6,align=off,sleep=off \
                -rtc clock=vm -chardev pty,id=hostS0 \
                -serial chardev:hostS0 \
                -kernel build/zephyr/zephyr.elf
```
加上`-s`，（`-gdb tcp::1234`的缩写）打开gdbserver，默认端口为：1234。也可以运行
```
west build -p auto -b mps2_an521 . -t debugserver_qemu
```
运行gdb命令：
```
arm-zephyr-eabi-gdb build/zephyr/zephyr.elf
(gdb) target remote localhost:1234
(gdb) dir ZEPHYR_BASE
```
#### 使用工具连接调试硬件板
参考：[Flash & Debug Host Tools](https://docs.zephyrproject.org/latest/develop/flash_debug/host-tools.html#j-link-debug-host-tools)
## API
汇总：[API Overview](https://docs.zephyrproject.org/latest/develop/api/overview.html)
使用说明：[API Guildlines](https://docs.zephyrproject.org/latest/develop/api/design_guidelines.html)

## 内核系统开发

### 点灯

以musca-s1为例，在其设备树文件boards/arm/v2m_musca_s1/v2m_musca_s1.dts中：
```
	aliases {
		led0 = &red_led;
		led1 = &green_led;
	};

	leds {
		compatible = "gpio-leds";
		red_led: led_0 {
			gpios = <&gpio 2 0>;
			label = "User LED1";
		};
		green_led: led_1 {
			gpios = <&gpio 3 0>;
			label = "User LED2";
		};
	};
```
使用led0和led1，用两个线程进行闪烁，调用的是toggle接口，使用fifo和输出线程通信，打印log，配置如下：
```py
CONFIG_PRINTK=y
CONFIG_HEAP_MEM_POOL_SIZE=256
CONFIG_ASSERT=y
CONFIG_GPIO=y
```

代码如下:
```c
/*
 * Copyright (c) 2016 Intel Corporation
 *
 * SPDX-License-Identifier: Apache-2.0
 */

#include <zephyr/kernel.h>
#include <zephyr/device.h>
#include <zephyr/drivers/gpio.h>
#include <zephyr/sys/printk.h>
#include <zephyr/sys/__assert.h>
#include <string.h>

/* 1000 msec = 1 sec */
#define SLEEP_TIME_MS 1000

/* size of stack area used by each thread */
#define STACKSIZE 1024

/* scheduling priority used by each thread */
#define PRIORITY 7

/* The devicetree node identifier for the "led0" alias. */
#define LED0_NODE DT_ALIAS(led0)
#define LED1_NODE DT_ALIAS(led1)

/*
 * A build error on this line means your board is unsupported.
 * See the sample documentation for information on how to fix this.
 */
static const struct gpio_dt_spec led0 = GPIO_DT_SPEC_GET(LED0_NODE, gpios);
static const struct gpio_dt_spec led1 = GPIO_DT_SPEC_GET(LED1_NODE, gpios);

struct printk_data_t
{
	void *fifo_reserved; /* 1st word reserved for use by fifo */
	uint32_t led;
	uint32_t cnt;
};

K_FIFO_DEFINE(printk_fifo);

void blink(const struct gpio_dt_spec *led, uint32_t sleep_ms, uint32_t id)
{
	int ret, cnt = 0;

	if (!gpio_is_ready_dt(led))
	{
		return;
	}

	ret = gpio_pin_configure_dt(led, GPIO_OUTPUT_ACTIVE);
	if (ret < 0)
	{
		return;
	}

	while (1)
	{

		ret = gpio_pin_toggle_dt(led);

		struct printk_data_t tx_data = {.led = id, .cnt = cnt};
		size_t size = sizeof(struct printk_data_t);

		char *mem_ptr = k_malloc(size);
		__ASSERT_NO_MSG(mem_ptr != 0);

		memcpy(mem_ptr, &tx_data, size);

		k_fifo_put(&printk_fifo, mem_ptr);

		k_msleep(sleep_ms);
		cnt++;
	}
}

void blink0(void)
{
	blink(&led0, SLEEP_TIME_MS, 0);
}

void blink1(void)
{
	blink(&led1, SLEEP_TIME_MS, 1);
}

void uart_out(void)
{
	while (1)
	{
		struct printk_data_t *rx_data = k_fifo_get(&printk_fifo,
												   K_FOREVER);
		printk("Toggled led%d; counter=%d\n",
			   rx_data->led, rx_data->cnt);
		k_free(rx_data);
	}
}

K_THREAD_DEFINE(blink0_id, STACKSIZE, blink0, NULL, NULL, NULL,	PRIORITY, 0, 0);
K_THREAD_DEFINE(blink1_id, STACKSIZE, blink1, NULL, NULL, NULL,	PRIORITY, 0, SLEEP_TIME_MS);
K_THREAD_DEFINE(uart_out_id, STACKSIZE, uart_out, NULL, NULL, NULL, PRIORITY, 0, 0);
```

### 线程开发

#### 生命周期

* 线程创建
* 线程终止，termination，入口函数执行返回，线程终止
  * 通过k_thread_join()睡眠当前线程，等待其他线程执行结束
* 线程中止，aborting，调用k_thread_abort()
* 线程挂起，suspension，调用k_thread_suspend()

#### 线程状态

zephyr的线程状态：![](/img/post_pics/os/thread_states.svg)

#### 线程特性

线程有以下重要特性：

* 堆栈区域
  * 内核堆栈
  * 线程堆栈
* 线程控制块
* 线程入口函数，及三个参数
* [线程优先级](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html#thread-priorities)
  * 越小优先级越高，可以为负数
* [线程配置选项](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html#configuration-options)
  * 如[K_USER](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html#c.K_USER)和[K_INHERIT_PERMS](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html#c.K_INHERIT_PERMS)
* 启动延时
* 执行模式

其他特性
* [自定义数据](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html#thread-custom-data)
* 用户线程，[CONFIG_USERSPACE](https://docs.zephyrproject.org/latest/kconfig.html#CONFIG_USERSPACE)，用户线程需要系统调度获取权限，参考[用户模式](https://docs.zephyrproject.org/latest/kernel/usermode/index.html#usermode-api)

#### 创建线程
参考[Spawning a Thread](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html#spawning-a-thread)

除了上述的[K_THREAD_DEFINE](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html#c.K_THREAD_DEFINE)创建以外，还可以使用[k_thread_create](https://docs.zephyrproject.org/latest/kernel/services/threads/index.html#c.k_thread_create)建立一个新的线程。

```c
K_THREAD_STACK_DEFINE(bk_stack0, STACKSIZE);
K_THREAD_STACK_DEFINE(bk_stack1, STACKSIZE);
K_THREAD_STACK_DEFINE(ptk_stack, STACKSIZE);

void main(void)
{
	struct k_thread bk0, bk1, ptk;
	k_timeout_t delay = K_MSEC(SLEEP_TIME_MS);

	k_thread_create(&bk0, bk_stack0, K_THREAD_STACK_SIZEOF(bk_stack0),
					(k_thread_entry_t)blink0, NULL, NULL, NULL, PRIORITY, 0, K_NO_WAIT);

	k_thread_create(&bk1, bk_stack1, K_THREAD_STACK_SIZEOF(bk_stack1),
					(k_thread_entry_t)blink1, NULL, NULL, NULL, PRIORITY, 0, delay);

	k_thread_create(&ptk, ptk_stack, K_THREAD_STACK_SIZEOF(ptk_stack),
					(k_thread_entry_t)uart_out, NULL, NULL, NULL, PRIORITY, 0, K_NO_WAIT);
}
```

### 调度
参考[schduling](https://docs.zephyrproject.org/latest/kernel/services/scheduling/index.html#)

支持以下三种：
* Simple linked-list ready queue (CONFIG_SCHED_DUMB)
* Red/black tree ready queue (CONFIG_SCHED_SCALABLE)
* Traditional multi-queue ready queue (CONFIG_SCHED_MULTIQ)

![](/img/post_pics/os/cooperative.svg)
其他相关：
* [抢占式线程](https://docs.zephyrproject.org/latest/kernel/services/scheduling/index.html#preemptive-time-slicing)
* [调度锁](https://docs.zephyrproject.org/latest/kernel/services/scheduling/index.html#scheduler-locking)，只执行当前线程
* [睡眠和唤醒](https://docs.zephyrproject.org/latest/kernel/services/scheduling/index.html#thread-sleeping)，释放cpu
* [忙等待](https://docs.zephyrproject.org/latest/kernel/services/scheduling/index.html#busy-waiting)，不释放cpu

要点：
* 将普通线程用于设备驱动程序和其他性能关键型工作。
* 使用普通线程实现相互排除，而无需内核对象（如互斥）。
* 使用抢占式线程优先考虑时间敏感的处理，而不是时间敏感度较低的处理。

### CPU idle
使CPU闲置会导致内核暂停所有操作，直到事件（通常是中断）唤醒CPU。在一般系统中，空闲线程负责这个事情。然而，在一些受约束的系统中，另一个线程可能会承担这一职责。使CPU闲置很简单：调用k_cpu_idle() API。CPU将停止执行指令，直到事件发生。最有可能的是，该函数将在循环中调用。请注意，在某些架构中，返回时，k_cpu_idle()无条件取消屏蔽中断。

```c
static k_sem my_sem;

void my_isr(void *unused)
{
    k_sem_give(&my_sem);
}

int main(void)
{
    k_sem_init(&my_sem, 0, 1);

    /* wait for semaphore from ISR, then do related work */

    for (;;) {

        /* wait for ISR to trigger work to perform */
        if (k_sem_take(&my_sem, K_NO_WAIT) == 0) {

            /* ... do processing */

        }

        /* put CPU to sleep to save power */
        k_cpu_idle();
    }
}
```

当线程除了空闲CPU等待事件外，还必须做一些实际工作时，请使用k_cpu_atomic_idle()。见下面的例子。

仅当线程仅负责空转CPU时，即不进行任何实际工作时，才使用k_cpu_idle()。

```c
static k_sem my_sem;

void my_isr(void *unused)
{
    k_sem_give(&my_sem);
}

int main(void)
{
    k_sem_init(&my_sem, 0, 1);

    for (;;) {

        unsigned int key = irq_lock();

        /*
         * Wait for semaphore from ISR; if acquired, do related work, then
         * go to next loop iteration (the semaphore might have been given
         * again); else, make the CPU idle.
         */

        if (k_sem_take(&my_sem, K_NO_WAIT) == 0) {

            irq_unlock(key);

            /* ... do processing */


        } else {
            /* put CPU to sleep to save power */
            k_cpu_atomic_idle(key);
        }
    }
}
```

### 系统线程
包括main线程和idle线程：
```c
int main(void)
{
    /* initialize a semaphore */
    ...

    /* register an ISR that gives the semaphore */
    ...

    /* monitor the semaphore forever */
    while (1) {
        /* wait for the semaphore to be given by the ISR */
        ...
        /* do whatever processing is now needed */
        ...
    }
}
```

### 工作队列
工作队列是一个内核对象，它使用专用线程以先入先出的方式处理工作项。每个工作项都通过调用工作项指定的函数进行处理。工作队列通常由ISR或高优先级线程使用，将非紧急处理卸载到低优先级线程，因此它不会影响时间敏感的处理。可以定义任意数量的工作队列（仅受可用RAM的限制）。每个工作队列都由其内存地址引用。参考：[workqueue-threads](https://docs.zephyrproject.org/latest/kernel/services/threads/workqueue.html#workqueue-threads)

### 中断
中断服务例程（ISR）是一种响应硬件或软件中断异步执行的函数。ISR通常抢占当前线程的执行，允许以非常低的开销进行响应。线程执行只有在所有ISR工作完成后才会恢复。参考：[interrupt service routine](https://docs.zephyrproject.org/latest/kernel/services/interrupts.html#interrupts)

多级中断处理：
```bash
          9             2   0
    _ _ _ _ _ _ _ _ _ _ _ _ _         (LEVEL 1)
  5       |         A   |
_ _ _ _ _ _ _         _ _ _ _ _ _ _   (LEVEL 2)
  |   C                       B
_ _ _ _ _ _ _                         (LEVEL 3)
        D
```
这里显示了三个中断级别。

* '-'表示中断行，从0开始编号（最右侧）。
* 1级有12条中断线，两条线（2和9）连接到嵌套控制器，4号线有一个设备“A”。
* 其中一个2级控制器的中断线5连接到3级嵌套控制器和3号线上的一个设备“C”。
* 另一个2级控制器没有嵌套控制器，但在2号线上有一个设备“B”。
* 3级控制器的2号线有一个设备“D”。

2级及以上控制器的编号都+1，于是有：
```
A -> 0x00000004
B -> 0x00000302 (2->2)
C -> 0x00000409 (3->9)
D -> 0x00030609 (2->5->9)
```

### 其他内核功能

* [轮训API](https://docs.zephyrproject.org/latest/kernel/services/polling.html)，轮询API的主要函数是k_poll()它在概念上与POSIX poll()函数非常相似，只是它在内核对象上运行，而不是在文件描述符上运行。
* [信号量](https://docs.zephyrproject.org/latest/kernel/services/synchronization/semaphores.html)，实现线程的互斥（多线程对资源的访问）或同步。
  * semaphore是使用k_sem类型的变量定义的。然后，它必须通过调用k_sem_init()进行初始化。
  * 或者，可以通过调用K_SEM_DEFINE在编译时定义和初始化信号量。
  * 通过调用k_sem_give()给出信号量。
  * 通过调用k_sem_take()来获取信号量
* [互斥锁](https://docs.zephyrproject.org/latest/kernel/services/synchronization/mutexes.html)，互斥锁允许多个线程通过确保相互排斥地访问资源来安全地共享相关的硬件或软件资源。
  * 互斥体是使用k_mutex类型的变量定义的。
  * 然后必须通过调用k_mutex_init()来初始化它。
  * 或者，可以通过调用K_MUTEX_DEFINE在编译时定义和初始化互斥体。
  * 互斥体通过调用k_mutex_lock()来锁定。通过调用k_mutex_unlock()来解锁互斥体。
* [条件变量](https://docs.zephyrproject.org/latest/kernel/services/synchronization/condvar.html)，使线程能够等到特定条件发生。
  * 线程可以通过调用k_condvar_wait()来等待条件。
  * 条件变量通过为一个线程调用k_condvar_signal()或为多个线程调用k_condvar_broadcast()来发出信号。
* [事件](https://docs.zephyrproject.org/latest/kernel/services/synchronization/events.html)，一个或多个线程可以等待事件对象，直到所需的事件集交付到事件对象。当新事件交付到事件对象时，所有等待条件已满足的线程都会同时准备就绪。
  * 使用k_event类型的变量定义事件对象。然后必须通过callkk_event_init()进行初始化。
  * 或者，可以通过调用K_EVENT_DEFINE在编译时定义和初始化事件对象。
  * 事件对象中的事件是通过调用k_event_set()来设置的。
  * 通过调用k_event_post()将事件发布到事件对象。
  * 线程通过调用k_event_wait()来等待事件。
* [多核对称多处理](https://docs.zephyrproject.org/latest/kernel/services/smp/smp.html)，在多处理器架构上，Zephyr支持使用运行Zephyr应用程序代码的多个物理CPU。这种支持是“对称的”，因为默认情况下没有特定的CPU被特别处理。任何处理器都能够运行任何Zephyr线程，并支持访问所有标准的Zephyr API。
  * SMP系统提供了一个更受约束的k_spin_lock()原语，它不仅像irq_lock()所做的那样在本地屏蔽中断，而且还在返回调用者之前原子地验证共享锁变量已被修改，如果需要等待另一个CPU退出锁，则在检查上“旋转”。k_spin_lock()和k_spin_unlock()的默认Zephyr实现建立在预先存在的atomic_层（本身通常使用编译器本质实现）之上，尽管出于性能原因，架构存在定义自己的设施。
  * IRQ锁和自旋锁之间的一个重要区别是，早期的API是自然递归的：锁是全局的，因此在关键部分内获取嵌套锁是合法的。Spinlocks是可分离的：您可以为单独的子系统或数据结构拥有许多锁，防止CPU在单个全局资源上竞争。但这意味着旋转锁不能递归使用。持有特定锁的代码不得试图重新获取它，否则它将陷入僵局（然而，嵌套不同的自旋锁是完全合法的）。验证层可用于检测和报告此类错误。

## 数据传输
下面汇总所有的数据传输方式，参考[Data passing](https://docs.zephyrproject.org/latest/kernel/services/index.html#data-passing)

| Object        | Bidirectional? | Data structure  | Data item size | Data Alignment | ISRs can receive? | ISRs can send? | Overrun handling             |
| ------------- | -------------- | --------------- | -------------- | -------------- | ----------------- | -------------- | ---------------------------- |
| FIFO          | No             | Queue           | Arbitrary [1]  | 4 B [2]        | Yes [3]           | Yes            | N/A                          |
| LIFO          | No             | Queue           | Arbitrary [1]  | 4 B [2]        | Yes [3]           | Yes            | N/A                          |
| Stack         | No             | Array           | Word           | Word           | Yes [3]           | Yes            | Undefined behavior           |
| Message queue | No             | Ring buffer     | Power of two   | Power of two   | Yes [3]           | Yes            | Pend thread or return -errno |
| Mailbox       | Yes            | Queue           | Arbitrary [1]  | Arbitrary      | No                | No             | N/A                          |
| Pipe          | No             | Ring buffer [4] | Arbitrary      | Arbitrary      | Yes [5]           | Yes [5]        | Pend thread or return -errno |

[1] Callers allocate space for queue overhead in the data elements themselves.
[2] Objects added with k_fifo_alloc_put() and k_lifo_alloc_put() do not have alignment constraints, but use temporary memory from the calling thread’s resource pool.
[3] ISRs can receive only when passing K_NO_WAIT as the timeout argument.
[4] Optional.
[5] ISRS can send and/or receive only when passing K_NO_WAIT as the timeout argument.
[6] Data item size must be a multiple of the data alignment.
