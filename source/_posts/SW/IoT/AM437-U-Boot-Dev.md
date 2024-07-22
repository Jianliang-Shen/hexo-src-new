---
title: AM437 U-Boot-2014修改
date: 2020-05-25 9:12:05
tags:
    - Linux
    - U-Boot
categories: 
    - 嵌入式
---

为了让ARM启动时从SD-card搬运数据然后转NAND Flash启动，需要重新配置创龙提供的2014版本的Uboot源码文件.

<!-- more -->  

主要思路包括：  

- [U-boot添加Ubi和Ubifs命令](#u-boot添加ubi和ubifs命令)
- [配置环境变量](#配置环境变量)
- [使用UBI命令](#使用ubi命令)
- [制作文件系统镜像](#制作文件系统镜像)
- [烧写文件](#烧写文件)
- [添加自定义三取二命令](#添加自定义三取二命令)
  - [添加自定义命令的方法](#添加自定义命令的方法)
  - [编写自定义命令源码](#编写自定义命令源码)
- [修改Uboot初始化配置](#修改uboot初始化配置)
  
## U-boot添加Ubi和Ubifs命令

在uboot-2014的include/configs/am43xx_evm.h中添加如下声明。  

```C
/* #define CONFIG_MTD_DEVICE            1 */
#define CONFIG_MTD_PARTITIONS        1
#define CONFIG_CMD_MTDPARTS         
#define CONFIG_CMD_UBIFS           
#define CONFIG_CMD_UBI         
#define CONFIG_LZO                   1
#define CONFIG_RBTREE                1
```
  
## 配置环境变量

在上述头文件中修改nandroot变量：  

```C
#define NANDARGS \
    "mtdids=" MTDIDS_DEFAULT "\0" \
    "mtdparts=" MTDPARTS_DEFAULT "\0" \
    "nandargs=setenv bootargs console=${console} " \
        "${optargs} " \
        "root=${nandroot} " \
        "rootfstype=${nandrootfstype}\0" \
    "nandroot=ubi0:NAND.file-system rw ubi.mtd=NAND.file-system,2048\0" \
    "nandrootfstype=ubifs rootwait=1\0" \
    "nandboot=echo Booting from nand ...; " \
        "run nandargs; " \
        "nand read ${fdtaddr} NAND.u-boot-spl-os; " \
        "nand read ${loadaddr} NAND.kernel; " \
        "bootz ${loadaddr} - ${fdtaddr}\0"
#define NANDBOOT            "run nandboot; "
```
  
其中，ubi0:NAND.file-system是由分区表决定的，ubi0指设备名，NAND.file-system是分区名，rw指分区可读可写，2048代表VID offset，不可或缺，下面需要使用这个变量。
分区表也在头文件中定义：  

```C
#define MTDPARTS_DEFAULT        "mtdparts=nand.0:" \
                    "256k(NAND.SPL)," \
                    "256k(NAND.SPL.backup1)," \
                    "256k(NAND.SPL.backup2)," \
                    "256k(NAND.SPL.backup3)," \
                    "512k(NAND.u-boot-spl-os)," \
                    "1m(NAND.u-boot)," \
                    "256k(NAND.u-boot-env)," \
                    "256k(NAND.u-boot-env.backup1)," \
                    "7m(NAND.kernel)," \
                    "-(NAND.file-system)"
```
  
注意mtd计数从0开始，因此文件系统分区为mtd=9。分区名称一定要与nandroot变量中一致，否则一定会出现无法加载的错误。  

## 使用UBI命令

UBI包括ubi part、ubi create、ubi write、ubi info (l)等命令。进入uboot后使用mtd可以查看分区表，接下来运行：  

```
nand erase.part NAND.file-system       //擦除分区
ubi part NAND.file-system 2048        //2048与环境变量一致，否则默认512
ubi create NAND.file-system           //制作卷
```
  
使用ubi info l命令查看分区，注意usable_leb_size参数为126976，在制作镜像要用。  

## 制作文件系统镜像

在rootfs文件系统上一级运行如下命令，并将ubifs.img拷贝到sd card的rootfs/boot路径下。  

```
sudo mkfs.ubifs -r ./rootfs/ -m 2048 -e 126976 -c 4300 -o ~/ubifs.img
```
  
注意-c, --max-leb-cnt=COUNT  maximum logical erase block count，主要和容量大小有关，如果太小会提示设置一个合理值。另外-e, --leb-size=SIZE:logical erase block size设置为126976。  

## 烧写文件  

将开发板设置为优先从SDIO（MMC 0）启动，启动进入uboot命令行，运行：  

```
env default -a
```
  
上述步骤还原编译的环境变量，下面依次运行下面指令（注意事先擦除各分区）。  

```
//烧写MLO至NAND.SPL，烧写u-boot.img至NAND.u-boot。
load mmc 0:1 0x80000000 /MLO                    //load MLO from sdcard
nand write 0x80000000 0x00000000 0x00016f30     
nand write 0x80000000 0x00040000 0x00016f30
nand write 0x80000000 0x00080000 0x00016f30
nand write 0x80000000 0x000c0000 0x00016f30
load mmc 0:1 0x90000000 /u-boot.img              //load u-boot.img from sdcard
nand write 0x90000000 0x00180000 0x000817e0
load mmc 0:2 0x88000000 /boot/am437x-gp-evm.dtb   //load dtb from sdcard

//烧写zImage至NAND.kernel，设备树至NAND.u-boot-spl-os。
load mmc 0:2 0x82000000 /boot/zImage             //load zImage from sdcard
load mmc 0:2 0x88080000 /boot/rootfs.tar.gz         //load gz rootf from sdcard 
nand write 0x82000000 0x00300000 0x00461568     //sizeof zImage  4593000
nand write 0x88000000 0x00100000 0x0000c774     //sizeof dtb   51060

//烧写ubifs镜像，UBI分区的步骤需要先做一遍。
load mmc 0:2 0x88080000 /boot/ubifs.img
ubi write 0x88080000 NAND.file-system 0x12414000
```

设置nandboot：  

```
setenv bootcmd nand read 0x88000000 NAND.u-boot-spl-os\;nand read 0x82000000 NAND.kernel\;bootz 0x82000000 - 0x88000000
```
  
主要从相关分区读取文件并保存到内存相应地址中，0x88000000和0x82000000也在环境变量中定义的。不赘述。  
保存环境变量：  

```
saveenv
```
  
关闭电源，设置从nandflash优先启动并拔出sdcard，上电后开发板从nandflash启动。  

## 添加自定义三取二命令  

### 添加自定义命令的方法

- 2014版本在common/文件目录下创建cmd_xx.c源码文件，内容如下：（注意U-boot没有C库）
在文件最后定义一个声明，主要包括命令名称和帮助信息，例如：  
  
```C
/*
nandinit-- cmd name 
6  -- arvc
0  -- DONOT repeat cmd while Enter
*/
U_BOOT_CMD(
 nandinit, 6, 0, do_nandinit,
 "2-out-of-3-voting commands\n",
 "[devname] [part] [filename] [flag] [nand_addr]");
```
  
表示命令名称为nandinit，包括本身一共最多6个参数，0表示按Enter键不重复命令，do_nandinit表示要执行的主体函数，后面信息为输入nandinit时弹出的帮助信息，一般由命令含义和命令使用说明组成。  
函数执行的主体为do_nandinit函数，形式如下：

```C
/* the main function of NANDINIT CMD */
static int do_nandinit(cmd_tbl_t *cmdtp, int flag, int argc, char *const argv[])
{
 if (argc < 2)
  return CMD_RET_USAGE;

 //函数将执行的内容

 return 0;
}
```
  
上面函数的参数，cmd_tbl_t是默认定义的，flag是重复标志，后面是命令行输入命令后紧跟的参数，与C语言命令行使用一致。  

- 在Makefile中添加编译  
  
```C
obj-$(CONFIG_CMD_NANDINIT) += cmd_nandinit.o
```
  
### 编写自定义命令源码  

<details>
<summary>cmd_nandinit.c</summary>  
  
```C
#include <common.h>
#include <command.h>
#include <exports.h>
#include <nand.h>
#include <onenand_uboot.h>
#include <linux/mtd/mtd.h>
#include <linux/mtd/partitions.h>
#include <ubi_uboot.h>
#include <asm/errno.h>
#include <jffs2/load_kernel.h>
#include <bootretry.h>
#include <cli.h>
#include <hash.h>
#include <watchdog.h>
#include <asm/io.h>
#include <linux/compiler.h>
#include <fs.h>
#include <malloc.h>
#include <asm/byteorder.h>
#include <jffs2/jffs2.h>

#define MLO_ADDR 0x80000000
#define UBOOTIMG_ADDR 0x81000000
#define FDT_ADDR 0x88000000
#define IMG_ADDR 0x82000000
#define FS_ADDR 0x88080000

#define ADDR1 0x85000000
#define ADDR2 0x8a000000
#define ADDR3 0x8f000000
#define ADDR_OUT 0x80000000

#define NAND_SPL 0x00000000
#define NAND_SPL_BACKUP1 0x00040000
#define NAND_SPL_BACKUP2 0x00080000
#define NAND_SPL_BACKUP3 0x000c0000
#define NAND_UBOOT_SPL_OS 0x00100000
#define NAND_UBOOT 0x00180000
#define NAND_KERNEL 0x00300000

static struct ubi_device *ubi;   //define ubi device

/*load file from eMMC*/
int load_file(const char *dev_name, const char *part, unsigned long addr, const char *filename)
{
 unsigned long bytes = 0;
 unsigned long pos = 0;
 int len_read;
 unsigned long time;
 fs_set_blk_dev(dev_name, part, FS_TYPE_ANY);   //judge the mmc part
 time = get_timer(0);
 len_read = fs_read(filename, addr, pos, bytes);
 time = get_timer(time);
 if (len_read <= 0)
  return -1;
 return len_read;
}


/*read 4 bytes  data from base_addr + addr, addr is the offset */
static uint32_t mem_read(uint addr, uint base_address)
{
 ulong bytes = 4;
 uint32_t x;
 const void *buf = map_sysmem(base_address + addr, bytes);
 x = *(volatile uint32_t *)buf;
 unmap_sysmem(buf);
 return x;
}


/* write 4 bytes data to base_addr + addr, addr is the offset */
static int mem_write(uint addr, uint base_address, uint32_t value)
{
 ulong writeval, bytes = 4;
 void *buf;
 writeval = (ulong)value;
 buf = map_sysmem(addr + base_address, bytes);
 *((u32 *)buf) = (u32)writeval;
 unmap_sysmem(buf);
 return 0;
}

/* nand write data into addr + offset of NAND flash */
static int nandwrite(int addr, loff_t offset, size_t length)
{
 nand_info_t *nand;
 int dev = nand_curr_device;
 nand = &nand_info[dev];
 return nand_write_skip_bad(nand, offset, &length, NULL, length, (u_char *)addr, 0);
}

/* function copied from cmd_ubi.c */
static struct ubi_volume *ubi_findvolume(char *volume)
{
 struct ubi_volume *vol = NULL;
 int i;

 for (i = 0; i < ubi->vtbl_slots; i++)
 {
  vol = ubi->volumes[i];
  if (vol && !strcmp(vol->name, volume))
   return vol;
 }

 printf("Volume %s not found!\n", volume);
 return NULL;
}

/* function copied from cmd_ubi.c */
int ubi_continue_write(char *volume, void *buf, size_t size)
{
 int err = 1;
 struct ubi_volume *vol;

 vol = ubi_findvolume(volume);
 if (vol == NULL)
  return ENODEV;

 err = ubi_more_update_data(ubi, vol, buf, size);
 if (err < 0)
 {
  printf("Couldnt or partially wrote data\n");
  return -err;
 }

 if (err)
 {
  size = err;

  err = ubi_check_volume(ubi, vol->vol_id);
  if (err < 0)
   return -err;

  if (err)
  {
   ubi_warn("volume %d on UBI device %d is corrupted",
      vol->vol_id, ubi->ubi_num);
   vol->corrupted = 1;
  }

  vol->checked = 1;
  ubi_gluebi_updated(vol);
 }

 return 0;
}

/* function copied from cmd_ubi.c */
int ubi_beginwrite(char *volume, void *buf, size_t size, size_t full_size)
{
 int err = 1;
 int rsvd_bytes = 0;
 struct ubi_volume *vol;

 vol = ubi_findvolume(volume);
 if (vol == NULL)
  return ENODEV;

 rsvd_bytes = vol->reserved_pebs * (ubi->leb_size - vol->data_pad);
 if (size < 0 || size > rsvd_bytes)
 {
  printf("size > volume size! Aborting!\n");
  return EINVAL;
 }

 err = ubi_start_update(ubi, vol, full_size);
 if (err < 0)
 {
  printf("Cannot start volume update\n");
  return -err;
 }

 return ubi_continue_write(volume, buf, size);
}

/*equal the cmd in uboot: ubi wirte ADDR(buf) UBI:volume_name(*volume) size */
static int ubiwrite(char *volume, void *buf, size_t size)
{
 return ubi_beginwrite(volume, buf, size, size);
}

/* the main function of NANDINIT CMD */
static int do_nandinit(cmd_tbl_t *cmdtp, int flag, int argc, char *const argv[])
{
 if (argc < 2)
  return CMD_RET_USAGE;

 int file_length = 0;

 /* 
 U-BOOT: nandinit mmc 0:1 /MLO 0 0x00000000
 # argv[1] = "mmc"    --flash device name
 # argv[2] = "0:1"    --the part of eMMC
 # argv[3] = "/MLO"    --path and name
 # argv[4] = "0"     --flag,0->use nand write;1->use ubi write
 # argv[5] = "0x00000000"  --the beginning address of mtd part
    "NAND.file-system" --the name of ubi volume
 */

 /*load file into three different address of DDRAM */
 file_length = load_file(argv[1], argv[2], ADDR1, argv[3]);
 load_file(argv[1], argv[2], ADDR2, argv[3]);
 load_file(argv[1], argv[2], ADDR3, argv[3]);

 ulong data1 = 0, data2 = 0, data3 = 0;
 ulong data_out = 0;
 uint offset = 0;
 int count, count_2k = 0;
 count = file_length / 4 + 1;

 unsigned long time = 0;

 /*flag = 0 ,start nand write file into mtd*/
 if (!strcmp(argv[4], "0"))
 {
  loff_t nand_off = simple_strtoul(argv[5], NULL, 16);
  printf("start\t3-to-2 [%s]\t\t\t\n", argv[3]);
  time = get_timer(0);
  while (count > 0)
  {
   data1 = mem_read(offset, ADDR1);
   data2 = mem_read(offset, ADDR2);
   data3 = mem_read(offset, ADDR3);
   data_out = (data1 & data2) | (data1 & data3) | (data2 & data3);
   mem_write(offset, ADDR_OUT, data_out);
   offset += 4;
   if ((offset % 2048 == 0) && (offset > 0))
   {
    nandwrite(ADDR_OUT + offset - 2048, (loff_t)(nand_off + offset - 2048), 2048);
   }
   count--;
  }
  count_2k = file_length / 2048;
  nandwrite(ADDR_OUT + 2048 * count_2k, (loff_t)(nand_off + 2048 * count_2k), file_length - count_2k * 2048);
  time = get_timer(time);
  printf("end\t3-to-2 [%s]\t\t\tin %lu ms\n", argv[3],time);
 }
 /*flag = 1 ,start ubi write file into NAND.file-system*/
 else if (!strcmp(argv[4], "1"))
 {
  ubi = ubi_devices[0];
  printf("start\t3-to-2 [%s]\t\t\t\n", argv[3]);
  time = get_timer(0);
  while (count > 0)
  {
   data1 = mem_read(offset, ADDR1);
   data2 = mem_read(offset, ADDR2);
   data3 = mem_read(offset, ADDR3);
   data_out = (data1 & data2) | (data1 & data3) | (data2 & data3);
   mem_write(offset, ADDR_OUT, data_out);
   offset += 4;
   count--;
  }

  printf("start ubi_write filesystem to [%s]\n", argv[5]);
  int ret;
  ret = ubiwrite(argv[5], (void *)ADDR_OUT, (size_t)file_length);
  if (!ret)
  {
   printf("%d MB written to volume %s\n", file_length / 1024 / 1024, "NAND.file-system");
  }
  time = get_timer(time);
  printf("end\t3-to-2 [%s]\t\t\tin %lu ms\n", argv[3],time);
 }
 else
 {
  return CMD_RET_USAGE;
 }

 return 0;
}

/*
nandinit-- cmd name 
6  -- arvc
0  -- DONOT repeat cmd while Enter
*/
U_BOOT_CMD(
 nandinit, 6, 0, do_nandinit,
 "2-out-of-3-voting commands\n",
 "[devname] [part] [filename] [flag] [nand_addr]");

```
  
</details>
上面的源码包括内存读写函数、ubi读写命令、nand读命令等，在do_nandinit()中进行三取二操作。  
  
## 修改Uboot初始化配置  

在am43xx_evm.h中重新配置自定义命令进行三取二操作，首先添加一个自定义的NAND_INIT命令，再将NAND_INIT命令加入到CONFIG_BOOTCOMMAND中。  
  
```C
#define CONFIG_BOOTCOMMAND \
 "if mmc rescan; then " \
  "echo SD/MMC found on device ${devnum};" \
  NAND_INIT \
 "fi; " \
 "run nandboot;" 

#define NAND_INIT "nand erase.chip;" \
 "ubi part NAND.file-system 2048 ;" \
 "ubi create NAND.file-system ;" \
 "nandinit mmc 0:1 /MLO 0 0x00000000;" \
 "nandinit mmc 0:1 /MLO 0 0x00040000;" \
 "nandinit mmc 0:1 /MLO 0 0x00080000;" \
 "nandinit mmc 0:1 /MLO 0 0x000c0000;" \
 "nandinit mmc 0:1 /u-boot.img 0 0x00180000; " \
 "nandinit mmc 0:2 /boot/am437x-gp-evm.dtb 0 0x00100000; " \
 "nandinit mmc 0:2 /boot/zImage 0 0x00300000; " \
 "nandinit mmc 0:2 /boot/ubifs.img 1 NAND.file-system ; " \
```
  
上述CONFIG_BOOTCOMMAN中首先扫描eMMC设备是否存在，如果存在SD卡，则运行NAND_INIT，主要是先加载再运行三取二命令；如果未连接eMMC设备，则直接nandboot启动。
