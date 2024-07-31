---
title: R329开发板上手体验
date: 2021-07-27 11:23:11
index_img: /img/r329/9.jpeg
tags: 
    - Linux
    - Algorithm
    - OpenCV
categories: 
    - AIoT
---

R329的板子到手后折腾了几天，将前期学习的步骤简单记录一下。

<!-- more -->  

## 0.硬件

板子的设计十分精巧，尤其是相机模组的Type-C连接方式，比起以前习惯的相机拽着排线前后翻动好用得多。

![图1](/img/r329/1.jpeg)

翻开显示屏下面是卡槽和两侧Type-C数据接口，左侧连相机，右侧连主机。显示屏是磁吸式的（指两块柔性吸铁石）。卡槽插卡时金手指向下。

![图2](/img/r329/2.jpeg)


最后再来个正面照~
![图3](/img/r329/3.jpeg)

调试板子至少需要：

* 开发板
* USB转Type-C数据线
* 大于512MB的SD卡及读卡器
* 一个Ubuntu主机
* WIFI

## 1.刷系统制卡

### 1.1编译系统

参考[R329 SDK 上手编译与烧录!](https://www.cnblogs.com/juwan/p/14650733.html)以及[【R329测评】全志 R329 Tina 系统镜像编译](https://aijishu.com/a/1060000000221484),遗憾的是本地编译u-boot的时候就失败了，遂采用[R329开发板系列教程之二|实机运行aipu程序](https://aijishu.com/a/1060000000220719)中网盘提供的镜像。

### 1.2制卡

使用网盘中PhoenixCard工具将img刷入sd卡，因为后面要用到OpenCV，所以先烧写tina_r329-evb5_uart0_0723.img。如下图步骤：
![图4](/img/r329/4.png)

制卡完成显示一个绿色的进度条，如果不行尝试换卡。

注：如果制卡后可以启动，则继续，要是和我一样在启动是找不到根目录，可以在上位机Ubuntu中将根目录删除，并将网盘中rootfs.tar.gz解压拷贝到根目录，拷贝完成后不要急着拔卡，sync一下，要优雅~

### 1.3扩展分区

系统根目录没有使用所有的存储，可以使用df -h命令查看。在ubuntu运行：

```bash
df -h   #查看卡挂载的位置，比如/dev/sdb
sudo fdisk /dev/sdb
p       #查看分区
d       #删除分区8
8
d       #删除分区7
7
n       #创建分区7
7
xxxx    #输入起始位置（原7的起始地址）
w       #保存
sudo umount /dev/sdb7
sudo e2fsck -f /dev/sdb7
sudo resize2fs /dev/sdb7
df -h
```

此时应该将分区扩大了，此处感谢[李洋](https://aijishu.com/u/liyang_dmdooa)。

### 1.4开机

数据线连接电脑后，主机打开串口，波特率115200，Ubuntu可以使用

```bash
sudo minicom -D /dev/ttyUSB0
```

开机进入系统：
![图5](/img/r329/5.png)

连接wifi：

```bash
wifi_connect_ap_test [wifiname] [wifipasswd]
```

传递文件用scp，密码是sipeed。

## 2.验证模型

```bash
cd /root/maix_sense
insmod aipu.ko
./run.sh -c mbnetv2
./zhouyi models/mbnetv2/aipu.bin test.bmp 0
```

将自己模型放到maxi_sense的models下，并且aipu.bin（来自于模型交叉编译），input0.bin（来自量化模型时的输入input.bin）以及output.bin（来自仿真时的输出）文件需要准备好，再回到maix_sense下运行脚本，-c后面跟着的是模型目录名。注意系统给模型预留了约100MB的内存，我的VGG模型过大，所以验证失败。解决办法要么编译系统时扩大预留空间，要么换成更轻量的模型。

可执行文件zhouyi来自于网盘zhouyi_bmp目录，使用交叉编译工具编译，运行时依赖相关的库文件，注意检查系统根目录是否配置。test.bmp的测试图片需要注意输入的尺寸与模型一致，这里是224x224x3。获得一个224x224x3通道的bmp位图可以借助PS软件，打开图片，裁剪为正方形，再点击菜单->图像大小：
![图6](/img/r329/6.jpeg)

然后另存为，选择BMP格式，在弹出的窗口选择高级模式，选择24位R8 G8 B8格式：
![图7](/img/r329/7.jpeg)

保存为test.bmp后发送到板子上运行即可，结果如下，分的类都是猫的类型。
![图8](/img/r329/8.png)

## 3.跑通OpenCV

### 3.1编译

参考[R329开发板教程之三|视觉模型实时运行](https://aijishu.com/a/1060000000221762)。

在上位机准备源码、toolchain和rootfs（内涵opencv的动态库和头文件），目录如下：

```bash
.
├── zhouyi_cam
├── rootfs
└── toolchain
```

在zhouyi_cam路径下CMakLists.txt中需要将编译链和`ROOTFS_root`配置一下再编译：

```bash
mkdir build && cd build
cmake .. && make -j32
```

下载到板子上，如果目标板系统缺少OpenCV的依赖，参考步骤1.2制卡，将zhouyi_cam拷贝到root下，运行：

```bash
cd /root
insmod maix_sense/aipu.ko
./zhouyi_cam maix_sense/models/resnet50/aipu.bin 1
rmmod maix_sense/aipu.ko
```

运行结果：
![图9](/img/r329/9.jpeg)

测试中发现：

* 帧率和模型有关，resnet50只有7-8帧，而mbnetv2可以达到20-21帧
* 注意参数使用int8还是uint8
* 多跑几次内存不足，所以需要卸载驱动模块。
