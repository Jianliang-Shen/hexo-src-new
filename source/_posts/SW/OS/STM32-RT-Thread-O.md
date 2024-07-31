---
title: STM32 RT-Thread OS实战
date: 2020-09-05 15:54:45
index_img: /img/index_img/rtthread.png
tags: 
    - RTOS
categories: 
    - OS
---
使用的平台：秉火STM32 Cortex-M3内核开发板，RT-Thread v3.1.3。
<!-- more -->
### RT-Thread移植
以秉火-指南者STM32F103VE6为例。参考[官方文档](https://www.rt-thread.org/document/site/)，[Git仓库](https://github.com/RT-Thread/rt-thread)，在keil5中添加RT-Thread内核，并下载源码：
```bash
git clone git@github.com:RT-Thread/rt-thread.git
```
用keil5打开rt-thread\bsp\stm32\stm32f103-fire-arbitrary\project.uvprojx，修改下载器配置，选择芯片STM32F103VE6，并修改main.c中的LED管脚，因为此例程使用的是STM32F103ZET6芯片版本。
```c
/* defined the LED0 pin: PF7 */
//#define LED0_PIN    GET_PIN(F, 7)
#define LED0_PIN    GET_PIN(B, 0) //LED_G
#define LED1_PIN    GET_PIN(B, 1) //LED_B
#define LED2_PIN    GET_PIN(B, 5) //LED_R
int main(void)
{
    int count = 1;
    /* set LED0 pin mode to output */
    rt_pin_mode(LED0_PIN, PIN_MODE_OUTPUT);
    rt_pin_mode(LED1_PIN, PIN_MODE_OUTPUT);
    rt_pin_mode(LED2_PIN, PIN_MODE_OUTPUT);
	
    rt_pin_write(LED0_PIN, PIN_LOW);
    rt_pin_write(LED1_PIN, PIN_LOW);
    rt_pin_write(LED2_PIN, PIN_LOW);

    while (count++)
    {
        rt_pin_write(LED0_PIN, PIN_LOW);
        //rt_kprintf("SET LOW\n");
        rt_thread_mdelay(100);
        rt_pin_write(LED0_PIN, PIN_HIGH);  //OFF ???!
        //rt_kprintf("SET HIGH\n");
        rt_thread_mdelay(100);
			
        rt_pin_write(LED1_PIN, PIN_LOW);
        rt_thread_mdelay(100);
        rt_pin_write(LED1_PIN, PIN_HIGH);
        rt_thread_mdelay(100);
			
        rt_pin_write(LED2_PIN, PIN_LOW);
        rt_thread_mdelay(100);
        rt_pin_write(LED2_PIN, PIN_HIGH);
        rt_thread_mdelay(100);
    }
    return RT_EOK;
}

```
值得注意的是，PIN_HIGH会使灯关闭，PIN_LOW会使灯关闭，这一点还没有搞清楚。在控制台输出使用rt_kprintf()函数，输出在UART串口，连接电脑即可显示。
### RT-Thread Qemu仿真
参考[QEMU-Ubuntu](https://github.com/RT-Thread/rtthread-manual-doc/blob/master/documentation/quick_start_qemu/quick_start_qemu_linux.md)，图片访问有问题，建议下载仓库查看readme。
### UART读取北斗数据
在STM32F103指南者上引脚如下，其中UART1为console串口，输出信息。
<style>
table th:nth-of-type(1) {
	width: 300px;
}
table th:nth-of-type(2) {
	width: 300px;
}
</style>
| PIN NAME | PIN ADDR |
| -------- | -------- |
| UART1_RX | PIN A9   |
| UART1_TX | PIN A10  |
| UART3_RX | PIN B10  |
| UART3_TX | PIN B11  |
  
连接北斗单元至UART3，在固件库版本中将UART1输出重定向为printf，将UART3输入重定向为scanf，这样就很轻松的读取到北斗的数据。同样的，在RT Thread版本中，在使用UART3之前首先需要定义设备和管脚，并打开：
```c
//board.c
/*------------------------ UART3 CONFIG BEGIN -------------------------*/
#define BSP_USING_UART3
#define UART3_TX_PORT GPIOB
#define UART3_RX_PORT GPIOB
#define UART3_TX_PIN GPIO_PIN_10
#define UART3_RX_PIN GPIO_PIN_11
/*------------------------  UART3 CONFIG END  -------------------------*/	
```
在main.c的路径下创建uart3.c文件，添加如下内容：
```c
#include <rtthread.h>
#include <rtdevice.h>

#define SAMPLE_UART_NAME "uart3"
static struct rt_semaphore rx_sem;
static rt_device_t serial;

static rt_err_t uart_input(rt_device_t dev, rt_size_t size)
{
    rt_sem_release(&rx_sem);

    return RT_EOK;
}

static void serial_thread_entry(void *parameter)
{
    char ch;

    while (1)
    {
        while (rt_device_read(serial, -1, &ch, 1) != 1)
        {
            rt_sem_take(&rx_sem, RT_WAITING_FOREVER);
        }
        rt_kprintf("%c", ch);
        ch = ch + 1;

        //rt_device_write(serial, 0, &ch, 1);
    }
}

static int uart_sample(int argc, char *argv[])
{
    struct serial_configure config = RT_SERIAL_CONFIG_DEFAULT;

    config.baud_rate = BAUD_RATE_38400;
    config.data_bits = DATA_BITS_8;
    config.stop_bits = STOP_BITS_1;
    config.bufsz = 128;
    config.parity = PARITY_NONE;

    rt_err_t ret = RT_EOK;

    char uart_name[RT_NAME_MAX];
    char str[] = "hello RT-Thread!\r\n";

    if (argc == 2)
    {
        rt_strncpy(uart_name, argv[1], RT_NAME_MAX);
    }
    else
    {
        rt_strncpy(uart_name, SAMPLE_UART_NAME, RT_NAME_MAX);
    }

    serial = rt_device_find(uart_name);
    if (!serial)
    {
        rt_kprintf("find %s failed!\n", uart_name);
        return RT_ERROR;
    }

    rt_sem_init(&rx_sem, "rx_sem", 0, RT_IPC_FLAG_FIFO);
    rt_device_control(serial, RT_DEVICE_CTRL_CONFIG, &config);
    rt_device_open(serial, RT_DEVICE_FLAG_INT_RX);
    rt_device_set_rx_indicate(serial, uart_input);
    rt_device_write(serial, 0, str, (sizeof(str) - 1));
    rt_thread_t thread = rt_thread_create("serial", serial_thread_entry, RT_NULL, 1024, 25, 10);

    if (thread != RT_NULL)
    {
        rt_thread_startup(thread);
    }
    else
    {
        ret = RT_ERROR;
    }

    return ret;
}

MSH_CMD_EXPORT(uart_sample, uart device sample);
```
该文件中，最后一句完成看自定义命令的添加。完成以上配置后，下载后打开串口输入命令查看结果：
![](/img/os/gps.PNG)


### 管脚中断
通过按键1和2改变灯的亮灭。这里主要使用了管脚输入输出和中断输入。
```c
#include <rtthread.h>
#include <rtdevice.h>

#define LED0_PIN_NUM 16  /* PB0  */
#define KEY0_PIN_NUM 0   /* PA0  */
#define KEY1_PIN_NUM 45  /* PC13 */

void led_on(void *args)
{
    rt_kprintf("turn on led!\n");
    rt_pin_write(LED0_PIN_NUM, PIN_LOW);
}

void led_off(void *args)
{
    rt_kprintf("turn off led!\n");
    rt_pin_write(LED0_PIN_NUM, PIN_HIGH);
}

static void key_led_sample(void)
{
    rt_pin_mode(LED0_PIN_NUM, PIN_MODE_OUTPUT); 
    rt_pin_write(LED0_PIN_NUM, PIN_LOW);
 
    rt_pin_mode(KEY0_PIN_NUM, PIN_MODE_INPUT_PULLUP); 
    /* 绑定中断，下降沿模式，回调函数名为led_off */
    rt_pin_attach_irq(KEY0_PIN_NUM, PIN_IRQ_MODE_FALLING, led_on, RT_NULL);
    /* 使能中断 */
    rt_pin_irq_enable(KEY0_PIN_NUM, PIN_IRQ_ENABLE);

    rt_pin_mode(KEY1_PIN_NUM, PIN_MODE_INPUT_PULLUP);
    rt_pin_attach_irq(KEY1_PIN_NUM, PIN_IRQ_MODE_FALLING, led_off, RT_NULL);
    rt_pin_irq_enable(KEY1_PIN_NUM, PIN_IRQ_ENABLE);
}

MSH_CMD_EXPORT(key_led_sample, key led sample);
```