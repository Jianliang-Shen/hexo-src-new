---
title: 编译内核用到的图形界面工具
date: 2019-07-22 17:48:33
index_img: /img/post_pics/index_img/gconfig.png
tags:
    - Linux
    - Kernel
categories: 
    - IoT
---

除了命令行外，还可以用图形UI来快速配置内核。

<!-- more -->

- [menuconfig](#menuconfig)
- [xconfig](#xconfig)
- [gconfig](#gconfig)


## menuconfig
运行在没有桌面环境的主机上，可以查看选项功能，不支持搜索，需要安装终端的图形包。
``` s
$ make menuconfig  
$ sudo apt-get install libncurses5-dev
```
<!-- more --> 



## xconfig
可以查看选项功能，支持搜索功能，需要安装QT依赖包。
``` s
$ make xconfig
$ CHECK   qt
$ Could not find Qt via pkg-config.
$ Please install either Qt 4.8 or 5.x. and make sure it's in PKG_CONFIG_PATH
$ apt-get install qt4-dev-tools
```
安装后运行如下：  
![](/img/post_pics/ui/xconfig.png)
## gconfig
可以查看选项功能，需要安装gtk的包。
``` s
$ make gconfig

$ Unable to find the GTK+ installation. Please make sure that
$ the GTK+ 2.0 development package is correctly installed...
$ You need gtk+-2.0, glib-2.0 and libglade-2.0.
$  sudo apt-get install libgtk2.0-dev libglib2.0-dev libglade2-dev
```
安装后运行如下：  
![](/img/post_pics/ui/gconfig.png)