---
title: MacOS安装ncnn和OpenCV
date: 2023-06-24 08:41:35
tags:
    - OpenCV
categories: 
    - AIoT
---

你已经学会安装ncnn和opencv来打开图片识别目标了，下面去拯救公主吧！🐛🦆

<!-- more -->

## 安装腾讯ncnn

参考：https://github.com/Tencent/ncnn/wiki/how-to-build#build-for-macos

安装完成后，在bashrc导入：

```bash
# ncnn_install
PATH="/Users/sjl/learn/ncnn/build/install/bin":"$PATH"
PATH="/Users/sjl/learn/ncnn/build/examples":"$PATH"
```

下载model，建议压缩包

```bash
git clone git@github.com:nihui/ncnn-assets.git
```

到models路径下

```bash
yolov3 test.jpeg
```

## 安装opencv

下载安装包

```bash
cd opencv-3.4.16
mkdir build
cd build
cmake-gui
make -j32
sudo make install
opencv2_version
 ```
