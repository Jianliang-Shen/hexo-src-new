---
title: AM5728配置DMM_LISA_MAP修改内存工作模式
date: 2019-11-08 13:30:34
index_img: /img/post_pics/index_img/am5728_ram.png
tags: 
    - Linux
    - 内存
categories: 
    - 嵌入式
---
在测试AM5728时发现，ARM读完SD card中的MLO文件后没有后续操作.

<!-- more -->

推断是由于我们的硬件更改导致的内存加载出错，研究TI给出的《AM572x Technical Reference Manual》，需要配置相关的寄存器。

- [硬件更改](#硬件更改)
- [寄存器详解](#寄存器详解)
- [U-BOOT配置寄存器](#u-boot配置寄存器)
  
### 硬件更改

原TI AM5728使用EMIF0接口和EMIF1接口，使用interleaving模式配置2GB的内存，每个EMIF接口连接两个512MB的内存芯片。为降低功耗，平衡性能和成本，现更改为单EMIF接口连接两个256MB的芯片，并且在ECC内存接口增设ECC DDR保证内存运行的可靠性。

|             | EMIF1     | EMIF2     | ECC   |
| ----------- | --------- | --------- | ----- |
| TI AM5728   | 2 x 512MB | 2 x 512MB | /     |
| ZHX星务主板 | 2 x 256MB | /         | 128MB |
  
<!-- more -->  
### 寄存器详解

根据TRM 15.2.3.5.1.2所述，在系统启动时，需要合理更改DDR的寄存器配置，主要有以下四种：

- DMM_LISA_MAP_3
- DMM_LISA_MAP_2
- DMM_LISA_MAP_1
- DMM_LISA_MAP_0
  
参考DMM_LISA_MAP_i的寄存器表格可以重置DDR的启动功能和映射大小。  

![](/img/post_pics/ram/pic1.png)
  
上图为寄存器的内容，SYS_SIZE为配置的内存总大小，SDRC_MAP为interleaving的配置方式，可选只使用EMIF1、只使用EMIF2、或者EMIF1和EMIF2交叉存取的方式，如果工作在非交叉存取的模式，SDRC_INTL不需要配置。  
  
![](/img/post_pics/ram/pic2.png)

### U-BOOT配置寄存器

在TI-SDK的U-BOOT源码board/ti/board.c中，以下程序完成了对TI AM5728 EVM的内存配置：  

```CPP
static const struct dmm_lisa_map_regs beagle_x15_lisa_regs = {
 .dmm_lisa_map_3 = 0x80740300,
 .is_ma_present  = 0x1
};

void emif_get_dmm_regs(const struct dmm_lisa_map_regs **dmm_lisa_regs)
{
 if (board_is_am571x_idk())
  *dmm_lisa_regs = &am571x_idk_lisa_regs;
 else
  *dmm_lisa_regs = &beagle_x15_lisa_regs;
}

```

根据原设置使用寄存器3的内容0x8074 0300可以看出内存的配置如下：  
![](/img/post_pics/ram/pic3.png)

根据硬件配置，我们的内存设置如下：  
![](/img/post_pics/ram/pic4.png)

参考：[TI wiki](http://processors.wiki.ti.com/index.php?oldid=127545&title=EZSDK_Memory_Map&keyMatch=MEMORY%20MAP&tisearch=Search-CN-everything)  
