---
title: R329 Debian系统安装体验
date: 2021-08-02 21:56:29
tags: 
    - Linux
    - Algorithm
    - OpenCV
categories: 
    - AIoT
---

## R329 MaixSense

近期矽速科技新移植了Debian系统到MaixSense(R329)上。

<!-- more -->  

本章节网盘资料：

```bash
链接：https://pan.baidu.com/s/1h-Lf6Y-xnvWUrOW6FSwitQ 
提取码：dl7s 
```

## 镜像下载烧录

从以上网盘地址获取镜像文件（r329-mainline-debian-20210802.img）后，使用 dd 或者 win32diskImager 烧录镜像到TF卡。  

```bash
dd if=<ubuntu image name>.iso of=/dev/sdX bs=32M 
```

镜像尺寸大约有2GB，请使用至少4GB的TF卡进行烧录。

## Debian系统基础使用

### 启动现象

将烧录好系统的卡插入卡槽，上电，可见屏幕上有两只企鹅图标，稍等片刻即进入了登录提示符界面，此时可以拔下摄像头，使用type-C转A母口转接头，插上键鼠进行操作；  
当然正常使用方式是直接在串口终端操作, 用户名 root，密码 sipeed。  
（附录附上了debian系统启动日志，启动有问题的可以参考该启动日志）  

### 根分区扩容

debian镜像默认为2个分区，第一分区是boot分区，第二分区为根文件系统所在分区。  
默认第二分区是按照根文件系统大小配置，所以启动后会发现可用空间为0，所以首次启动后要先进行分区扩容。  

1. fdisk /dev/mmcblk0 
   1. d 2   #删除第二分区
   2. n p 2  回车 回车 No  #重建第二分区，剩余空间全部给第二分区
   3. w   #保存配置
2. partx   #查看，高速内核去识别等级硬盘信息
3. resize2fs /dev/mmcblk0p2

至此根分区扩容完成：

```bash
root@maixsense:~## df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        29G  1.9G   26G   7% /
devtmpfs         55M     0   55M   0% /dev
tmpfs           120M     0  120M   0% /dev/shm
tmpfs            48M  848K   47M   2% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs            24M     0   24M   0% /run/user/0
```

### SWAP分区设置

由于R329内置内存较小，运行debian系统没有问题，但是要进行本地编译的话，就必须开启swap分区。  

1. 设置swapnees(可选)
   1. swappiness参数值可设置范围在0到100之间。 此参数值越低,就会让Linux系统尽量少用swap分区,多用内存;参数值越高就是反过来,使内核更多的去使用swap空间
   2. cat /proc/sys/vm/swappiness  
   3. echo "vm.swappiness = 50" >> /etc/sysctl.conf  #永久修改
   4. sysctl -p   #启用内存阈值设置
2. 创建swap分区
   1. cd /opt && dd if=/dev/zero of=swapfile bs=1M count=1024  #1GB swap空间，约2~3分钟
   2. mkswap swapfile  
3. 开启关闭swap分区
   1. swapon swapfile /  swapoff swapfile
4. 开机自动挂载swap分区（可选）
   1. 在/etc/fstab内加入
      1. /opt/swapfile    swap        swap    defaults        0 0

至此已完成swap分区设置，可使用free查看当前内存情况：  

```bash
root@maixsense:/opt## free -h
               total        used        free      shared  buff/cache   available
Mem:           238Mi        82Mi       3.0Mi       0.0Ki       151Mi       146Mi
Swap:          1.0Gi          0B       1.0Gi
```

### 联网操作

使用 nmtui 指令可以进入可视化的配网界面，  
选择 Activate a connection ， 选择对应的SSID，输入连接密码，确认即可。

### 安装软件包

debian系统内置了便捷的软件包管理系统，按照正常PC端的操作方式操作即可，比如我们要按照ifconfig 指令：

```bash
apt-get install net-tools
```

等待片刻即可安装完成

### 手工生成ssh秘钥

系统暂时没有自动生成ssh秘钥，需要手工生成秘钥才能使能ssh。

## Debian系统下zhouyi使用

默认镜像的 ~/zhouyi_test 目录下放置了zhouyi_cam 例程和模型，在此用户可以体验本机编译zhouyi程序：

```bash
cd ~/zhouyi_test/build
make clean
cmake ..
time make -j2  #注意必须开启swap才能编译通过
```

经过漫长的等待，即可得到zhouyi_cam可执行文件，可以本机执行测试。
